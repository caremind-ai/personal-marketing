# EPIC-4: Provider AI OS v1

**Priority**: P0 (Critical)  
**Phase**: 3 — GP Layer  
**Timeline**: M7–M8 (December 2026–January 2027)

## Overview

Build the web-based Provider AI OS that enables GPs to manage their patient panel at scale. The GP sees an AI-digested view of every patient, a priority queue of who needs attention today, and tools to review/approve AI-drafted messages, Rx requests, and referrals. Launch with 2 contracted GPs in beta.

## User Stories

As a GP, I want to see which patients need my attention today — ranked by urgency — so I can focus my limited async time where it matters most.

As a GP, I want to read an AI-generated summary of a patient before I respond to their message, so I don't have to review months of raw data before every interaction.

As a GP, I want to review and approve AI-drafted Rx requests with one click, so I can authorize refills in seconds rather than minutes.

## Child Tickets

### TICKET-4.1: GP Authentication + Onboarding
**Story Points**: 3  
**Dependencies**: EPIC-1 (HealthTrack GraphQL API)

**Description:**
GP-facing web app (Next.js) with GP authentication separate from patient auth. GPs onboard with license verification (state, NPI number). Each GP is assigned a patient panel.

**Acceptance Criteria:**
- [ ] GP login via email + password (separate from patient Sign in with Apple)
- [ ] Onboarding: GP enters name, NPI, state license(s), specialty
- [ ] NPI verified against NPPES registry (NPI lookup API)
- [ ] GP role assigned in HealthTrack; GP can only see their assigned patients
- [ ] Session: JWT with 8-hour expiry, refresh on activity
- [ ] GP dashboard accessible at `app.personal.health` (or subdomain TBD)

---

### TICKET-4.2: Patient Panel Dashboard
**Story Points**: 8  
**Dependencies**: TICKET-4.1

**Description:**
The GP's home screen: a list of all assigned patients sorted by AI-computed urgency. Each patient row shows name, last activity, current alert status, and one-line AI summary. GP can see who needs attention today at a glance.

**Acceptance Criteria:**
- [ ] Patient list: all assigned patients, sorted by urgency (highest → lowest)
- [ ] Urgency computed by AI: considers active alerts, unread messages, upcoming appointments, overdue follow-ups
- [ ] Each row shows: patient name, age, primary conditions, last interaction date, current alert badge (none/caution/urgent/emergency)
- [ ] One-line AI patient summary: "Alex, 34M, hypertension + elevated stress. HRV declining 5 days."
- [ ] Filter: "Needs attention today" (urgent+), "All patients", "Recently active"
- [ ] Real-time updates: new alerts surface without page refresh (SSE subscription)
- [ ] Panel count visible: "Your panel: 47 patients"

---

### TICKET-4.3: Patient Health Graph View
**Story Points**: 8  
**Dependencies**: TICKET-4.2

**Description:**
Full longitudinal patient view for the GP. Shows AI-generated patient summary, all health metrics with charts, complete alert history, visit history, current medications, and open action items. This is the GP's primary context before any interaction.

**Acceptance Criteria:**
- [ ] AI-generated patient summary: 3–5 sentences, updated daily, includes recent trends and open concerns
- [ ] Health metrics charts: HRV, resting HR, sleep, SpO2, steps — last 30/90/365 days (toggleable)
- [ ] Alert history: all alerts with timestamps, severity, GP response (if any)
- [ ] Medication list: from Flexpa EHR + any GP-prescribed
- [ ] Visit history: all past interactions (async messages, video calls) with AI summaries
- [ ] Open action items: pending Rx approvals, pending referrals, scheduled appointments
- [ ] "Share with patient" toggle: GP can mark notes as patient-visible

---

### TICKET-4.4: Async Messaging with AI Draft Assist
**Story Points**: 8  
**Dependencies**: TICKET-4.3

**Description:**
Patient sends a message → AI generates a GP reply draft grounded in the patient's full health graph → GP reviews, edits, and sends. GP never sends raw AI text without reviewing. Response time target: GP responds within 4 hours during business hours.

**Acceptance Criteria:**
- [ ] Inbox: all unread patient messages, sorted by urgency
- [ ] Message thread view: full conversation history between patient and GP
- [ ] AI draft generated automatically when GP opens a new message (< 3 seconds)
- [ ] AI draft clearly labeled "AI Draft — Review before sending"
- [ ] GP can: edit draft, discard and write from scratch, or send as-is
- [ ] Sent messages visible to patient immediately in their app
- [ ] Push notification to GP on new urgent patient message (outside business hours: flagged for next business day unless emergency)
- [ ] Response time SLA: 4 hours (business hours); shown to patient as expected response time

---

### TICKET-4.5: Rx and Referral Authorization Workflow
**Story Points**: 5  
**Dependencies**: TICKET-4.4, EPIC-3 TICKET-3.3

**Description:**
Queue of pending Rx refill requests and referral letters for GP review. GP sees AI-drafted Rx/referral with full patient context, approves or modifies, and transmits. This closes the loop on EPIC-3's agentic Rx requests.

**Acceptance Criteria:**
- [ ] Pending Rx queue: all pending refill requests with patient context and AI draft
- [ ] GP one-click approve → triggers GoodRx routing and pharmacy notification
- [ ] GP can modify dosage/notes before approving
- [ ] GP can reject with reason → patient notified with reason
- [ ] Pending referral queue: AI-drafted referral letter for GP review and signature
- [ ] Signed referrals sent to specialist via secure fax (Updox API) or Direct messaging
- [ ] Controlled substance requests: blocked from queue with clear label
- [ ] Audit log: all GP actions (approve/reject/modify) logged with timestamp

---

### TICKET-4.6: GP Beta Onboarding — 2 Contracted GPs
**Story Points**: 3  
**Dependencies**: TICKET-4.5

**Description:**
Recruit and onboard 2 contracted GPs for beta. GPs sign contractor agreement covering scope, compensation, response SLAs, and malpractice coverage confirmation. Assign initial patient panels of 10–20 Care tier subscribers.

**Acceptance Criteria:**
- [ ] 2 GPs contracted with signed agreement (NDA, contractor terms, SLA, malpractice confirmation)
- [ ] Each GP completes onboarding in Provider AI OS: NPI verified, state licenses confirmed
- [ ] Each GP assigned initial panel of 10–20 Care tier users (from beta waitlist)
- [ ] GP training session: 1-hour walkthrough of Provider AI OS with Q&A
- [ ] GP feedback loop: weekly async check-in for first 4 weeks
- [ ] Emergency coverage: GPs coordinate coverage between themselves; escalation to urgent care for after-hours

---

## Definition of Done

- [ ] Provider AI OS live at production URL
- [ ] 2 GPs onboarded and actively managing panels
- [ ] Patient panel, health graph, messaging, Rx authorization all working
- [ ] GP response time < 4 hours for 90% of messages
- [ ] Zero patient PHI accessible to wrong GP (row-level security validated)
- [ ] North Star metric now trackable: "Monthly Active Patients with meaningful GP interaction"

## Notes

- Row-level security is critical — a GP must never see another GP's patients. Test this aggressively.
- The AI draft (TICKET-4.4) sets the GP's productivity floor. If it's wrong or unhelpful, GPs will stop using it and hit burnout. Invest in prompt quality.
- GP beta (TICKET-4.6) requires legal review of contractor agreement and malpractice structure. Start this in M6 in parallel with build.
