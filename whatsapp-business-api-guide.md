# WhatsApp Business API — Setup & Integration Guide

## 1. Overview
The WhatsApp Business Platform (Cloud API) lets businesses send/receive messages programmatically. Two access routes exist:

- **Meta Cloud API** (recommended) — hosted by Meta, free tier available, fastest to set up.
- **On-Premise API** — deprecated, self-hosted, not recommended for new projects.

This guide covers the Cloud API path.

## 2. Prerequisites
- A Meta Business Account (business.facebook.com)
- A verified business (business documents may be required)
- A phone number not currently active on regular WhatsApp/Business app (or willing to migrate)
- A Facebook Developer account (developers.facebook.com)
- A server/backend to host your webhook (Node.js, Python, PHP, etc.)
- HTTPS endpoint with a valid SSL certificate (required for webhooks)

## 3. Account Setup Steps

1. **Create a Meta App**
   - Go to developers.facebook.com → My Apps → Create App → select "Business" type.
2. **Add WhatsApp Product**
   - Inside the app dashboard, add the "WhatsApp" product.
   - Meta auto-generates a **test phone number** and **temporary access token** (valid 24h) for development.
3. **Link/Create a Business Phone Number**
   - Add your real business number under WhatsApp → API Setup.
   - Verify via SMS/voice OTP.
4. **Generate a Permanent Access Token**
   - Create a System User in Meta Business Settings → assign WhatsApp asset permissions → generate a token with `whatsapp_business_messaging` and `whatsapp_business_management` scopes.
5. **Get your IDs**
   - `Phone Number ID` and `WhatsApp Business Account ID (WABA ID)` — both found in API Setup page.
6. **Business Verification** (required to remove messaging limits)
   - Submit business documents under Meta Business Settings → Security Center.

## 4. Core API Concepts

| Concept | Description |
|---|---|
| Phone Number ID | Identifies the sending number in API calls |
| WABA ID | The WhatsApp Business Account container |
| Access Token | Bearer token for authenticating API requests |
| Webhook | Endpoint where Meta sends incoming messages/status updates |
| Message Templates | Pre-approved message formats required for business-initiated conversations |
| Session Messages | Free-form replies allowed only within 24h of a user's last message |

## 5. Sending a Message (Example)

```bash
curl -X POST "https://graph.facebook.com/v20.0/<PHONE_NUMBER_ID>/messages" \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "messaging_product": "whatsapp",
    "to": "<RECIPIENT_PHONE_NUMBER>",
    "type": "text",
    "text": { "body": "Hello from our business!" }
  }'
```

For first-contact/business-initiated messages, you must use an **approved template** instead of `type: text`.

## 6. Message Templates
- Created in Meta Business Manager → WhatsApp Manager → Message Templates.
- Categories: Marketing, Utility, Authentication.
- Must be approved by Meta (usually minutes to a few hours).
- Required for any message sent outside the 24-hour customer service window.

## 7. Webhooks (Receiving Messages)
1. Set Webhook URL + Verify Token in App Dashboard → WhatsApp → Configuration.
2. Meta sends a GET request to verify ownership (`hub.challenge` echo).
3. Incoming messages/status updates arrive as POST requests (JSON payload) to your endpoint.
4. Your server must respond `200 OK` within a few seconds.

## 8. Pricing Model (as of current Meta policy — verify before launch)
- Conversation-based pricing, billed per 24-hour conversation window, categorized as Marketing, Utility, Authentication, or Service.
- Service (user-initiated) conversations are often free up to a monthly threshold.
- Always check Meta's official pricing page at launch time since rates change.

## 9. Rate Limits & Scaling
- New numbers start in a limited messaging tier (e.g., 250 unique users/24h).
- Tier increases automatically based on quality rating and volume sent.
- Use a message queue (e.g., Redis/Bull, RabbitMQ) on your backend to avoid hitting bursts.

## 10. Recommended Tech Stack
- **Backend:** Node.js (Express) or Python (FastAPI) to handle webhook + send logic.
- **Database:** Postgres/MySQL for storing conversation state, contacts, message logs.
- **Queue:** Redis-based queue for outbound message throttling.
- **Hosting:** Any cloud VM/container service with HTTPS (e.g., a reverse proxy via Nginx + Let's Encrypt).

## 11. Alternative: Business Solution Providers (BSPs)
If you prefer not to integrate directly with Meta, official BSPs (Twilio, 360dialog, Gupshup, MessageBird) offer simplified onboarding, dashboards, and support — at an added per-message markup. Recommended if you lack dev resources to manage tokens/webhooks yourself.

## 12. Next Steps Checklist
- [ ] Create Meta Developer App
- [ ] Verify business phone number
- [ ] Complete Meta Business Verification
- [ ] Generate permanent system-user access token
- [ ] Build & deploy webhook endpoint
- [ ] Create & get message templates approved
- [ ] Implement send/receive logic in backend
- [ ] Set up message queue for scale
- [ ] Test in sandbox before going live

---
*Note: Meta updates WhatsApp Business Platform policies and pricing periodically — confirm current terms at developers.facebook.com/docs/whatsapp before production launch.*
