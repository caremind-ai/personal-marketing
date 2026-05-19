# EPIC-2: Consumer App MVP

**Priority**: P0 (Critical)  
**Phase**: 1 — Consumer Layer  
**Timeline**: M3–M4 (August–September 2026)

## Overview

Ship the Layer 1 consumer app to a closed beta of 200 users. The MVP delivers the core value proposition: continuous HealthKit monitoring → AI health score and proactive alerts → shareable clinical reports. No GP layer yet. Success = 35% D30 retention.

## User Stories

As a health-engaged adult, I want to see a daily health score with a plain-language explanation of what's driving it, so I understand my health without being a doctor.

As a user, I want Personal to alert me when a health trend needs attention — before I notice symptoms — so I can act proactively rather than reactively.

As a user, I want to generate a clinical-grade health report I can share with my doctor, so I don't have to explain my history from scratch at every appointment.

## Child Tickets

### TICKET-2.1: Daily Health Score
**Story Points**: 8  
**Dependencies**: EPIC-1

**Description:**
Compute a 0–100 daily health score from the user's FHIR observation data. Score should reflect sleep quality, HRV trend, resting heart rate, activity, and any anomalies. AI explains the score in 2–3 sentences. Score is the app's home screen centerpiece.

**Acceptance Criteria:**
- [ ] Score computed daily from available HealthKit data (gracefully handles missing types)
- [ ] Score has a trend indicator (up/down/flat vs. last 7 days)
- [ ] AI explanation is ≤3 sentences, plain language, references specific metrics
- [ ] Score updates when new HealthKit sync completes (not just once/day)
- [ ] Empty state: meaningful score shown even with only 1–2 data types available
- [ ] Score history chart shows last 30 days

**Technical Notes:**
- Scoring algorithm: weighted composite; weights configurable per data type
- AI prompt: use HealthTrack `ai-worker` synthesis endpoint; pass last 7 days of observations

---

### TICKET-2.2: Proactive Alert System
**Story Points**: 8  
**Dependencies**: TICKET-2.1

**Description:**
Detect health trend anomalies and deliver proactive push notifications. Alerts are ranked by severity (info, caution, urgent, emergency). Emergency alerts (critical vitals) include a "Call 911" prompt. Alert fatigue prevention: max 1 non-urgent alert per day.

**Acceptance Criteria:**
- [ ] Alert triggers defined for: HRV decline >10% over 5 days, resting HR elevated >15% for 3 days, sleep <5hrs for 3 consecutive nights, SpO2 <94% any reading
- [ ] Emergency threshold: SpO2 <90% or resting HR >150 or <40 → in-app banner + push + "Call 911" button
- [ ] Push notification delivered within 15 minutes of threshold crossing
- [ ] User can configure alert thresholds for non-emergency alerts
- [ ] Max 1 caution/info alert per 24 hours (prevent fatigue)
- [ ] Alert history visible in app (last 30 days)
- [ ] APNs background delivery configured for real-time alerts

**Technical Notes:**
- Emergency alerts bypass the daily rate limit
- Use HealthTrack GraphQL `observationAdded` subscription for real-time monitoring
- APNs requires push notification entitlement in Xcode

---

### TICKET-2.3: AI Chat — "Ask Personal"
**Story Points**: 5  
**Dependencies**: TICKET-2.1

**Description:**
In-app AI chat grounded in the user's health data. User asks questions ("Why is my HRV low?", "Should I be worried about my sleep?") and AI responds with context from their actual data. Clearly labeled as AI, not medical advice. Free tier: 10 messages/month.

**Acceptance Criteria:**
- [ ] Chat UI: message bubbles, streaming response, conversation history (last 30 days)
- [ ] AI response is grounded in user's actual FHIR observations (not generic)
- [ ] Every response includes disclaimer: "This is AI-generated wellness information, not medical advice."
- [ ] Free tier: message counter visible, soft limit at 10/month with upgrade prompt
- [ ] Plus/Premium: unlimited messages
- [ ] Response time: <5 seconds for first token (streaming)
- [ ] No PHI leaked in error messages or logs

---

### TICKET-2.4: Clinical Report Generation
**Story Points**: 5  
**Dependencies**: TICKET-2.1

**Description:**
Generate a shareable PDF clinical health report from the user's health graph. Templates: annual summary, pre-visit summary, specialist referral context. Free tier: 1 report/month. Reports can be shared via link (public URL, no auth required) or downloaded as PDF.

**Acceptance Criteria:**
- [ ] Report types available: Annual Summary, Pre-Visit Summary
- [ ] Report includes: health score trend, all tracked metrics with 30-day chart, notable alerts, medications (if entered), open questions for doctor
- [ ] PDF generated server-side (not client-side); link expires after 30 days
- [ ] Shareable link works on desktop browser without app install
- [ ] Free tier: counter shown, month resets on billing anniversary
- [ ] Report generation < 10 seconds
- [ ] Report clearly marked "Patient-Generated Health Data — Not a Medical Record"

**Technical Notes:**
- Use existing Chiron report generation backend (`caremind-ai/chiron`)
- PDF rendering: Puppeteer on Fly.io or CF Worker with html-pdf-node

---

### TICKET-2.5: Onboarding Flow
**Story Points**: 5  
**Dependencies**: TICKET-2.2, TICKET-2.3

**Description:**
First-run experience: account creation, HealthKit permissions, historical sync, and first health score reveal. Onboarding must achieve >80% HealthKit permission grant rate and leave users with a meaningful health score on Day 1.

**Acceptance Criteria:**
- [ ] Step 1: Sign in with Apple (primary) or Google
- [ ] Step 2: HealthKit permission request — all 16 types in a single permission sheet with plain-language explanation of why each matters
- [ ] Step 3: Historical sync progress indicator ("Importing 6 months of health data...")
- [ ] Step 4: First health score reveal with animation and explanation
- [ ] Step 5: First proactive insight shown ("Your sleep has averaged 6.2 hrs — let's watch that")
- [ ] Skip path: user can skip HealthKit but sees reduced experience messaging
- [ ] <80% HealthKit grant rate triggers A/B test flag for permission copy

---

### TICKET-2.6: Beta Distribution (TestFlight)
**Story Points**: 2  
**Dependencies**: TICKET-2.5

**Description:**
Set up TestFlight beta distribution for 200 beta users. Waitlist-to-beta conversion: send TestFlight invite links to first 200 waitlist signups. Capture in-app feedback.

**Acceptance Criteria:**
- [ ] TestFlight build passes Apple review
- [ ] Beta invite links sent to first 200 waitlist emails via Loops.so
- [ ] In-app feedback button (long-press anywhere → feedback modal)
- [ ] Crash reporting enabled (Firebase Crashlytics or Sentry)
- [ ] Analytics events firing for: app open, health score view, alert received, report generated, chat message sent

---

## Definition of Done

- [ ] 200 beta users onboarded
- [ ] D7 retention ≥ 40%, D30 retention ≥ 35%
- [ ] Health score, alerts, AI chat, and clinical reports all working
- [ ] Zero critical crashes in beta (P0 crash rate < 0.1%)
- [ ] All BAAs confirmed signed before any beta user data is processed

## Notes

- D30 retention of 35% is the gate for Phase 2 (public App Store launch). If retention is below 30%, hold launch and iterate on core loop.
- Emergency escalation (TICKET-2.2) is P0 — do not ship without it.
- "Ask Personal" chat must clearly distinguish AI from GP throughout — this matters for regulatory posture.
