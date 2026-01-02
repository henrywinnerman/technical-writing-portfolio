# Flutterwave Virtual Cards API Reference

## Overview

The Flutterwave Virtual Cards API enables you to programmatically create, fund, and manage virtual cards for your users. Virtual cards can be used for online purchases, subscription payments, and expense management.

**Base URL:** `https://api.flutterwave.com/v3`

**Use cases:**
- Issue virtual cards to customers for online payments
- Create expense cards for employee spending
- Enable users to make international purchases
- Build fintech applications with card issuing capabilities

---

## Authentication

All API requests require authentication using your secret key in the Authorization header.

```
Authorization: Bearer YOUR_SECRET_KEY
```

**Request headers:**

| Header | Value | Required |
|--------|-------|----------|
| Authorization | Bearer {secret_key} | Yes |
| Content-Type | application/json | Yes |

> ⚠️ **Security:** Never expose your secret key in client-side code. All API calls must be made from your server.

---

## Supported Currencies

| Currency | Card Type | Availability |
|----------|-----------|--------------|
| NGN | Naira Card | Nigeria |
| USD | Dollar Card | All supported countries |

---

## Endpoints

### Create Virtual Card

Creates a new virtual card for a user.

**Endpoint:** `POST /virtual-cards`

**Request body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| currency | string | Yes | Card currency (`NGN` or `USD`) |
| amount | number | Yes | Initial funding amount (minimum varies by currency) |
| billing_name | string | Yes | Cardholder name (as it appears on card) |
| billing_address | string | Yes | Cardholder billing address |
| billing_city | string | Yes | Billing city |
| billing_state | string | Yes | Billing state |
| billing_postal_code | string | Yes | Billing postal/ZIP code |
| billing_country | string | Yes | Billing country (ISO 3166-1 alpha-2) |
| callback_url | string | No | URL for transaction webhooks |
| debit_currency | string | No | Currency to debit from your balance |

**Example request:**

```bash
curl -X POST https://api.flutterwave.com/v3/virtual-cards \
  -H "Authorization: Bearer YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "currency": "USD",
    "amount": 50,
    "billing_name": "Adaeze Okonkwo",
    "billing_address": "123 Main Street",
    "billing_city": "Lagos",
    "billing_state": "Lagos",
    "billing_postal_code": "100001",
    "billing_country": "NG",
    "callback_url": "https://yourapp.com/webhooks/card"
  }'
```

**Example response (201 Created):**

```json
{
  "status": "success",
  "message": "Virtual card created successfully",
  "data": {
    "id": "vc_1234567890abcdef",
    "account_id": 12345,
    "amount": 50,
    "currency": "USD",
    "card_hash": "vc_abcdef1234567890",
    "card_pan": "5366 1234 5678 9012",
    "masked_pan": "5366 **** **** 9012",
    "cvv": "123",
    "expiration": "12/28",
    "card_type": "mastercard",
    "name_on_card": "ADAEZE OKONKWO",
    "billing_address": "123 Main Street",
    "billing_city": "Lagos",
    "billing_state": "Lagos",
    "billing_postal_code": "100001",
    "billing_country": "NG",
    "state": "active",
    "created_at": "2026-01-02T10:30:00.000Z",
    "callback_url": "https://yourapp.com/webhooks/card"
  }
}
```

**Error responses:**

| Status | Code | Description |
|--------|------|-------------|
| 400 | BAD_REQUEST | Invalid or missing parameters |
| 401 | UNAUTHORIZED | Invalid or missing API key |
| 402 | INSUFFICIENT_FUNDS | Insufficient balance to fund card |
| 422 | VALIDATION_ERROR | Validation failed for one or more fields |

---

### Get All Virtual Cards

Retrieves a paginated list of all virtual cards on your account.

**Endpoint:** `GET /virtual-cards`

**Query parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number (default: 1) |
| limit | integer | No | Results per page (default: 10, max: 100) |

**Example request:**

```bash
curl -X GET "https://api.flutterwave.com/v3/virtual-cards?page=1&limit=10" \
  -H "Authorization: Bearer YOUR_SECRET_KEY"
```

**Example response (200 OK):**

