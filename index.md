---
layout: default
title: Safelog.ai Documentation
---

# Safelog.ai Documentation

Welcome to the comprehensive documentation for Safelog.ai background check services and integration APIs.

## Overview

Safelog.ai provides secure, reliable background check services with easy integration options for partners. Our platform offers:

- **Employee Screening**: Comprehensive background checks with risk scoring
- **Identity Verification**: KTP OCR, face verification, and address verification
- **Partner Integration**: Simple webhook-based integration with your existing systems
- **Real-time Processing**: Fast turnaround times with automated workflows

## Quick Navigation

### Integration
- [Partner Integration Guide](integration/partner-integration/) - Complete guide for integrating background check forms
- [Webhook Specification](integration/webhook-spec/) - Report file delivery webhook documentation

## Getting Started

1. **Contact Safelog.ai** to obtain your API credentials
2. **Review the Integration Guide** to understand the implementation flow
3. **Set up webhooks** for receiving background check results
4. **Test your integration** using our sandbox environment

## Architecture Overview

```
Partner Site → Payment Success → Safelog Webhook → JWT Session → Secure Form → Background Check Process
```

## Features

### Security
- HMAC signature validation
- JWT-based session management
- Rate limiting and DDoS protection
- Secure form proxying

### Integration Options
- Direct redirect integration
- POST-based API calls
- Webhook notifications
- Custom form embedding

### Supported Services
- Indonesian NIK verification
- KTP OCR processing
- Address verification with maps
- Risk scoring algorithms
- PDF report generation

## Support

For technical support and questions:
- **Email**: support@safelog.ai
- **Documentation**: This site
- **Status**: [status.safelog.ai](https://status.safelog.ai)

## API Reference

All API endpoints use HTTPS and require proper authentication. See the integration guides for detailed implementation examples.