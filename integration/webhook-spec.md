---
layout: default
title: Webhook Specification
parent: Integration
nav_order: 2
---

# Safelog.ai Partner Webhook Specification

## Report File Delivery Webhook

When a background check is completed and the report file is ready, Safelog.ai will send a webhook notification to your specified endpoint.

### Webhook Payload

**HTTP Method:** `POST`  
**Content-Type:** `application/json`

#### Headers

- `X-Safelog-Signature`: HMAC-SHA256 signature of the payload
- `X-Safelog-Timestamp`: Unix timestamp when the webhook was sent
- `User-Agent`: `Safelog-Webhook/1.0`

#### Payload Structure

```json
{
  "screeningId": "clxyz123456789",
  "customerId": "your-customer-id-123", 
  "voucherCode": "VOUCHER123ABC",
  "partnerId": "your-partner-id",
  "reportFileUrl": "https://safelog.ai/reports/screening-report.pdf",
  "completedAt": "2024-08-18T10:30:00.000Z",
  "riskScore": {
    "totalScore": 2.5,
    "level": "LOW_RISK"
  },
  "status": "completed",
  "timestamp": 1692357000
}
```

#### Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| `screeningId` | String | Unique identifier for the screening |
| `customerId` | String | Your customer identifier from the original request |
| `voucherCode` | String | The voucher code that was used |
| `partnerId` | String | Your partner identifier |
| `reportFileUrl` | String | Direct URL to download the report file |
| `completedAt` | String | ISO 8601 timestamp when screening was completed |
| `riskScore` | Object\|null | Risk assessment results (null if still processing) |
| `status` | String | Always "completed" for report file delivery webhooks |
| `timestamp` | Number | Unix timestamp when webhook was sent |

### Signature Verification

To verify the webhook is from Safelog.ai, validate the HMAC signature:

```javascript
const crypto = require('crypto');

function verifySignature(payload, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  return signature === expectedSignature;
}

// Usage
const signature = request.headers['x-safelog-signature'];
const isValid = verifySignature(payload, signature, YOUR_PARTNER_SECRET);
```

### Response Requirements

Your webhook endpoint should:

1. **Respond quickly** (within 30 seconds)
2. **Return HTTP 200** for successful receipt
3. **Be idempotent** (handle duplicate deliveries gracefully)
4. **Validate the signature** before processing

#### Success Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "received": true,
  "processedAt": "2024-08-18T10:30:15.000Z"
}
```

#### Error Response

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid signature",
  "code": "SIGNATURE_INVALID"
}
```

### Retry Policy

If your webhook endpoint returns a non-2xx response or doesn't respond within 30 seconds:

- We will retry up to **3 times**
- With exponential backoff: 5s, 25s, 125s
- After all retries fail, the webhook is marked as failed

### Security Considerations

1. **Always verify signatures** to ensure authenticity
2. **Use HTTPS** for your webhook URL
3. **Store your partner secret securely**
4. **Log webhook receipts** for debugging
5. **Handle replay attacks** by checking timestamps

### Testing Your Webhook

You can test your webhook implementation using the partner integration test file:

```bash
# Open the test file
open test-partner-integration.html

# Configure your webhook URL and test the flow
# The test will simulate the complete integration flow
```

### Example Webhook Implementations

#### Node.js/Express

```javascript
const express = require('express');
const crypto = require('crypto');
const app = express();

app.use(express.json());

app.post('/webhook/safelog-callback', (req, res) => {
  const signature = req.headers['x-safelog-signature'];
  const timestamp = req.headers['x-safelog-timestamp'];
  const payload = req.body;
  
  // Verify signature
  const expectedSignature = crypto
    .createHmac('sha256', process.env.PARTNER_SECRET)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  if (signature !== expectedSignature) {
    return res.status(400).json({ error: 'Invalid signature' });
  }
  
  // Process the webhook
  console.log('Received screening result:', {
    customerId: payload.customerId,
    reportFileUrl: payload.reportFileUrl,
    riskScore: payload.riskScore
  });
  
  // Download and process the report file
  // Update your system with the screening results
  
  res.json({ received: true, processedAt: new Date().toISOString() });
});
```

#### PHP

```php
<?php
function handleSafelogWebhook() {
    $signature = $_SERVER['HTTP_X_SAFELOG_SIGNATURE'] ?? '';
    $timestamp = $_SERVER['HTTP_X_SAFELOG_TIMESTAMP'] ?? '';
    $payload = json_decode(file_get_contents('php://input'), true);
    
    // Verify signature
    $expectedSignature = hash_hmac('sha256', json_encode($payload), getenv('PARTNER_SECRET'));
    
    if (!hash_equals($signature, $expectedSignature)) {
        http_response_code(400);
        echo json_encode(['error' => 'Invalid signature']);
        return;
    }
    
    // Process the webhook
    error_log('Received screening result for customer: ' . $payload['customerId']);
    
    // Download and process the report file
    // Update your system with the screening results
    
    echo json_encode(['received' => true, 'processedAt' => date('c')]);
}

handleSafelogWebhook();
?>
```

### Support

For webhook integration support:
- Check the logs in your Safelog.ai partner dashboard
- Use the test integration tool to debug issues
- Contact support with your partner ID and webhook URL