```json
{
  "status": "success",
  "message": "Virtual cards retrieved successfully",
  "meta": {
    "page_info": {
      "total": 25,
      "current_page": 1,
      "total_pages": 3
    }
  },
  "data": [
    {
      "id": "vc_1234567890abcdef",
      "account_id": 12345,
      "currency": "USD",
      "masked_pan": "5366 **** **** 9012",
      "card_type": "mastercard",
      "name_on_card": "ADAEZE OKONKWO",
      "state": "active",
      "balance": 45.50,
      "created_at": "2026-01-02T10:30:00.000Z"
    },
    {
      "id": "vc_0987654321fedcba",
      "account_id": 12345,
      "currency": "NGN",
      "masked_pan": "5061 **** **** 3456",
      "card_type": "mastercard",
      "name_on_card": "CHUKWUEMEKA NWOSU",
      "state": "active",
      "balance": 25000.00,
      "created_at": "2026-01-01T14:20:00.000Z"
    }
  ]
}
```

---

### Get Virtual Card

Retrieves details of a specific virtual card including full card details.

**Endpoint:** `GET /virtual-cards/{id}`

**Path parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | Virtual card ID |

**Example request:**

```bash
curl -X GET https://api.flutterwave.com/v3/virtual-cards/vc_1234567890abcdef \
  -H "Authorization: Bearer YOUR_SECRET_KEY"
```

**Example response (200 OK):**

```json
{
  "status": "success",
  "message": "Virtual card retrieved successfully",
  "data": {
    "id": "vc_1234567890abcdef",
    "account_id": 12345,
    "amount": 50,
    "currency": "USD",
    "card_hash": "vc_abcdef1234567890",
    "card_pan": "5366 1234 5678 9012",
    "masked_pan": "5366 **** **** 9012",
    "cvv": "123",
    "expiration": "12/28",
    "card_type": "mastercard",
    "name_on_card": "ADAEZE OKONKWO",
    "billing_address": "123 Main Street",
    "billing_city": "Lagos",
    "billing_state": "Lagos",
    "billing_postal_code": "100001",
    "billing_country": "NG",
    "state": "active",
    "balance": 45.50,
    "spend_limit": 5000,
    "created_at": "2026-01-02T10:30:00.000Z",
    "callback_url": "https://yourapp.com/webhooks/card"
  }
}
```

**Error responses:**

| Status | Code | Description |
|--------|------|-------------|
| 404 | NOT_FOUND | Virtual card not found |

---

### Fund Virtual Card

Adds funds to an existing virtual card.

**Endpoint:** `POST /virtual-cards/{id}/fund`

**Path parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | Virtual card ID |

**Request body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| amount | number | Yes | Amount to fund |
| debit_currency | string | No | Currency to debit from your balance |

**Example request:**

```bash
curl -X POST https://api.flutterwave.com/v3/virtual-cards/vc_1234567890abcdef/fund \
  -H "Authorization: Bearer YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 100,
    "debit_currency": "NGN"
  }'
```

**Example response (200 OK):**

```json
{
  "status": "success",
  "message": "Card funded successfully",
  "data": {
    "id": "vc_1234567890abcdef",
    "currency": "USD",
    "amount_funded": 100,
    "previous_balance": 45.50,
    "new_balance": 145.50,
    "debit_currency": "NGN",
    "amount_debited": 160000,
    "exchange_rate": 1600,
    "funded_at": "2026-01-02T12:00:00.000Z"
  }
}
```

**Error responses:**

| Status | Code | Description |
|--------|------|-------------|
| 400 | BAD_REQUEST | Invalid amount |
| 402 | INSUFFICIENT_FUNDS | Insufficient balance |
| 404 | NOT_FOUND | Virtual card not found |
| 422 | CARD_INACTIVE | Cannot fund inactive or blocked card |

---

### Withdraw from Virtual Card

Withdraws funds from a virtual card back to your Flutterwave balance.

**Endpoint:** `POST /virtual-cards/{id}/withdraw`

**Path parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | Virtual card ID |

**Request body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| amount | number | Yes | Amount to withdraw |

**Example request:**

```bash
curl -X POST https://api.flutterwave.com/v3/virtual-cards/vc_1234567890abcdef/withdraw \
  -H "Authorization: Bearer YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50
  }'
```

