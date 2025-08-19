---
layout: default
title: Home
nav_order: 1
description: "Comprehensive documentation for Safelog.ai background check services and integration APIs"
permalink: /
---

# Safelog.ai Documentation
{: .fs-9 }

Comprehensive documentation for Safelog.ai background check services and integration APIs
{: .fs-6 .fw-300 }

[Get Started with Integration](integration/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View on GitHub](https://github.com/safelog-ai/docs-safelog.ai){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## 🎯 What is Safelog.ai?

Safelog.ai provides secure, reliable background check services with easy integration options for partners. Our platform offers comprehensive screening solutions with minimal technical implementation required.

### Key Features

**🔍 Employee Screening**
: Comprehensive background checks with intelligent risk scoring algorithms

**🆔 Identity Verification** 
: KTP OCR processing, face verification, and address validation

**🔗 Partner Integration**
: Simple webhook-based integration with your existing payment systems

**⚡ Real-time Processing**
: Fast turnaround times with automated workflows and instant notifications

---

## 🚀 Quick Start

{: .important }
> Contact Safelog.ai to obtain your partner credentials before starting integration

1. **Get API Credentials** - Contact our team for partner ID and HMAC secret
2. **Review Integration Guide** - Understand the implementation flow and security requirements  
3. **Set Up Webhooks** - Configure endpoints for receiving background check results
4. **Test Integration** - Use our sandbox environment to validate your implementation

---

## 📚 Documentation Sections

### Integration
Complete guides for integrating Safelog.ai services into your platform

- [Partner Integration Guide](integration/partner-integration/) - Step-by-step implementation guide
- [Webhook Specification](integration/webhook-spec/) - Report delivery webhook documentation

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