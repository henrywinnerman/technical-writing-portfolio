# Integrating Paystack Payments in Next.js

A complete guide to accepting payments in your Next.js application using the Paystack API with the App Router and TypeScript.

## Introduction

This guide walks you through integrating Paystack payment processing into a Next.js 14+ application. You'll learn to build a secure payment flow using Next.js API routes, server actions, and the react-paystack library.

**What you'll build:**
- A checkout page with Paystack's popup integration
- Secure server-side transaction initialization
- Payment verification via API routes
- Webhook handling for reliable payment confirmation

**Prerequisites:**
- Node.js 18+ installed
- Basic knowledge of React and Next.js
- A Paystack account ([create one here](https://dashboard.paystack.co/#/signup))
- Familiarity with TypeScript (optional but recommended)

**Estimated time:** 45-60 minutes

---

## Table of Contents

1. [Project Setup](#1-project-setup)
2. [Get Your API Keys](#2-get-your-api-keys)
3. [Configure Environment Variables](#3-configure-environment-variables)
4. [Understanding the Payment Flow](#4-understanding-the-payment-flow)
5. [Create the Payment API Route](#5-create-the-payment-api-route)
6. [Build the Checkout Component](#6-build-the-checkout-component)
7. [Verify Transactions](#7-verify-transactions)
8. [Handle Webhooks](#8-handle-webhooks)
9. [Test Your Integration](#9-test-your-integration)
10. [Deploy to Production](#10-deploy-to-production)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Project Setup

### Create a new Next.js project

If you're starting fresh, create a new Next.js project with TypeScript:

```bash
npx create-next-app@latest my-paystack-app --typescript --tailwind --app
cd my-paystack-app
```

Select the following options when prompted:
- Would you like to use ESLint? **Yes**
- Would you like to use `src/` directory? **Yes**
- Would you like to customize the default import alias? **No**

### Install dependencies

Install the react-paystack library:

```bash
npm install react-paystack
```

---

## 2. Get Your API Keys

1. Log in to your [Paystack Dashboard](https://dashboard.paystack.co/)
2. Navigate to **Settings** → **API Keys & Webhooks**
3. Copy your **Test Public Key** (starts with `pk_test_`)
4. Copy your **Test Secret Key** (starts with `sk_test_`)

> ⚠️ **Security Note:** Your secret key must never be exposed in client-side code. Only use it in API routes and server components.

---

## 3. Configure Environment Variables

Create a `.env.local` file in your project root:

```env
# Paystack API Keys
NEXT_PUBLIC_PAYSTACK_PUBLIC_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
PAYSTACK_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# App URL (update for production)
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

**Important:** 
- Variables prefixed with `NEXT_PUBLIC_` are exposed to the browser
- `PAYSTACK_SECRET_KEY` has no prefix, keeping it server-side only

Add `.env.local` to your `.gitignore` if not already present:

```gitignore
# local env files
.env*.local
```

---

## 4. Understanding the Payment Flow

Before writing code, understand how payments flow through your application:

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│   Customer   │──────▶│  Your Next.js │──────▶│   Paystack   │
│  clicks Pay  │       │   API Route   │       │     API      │
└──────────────┘       └──────────────┘       └──────────────┘
                              │                      │
                              │    access_code       │
                              │◀─────────────────────│
                              │
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│   Paystack   │◀──────│   Customer   │◀──────│    Popup     │
│   Checkout   │       │ enters card  │       │   displays   │
└──────────────┘       └──────────────┘       └──────────────┘
        │
        │ Payment complete
        ▼
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│   Callback   │──────▶│   Verify     │──────▶│   Deliver    │
│   triggered  │       │  transaction │       │    value     │
└──────────────┘       └──────────────┘       └──────────────┘
```

**Flow summary:**
1. Customer initiates payment on your site
2. Your server initializes the transaction with Paystack
3. Paystack returns an `access_code`
4. The popup displays Paystack's checkout
5. Customer completes payment
6. Your callback verifies the transaction
7. You deliver value to the customer

---

## 5. Create the Payment API Route

Create an API route to initialize transactions securely on the server.

### Create the initialization endpoint

Create `src/app/api/paystack/initialize/route.ts`:

```typescript
import { NextRequest, NextResponse } from 'next/server';

const PAYSTACK_SECRET_KEY = process.env.PAYSTACK_SECRET_KEY;

interface InitializePaymentBody {
  email: string;
  amount: number; // Amount in Naira
  metadata?: Record<string, unknown>;
}

export async function POST(request: NextRequest) {
  try {
    const body: InitializePaymentBody = await request.json();
    const { email, amount, metadata } = body;

    // Validate required fields
    if (!email || !amount) {
      return NextResponse.json(
        { error: 'Email and amount are required' },
        { status: 400 }
      );
    }

    // Validate amount (minimum ₦100)
    if (amount < 100) {
      return NextResponse.json(
        { error: 'Minimum amount is ₦100' },
        { status: 400 }
      );
    }

    // Convert Naira to kobo (Paystack uses smallest currency unit)
    const amountInKobo = Math.round(amount * 100);

    // Generate unique reference
    const reference = `TXN_${Date.now()}_${Math.random().toString(36).substring(2, 9)}`;

    // Initialize transaction with Paystack
    const response = await fetch('https://api.paystack.co/transaction/initialize', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${PAYSTACK_SECRET_KEY}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        email,
        amount: amountInKobo,
        reference,
        callback_url: `${process.env.NEXT_PUBLIC_APP_URL}/payment/verify`,
        metadata: {
          ...metadata,
          custom_fields: [
            {
              display_name: 'Payment For',
              variable_name: 'payment_for',
              value: metadata?.productName || 'Purchase',
            },
          ],
        },
      }),
    });

    const data = await response.json();

    if (!data.status) {
      return NextResponse.json(
        { error: data.message || 'Failed to initialize payment' },
        { status: 400 }
      );
    }

    return NextResponse.json({
      status: true,
      message: 'Payment initialized',
      data: {
        authorization_url: data.data.authorization_url,
        access_code: data.data.access_code,
        reference: data.data.reference,
      },
    });
  } catch (error) {
    console.error('Payment initialization error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Create the verification endpoint

Create `src/app/api/paystack/verify/route.ts`:

```typescript
import { NextRequest, NextResponse } from 'next/server';

const PAYSTACK_SECRET_KEY = process.env.PAYSTACK_SECRET_KEY;

export async function GET(request: NextRequest) {
  try {
    const searchParams = request.nextUrl.searchParams;
    const reference = searchParams.get('reference');

    if (!reference) {
      return NextResponse.json(
        { error: 'Transaction reference is required' },
        { status: 400 }
      );
    }

    // Verify transaction with Paystack
    const response = await fetch(
      `https://api.paystack.co/transaction/verify/${encodeURIComponent(reference)}`,
      {
        method: 'GET',
        headers: {
          Authorization: `Bearer ${PAYSTACK_SECRET_KEY}`,
        },
      }
    );

    const data = await response.json();

    if (!data.status) {
      return NextResponse.json(
        { error: data.message || 'Verification failed' },
        { status: 400 }
      );
    }

    const transaction = data.data;

    return NextResponse.json({
      status: true,
      message: 'Verification successful',
      data: {
        reference: transaction.reference,
        amount: transaction.amount / 100, // Convert back to Naira
        currency: transaction.currency,
        status: transaction.status,
        channel: transaction.channel,
        paid_at: transaction.paid_at,
        customer: {
          email: transaction.customer.email,
        },
      },
    });
  } catch (error) {
    console.error('Payment verification error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

---

## 6. Build the Checkout Component

### Create the PaystackButton component

Create `src/components/PaystackButton.tsx`:

```typescript
'use client';

import { usePaystackPayment } from 'react-paystack';
import { useRouter } from 'next/navigation';

interface PaystackButtonProps {
  email: string;
  amount: number; // Amount in Naira
  onSuccess?: (reference: string) => void;
  onClose?: () => void;
  disabled?: boolean;
  className?: string;
  children?: React.ReactNode;
}

export default function PaystackButton({
  email,
  amount,
  onSuccess,
  onClose,
  disabled = false,
  className = '',
  children = 'Pay Now',
}: PaystackButtonProps) {
  const router = useRouter();

  const config = {
    reference: `TXN_${Date.now()}_${Math.random().toString(36).substring(2, 9)}`,
    email,
    amount: amount * 100, // Convert to kobo
    publicKey: process.env.NEXT_PUBLIC_PAYSTACK_PUBLIC_KEY!,
    currency: 'NGN',
  };

  const handleSuccess = (response: { reference: string }) => {
    if (onSuccess) {
      onSuccess(response.reference);
    } else {
      // Default: redirect to verification page
      router.push(`/payment/verify?reference=${response.reference}`);
    }
  };

  const handleClose = () => {
    if (onClose) {
      onClose();
    }
    console.log('Payment dialog closed');
  };

  const initializePayment = usePaystackPayment(config);

  const handleClick = () => {
    initializePayment({ onSuccess: handleSuccess, onClose: handleClose });
  };

  return (
    <button
      onClick={handleClick}
      disabled={disabled}
      className={`bg-green-600 text-white px-6 py-3 rounded-lg font-medium 
        hover:bg-green-700 transition disabled:opacity-50 disabled:cursor-not-allowed 
        ${className}`}
    >
      {children}
    </button>
  );
}
```

### Create the checkout page

Create `src/app/checkout/page.tsx`:

```typescript
'use client';

import { useState } from 'react';
import PaystackButton from '@/components/PaystackButton';

interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
}

// Sample product - in production, fetch from your database/API
const product: Product = {
  id: 'prod_001',
  name: 'Premium Subscription',
  price: 5000,
  description: 'Get access to all premium features for one month',
};

export default function CheckoutPage() {
  const [email, setEmail] = useState('');
  const [isValid, setIsValid] = useState(false);

  const handleEmailChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setEmail(value);
    // Basic email validation
    setIsValid(/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value));
  };

  const handleSuccess = (reference: string) => {
    console.log('Payment successful! Reference:', reference);
    // Redirect to success page or verify payment
    window.location.href = `/payment/verify?reference=${reference}`;
  };

  const handleClose = () => {
    console.log('Payment cancelled');
  };

  return (
    <main className="min-h-screen bg-gray-100 py-12 px-4">
      <div className="max-w-md mx-auto">
        {/* Product Card */}
        <div className="bg-white rounded-lg shadow-md p-6 mb-6">
          <h1 className="text-2xl font-bold text-gray-800 mb-2">
            {product.name}
          </h1>
          <p className="text-gray-600 mb-4">{product.description}</p>
          <p className="text-3xl font-bold text-green-600">
            ₦{product.price.toLocaleString()}
          </p>
        </div>

        {/* Checkout Form */}
        <div className="bg-white rounded-lg shadow-md p-6">
          <h2 className="text-xl font-semibold text-gray-800 mb-4">
            Complete Payment
          </h2>

          <div className="mb-6">
            <label
              htmlFor="email"
              className="block text-sm font-medium text-gray-700 mb-2"
            >
              Email Address
            </label>
            <input
              type="email"
              id="email"
              value={email}
              onChange={handleEmailChange}
              placeholder="Enter your email"
              className="w-full px-4 py-3 border border-gray-300 rounded-lg 
                focus:ring-2 focus:ring-green-500 focus:border-transparent
                outline-none transition"
            />
            {email && !isValid && (
              <p className="text-red-500 text-sm mt-1">
                Please enter a valid email address
              </p>
            )}
          </div>

          {/* Order Summary */}
          <div className="border-t border-gray-200 pt-4 mb-6">
            <div className="flex justify-between text-gray-600 mb-2">
              <span>Subtotal</span>
              <span>₦{product.price.toLocaleString()}</span>
            </div>
            <div className="flex justify-between text-gray-600 mb-2">
              <span>Processing Fee</span>
              <span>₦0</span>
            </div>
            <div className="flex justify-between text-lg font-bold text-gray-800">
              <span>Total</span>
              <span>₦{product.price.toLocaleString()}</span>
            </div>
          </div>

          {/* Pay Button */}
          <PaystackButton
            email={email}
            amount={product.price}
            onSuccess={handleSuccess}
            onClose={handleClose}
            disabled={!isValid}
            className="w-full"
          >
            Pay ₦{product.price.toLocaleString()}
          </PaystackButton>

          {/* Security Note */}
          <p className="text-center text-sm text-gray-500 mt-4">
            🔒 Secured by Paystack
          </p>
        </div>
      </div>
    </main>
  );
}
```

---

## 7. Verify Transactions

Create a verification page that confirms payment status.

### Create the verification page

Create `src/app/payment/verify/page.tsx`:

```typescript
'use client';

import { useEffect, useState } from 'react';
import { useSearchParams } from 'next/navigation';
import Link from 'next/link';

interface VerificationData {
  reference: string;
  amount: number;
  currency: string;
  status: string;
  channel: string;
  paid_at: string;
  customer: {
    email: string;
  };
}

export default function VerifyPaymentPage() {
  const searchParams = useSearchParams();
  const reference = searchParams.get('reference');

  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [data, setData] = useState<VerificationData | null>(null);

  useEffect(() => {
    if (!reference) {
      setError('No transaction reference provided');
      setLoading(false);
      return;
    }

    const verifyPayment = async () => {
      try {
        const response = await fetch(
          `/api/paystack/verify?reference=${encodeURIComponent(reference)}`
        );
        const result = await response.json();

        if (!result.status) {
          setError(result.error || 'Verification failed');
        } else {
          setData(result.data);
        }
      } catch (err) {
        setError('An error occurred while verifying payment');
        console.error(err);
      } finally {
        setLoading(false);
      }
    };

    verifyPayment();
  }, [reference]);

  if (loading) {
    return (
      <main className="min-h-screen bg-gray-100 flex items-center justify-center">
        <div className="text-center">
          <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-green-600 mx-auto mb-4"></div>
          <p className="text-gray-600">Verifying your payment...</p>
        </div>
      </main>
    );
  }

  if (error) {
    return (
      <main className="min-h-screen bg-gray-100 flex items-center justify-center px-4">
        <div className="bg-white rounded-lg shadow-md p-8 max-w-md w-full text-center">
          <div className="w-16 h-16 bg-red-100 rounded-full flex items-center justify-center mx-auto mb-4">
            <svg
              className="w-8 h-8 text-red-500"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                strokeLinecap="round"
                strokeLinejoin="round"
                strokeWidth={2}
                d="M6 18L18 6M6 6l12 12"
              />
            </svg>
          </div>
          <h1 className="text-2xl font-bold text-gray-800 mb-2">
            Verification Failed
          </h1>
          <p className="text-gray-600 mb-6">{error}</p>
          <Link
            href="/checkout"
            className="inline-block bg-green-600 text-white px-6 py-2 rounded-lg hover:bg-green-700 transition"
          >
            Try Again
          </Link>
        </div>
      </main>
    );
  }

  const isSuccess = data?.status === 'success';

  return (
    <main className="min-h-screen bg-gray-100 flex items-center justify-center px-4">
      <div className="bg-white rounded-lg shadow-md p-8 max-w-md w-full">
        {/* Status Icon */}
        <div
          className={`w-16 h-16 rounded-full flex items-center justify-center mx-auto mb-4 ${
            isSuccess ? 'bg-green-100' : 'bg-yellow-100'
          }`}
        >
          {isSuccess ? (
            <svg
              className="w-8 h-8 text-green-500"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                strokeLinecap="round"
                strokeLinejoin="round"
                strokeWidth={2}
                d="M5 13l4 4L19 7"
              />
            </svg>
          ) : (
            <svg
              className="w-8 h-8 text-yellow-500"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                strokeLinecap="round"
                strokeLinejoin="round"
                strokeWidth={2}
                d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"
              />
            </svg>
          )}
        </div>

        {/* Status Message */}
        <h1 className="text-2xl font-bold text-gray-800 text-center mb-2">
          {isSuccess ? 'Payment Successful!' : 'Payment Pending'}
        </h1>
        <p className="text-gray-600 text-center mb-6">
          {isSuccess
            ? 'Your transaction has been completed successfully.'
            : 'Your payment is being processed.'}
        </p>

        {/* Transaction Details */}
        {data && (
          <div className="border-t border-gray-200 pt-4 space-y-3">
            <div className="flex justify-between">
              <span className="text-gray-600">Reference</span>
              <span className="font-mono text-sm">{data.reference}</span>
            </div>
            <div className="flex justify-between">
              <span className="text-gray-600">Amount</span>
              <span className="font-semibold">
                ₦{data.amount.toLocaleString()}
              </span>
            </div>
            <div className="flex justify-between">
              <span className="text-gray-600">Email</span>
              <span>{data.customer.email}</span>
            </div>
            <div className="flex justify-between">
              <span className="text-gray-600">Channel</span>
              <span className="capitalize">{data.channel}</span>
            </div>
            <div className="flex justify-between">
              <span className="text-gray-600">Status</span>
              <span
                className={`px-2 py-1 rounded text-sm ${
                  isSuccess
                    ? 'bg-green-100 text-green-800'
                    : 'bg-yellow-100 text-yellow-800'
                }`}
              >
                {data.status}
              </span>
            </div>
          </div>
        )}

        {/* Action Button */}
        <div className="mt-6">
          <Link
            href="/"
            className="block w-full text-center bg-green-600 text-white px-6 py-3 rounded-lg hover:bg-green-700 transition"
          >
            Return Home
          </Link>
        </div>
      </div>
    </main>
  );
}
```

---

## 8. Handle Webhooks

Webhooks provide reliable payment confirmation even if the customer closes their browser.

### Create the webhook handler

Create `src/app/api/paystack/webhook/route.ts`:

```typescript
import { NextRequest, NextResponse } from 'next/server';
import crypto from 'crypto';

const PAYSTACK_SECRET_KEY = process.env.PAYSTACK_SECRET_KEY!;

// Verify webhook signature
function verifySignature(body: string, signature: string): boolean {
  const hash = crypto
    .createHmac('sha512', PAYSTACK_SECRET_KEY)
    .update(body)
    .digest('hex');
  return hash === signature;
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.text();
    const signature = request.headers.get('x-paystack-signature');

    // Verify webhook signature
    if (!signature || !verifySignature(body, signature)) {
      console.error('Invalid webhook signature');
      return NextResponse.json(
        { error: 'Invalid signature' },
        { status: 401 }
      );
    }

    const event = JSON.parse(body);
    console.log('Webhook received:', event.event);

    // Handle different event types
    switch (event.event) {
      case 'charge.success':
        await handleChargeSuccess(event.data);
        break;

      case 'transfer.success':
        await handleTransferSuccess(event.data);
        break;

      case 'transfer.failed':
        await handleTransferFailed(event.data);
        break;

      default:
        console.log('Unhandled event type:', event.event);
    }

    // Always return 200 to acknowledge receipt
    return NextResponse.json({ received: true }, { status: 200 });
  } catch (error) {
    console.error('Webhook error:', error);
    // Still return 200 to prevent retries for parsing errors
    return NextResponse.json({ received: true }, { status: 200 });
  }
}

async function handleChargeSuccess(data: {
  reference: string;
  amount: number;
  customer: { email: string };
  metadata?: Record<string, unknown>;
}) {
  console.log('Payment successful:', data.reference);

  // TODO: Implement your business logic here
  // Examples:
  // - Update order status in database
  // - Send confirmation email
  // - Activate subscription
  // - Credit user wallet

  // Verify the transaction (recommended)
  const verifyResponse = await fetch(
    `https://api.paystack.co/transaction/verify/${data.reference}`,
    {
      headers: {
        Authorization: `Bearer ${PAYSTACK_SECRET_KEY}`,
      },
    }
  );
  const verifyData = await verifyResponse.json();

  if (verifyData.data.status === 'success') {
    // Safe to deliver value
    console.log('Transaction verified:', {
      reference: data.reference,
      amount: data.amount / 100,
      email: data.customer.email,
    });
  }
}

async function handleTransferSuccess(data: { reference: string }) {
  console.log('Transfer successful:', data.reference);
  // Handle successful transfer (for payouts)
}

async function handleTransferFailed(data: { reference: string }) {
  console.error('Transfer failed:', data.reference);
  // Handle failed transfer
}
```

### Configure webhook URL in Paystack

1. Go to [Paystack Dashboard](https://dashboard.paystack.co/)
2. Navigate to **Settings** → **API Keys & Webhooks**
3. Enter your webhook URL: `https://yourdomain.com/api/paystack/webhook`
4. Click **Save**

> **For local testing:** Use [ngrok](https://ngrok.com/) to expose your local server:
> ```bash
> ngrok http 3000
> ```
> Then use the ngrok URL as your webhook endpoint.

---

## 9. Test Your Integration

### Start the development server

```bash
npm run dev
```

Visit `http://localhost:3000/checkout` to see your checkout page.

### Test card numbers

Paystack provides test cards for different scenarios:

| Card Number | CVV | Expiry | PIN | OTP | Result |
|-------------|-----|--------|-----|-----|--------|
| 4084 0840 8408 4081 | 408 | Any future date | Any 4 digits | 123456 | Success |
| 4084 0840 8408 4081 | 400 | Any future date | Any 4 digits | - | Failed |
| 5060 6666 6666 6666 666 | 123 | Any future date | Any 4 digits | 123456 | Verve Success |

### Testing checklist

- [ ] Checkout page loads correctly
- [ ] Email validation works
- [ ] Paystack popup opens on button click
- [ ] Test payment completes successfully
- [ ] Verification page shows correct status
- [ ] Failed payment shows appropriate error
- [ ] Webhook receives events (check server logs)

---

## 10. Deploy to Production

### Prepare for deployment

1. **Update environment variables** with live keys:

```env
NEXT_PUBLIC_PAYSTACK_PUBLIC_KEY=pk_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
PAYSTACK_SECRET_KEY=sk_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
NEXT_PUBLIC_APP_URL=https://yourdomain.com
```

2. **Update webhook URL** in Paystack dashboard to your production URL.

3. **Complete Paystack activation** by verifying your business in the dashboard.

### Deploy to Vercel

```bash
npm install -g vercel
vercel
```

Or connect your GitHub repository to Vercel for automatic deployments.

### Security checklist

- [ ] HTTPS enabled
- [ ] Secret key only used server-side
- [ ] Webhook signature verification enabled
- [ ] Amount verified on server before delivering value
- [ ] Environment variables set in hosting platform
- [ ] No sensitive data logged in production

---

## 11. Troubleshooting

### Common issues and solutions

**"Invalid public key"**
- Verify `NEXT_PUBLIC_PAYSTACK_PUBLIC_KEY` is set correctly
- Restart your development server after adding env variables
- Ensure the key starts with `pk_test_` or `pk_live_`

**Popup not opening**
- Check browser console for errors
- Ensure react-paystack is installed
- Verify the email is valid

**"Invalid signature" on webhooks**
- Confirm `PAYSTACK_SECRET_KEY` matches your dashboard
- Ensure no middleware is modifying the request body
- Check that you're reading the raw body for verification

**Verification returns "Transaction not found"**
- Wait a few seconds after payment before verifying
- Ensure the reference matches exactly (case-sensitive)

**CORS errors**
- Paystack API calls should only happen server-side
- Use API routes, not direct fetch from client components

### Getting help

- [Paystack Documentation](https://paystack.com/docs/)
- [Paystack API Reference](https://paystack.com/docs/api/)
- [react-paystack GitHub](https://github.com/iamraphson/react-paystack)
- [Next.js Documentation](https://nextjs.org/docs)

---

## Next Steps

Now that you have payments working, consider implementing:

- **Subscription payments** with Paystack Plans
- **Split payments** for marketplace applications
- **Saved cards** for returning customers
- **Multiple currencies** for international customers
- **Refund handling** via the Refund API

---

## Summary

In this guide, you learned how to:

1. Set up a Next.js project with Paystack
2. Create secure API routes for payment operations
3. Build a checkout component with the Paystack popup
4. Verify transactions server-side
5. Handle webhooks for reliable payment confirmation
6. Test and deploy your integration

Your Next.js application is now ready to accept payments securely through Paystack.

---

## References

- [Paystack API Documentation](https://paystack.com/docs/api/)
- [Accept Payments Guide](https://paystack.com/docs/payments/accept-payments/)
- [react-paystack Library](https://github.com/iamraphson/react-paystack)
- [Next.js App Router Documentation](https://nextjs.org/docs/app)

---

*Last updated: January 2026*