**Example response (200 OK):**

```json
{
  "status": "success",
  "message": "Withdrawal successful",
  "data": {
    "id": "vc_1234567890abcdef",
    "currency": "USD",
    "amount_withdrawn": 50,
    "previous_balance": 145.50,
    "new_balance": 95.50,
    "withdrawn_at": "2026-01-02T14:30:00.000Z"
  }
}
```

**Error responses:**

| Status | Code | Description |
|--------|------|-------------|
| 400 | INSUFFICIENT_CARD_BALANCE | Card balance is less than withdrawal amount |
| 404 | NOT_FOUND | Virtual card not found |
| 422 | CARD_INACTIVE | Cannot withdraw from inactive or blocked card |

---

### Block/Unblock Virtual Card

Temporarily blocks or unblocks a virtual card.

**Endpoint:** `PUT /virtual-cards/{id}/status/{action}`

**Path parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | Virtual card ID |
| action | string | Yes | `block` or `unblock` |

**Example request (Block):**

```bash
curl -X PUT https://api.flutterwave.com/v3/virtual-cards/vc_1234567890abcdef/status/block \
  -H "Authorization: Bearer YOUR_SECRET_KEY"
```

**Example response (200 OK):**

```json
{
  "status": "success",
  "message": "Card blocked successfully",
  "data": {
    "id": "vc_1234567890abcdef",
    "state": "blocked",
    "blocked_at": "2026-01-02T15:00:00.000Z"
  }
}
```

**Example request (Unblock):**

```bash
curl -X PUT https://api.flutterwave.com/v3/virtual-cards/vc_1234567890abcdef/status/unblock \
  -H "Authorization: Bearer YOUR_SECRET_KEY"
```

**Example response (200 OK):**

```json
{
  "status": "success",
  "message": "Card unblocked successfully",
  "data": {
    "id": "vc_1234567890abcdef",
    "state": "active",
    "unblocked_at": "2026-01-02T16:00:00.000Z"
  }
}
```

---

### Terminate Virtual Card

Permanently terminates a virtual card. This action cannot be undone.

**Endpoint:** `PUT /virtual-cards/{id}/terminate`

**Path parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | Virtual card ID |

**Example request:**

```bash
curl -X PUT https://api.flutterwave.com/v3/virtual-cards/vc_1234567890abcdef/terminate \
  -H "Authorization: Bearer YOUR_SECRET_KEY"
```

**Example response (200 OK):**

```json
{
  "status": "success",
  "message": "Card terminated successfully",
  "data": {
    "id": "vc_1234567890abcdef",
    "state": "terminated",
    "remaining_balance": 95.50,
    "balance_refunded": true,
    "terminated_at": "2026-01-02T17:00:00.000Z"
  }
}
```

> **Note:** Remaining balance on terminated cards is automatically refunded to your Flutterwave balance.

---

### Get Card Transactions

Retrieves transaction history for a specific virtual card.

**Endpoint:** `GET /virtual-cards/{id}/transactions`

**Path parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | Virtual card ID |

**Query parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| from | string | No | Start date (YYYY-MM-DD) |
| to | string | No | End date (YYYY-MM-DD) |
| page | integer | No | Page number (default: 1) |
| limit | integer | No | Results per page (default: 20, max: 100) |

**Example request:**

```bash
curl -X GET "https://api.flutterwave.com/v3/virtual-cards/vc_1234567890abcdef/transactions?from=2026-01-01&to=2026-01-31&page=1&limit=20" \
  -H "Authorization: Bearer YOUR_SECRET_KEY"
```

**Example response (200 OK):**

```json
{
  "status": "success",
  "message": "Card transactions retrieved successfully",
  "meta": {
    "page_info": {
      "total": 15,
      "current_page": 1,
      "total_pages": 1
    }
  },
  "data": [
    {
      "id": "txn_abc123def456",
      "card_id": "vc_1234567890abcdef",
      "amount": 12.99,
      "currency": "USD",
      "merchant_name": "NETFLIX.COM",
      "merchant_city": "LOS GATOS",
      "merchant_country": "US",
      "merchant_category": "Streaming Services",
      "mcc": "5815",
      "type": "debit",
      "status": "successful",
      "reference": "FLW-TXN-1234567890",
      "narration": "Card payment to NETFLIX.COM",
      "balance_after": 132.51,
      "created_at": "2026-01-15T08:30:00.000Z"
    },
    {
      "id": "txn_def456ghi789",
      "card_id": "vc_1234567890abcdef",
      "amount": 100,
      "currency": "USD",
      "type": "credit",
      "status": "successful",
      "reference": "FLW-FUND-0987654321",
      "narration": "Card funding",
      "balance_after": 145.50,
      "created_at": "2026-01-02T12:00:00.000Z"
    }
  ]
}
```

