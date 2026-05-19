# EPIC-5: Care Tier + Telemedicine

**Priority**: P0 (Critical)  
**Phase**: 3 — GP Relationship at Scale  
**Timeline**: M9–M10 (February–March 2027)

## Overview

Launch the Care tier ($79/mo) with a dedicated GP relationship and embedded telemedicine via Doxy.me. AI generates pre-visit summaries automatically before every video call and post-visit drafts (SOAP note, Rx, follow-up plan) for GP review. Target: 500 Care subscribers, 10 GPs.

## User Stories

As a Care subscriber, I want a scheduled video call with my own GP who already knows everything about my health before we start, so I don't have to spend the first 10 minutes explaining my history.

As a GP, I want the AI to generate a pre-visit summary of my patient's recent trends, medications, and concerns before every video call, so I walk in with full context and can focus on the clinical conversation.

As a Care subscriber, I want to receive an after-visit summary and follow-up plan within an hour of my video call, so I know exactly what to do next.

## Child Tickets

### TICKET-5.1: Care Tier Billing + GP Assignment
**Story Points**: 5  
**Dependencies**: EPIC-3 (billing infrastructure), EPIC-4 (GP panel)

**Description:**
Launch Care tier ($79/mo) billing. On subscription, automatically assign the user to a GP with available panel capacity. GP is notified of new patient. Patient sees their GP's name, photo, and bio immediately after subscribing.

**Acceptance Criteria:**
- [ ] Care tier product in RevenueCat: `personal_care_monthly` at $79
- [ ] On subscription: GP assignment algorithm runs (assign to GP with lowest panel utilization)
- [ ] Patient sees "Your doctor: Dr. [Name]" with photo and bio within 60 seconds of subscribing
- [ ] GP notified of new patient via push notification (Provider AI OS)
- [ ] GP assignment is sticky — patient keeps same GP unless GP leaves platform or patient requests change
- [ ] Panel capacity limit: GP not assigned new patients if panel > 200 (configurable)
- [ ] Cancellation: patient retains GP access until billing period ends

---

### TICKET-5.2: Video Visit Scheduling
**Story Points**: 5  
**Dependencies**: TICKET-5.1

**Description:**
Patient books a video visit with their assigned GP. GP sets their availability in the Provider AI OS. Patient sees open slots and books. Reminders sent 24 hours and 1 hour before the visit.

**Acceptance Criteria:**
- [ ] GP sets availability: weekly recurring schedule (e.g., Mon/Wed/Fri 9am-12pm)
- [ ] Patient sees GP's available slots (next 14 days) and selects one
- [ ] Booking confirmation: push notification + calendar invite (EventKit)
- [ ] 24-hour reminder: push notification + in-app
- [ ] 1-hour reminder: push notification with "Join call" deep link
- [ ] Rescheduling: patient can reschedule up to 2 hours before visit
- [ ] Cancellation: patient can cancel up to 2 hours before; GP slot opens
- [ ] Care plan: 2 video visits/month included; additional visits $49 each

---

### TICKET-5.3: Doxy.me Video Integration
**Story Points**: 8  
**Dependencies**: TICKET-5.2

**Description:**
Embed Doxy.me HIPAA-compliant video in the iOS app for the patient side and in the Provider AI OS for the GP side. Both parties join through the app — no external downloads or links.

**Acceptance Criteria:**
- [ ] Patient: "Join call" button in app at visit time → embedded Doxy.me room opens
- [ ] GP: "Start visit" button in Provider AI OS → Doxy.me room opens in browser
- [ ] Patient waiting room: shows patient vitals (current health score, recent alerts) while waiting for GP
- [ ] GP waiting room: shows AI pre-visit summary while waiting for patient to join
- [ ] Video quality: minimum 720p; fallback to audio-only if bandwidth < 500kbps
- [ ] End call: both sides see "Visit ended" confirmation
- [ ] Recording: disabled by default; opt-in only with mutual consent + explicit consent confirmation
- [ ] HIPAA: Doxy.me BAA in place; no video data stored on Personal infrastructure

