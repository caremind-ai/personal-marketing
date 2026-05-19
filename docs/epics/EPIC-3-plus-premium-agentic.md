# EPIC-3: Plus + Premium Tiers + Agentic Actions

**Priority**: P0 (Critical)  
**Phase**: 2 — Monetization + Agentic  
**Timeline**: M5–M6 (October–November 2026)

## Overview

Launch paid tiers (Plus $9, Premium $29) with subscription billing and introduce agentic capabilities: appointment booking, Rx refill requests (non-controlled only), and lab orders. Multi-LLM consensus for high-severity findings. App Store submission for public launch.

## User Stories

As a Plus user, I want Personal to book my appointments automatically when I agree to a suggested action, so I don't have to navigate a scheduling system myself.

As a Premium user, I want Personal to use multiple AI models to cross-check serious findings before alerting me, so I know the alert is real and not a false positive.

As a Care-bound user, I want my Rx refill request to go through Personal instead of calling the pharmacy myself, with my GP approving it before it's sent.

## Child Tickets

### TICKET-3.1: Subscription Billing — RevenueCat + Stripe
**Story Points**: 5  
**Dependencies**: EPIC-2

**Description:**
Integrate RevenueCat for in-app purchase management (iOS App Store subscriptions). Stripe for web/B2B billing (future). Entitlements gate features per tier. Free tier users see upgrade prompts at feature limits.

**Acceptance Criteria:**
- [ ] RevenueCat configured with products: `personal_plus_monthly`, `personal_premium_monthly`
- [ ] Purchase flow: tap upgrade → App Store sheet → subscription active immediately
- [ ] Entitlements checked server-side (not just client-side) via RevenueCat API
- [ ] Downgrade: user retains tier access until billing period ends
- [ ] Free trial: 14-day free trial on Plus and Premium (configurable)
- [ ] Upgrade prompts at free tier limits: AI chat (10 msgs), reports (1/month)
- [ ] Subscription status synced to HealthTrack user record
- [ ] Webhook: RevenueCat → gateway on subscription change (for server-side entitlement update)

**Technical Notes:**
- RevenueCat handles receipt validation, renewal, churn tracking
- Server-side entitlement check: `GET /users/:id/entitlements` in gateway

---

### TICKET-3.2: Agentic — Appointment Booking
**Story Points**: 8  
**Dependencies**: TICKET-3.1

**Description:**
AI suggests appointments based on health trends and the user can confirm with one tap. Personal books the appointment with the user's GP (once GP layer exists) or finds an available external provider. For now (pre-GP layer): book with external providers via Zocdoc API or similar.

**Acceptance Criteria:**
- [ ] AI generates appointment suggestion with reason: "Your HRV trend suggests a check-in with your doctor. Want me to book?"
- [ ] User taps "Book it" → confirmation of appointment time and provider
- [ ] Appointment appears in iOS Calendar (EventKit) with provider details
- [ ] Cancellation available up to 24 hours before (triggers provider API cancel)
- [ ] Booking confirmation sent via push notification + in-app
- [ ] Available for Plus+ tier only; free tier sees prompt to upgrade
- [ ] Failure case: if no slots available, suggests top 3 alternatives

**Technical Notes:**
- Zocdoc API (or Kyruus) for external provider search and booking
- Pre-GP layer: all bookings go to external providers
- Post-GP layer (EPIC-4): GP calendar takes priority

---

### TICKET-3.3: Agentic — Rx Refill Request
**Story Points**: 5  
**Dependencies**: TICKET-3.1

