---
layout: default
title: Partner Integration Guide
parent: Integration
nav_order: 1
permalink: /integration/partner-integration/
---

# Partner Integration Guide - Safelog.ai Background Check Forms

## Overview

This document provides comprehensive instructions for partners to integrate Safelog.ai background check forms into their websites with minimal technical implementation.

## Architecture

```
Partner Site → Payment Success → Safelog Webhook → JWT Session → Secure Form Proxy → Tally Form → Background Check Process
```

## Integration Flow

1. **Customer completes payment** on partner site
2. **Partner redirects/POSTs** to Safelog webhook endpoint with voucher details
3. **Safelog validates** partner signature and voucher status, creates secure session
4. **Customer is redirected** to form with JWT token (3-day expiry)
5. **Secure form proxy** loads Tally form with session context
6. **Form submission** processed through Tally webhooks, triggers background check
7. **Voucher marked as used** and results are processed in database
8. **Results** can be delivered via webhook notifications (optional)

## Partner Implementation

### Step 1: Obtain API Credentials

Contact Safelog.ai to obtain:
- Partner ID
- HMAC Secret Key
- Webhook endpoint URL

### Step 2: Post-Payment Redirect

After successful payment, redirect customer to Safelog webhook:

```javascript
// JavaScript example
function redirectToBackgroundCheck(customerId, voucherCode, customerEmail = '') {
  const partnerId = 'YOUR_PARTNER_ID';
  const secret = 'YOUR_HMAC_SECRET';
  
  // Create HMAC signature (includes customerEmail)
  const data = customerId + partnerId + voucherCode + customerEmail;
  const signature = crypto.createHmac('sha256', secret).update(data).digest('hex');
  
  // Redirect to Safelog  
  const url = new URL(`https://your-safelog-domain.com/api/v1/partner/${partnerId}/form/${voucherCode}/links`);
  url.searchParams.set('customer_id', customerId);
  url.searchParams.set('customer_email', customerEmail);
  url.searchParams.set('signature', signature);
  
  window.location.href = url.toString();
}
```

### Step 3: Alternative POST Method

You can also use POST request instead of redirect:

```javascript
// POST method example
async function initiateBackgroundCheck(customerId, voucherCode, customerEmail = '') {
  const partnerId = 'YOUR_PARTNER_ID';
  const secret = 'YOUR_HMAC_SECRET';
  
  // Create HMAC signature (includes customerEmail)
  const data = customerId + partnerId + voucherCode + customerEmail;
  const signature = crypto.createHmac('sha256', secret).update(data).digest('hex');
  
  const response = await fetch(`https://your-safelog-domain.com/api/v1/partner/${partnerId}/form/${voucherCode}/links`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      customerId,
      customerEmail,
      signature
    })
  });
  
  const result = await response.json();
  
  if (result.success) {
    // Redirect customer to form
    window.location.href = result.formUrl;
  }
}
```

## Security Features

### HMAC Signature Verification
All requests must include valid HMAC signature to prevent tampering.

### Rate Limiting
- 5 requests per minute per IP for payment webhook
- 3 form submissions per minute per IP
- 20 general requests per minute per IP

### Voucher Management
- Voucher codes are tracked per partner and prevent duplicate usage
- Vouchers expire after 3 days if not used
- Each voucher can only be used once for form submission
- Voucher status tracking with usage timestamps

### Session Management
- JWT tokens expire after 3 days (72 hours)
- Session tokens include voucher and partner context
- Secure form proxy prevents direct Tally form access
- Session validation on every form interaction

### Input Validation
- All form inputs are sanitized
- Email and phone format validation
- Indonesian NIK (16-digit) validation
- Bot detection with honeypot fields
- Enhanced rate limiting per IP address

## Form Configuration

The background check form is powered by Tally.so and includes these fields:
- Full Name (required)
- Email Address
- Phone Number  
- NIK (Indonesian Identity Number, required)
- Address

### Tally Form Integration
- Form ID: `wv8RVX` (configurable via environment)
- Event forwarding enabled for real-time processing
- Transparent background and responsive design
- Secure iframe embedding with CSP headers

## API Endpoints

### GET /api/v1/partner/{partnerId}/form/{voucherCode}/links
Initiates background check session and redirects directly to form.

**Path Parameters:**
- `partnerId` (string): Your partner ID
- `voucherCode` (string): Voucher code for validation

**Query Parameters:**
- `customer_id` (string): Unique customer identifier
- `customer_email` (string): Customer email address (optional)
- `signature` (string): HMAC-SHA256 signature

**Response:** HTTP 302 redirect to form URL

### POST /api/v1/partner/{partnerId}/form/{voucherCode}/links
Initiates background check session and returns form URL.

**Path Parameters:**
- `partnerId` (string): Your partner ID
- `voucherCode` (string): Voucher code for validation

**Body Parameters:**
- `customerId` (string): Unique customer identifier
- `customerEmail` (string): Customer email address (optional)
- `signature` (string): HMAC-SHA256 signature

**Response:**
```json
{
  "success": true,
  "sessionToken": "jwt_token_here",
  "formUrl": "https://domain.com/form/jwt_token_here",
  "partnerId": "your_partner_id",
  "voucherCode": "VOUCHER123ABC",
  "expiresIn": 259200
}
```

### GET /form/[token]
Displays the background check form with JWT validation and secure Tally proxy.

### GET /api/form/proxy?token=[token]
Secure proxy endpoint that loads Tally form with session context.

## Error Handling

### Common Error Responses

**400 Bad Request**
```json
{
  "error": "Missing required parameters"
}
```

**403 Forbidden**
```json
{
  "error": "Invalid signature"
}
```

**429 Too Many Requests**
```json
{
  "error": "Rate limit exceeded"
}
```

**401 Unauthorized**
```json
{
  "error": "Invalid or missing authentication token"
}
```

**409 Conflict**
```json
{
  "error": "Voucher has already been used"
}
```

## Testing

### Test Credentials
For testing purposes, use:
- Partner ID: `test-partner`
- HMAC Secret: `test-secret-key`
- Endpoint: `https://test-safelog-domain.com/api/v1/partner/test-partner/form/VOUCHER123ABC/links`