**Technical Notes:**
- Doxy.me has a HIPAA BAA available for paid plans
- Embed via WKWebView (iOS) and iframe (Provider OS web)
- Room URL format: `https://doxy.me/dr-[gp-id]/[appointment-id]`

---

### TICKET-5.4: AI Pre-Visit Summary
**Story Points**: 5  
**Dependencies**: TICKET-5.2

**Description:**
AI automatically generates a pre-visit summary 30 minutes before every scheduled video call. Summary is shown in the GP's waiting room view and is optionally shared with the patient. GP can annotate or override before the call begins.

**Acceptance Criteria:**
- [ ] Summary generated automatically 30 minutes before scheduled call time
- [ ] Summary includes: recent health trend (7 days), notable alerts since last visit, current medications, open action items, patient's stated reason for visit (if provided in booking)
- [ ] Summary is 200–400 words, plain prose with a brief bullet summary
- [ ] GP can add notes to summary before call
- [ ] Patient-visible version: shorter (5–7 bullets), no clinical terminology
- [ ] Summary archived with visit record for future reference
- [ ] Generation time: < 15 seconds

---

### TICKET-5.5: Post-Visit AI Drafts
**Story Points**: 5  
**Dependencies**: TICKET-5.3

**Description:**
After the video call ends, AI generates: (1) draft SOAP note for GP review, (2) after-visit summary for patient, (3) follow-up Rx request if applicable, (4) referral letter if applicable. GP reviews and sends within 2 hours of visit.

**Acceptance Criteria:**
- [ ] SOAP note draft generated within 5 minutes of call ending
- [ ] SOAP note: Subjective, Objective (from health data), Assessment, Plan — GP-facing
- [ ] After-visit summary: patient-facing, plain language, includes: what was discussed, action items, next steps, follow-up date
- [ ] Patient receives after-visit summary push notification within 2 hours
- [ ] If Rx discussed: draft Rx request added to GP's pending queue
- [ ] If referral discussed: draft referral letter added to GP's pending queue
- [ ] GP can edit any draft before sending/signing
- [ ] All drafts and final versions stored in patient record

---

### TICKET-5.6: GP Scale — 10 GPs
**Story Points**: 3  
**Dependencies**: TICKET-5.1

**Description:**
Scale from 2 GP beta to 10 GPs. Standardize GP onboarding process. Target: 500 Care subscribers across 10 GPs (avg 50 patients/GP at launch, scaling to 200+).

**Acceptance Criteria:**
- [ ] GP onboarding flow self-serve: NPI verification, license upload, bio/photo, availability setup
- [ ] GP agreement template finalized (legal review complete)
- [ ] 10 GPs contracted and onboarded
- [ ] GP marketplace page: internal directory for panel assignment (not patient-facing yet)
- [ ] Panel balancing: new Care subscribers auto-assigned to GP with lowest utilization
- [ ] GP offboarding: if GP leaves, patients reassigned within 48 hours with notification

---

## Definition of Done

- [ ] Care tier live at $79/mo
- [ ] 10 GPs active with assigned panels
- [ ] Video visits working end-to-end (Doxy.me embedded)
- [ ] Pre-visit summaries generated before 100% of scheduled visits
- [ ] Post-visit summaries delivered to patients within 2 hours for 90% of visits
- [ ] 500 Care subscribers reached
- [ ] Patient NPS ≥ 50

## Notes

- The pre-visit summary (TICKET-5.4) is the core differentiation. If the GP walks into a call with full context, patients feel the difference immediately. This is what makes Personal feel like concierge medicine.
- GP offboarding (TICKET-5.6) must be planned before launch — patients need assurance their GP isn't going away without notice.
- Doxy.me BAA and full HIPAA vendor sign-off are handled in EPIC-6 (TICKET-6.7–6.9) before commercial launch. Beta video visits use test accounts only.