**Description:**
User initiates Rx refill through the app. AI drafts the refill request. For Care tier: GP reviews and authorizes before transmitting. For Plus/Premium: request goes to the prescribing physician on file (via patient's EHR). Non-controlled medications only.

**Acceptance Criteria:**
- [ ] Medication list pulled from Flexpa EHR (existing medications with active Rx)
- [ ] User selects medication → "Request refill" → AI drafts request with clinical context
- [ ] Care tier: request queued for GP review in Provider AI OS (EPIC-4); patient notified when GP approves
- [ ] Plus/Premium (pre-GP): request drafted and user sends to their existing physician via secure message
- [ ] Controlled substances (Schedule II–V) explicitly blocked with explanation
- [ ] GoodRx price comparison shown at confirmation: cheapest pharmacy option
- [ ] Status tracking: pending → approved → pharmacy notified → ready for pickup

---

### TICKET-3.4: Agentic — Lab Order Request
**Story Points**: 5  
**Dependencies**: TICKET-3.1

**Description:**
User requests a lab order (e.g., annual bloodwork, HbA1c, thyroid panel). AI suggests relevant panels based on health graph. Care tier: GP signs the order. Plus/Premium (pre-GP): patient-directed lab testing via Everlywell or similar direct-to-consumer lab.

**Acceptance Criteria:**
- [ ] Suggested lab panels: AI recommends based on conditions, last lab date, age
- [ ] User selects panel → sees nearest LabCorp/Quest locations (or DTC option)
- [ ] Care tier: lab request queued for GP approval (EPIC-4 dependency)
- [ ] Plus/Premium: direct-to-consumer via Everlywell/LabCorp DTC; user pays out of pocket
- [ ] Results ingested automatically when available (Flexpa or manual upload fallback)
- [ ] Results trigger AI analysis and update health score

---

### TICKET-3.5: Multi-LLM Consensus for High-Severity Alerts
**Story Points**: 5  
**Dependencies**: TICKET-3.1 (Premium tier)

**Description:**
When the primary AI (Claude) assigns highest alert urgency to a finding, run the same prompt through Gemini and GPT-4o as independent reviewers. Alert fires only if 2/3 models agree. Reduces false positives for critical alerts.

**Acceptance Criteria:**
- [ ] High-severity threshold: defined as top 5% of alert urgency scores
- [ ] Consensus flow: Claude primary → if urgent, send same context to Gemini + GPT-4o in parallel
- [ ] Alert fires if ≥2/3 models classify as urgent (majority vote)
- [ ] Disagreement logged with all three model responses (for audit and model improvement)
- [ ] Total latency for consensus check: < 8 seconds (parallel calls)
- [ ] Consensus used only for non-emergency alerts; true emergencies (SpO2 <90% etc.) fire immediately without waiting for consensus
- [ ] Available for Premium+ tier only

**Technical Notes:**
- Gemini: Vertex AI (existing OpenRouter or direct Vertex API)
- GPT-4o: OpenRouter
- Parallel calls via `Promise.all` in AI worker

---

### TICKET-3.6: App Store Submission
**Story Points**: 3  
**Dependencies**: TICKET-3.1 through 3.5

**Description:**
Submit Personal to the App Store for public launch. Category: Medical (primary), Health & Fitness (secondary). Requires App Store review of health app metadata and privacy labels.

**Acceptance Criteria:**
- [ ] App Store Connect metadata complete: screenshots (all device sizes), description, keywords, support URL
- [ ] Privacy nutrition label accurate (HealthKit data, usage analytics)
- [ ] HealthKit usage string explains each data type used
- [ ] App Store Review Guidelines compliance: medical disclaimer present, no diagnosis claims
- [ ] Age rating: 17+ (medical)
- [ ] App approved and live in App Store
- [ ] App Store Optimization (ASO): target keywords "AI health app", "personal doctor app", "health monitoring"

---

## Definition of Done

- [ ] Plus and Premium tiers live with billing
- [ ] Agentic: appointment booking, Rx refill, lab order all working
- [ ] Multi-LLM consensus working for Premium users
- [ ] App Store approved and public
- [ ] Revenue > $0 (first paid subscriber)
- [ ] Churn < 15% monthly for paid tiers

## Notes

- Agentic actions (TICKET-3.2–3.4) are the primary upgrade driver for Plus. Make them feel magical.
- Rx refill (TICKET-3.3) requires clear communication that GP approval is needed before pharmacy transmission — set expectations to avoid user frustration.
- Controlled substance exclusion is non-negotiable legally — ensure UI clearly explains this.
