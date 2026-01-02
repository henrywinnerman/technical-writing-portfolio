# Integrating Paystack Payments in Laravel

A step-by-step guide to accepting payments in your Laravel application using the Paystack API.

## Introduction

This guide walks you through integrating Paystack payment processing into a Laravel application. By the end, you'll be able to:

- Accept card, bank transfer, and USSD payments
- Verify transactions securely
- Handle payment webhooks
- Store transaction records in your database

**Prerequisites:**
- Laravel 10 or 11 installed
- Composer
- A Paystack account ([signup here](https://dashboard.paystack.co/#/signup))
- Basic knowledge of Laravel (routes, controllers, migrations)

**Estimated time:** 45-60 minutes

---

## Table of Contents

1. [Get Your API Keys](#1-get-your-api-keys)
2. [Install and Configure](#2-install-and-configure)
3. [Set Up the Database](#3-set-up-the-database)
4. [Create the Payment Flow](#4-create-the-payment-flow)
5. [Initialize Transactions](#5-initialize-transactions)
6. [Handle Callbacks and Verify Payments](#6-handle-callbacks-and-verify-payments)
7. [Implement Webhooks](#7-implement-webhooks)
8. [Test Your Integration](#8-test-your-integration)
9. [Go Live](#9-go-live)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Get Your API Keys

Before writing any code, you need your Paystack API keys.

1. Log in to your [Paystack Dashboard](https://dashboard.paystack.co/)
2. Navigate to **Settings** → **API Keys & Webhooks**
3. Copy your **Test Secret Key** (starts with `sk_test_`)
4. Copy your **Test Public Key** (starts with `pk_test_`)

> **Important:** Never expose your secret key in frontend code or commit it to version control. The secret key should only be used on your server.

---

## 2. Install and Configure

### Step 2.1: Install Guzzle HTTP Client

Laravel uses Guzzle for HTTP requests. If not already installed:

```bash
composer require guzzlehttp/guzzle
```

### Step 2.2: Add Environment Variables

Open your `.env` file and add:

```env
PAYSTACK_PUBLIC_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
PAYSTACK_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
PAYSTACK_PAYMENT_URL=https://api.paystack.co
```

### Step 2.3: Create a Configuration File

Create `config/paystack.php`:

```php
<?php

return [
    'public_key' => env('PAYSTACK_PUBLIC_KEY'),
    'secret_key' => env('PAYSTACK_SECRET_KEY'),
    'payment_url' => env('PAYSTACK_PAYMENT_URL', 'https://api.paystack.co'),
];
```

### Step 2.4: Create a Paystack Service Class

Create `app/Services/PaystackService.php`:

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Http;

class PaystackService
{
    protected string $secretKey;
    protected string $baseUrl;

    public function __construct()
    {
        $this->secretKey = config('paystack.secret_key');
        $this->baseUrl = config('paystack.payment_url');
    }

    /**
     * Initialize a transaction
     */
    public function initializeTransaction(array $data): array
    {
        $response = Http::withToken($this->secretKey)
            ->post("{$this->baseUrl}/transaction/initialize", $data);

        return $response->json();
    }

    /**
     * Verify a transaction
     */
    public function verifyTransaction(string $reference): array
    {
        $response = Http::withToken($this->secretKey)
            ->get("{$this->baseUrl}/transaction/verify/{$reference}");

        return $response->json();
    }

    /**
     * List transactions
     */
    public function listTransactions(array $params = []): array
    {
        $response = Http::withToken($this->secretKey)
            ->get("{$this->baseUrl}/transaction", $params);

        return $response->json();
    }

    /**
     * Fetch a transaction
     */
    public function fetchTransaction(int $id): array
    {
        $response = Http::withToken($this->secretKey)
            ->get("{$this->baseUrl}/transaction/{$id}");

        return $response->json();
    }

    /**
     * Charge an authorization (for recurring payments)
     */
    public function chargeAuthorization(array $data): array
    {
        $response = Http::withToken($this->secretKey)
            ->post("{$this->baseUrl}/transaction/charge_authorization", $data);

        return $response->json();
    }
}
```

### Step 2.5: Register the Service

In `app/Providers/AppServiceProvider.php`, add:

```php
use App\Services\PaystackService;

public function register(): void
{
    $this->app->singleton(PaystackService::class, function ($app) {
        return new PaystackService();
    });
}
```

---

## 3. Set Up the Database

### Step 3.1: Create a Migration for Transactions

```bash
php artisan make:migration create_transactions_table
```

Edit the migration file:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('transactions', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->string('reference')->unique();
            $table->integer('amount'); // Amount in kobo
            $table->string('email');
            $table->string('status')->default('pending');
            $table->string('channel')->nullable(); // card, bank, ussd, etc.
            $table->string('currency')->default('NGN');
            $table->string('ip_address')->nullable();
            $table->json('metadata')->nullable();
            $table->timestamp('paid_at')->nullable();
            $table->timestamps();

            $table->index('reference');
            $table->index('status');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('transactions');
    }
};
```

Run the migration:

```bash
php artisan migrate
```

### Step 3.2: Create the Transaction Model

```bash
php artisan make:model Transaction
```

Edit `app/Models/Transaction.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Transaction extends Model
{
    protected $fillable = [
        'user_id',
        'reference',
        'amount',
        'email',
        'status',
        'channel',
        'currency',
        'ip_address',
        'metadata',
        'paid_at',
    ];

    protected $casts = [
        'metadata' => 'array',
        'paid_at' => 'datetime',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Get amount in Naira (human-readable)
     */
    public function getAmountInNairaAttribute(): float
    {
        return $this->amount / 100;
    }

    /**
     * Generate a unique transaction reference
     */
    public static function generateReference(): string
    {
        return 'TXN_' . strtoupper(uniqid()) . '_' . time();
    }
}
```

---

## 4. Create the Payment Flow

### Understanding the Payment Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Customer  │────▶│ Your Server │────▶│  Paystack   │────▶│  Customer   │
│ clicks Pay  │     │ Initialize  │     │  Checkout   │     │ completes   │
└─────────────┘     └─────────────┘     └─────────────┘     │  payment    │
                                                            └──────┬──────┘
                                                                   │
┌─────────────┐     ┌─────────────┐     ┌─────────────┐            │
│   Deliver   │◀────│   Verify    │◀────│  Callback   │◀───────────┘
│   Value     │     │ Transaction │     │    URL      │
└─────────────┘     └─────────────┘     └─────────────┘
```

1. Customer clicks "Pay" on your site
2. Your server initializes the transaction with Paystack
3. Customer is redirected to Paystack Checkout
4. Customer completes payment
5. Paystack redirects to your callback URL
6. Your server verifies the transaction
7. You deliver value to the customer

---

## 5. Initialize Transactions

### Step 5.1: Create the Payment Controller

```bash
php artisan make: controller PaymentController
```

Edit `app/Http/Controllers/PaymentController.php`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Transaction;
use App\Services\PaystackService;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Log;

class PaymentController extends Controller
{
    public function __construct(
        protected PaystackService $paystack
    ) {}

    /**
     * Show the payment page
     */
    public function showPaymentPage()
    {
        return view('payment.checkout');
    }

    /**
     * Initialize a payment
     */
    public function initializePayment(Request $request)
    {
        $request->validate([
            'amount' => 'required|numeric|min:100', // Minimum ₦100
            'email' => 'required|email',
        ]);

        $user = Auth::user();
        $reference = Transaction::generateReference();
        $amountInKobo = $request->amount * 100; // Convert to kobo

        // Create a pending transaction record
        $transaction = Transaction::create([
            'user_id' => $user->id,
            'reference' => $reference,
            'amount' => $amountInKobo,
            'email' => $request->email,
            'status' => 'pending',
            'ip_address' => $request->ip(),
            'metadata' => [
                'user_id' => $user->id,
                'custom_fields' => [
                    [
                        'display_name' => 'Customer Name',
                        'variable_name' => 'customer_name',
                        'value' => $user->name,
                    ],
                ],
            ],
        ]);

        // Initialize with Paystack
        $response = $this->paystack->initializeTransaction([
            'email' => $request->email,
            'amount' => $amountInKobo,
            'reference' => $reference,
            'callback_url' => route('payment.callback'),
            'metadata' => $transaction->metadata,
            'channels' => ['card', 'bank', 'ussd', 'bank_transfer'],
        ]);

        if (!$response['status']) {
            Log::error('Paystack initialization failed', $response);
            
            $transaction->update(['status' => 'failed']);
            
            return back()->with('error', 'Unable to initialize payment. Please try again.');
        }

        // Redirect to Paystack Checkout
        return redirect($response['data']['authorization_url']);
    }
}
```

### Step 5.2: Create the Payment View

Create `resources/views/payment/checkout.blade.php`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Checkout</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center">
    <div class="bg-white p-8 rounded-lg shadow-md w-full max-w-md">
        <h1 class="text-2xl font-bold mb-6 text-center">Complete Payment</h1>
        
        @if(session('error'))
            <div class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded mb-4">
                {{ session('error') }}
            </div>
        @endif

        @if(session('success'))
            <div class="bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded mb-4">
                {{ session('success') }}
            </div>
        @endif

        <form action="{{ route('payment.initialize') }}" method="POST">
            @csrf
            
            <div class="mb-4">
                <label for="email" class="block text-gray-700 font-medium mb-2">
                    Email Address
                </label>
                <input 
                    type="email" 
                    id="email" 
                    name="email" 
                    value="{{ auth()->user()->email }}"
                    class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                    required
                >
            </div>

            <div class="mb-6">
                <label for="amount" class="block text-gray-700 font-medium mb-2">
                    Amount (₦)
                </label>
                <input 
                    type="number" 
                    id="amount" 
                    name="amount" 
                    min="100"
                    placeholder="Enter amount"
                    class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                    required
                >
                <p class="text-sm text-gray-500 mt-1">Minimum: ₦100</p>
            </div>

            <button 
                type="submit"
                class="w-full bg-blue-600 text-white py-3 rounded-lg font-medium hover:bg-blue-700 transition"
            >
                Pay with Paystack
            </button>
        </form>

        <div class="mt-6 flex items-center justify-center space-x-4 text-gray-400">
            <span class="text-sm">Secured by</span>
            <span class="font-semibold text-gray-600">Paystack</span>
        </div>
    </div>
</body>
</html>
```

### Step 5.3: Set Up Routes

Add to `routes/web.php`:

```php
use App\Http\Controllers\PaymentController;

Route::middleware('auth')->group(function () {
    Route::get('/payment', [PaymentController::class, 'showPaymentPage'])
        ->name('payment.page');
    
    Route::post('/payment/initialize', [PaymentController::class, 'initializePayment'])
        ->name('payment.initialize');
    
    Route::get('/payment/callback', [PaymentController::class, 'handleCallback'])
        ->name('payment.callback');
});
```

---

## 6. Handle Callbacks and Verify Payments

When a customer completes payment, Paystack redirects them to your callback URL with the transaction reference.

### Step 6.1: Add the Callback Method

Add to `PaymentController.php`:

```php
/**
 * Handle the payment callback
 */
public function handleCallback(Request $request)
{
    $reference = $request->query('reference');

    if (!$reference) {
        return redirect()->route('payment.page')
            ->with('error', 'No transaction reference provided.');
    }

    // Find the transaction
    $transaction = Transaction::where('reference', $reference)->first();

    if (!$transaction) {
        return redirect()->route('payment.page')
            ->with('error', 'Transaction not found.');
    }

    // Verify with Paystack
    $response = $this->paystack->verifyTransaction($reference);

    if (!$response['status']) {
        Log::error('Paystack verification failed', $response);
        
        return redirect()->route('payment.page')
            ->with('error', 'Unable to verify payment.');
    }

    $data = $response['data'];

    // Check if the transaction was successful
    if ($data['status'] === 'success') {
        // Verify the amount matches
        if ($data['amount'] !== $transaction->amount) {
            Log::warning('Amount mismatch', [
                'expected' => $transaction->amount,
                'received' => $data['amount'],
                'reference' => $reference,
            ]);
            
            return redirect()->route('payment.page')
                ->with('error', 'Payment amount mismatch. Please contact support.');
        }

        // Update transaction record
        $transaction->update([
            'status' => 'success',
            'channel' => $data['channel'],
            'paid_at' => now(),
        ]);

        // TODO: Deliver value to customer here
        // e.g., activate subscription, credit wallet, send confirmation email

        return redirect()->route('payment.success')
            ->with('success', 'Payment successful!');
    }

    // Handle failed or abandoned transactions
    $transaction->update([
        'status' => $data['status'], // 'failed' or 'abandoned'
    ]);

    return redirect()->route('payment.page')
        ->with('error', 'Payment was not successful. Please try again.');
}
```

### Step 6.2: Create Success Page

Add to `routes/web.php`:

```php
Route::get('/payment/success', function () {
    return view('payment.success');
})->name('payment.success');
```

Create `resources/views/payment/success.blade.php`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Payment Successful</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center">
    <div class="bg-white p-8 rounded-lg shadow-md w-full max-w-md text-center">
        <div class="w-16 h-16 bg-green-100 rounded-full flex items-center justify-center mx-auto mb-4">
            <svg class="w-8 h-8 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
            </svg>
        </div>
        
        <h1 class="text-2xl font-bold text-gray-800 mb-2">Payment Successful!</h1>
        <p class="text-gray-600 mb-6">Your transaction has been completed successfully.</p>
        
        <a href="{{ route('payment.page') }}" class="inline-block bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700 transition">
            Make Another Payment
        </a>
    </div>
</body>
</html>
```

---

## 7. Implement Webhooks

Webhooks provide a reliable way to receive payment notifications. Unlike callbacks, webhooks are server-to-server and don't depend on the customer's browser.

### Step 7.1: Create Webhook Controller

```bash
php artisan make:controller WebhookController
```

Edit `app/Http/Controllers/WebhookController.php`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Transaction;
use App\Services\PaystackService;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;

class WebhookController extends Controller
{
    public function __construct(
        protected PaystackService $paystack
    ) {}

    /**
     * Handle Paystack webhooks
     */
    public function handlePaystackWebhook(Request $request)
    {
        // Verify the webhook signature
        if (!$this->verifyWebhookSignature($request)) {
            Log::warning('Invalid Paystack webhook signature');
            return response()->json(['error' => 'Invalid signature'], 401);
        }

        $payload = $request->all();
        $event = $payload['event'];

        Log::info('Paystack webhook received', ['event' => $event]);

        switch ($event) {
            case 'charge.success':
                $this->handleChargeSuccess($payload['data']);
                break;

            case 'transfer.success':
                $this->handleTransferSuccess($payload['data']);
                break;

            case 'transfer.failed':
                $this->handleTransferFailed($payload['data']);
                break;

            default:
                Log::info('Unhandled Paystack event', ['event' => $event]);
        }

        return response()->json(['status' => 'success'], 200);
    }

    /**
     * Verify the webhook signature
     */
    protected function verifyWebhookSignature(Request $request): bool
    {
        $signature = $request->header('x-paystack-signature');
        
        if (!$signature) {
            return false;
        }

        $computedSignature = hash_hmac(
            'sha512',
            $request->getContent(),
            config('paystack.secret_key')
        );

        return hash_equals($signature, $computedSignature);
    }

    /**
     * Handle successful charge
     */
    protected function handleChargeSuccess(array $data): void
    {
        $reference = $data['reference'];
        
        $transaction = Transaction::where('reference', $reference)->first();

        if (!$transaction) {
            Log::warning('Transaction not found for webhook', ['reference' => $reference]);
            return;
        }

        // Prevent duplicate processing
        if ($transaction->status === 'success') {
            Log::info('Transaction already processed', ['reference' => $reference]);
            return;
        }

        // Verify the transaction
        $verifyResponse = $this->paystack->verifyTransaction($reference);

        if (!$verifyResponse['status'] || $verifyResponse['data']['status'] !== 'success') {
            Log::error('Transaction verification failed in webhook', ['reference' => $reference]);
            return;
        }

        // Update transaction
        $transaction->update([
            'status' => 'success',
            'channel' => $data['channel'],
            'paid_at' => now(),
        ]);

        // TODO: Deliver value to customer
        // Send confirmation email, activate service, etc.

        Log::info('Transaction processed via webhook', ['reference' => $reference]);
    }

    /**
     * Handle successful transfer
     */
    protected function handleTransferSuccess(array $data): void
    {
        // Handle transfer success if you're doing payouts
        Log::info('Transfer successful', ['reference' => $data['reference']]);
    }

    /**
     * Handle failed transfer
     */
    protected function handleTransferFailed(array $data): void
    {
        // Handle transfer failure
        Log::error('Transfer failed', ['reference' => $data['reference']]);
    }
}
```

### Step 7.2: Add Webhook Route

Add to `routes/web.php`:

```php
use App\Http\Controllers\WebhookController;

Route::post('/webhooks/paystack', [WebhookController::class, 'handlePaystackWebhook'])
    ->name('webhooks.paystack');
```

### Step 7.3: Disable CSRF for Webhooks

Edit `app/Http/Middleware/VerifyCsrfToken.php`:

```php
protected $except = [
    'webhooks/*',
];
```

### Step 7.4: Configure Webhook URL in Paystack

1. Go to your [Paystack Dashboard](https://dashboard.paystack.co/)
2. Navigate to **Settings** → **API Keys & Webhooks**
3. Enter your webhook URL: `https://yourdomain.com/webhooks/paystack`
4. Save the settings

---

## 8. Test Your Integration

### Step 8.1: Test Card Numbers

Paystack provides test cards for different scenarios:

| Card Number | Expiry | CVV | Result |
|------------|--------|-----|--------|
| 4084 0840 8408 4081 | Any future date | 408 | Successful transaction |
| 4084 0840 8408 4081 | Any future date | 400 | Failed transaction |
| 5060 6666 6666 6666 666 | Any future date | 123 | Verve card test |

### Step 8.2: Test Bank Transfer

When testing bank transfers, Paystack displays test bank account details. The transfer is automatically marked as successful in test mode.

### Step 8.3: Testing Checklist

- [ ] Payment page loads correctly
- [ ] Transaction initializes and redirects to Paystack
- [ ] Successful payment redirects to callback URL
- [ ] Transaction status updates to "success"
- [ ] Failed payment shows appropriate error
- [ ] Webhook receives and processes events
- [ ] Amount verification works correctly
- [ ] Duplicate transactions are prevented

---

## 9. Go Live

When you're ready to accept real payments:

### Step 9.1: Complete Paystack Activation

1. Log in to [Paystack Dashboard](https://dashboard.paystack.co/)
2. Complete your business verification
3. Wait for approval (usually 24-48 hours)

### Step 9.2: Update Environment Variables

Replace test keys with live keys in `.env`:

```env
PAYSTACK_PUBLIC_KEY=pk_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
PAYSTACK_SECRET_KEY=sk_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Step 9.3: Update Webhook URL

Update your webhook URL in Paystack dashboard to your production URL.

### Step 9.4: Security Checklist

- [ ] HTTPS enabled on your domain
- [ ] Secret key stored securely (not in code)
- [ ] Webhook signature verification enabled
- [ ] Amount verification implemented
- [ ] Error logging configured
- [ ] Database backups configured

---

## 10. Troubleshooting

### Common Issues and Solutions

**"Authorization URL not received"**
- Check that your secret key is correct
- Ensure your Paystack account is active
- Verify the amount is in kobo (multiply by 100)

**"Invalid signature" on webhooks**
- Confirm you're using the correct secret key
- Check that the webhook URL matches exactly
- Ensure no middleware is modifying the request body

**"Transaction not found"**
- Verify the reference is being stored correctly
- Check database connection
- Ensure the reference matches exactly (case-sensitive)

**Callback not redirecting**
- Verify callback URL is registered in Paystack dashboard
- Check route is correctly defined
- Ensure no authentication middleware is blocking the route

### Getting Help

- [Paystack Documentation](https://paystack.com/docs/)
- [Paystack API Reference](https://paystack.com/docs/api/)
- [Paystack Support](https://support.paystack.com/)

---

## Next Steps

Now that you have basic payments working, consider implementing:

- **Recurring payments** using Paystack subscriptions
- **Split payments** for marketplace applications
- **Refunds** via the Refund API
- **Payment links** for invoice-based payments
- **Multi-currency** support for international customers

---

## References

- [Paystack API Documentation](https://paystack.com/docs/api/)
- [Accept Payments Guide](https://paystack.com/docs/payments/accept-payments/)
- [Verify Payments Guide](https://paystack.com/docs/payments/verify-payments/)
- [Laravel HTTP Client](https://laravel.com/docs/http-client)

---

*Last updated: January 2026*
