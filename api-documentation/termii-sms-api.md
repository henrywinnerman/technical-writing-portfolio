# Termii SMS & OTP Verification API Reference

## Overview

The Termii API enables businesses to send SMS messages, deliver OTP (One-Time Password) tokens, and verify user phone numbers across multiple channels including SMS, WhatsApp, and Voice.

**Base URL:** `https://api.ng.termii.com`

**Use cases:**
- User phone number verification during registration
- Two-factor authentication (2FA)
- Transaction alerts and notifications
- Marketing campaigns and bulk messaging
- Password reset via OTP

---

## Authentication

All API requests require your API key passed in the request body.

```json
{
  "api_key": "YOUR_API_KEY"
}
```

**Getting your API key:**
1. Log in to your [Termii Dashboard](https://accounts.termii.com/)
2. Navigate to **Settings** → **API Keys**
3. Copy your API key

> ⚠️ **Security:** Never expose your API key in client-side code. All API calls must be made from your server.

---

## Messaging Channels

| Channel | Description | Use Case |
|---------|-------------|----------|
| `generic` | Promotional SMS route | Marketing messages, non-urgent notifications |
| `dnd` | Transactional SMS route (DND-enabled) | OTPs, alerts, transaction notifications |
| `whatsapp` | WhatsApp messaging | Conversational messages, rich media |
| `voice` | Voice call with text-to-speech | OTP delivery via phone call |

> **Note:** The `generic` route does not deliver to numbers on Do-Not-Disturb (DND) and has time restrictions in Nigeria (no delivery 8PM-8AM WAT for MTN). Use `dnd` route for OTPs and transactional messages.

---

## Endpoints

### Send SMS

Sends a single SMS message to a recipient.

**Endpoint:** `POST /api/sms/send`

**Request body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| api_key | string | Yes | Your Termii API key |
| to | string | Yes | Recipient phone number (with country code) |
| from | string | Yes | Registered Sender ID |
| sms | string | Yes | Message content |
| type | string | Yes | `plain` for text messages |
| channel | string | Yes | `generic`, `dnd`, or `whatsapp` |

**Example request:**

```bash
curl -X POST https://api.ng.termii.com/api/sms/send \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_API_KEY",
    "to": "2348012345678",
    "from": "MyApp",
    "sms": "Welcome to MyApp! Your account has been created successfully.",
    "type": "plain",
    "channel": "generic"
  }'
```

**Example response (200 OK):**

```json
{
  "code": "ok",
  "message_id": "9122821270554876574",
  "message": "Successfully Sent",
  "balance": 1047.57,
  "user": "John Doe",
  "message_id_str": "9122821270554876574"
}
```

**Error responses:**

| Code | Description |
|------|-------------|
| `INVALID_API_KEY` | API key is invalid or missing |
| `INSUFFICIENT_BALANCE` | Account balance is too low |
| `INVALID_SENDER_ID` | Sender ID not registered |
| `INVALID_PHONE_NUMBER` | Phone number format is invalid |

---

### Send Bulk SMS

Sends SMS to multiple recipients (up to 10,000 numbers).

**Endpoint:** `POST /api/sms/send/bulk`

**Request body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| api_key | string | Yes | Your Termii API key |
| to | array | Yes | Array of recipient phone numbers |
| from | string | Yes | Registered Sender ID |
| sms | string | Yes | Message content |
| type | string | Yes | `plain` for text messages |
| channel | string | Yes | `generic` or `dnd` |

**Example request:**

```bash
curl -X POST https://api.ng.termii.com/api/sms/send/bulk \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_API_KEY",
    "to": ["2348012345678", "2348098765432", "2349011223344"],
    "from": "MyApp",
    "sms": "Flash Sale! Get 50% off all items today only. Shop now at myapp.com",
    "type": "plain",
    "channel": "generic"
  }'
```

**Example response (200 OK):**

```json
{
  "code": "ok",
  "message_id": "9122821270554876575",
  "message": "Successfully Sent",
  "balance": 1041.57,
  "user": "John Doe",
  "message_id_str": "9122821270554876575"
}
```

---

### Send OTP Token

Generates and sends a one-time password to a phone number.

**Endpoint:** `POST /api/sms/otp/send`

**Request body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| api_key | string | Yes | Your Termii API key |
| message_type | string | Yes | `NUMERIC`, `ALPHANUMERIC`, or `ALPHA` |
| to | string | Yes | Recipient phone number |
| from | string | Yes | Registered Sender ID |
| channel | string | Yes | `generic`, `dnd`, `whatsapp`, or `voice` |
| pin_attempts | integer | Yes | Maximum verification attempts (1-10) |
| pin_time_to_live | integer | Yes | Token validity in minutes |
| pin_length | integer | Yes | Length of OTP (4-8) |
| pin_placeholder | string | Yes | Placeholder for OTP in message (e.g., `< 1234 >`) |
| message_text | string | Yes | Message template with pin_placeholder |
| pin_type | string | Yes | `NUMERIC`, `ALPHANUMERIC`, or `ALPHA` |

**Example request:**

```bash
curl -X POST https://api.ng.termii.com/api/sms/otp/send \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_API_KEY",
    "message_type": "NUMERIC",
    "to": "2348012345678",
    "from": "MyApp",
    "channel": "dnd",
    "pin_attempts": 3,
    "pin_time_to_live": 10,
    "pin_length": 6,
    "pin_placeholder": "< 1234 >",
    "message_text": "Your MyApp verification code is < 1234 >. Valid for 10 minutes. Do not share.",
    "pin_type": "NUMERIC"
  }'
```

**Example response (200 OK):**

```json
{
  "pinId": "29ae67c2-c8e1-4165-8a51-8d3d7c298081",
  "to": "2348012345678",
  "smsStatus": "Message Sent"
}
```

**Response fields:**

| Field | Description |
|-------|-------------|
| pinId | Unique identifier for OTP verification |
| to | Recipient phone number |
| smsStatus | Delivery status |

> **Important:** Store the `pinId` — you'll need it to verify the OTP.

---

### Verify OTP Token

Validates an OTP entered by the user.

**Endpoint:** `POST /api/sms/otp/verify`

**Request body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| api_key | string | Yes | Your Termii API key |
| pin_id | string | Yes | The pinId from send OTP response |
| pin | string | Yes | OTP entered by user |

**Example request:**

```bash
curl -X POST https://api.ng.termii.com/api/sms/otp/verify \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_API_KEY",
    "pin_id": "29ae67c2-c8e1-4165-8a51-8d3d7c298081",
    "pin": "123456"
  }'
```

**Example response (200 OK) - Verified:**

```json
{
  "pinId": "29ae67c2-c8e1-4165-8a51-8d3d7c298081",
  "verified": "True",
  "msisdn": "2348012345678"
}
```

**Example response - Expired:**

```json
{
  "pinId": "29ae67c2-c8e1-4165-8a51-8d3d7c298081",
  "verified": "Expired",
  "msisdn": "2348012345678"
}
```

**Example response - Failed (wrong PIN):**

```json
{
  "pinId": "29ae67c2-c8e1-4165-8a51-8d3d7c298081",
  "verified": "False",
  "attemptsRemaining": 2,
  "msisdn": "2348012345678"
}
```

**Verification statuses:**

| Status | Description |
|--------|-------------|
| `True` | OTP verified successfully |
| `False` | Incorrect OTP entered |
| `Expired` | OTP has expired |

---

### Send Voice OTP

Delivers OTP via automated voice call.

**Endpoint:** `POST /api/sms/otp/send/voice`

**Request body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| api_key | string | Yes | Your Termii API key |
| phone_number | string | Yes | Recipient phone number |
| pin_attempts | integer | Yes | Maximum verification attempts |
| pin_time_to_live | integer | Yes | Token validity in minutes |
| pin_length | integer | Yes | Length of OTP (4-8) |

**Example request:**

```bash
curl -X POST https://api.ng.termii.com/api/sms/otp/send/voice \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_API_KEY",
    "phone_number": "2348012345678",
    "pin_attempts": 3,
    "pin_time_to_live": 5,
    "pin_length": 6
  }'
```

**Example response (200 OK):**

```json
{
  "code": "ok",
  "balance": 793.57,
  "message_id": "3017545716011763238246705",
  "pinId": "19d41fdc-d38f-458e-89fc-1fa3a3b5e257",
  "message": "Successfully Sent"
}
```

---

### In-App OTP Generation

Generates an OTP without sending it (for in-app display or custom delivery).

**Endpoint:** `POST /api/sms/otp/generate`

**Request body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| api_key | string | Yes | Your Termii API key |
| pin_type | string | Yes | `NUMERIC`, `ALPHANUMERIC`, or `ALPHA` |
| phone_number | string | Yes | Phone number to associate OTP with |
| pin_attempts | integer | Yes | Maximum verification attempts |
| pin_time_to_live | integer | Yes | Token validity in minutes |
| pin_length | integer | Yes | Length of OTP (4-8) |

**Example request:**

```bash
curl -X POST https://api.ng.termii.com/api/sms/otp/generate \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_API_KEY",
    "pin_type": "NUMERIC",
    "phone_number": "2348012345678",
    "pin_attempts": 3,
    "pin_time_to_live": 10,
    "pin_length": 6
  }'
```

**Example response (200 OK):**

```json
{
  "status": "success",
  "data": {
    "pin_id": "c8dcd048-5e7f-4347-8c89-4470c3af0b",
    "otp": "195558",
    "phone_number": "2348012345678",
    "phone_number_other": "Termii"
  }
}
```

> **Use case:** Display OTP in your mobile app or send via your own channel.

---

### Get Account Balance

Retrieves your current wallet balance.

**Endpoint:** `GET /api/get-balance?api_key=YOUR_API_KEY`

**Example request:**

```bash
curl -X GET "https://api.ng.termii.com/api/get-balance?api_key=YOUR_API_KEY"
```

**Example response (200 OK):**

```json
{
  "user": "John Doe",
  "balance": 1047.57,
  "currency": "NGN"
}
```

---

### Fetch Sender IDs

Retrieves all registered Sender IDs on your account.

**Endpoint:** `GET /api/sender-id?api_key=YOUR_API_KEY`

**Example request:**

```bash
curl -X GET "https://api.ng.termii.com/api/sender-id?api_key=YOUR_API_KEY"
```

**Example response (200 OK):**

```json
{
  "current_page": 1,
  "data": [
    {
      "sender_id": "MyApp",
      "status": "active",
      "company": "My Company Ltd",
      "usecase": "OTP and transaction alerts",
      "country": "Nigeria",
      "created_at": "2024-01-15 10:30:00"
    },
    {
      "sender_id": "MyBrand",
      "status": "pending",
      "company": "My Company Ltd",
      "usecase": "Marketing campaigns",
      "country": "Nigeria",
      "created_at": "2024-06-20 14:45:00"
    }
  ],
  "first_page_url": "https://api.ng.termii.com/api/sender-id?page=1",
  "from": 1,
  "last_page": 1,
  "last_page_url": "https://api.ng.termii.com/api/sender-id?page=1",
  "next_page_url": null,
  "per_page": 15,
  "prev_page_url": null,
  "to": 2,
  "total": 2
}
```

---

### Request Sender ID

Submits a new Sender ID for registration.

**Endpoint:** `POST /api/sender-id/request`

**Request body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| api_key | string | Yes | Your Termii API key |
| sender_id | string | Yes | Desired Sender ID (3-11 characters) |
| usecase | string | Yes | Description of how it will be used |
| company | string | Yes | Your company name |

**Example request:**

```bash
curl -X POST https://api.ng.termii.com/api/sender-id/request \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_API_KEY",
    "sender_id": "MyBrand",
    "usecase": "OTP verification and transaction alerts for MyBrand mobile app",
    "company": "MyBrand Technologies Ltd"
  }'
```

**Example response (200 OK):**

```json
{
  "code": "ok",
  "message": "Sender ID requested successfully. Pending admin approval."
}
```

> **Note:** Sender ID approval typically takes 24-48 hours.

---

### Phone Number Search

Verifies a phone number's status and network information.

**Endpoint:** `GET /api/insight/number/query?api_key=YOUR_API_KEY&phone_number=PHONE_NUMBER`

**Example request:**

```bash
curl -X GET "https://api.ng.termii.com/api/insight/number/query?api_key=YOUR_API_KEY&phone_number=2348012345678"
```

**Example response (200 OK):**

```json
{
  "result": [
    {
      "number": "2348012345678",
      "status": "DELIVERED",
      "network": "MTN Nigeria",
      "network_code": "62130",
      "dnd_active": false,
      "routing_country": "Nigeria",
      "country_code": "234"
    }
  ]
}
```

**Response fields:**

| Field | Description |
|-------|-------------|
| status | Number status (DELIVERED, ABSENT, etc.) |
| network | Mobile network operator name |
| dnd_active | Whether Do-Not-Disturb is enabled |
| routing_country | Country the number is registered in |

---

## Message Character Limits

| Message Type | Characters per Page |
|--------------|---------------------|
| Plain text (GSM-7) | 160 characters |
| Unicode (emojis, special chars) | 70 characters |

**Special characters that reduce message length:**
- Emojis
- Characters: `^ { } \ [ ] ~ | €`
- Non-Latin scripts (Arabic, Chinese, etc.)

---

## Webhooks / Delivery Reports

Configure webhook URLs in your Termii dashboard to receive delivery reports.

### Webhook Payload

```json
{
  "message_id": "9122821270554876574",
  "message_id_str": "9122821270554876574",
  "status": "DELIVERED",
  "to": "2348012345678",
  "from": "MyApp",
  "channel": "dnd",
  "delivered_at": "2026-01-02T10:35:00.000Z"
}
```

### Delivery Statuses

| Status | Description |
|--------|-------------|
| `DELIVERED` | Message successfully delivered |
| `SENT` | Message sent to carrier |
| `PENDING` | Message queued for delivery |
| `FAILED` | Delivery failed |
| `REJECTED` | Message rejected by carrier |
| `EXPIRED` | Delivery timeout |

---

## Code Examples

### Node.js

```javascript
const axios = require('axios');

const termii = axios.create({
  baseURL: 'https://api.ng.termii.com',
  headers: { 'Content-Type': 'application/json' }
});

const API_KEY = process.env.TERMII_API_KEY;

// Send SMS
async function sendSMS(to, message, senderId = 'MyApp') {
  const response = await termii.post('/api/sms/send', {
    api_key: API_KEY,
    to,
    from: senderId,
    sms: message,
    type: 'plain',
    channel: 'dnd'
  });
  return response.data;
}

// Send OTP
async function sendOTP(phoneNumber) {
  const response = await termii.post('/api/sms/otp/send', {
    api_key: API_KEY,
    message_type: 'NUMERIC',
    to: phoneNumber,
    from: 'MyApp',
    channel: 'dnd',
    pin_attempts: 3,
    pin_time_to_live: 10,
    pin_length: 6,
    pin_placeholder: '< 1234 >',
    message_text: 'Your verification code is < 1234 >. Valid for 10 minutes.',
    pin_type: 'NUMERIC'
  });
  return response.data;
}

// Verify OTP
async function verifyOTP(pinId, pin) {
  const response = await termii.post('/api/sms/otp/verify', {
    api_key: API_KEY,
    pin_id: pinId,
    pin
  });
  return response.data;
}

// Usage
(async () => {
  // Send OTP
  const otpResponse = await sendOTP('2348012345678');
  console.log('OTP sent:', otpResponse.pinId);
  
  // Later, verify OTP entered by user
  const verifyResponse = await verifyOTP(otpResponse.pinId, '123456');
  console.log('Verified:', verifyResponse.verified);
})();
```

### Python

```python
import requests
import os

class TermiiClient:
    def __init__(self):
        self.base_url = 'https://api.ng.termii.com'
        self.api_key = os.environ.get('TERMII_API_KEY')
    
    def send_sms(self, to, message, sender_id='MyApp', channel='dnd'):
        """Send a single SMS message"""
        payload = {
            'api_key': self.api_key,
            'to': to,
            'from': sender_id,
            'sms': message,
            'type': 'plain',
            'channel': channel
        }
        
        response = requests.post(
            f'{self.base_url}/api/sms/send',
            json=payload
        )
        return response.json()
    
    def send_otp(self, phone_number, sender_id='MyApp'):
        """Send OTP to phone number"""
        payload = {
            'api_key': self.api_key,
            'message_type': 'NUMERIC',
            'to': phone_number,
            'from': sender_id,
            'channel': 'dnd',
            'pin_attempts': 3,
            'pin_time_to_live': 10,
            'pin_length': 6,
            'pin_placeholder': '< 1234 >',
            'message_text': f'Your {sender_id} verification code is < 1234 >. Valid for 10 minutes.',
            'pin_type': 'NUMERIC'
        }
        
        response = requests.post(
            f'{self.base_url}/api/sms/otp/send',
            json=payload
        )
        return response.json()
    
    def verify_otp(self, pin_id, pin):
        """Verify OTP entered by user"""
        payload = {
            'api_key': self.api_key,
            'pin_id': pin_id,
            'pin': pin
        }
        
        response = requests.post(
            f'{self.base_url}/api/sms/otp/verify',
            json=payload
        )
        return response.json()
    
    def get_balance(self):
        """Get account balance"""
        response = requests.get(
            f'{self.base_url}/api/get-balance',
            params={'api_key': self.api_key}
        )
        return response.json()


# Usage
client = TermiiClient()

# Send OTP
otp_response = client.send_otp('2348012345678')
print(f"OTP Pin ID: {otp_response['pinId']}")

# Verify OTP
verify_response = client.verify_otp(otp_response['pinId'], '123456')
if verify_response['verified'] == 'True':
    print('Phone number verified!')
else:
    print('Verification failed')
```

### PHP

```php
<?php

class TermiiClient {
    private $baseUrl = 'https://api.ng.termii.com';
    private $apiKey;
    
    public function __construct($apiKey) {
        $this->apiKey = $apiKey;
    }
    
    private function request($endpoint, $data, $method = 'POST') {
        $curl = curl_init();
        
        $options = [
            CURLOPT_URL => $this->baseUrl . $endpoint,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_TIMEOUT => 30,
            CURLOPT_HTTPHEADER => ['Content-Type: application/json'],
        ];
        
        if ($method === 'POST') {
            $options[CURLOPT_POST] = true;
            $options[CURLOPT_POSTFIELDS] = json_encode($data);
        }
        
        curl_setopt_array($curl, $options);
        $response = curl_exec($curl);
        curl_close($curl);
        
        return json_decode($response, true);
    }
    
    public function sendSMS($to, $message, $senderId = 'MyApp', $channel = 'dnd') {
        return $this->request('/api/sms/send', [
            'api_key' => $this->apiKey,
            'to' => $to,
            'from' => $senderId,
            'sms' => $message,
            'type' => 'plain',
            'channel' => $channel
        ]);
    }
    
    public function sendOTP($phoneNumber, $senderId = 'MyApp') {
        return $this->request('/api/sms/otp/send', [
            'api_key' => $this->apiKey,
            'message_type' => 'NUMERIC',
            'to' => $phoneNumber,
            'from' => $senderId,
            'channel' => 'dnd',
            'pin_attempts' => 3,
            'pin_time_to_live' => 10,
            'pin_length' => 6,
            'pin_placeholder' => '< 1234 >',
            'message_text' => "Your $senderId code is < 1234 >. Valid for 10 minutes.",
            'pin_type' => 'NUMERIC'
        ]);
    }
    
    public function verifyOTP($pinId, $pin) {
        return $this->request('/api/sms/otp/verify', [
            'api_key' => $this->apiKey,
            'pin_id' => $pinId,
            'pin' => $pin
        ]);
    }
}

// Usage
$termii = new TermiiClient('YOUR_API_KEY');

// Send OTP
$otpResponse = $termii->sendOTP('2348012345678');
$pinId = $otpResponse['pinId'];

// Verify OTP
$verifyResponse = $termii->verifyOTP($pinId, '123456');
if ($verifyResponse['verified'] === 'True') {
    echo 'Phone number verified!';
}
```

---

## Error Handling

### Common Error Codes

| Error | Description | Solution |
|-------|-------------|----------|
| `INVALID_API_KEY` | API key is missing or invalid | Check your API key in the dashboard |
| `INSUFFICIENT_BALANCE` | Wallet balance too low | Top up your account |
| `INVALID_SENDER_ID` | Sender ID not registered | Register Sender ID first |
| `INVALID_PHONE_NUMBER` | Phone number format wrong | Use international format with country code |
| `MESSAGE_TOO_LONG` | Message exceeds limit | Reduce message length |
| `RATE_LIMIT_EXCEEDED` | Too many requests | Implement backoff strategy |

### Error Response Format

```json
{
  "code": "error",
  "message": "Description of what went wrong"
}
```

---

## Rate Limits

| Endpoint | Limit |
|----------|-------|
| Send SMS | 100 requests/second |
| Send OTP | 50 requests/second |
| Verify OTP | 100 requests/second |
| Bulk SMS | 10 requests/second |

---

## Best Practices

1. **Use DND route for OTPs** — Generic route may not deliver to DND-enabled numbers

2. **Store pinId securely** — Required for OTP verification

3. **Set appropriate TTL** — Balance security (short) with user convenience (longer)

4. **Handle all verification statuses** — Check for `True`, `False`, and `Expired`

5. **Implement retry logic** — For failed deliveries, retry with exponential backoff

6. **Monitor your balance** — Set up low balance alerts to avoid service interruption

7. **Use webhooks** — Don't rely solely on synchronous responses for delivery status

---

## Testing

Use Termii's test environment with your test API key to simulate messages without actual delivery or charges.

**Test phone numbers:**
- Use any valid format phone number
- Messages won't be delivered in test mode
- Balance won't be deducted

---

## Support

- **Documentation:** [developers.termii.com](https://developers.termii.com)
- **Dashboard:** [accounts.termii.com](https://accounts.termii.com)
- **Email Support:** support@termii.com
- **Developer Forum:** [community.termii.com](https://community.termii.com)

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2024-06-01 | Added WhatsApp templates |
| 1.5 | 2024-01-15 | Added bulk SMS (10k limit) |
| 1.0 | 2023-01-01 | Initial API release |

---

*Last updated: January 2026*