### Test Signature Generation
```javascript
const crypto = require('crypto');

const customerId = 'test-customer-123';
const partnerId = 'test-partner';
const voucherCode = 'VOUCHER123ABC';
const customerEmail = 'test@example.com'; // Optional
const secret = 'test-secret-key';

// Note: Include customerEmail in signature calculation
const signature = crypto
  .createHmac('sha256', secret)
  .update(customerId + partnerId + voucherCode + customerEmail)
  .digest('hex');

console.log('Test signature:', signature);
```

## Monitoring & Logging

All requests are logged with comprehensive audit trails including:
- Timestamp and request ID
- IP address and user agent
- Request parameters and session data
- Voucher usage tracking
- Success/failure status with error details
- Form submission events and processing results

## Webhook Notifications (Optional)

Safelog can send webhook notifications to partners when:
- Background check is completed
- Results are ready
- Process encounters errors
- Report files are generated

### Webhook Configuration
Partners can configure webhook URLs in the `PartnerVoucherUsage` model:
- `webhookUrl`: Endpoint for status notifications
- `webhookStatus`: HTTP status code of last webhook attempt
- `reportFileUrl`: Direct link to generated PDF report
- `reportFileDeliveredAt`: Timestamp of successful delivery

Contact support to configure webhook endpoints and delivery preferences.

## Support

For technical support and API questions:
- Email: support@safelog.ai
- Documentation: https://docs.safelog.ai
- Status Page: https://status.safelog.ai

## Security Best Practices

1. **Store secrets securely** - Never expose HMAC secrets in client-side code
2. **Use HTTPS** - All communications must use secure connections
3. **Validate responses** - Always check response status and validate data
4. **Monitor logs** - Review integration logs regularly
5. **Rate limiting** - Implement your own rate limiting if needed
6. **Error handling** - Implement proper error handling for all scenarios

## Environment Variables

Required environment variables for Safelog application:

```bash
# JWT Configuration
JWT_SECRET=your-jwt-secret-key

# Partner Configuration
PARTNER_SECRET=your-partner-hmac-secret

# Database
DATABASE_URL=postgresql://...

# CORS (optional, defaults to *)
ALLOWED_ORIGINS=https://partner1.com,https://partner2.com
```

## Troubleshooting

### Common Issues

**Invalid Signature Error**
- Verify HMAC secret key
- Check parameter order in signature generation: `customerId + partnerId + voucherCode + customerEmail`
- Ensure no URL encoding of parameters before signing
- Include customerEmail parameter even if empty string

**Session Expired**
- JWT tokens expire after 3 days (72 hours)
- Customer must complete form within time limit
- Generate new session if expired

**Voucher Already Used**
- Each voucher can only be used once for form submission
- Check voucher status before redirecting customers
- Generate new voucher codes for repeat customers

**Rate Limit Exceeded**
- Implement delays between requests
- Monitor request frequency
- Contact support for higher limits if needed

**Form Not Loading**
- Check JWT token validity and expiration
- Verify Content Security Policy settings
- Ensure Tally.so domains are not blocked by firewall
- Check secure proxy endpoint `/api/form/proxy` accessibility
- Verify session data includes voucher and partner context

### Debug Mode

Add `debug=true` parameter to webhook URL for detailed error messages (test environment only).

## Changelog

### Version 2.0.0 (2025-08-18)
- Enhanced voucher management with `PartnerVoucherUsage` model
- Added customer email parameter to signature calculation
- Extended JWT token expiry to 3 days
- Improved form proxy system with Tally integration
- Enhanced error handling and voucher conflict detection
- Added webhook notification support for partners
- Comprehensive audit logging and monitoring

### Version 1.0.0 (2025-01-XX)
- Initial partner integration implementation
- JWT-based session management
- Tally form integration with event forwarding
- HMAC signature validation
- Rate limiting and security features