**Transaction types:**

| Type | Description |
|------|-------------|
| credit | Funds added to card (funding) |
| debit | Funds deducted from card (purchase or withdrawal) |

**Transaction statuses:**

| Status | Description |
|--------|-------------|
| successful | Transaction completed |
| pending | Transaction is being processed |
| failed | Transaction failed |
| reversed | Transaction was reversed |

---

## Webhooks

Receive real-time notifications for card events by configuring a webhook URL.

### Webhook Events

| Event | Description |
|-------|-------------|
| virtualcard.transaction | Card was used for a transaction |
| virtualcard.funding | Card was funded |
| virtualcard.withdrawal | Funds withdrawn from card |
| virtualcard.blocked | Card was blocked |
| virtualcard.terminated | Card was terminated |

### Webhook Payload Structure

```json
{
  "event": "virtualcard.transaction",
  "data": {
    "id": "txn_abc123def456",
    "card_id": "vc_1234567890abcdef",
    "amount": 12.99,
    "currency": "USD",
    "merchant_name": "NETFLIX.COM",
    "type": "debit",
    "status": "successful",
    "reference": "FLW-TXN-1234567890",
    "balance_after": 132.51,
    "created_at": "2026-01-15T08:30:00.000Z"
  },
  "event.type": "virtualcard.transaction"
}
```

### Verifying Webhook Signatures

Verify webhook authenticity using the `verif-hash` header:

```javascript
const crypto = require('crypto');

function verifyWebhook(payload, signature, secretHash) {
  return signature === secretHash;
}

// In your webhook handler
app.post('/webhooks/card', (req, res) => {
  const signature = req.headers['verif-hash'];
  const secretHash = process.env.FLUTTERWAVE_SECRET_HASH;
  
  if (!verifyWebhook(req.body, signature, secretHash)) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process webhook
  const event = req.body;
  console.log('Received event:', event.event);
  
  res.status(200).send('OK');
});
```

---

## Error Handling

### Error Response Format

All errors follow a consistent format:

```json
{
  "status": "error",
  "message": "Description of what went wrong",
  "data": null
}
```

### Common Error Codes

| HTTP Status | Code | Description |
|-------------|------|-------------|
| 400 | BAD_REQUEST | Invalid request parameters |
| 401 | UNAUTHORIZED | Missing or invalid API key |
| 402 | INSUFFICIENT_FUNDS | Not enough balance for operation |
| 403 | FORBIDDEN | Action not permitted |
| 404 | NOT_FOUND | Resource not found |
| 422 | VALIDATION_ERROR | Request validation failed |
| 429 | RATE_LIMITED | Too many requests |
| 500 | SERVER_ERROR | Internal server error |

---

## Rate Limits

| Endpoint Category | Limit |
|-------------------|-------|
| Card creation | 100 requests/minute |
| Card funding | 200 requests/minute |
| Card queries | 500 requests/minute |

Rate limit headers are included in all responses:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1704207600
```

---

## Code Examples

### Node.js

```javascript
const axios = require('axios');

const flutterwave = axios.create({
  baseURL: 'https://api.flutterwave.com/v3',
  headers: {
    'Authorization': `Bearer ${process.env.FLUTTERWAVE_SECRET_KEY}`,
    'Content-Type': 'application/json'
  }
});

// Create a virtual card
async function createVirtualCard(data) {
  try {
    const response = await flutterwave.post('/virtual-cards', {
      currency: data.currency,
      amount: data.amount,
      billing_name: data.name,
      billing_address: data.address,
      billing_city: data.city,
      billing_state: data.state,
      billing_postal_code: data.postalCode,
      billing_country: data.country
    });
    
    return response.data;
  } catch (error) {
    console.error('Error creating card:', error.response?.data);
    throw error;
  }
}

// Fund a virtual card
async function fundCard(cardId, amount) {
  try {
    const response = await flutterwave.post(`/virtual-cards/${cardId}/fund`, {
      amount: amount
    });
    
    return response.data;
  } catch (error) {
    console.error('Error funding card:', error.response?.data);
    throw error;
  }
}

// Get card transactions
async function getCardTransactions(cardId, options = {}) {
  try {
    const response = await flutterwave.get(`/virtual-cards/${cardId}/transactions`, {
      params: {
        from: options.from,
        to: options.to,
        page: options.page || 1,
        limit: options.limit || 20
      }
    });
    
    return response.data;
  } catch (error) {
    console.error('Error fetching transactions:', error.response?.data);
    throw error;
  }
}
```

### Python

```python
import requests
import os

class FlutterwaveCards:
    def __init__(self):
        self.base_url = 'https://api.flutterwave.com/v3'
        self.headers = {
            'Authorization': f'Bearer {os.environ.get("FLUTTERWAVE_SECRET_KEY")}',
            'Content-Type': 'application/json'
        }
    
    def create_card(self, data):
        """Create a new virtual card"""
        payload = {
            'currency': data['currency'],
            'amount': data['amount'],
            'billing_name': data['name'],
            'billing_address': data['address'],
            'billing_city': data['city'],
            'billing_state': data['state'],
            'billing_postal_code': data['postal_code'],
            'billing_country': data['country']
        }
        
        response = requests.post(
            f'{self.base_url}/virtual-cards',
            headers=self.headers,
            json=payload
        )
        
        return response.json()
    
    def fund_card(self, card_id, amount):
        """Fund an existing virtual card"""
        response = requests.post(
            f'{self.base_url}/virtual-cards/{card_id}/fund',
            headers=self.headers,
            json={'amount': amount}
        )
        
        return response.json()
    
    def get_card(self, card_id):
        """Retrieve card details"""
        response = requests.get(
            f'{self.base_url}/virtual-cards/{card_id}',
            headers=self.headers
        )
        
        return response.json()
    
    def block_card(self, card_id):
        """Block a virtual card"""
        response = requests.put(
            f'{self.base_url}/virtual-cards/{card_id}/status/block',
            headers=self.headers
        )
        
        return response.json()


# Usage
cards = FlutterwaveCards()

# Create a card
new_card = cards.create_card({
    'currency': 'USD',
    'amount': 50,
    'name': 'Adaeze Okonkwo',
    'address': '123 Main Street',
    'city': 'Lagos',
    'state': 'Lagos',
    'postal_code': '100001',
    'country': 'NG'
})

print(new_card)
```

---

## Testing

### Test Mode

Use your test API keys to simulate transactions without real money movement.

**Test API key format:** `FLWSECK_TEST-xxxxxxxxxxxxxxxx`

### Test Scenarios

| Scenario | How to Test |
|----------|-------------|
| Successful card creation | Use valid billing details |
| Insufficient funds | Attempt to create card with amount exceeding balance |
| Card funding | Fund card after creation |
| Card transactions | Use card details on test merchant sites |

---

## Best Practices

1. **Store card IDs securely** — Never expose full card details in logs or client-side code

2. **Implement webhooks** — Don't rely solely on synchronous responses for transaction status

3. **Handle rate limits** — Implement exponential backoff for retry logic

4. **Verify webhook signatures** — Always validate webhook authenticity before processing

5. **Monitor card balances** — Set up alerts for low balances to ensure uninterrupted service

6. **Use idempotency keys** — Prevent duplicate transactions by including unique references

---

## Support

- **Documentation:** [developer.flutterwave.com](https://developer.flutterwave.com)
- **API Status:** [status.flutterwave.com](https://status.flutterwave.com)
- **Developer Support:** developers@flutterwave.com
- **Community Forum:** [community.flutterwave.com](https://community.flutterwave.com)

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 3.0 | 2024-01-15 | Added USD card support |
| 2.5 | 2023-09-01 | Added webhook events |
| 2.0 | 2023-03-10 | Initial virtual cards release |

---

*Last updated: January 2026*
