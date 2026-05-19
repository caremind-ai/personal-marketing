# EPIC-6: Scale Foundations

**Priority**: P1 (High)  
**Phase**: 4 — Scale  
**Timeline**: M11–M12 (April–May 2027)

## Overview

Lay the commercial and technical foundations for scale: Family plan, Employer B2B pilot, GoodRx eRx integration, lab order routing, and 50-state telemedicine coverage. By end of M12, Personal should have a clear path to $5M ARR.

## User Stories

As a family, I want to add my elderly parents to my Personal plan so my whole family has a GP who knows each of us individually.

As an employer, I want to offer Personal as an employee health benefit so my workforce gets proactive, continuous care — reducing sick days and ER visits.

As a Care subscriber, I want my GP to send my Rx electronically to a pharmacy that offers the best price, so I don't have to figure out GoodRx myself.

## Child Tickets

### TICKET-6.1: Family Plan
**Story Points**: 8  
**Dependencies**: EPIC-5

**Description:**
Launch Family plan ($79 base + $35/additional member). Primary member invites additional adult members. Each member gets their own health profile and GP (shared GP by default; individual GP option available). Elderly parents supported; minors deferred to v2.

**Acceptance Criteria:**
- [ ] Family plan product in RevenueCat: `personal_family_monthly` at $79 + $35/member
- [ ] Primary member sends invite link (email) to additional members
- [ ] Additional member creates their own account and links to family plan
- [ ] Shared GP option: all family members assigned to same GP (default)
- [ ] Individual GP option: each member assigned their own GP (+$35/member difference still applies)
- [ ] Family health dashboard: primary member can see all members' health scores and alerts
- [ ] Caregiver mode: primary member can view (not edit) elderly parent's full health graph (with explicit consent from parent)
- [ ] Billing: single charge to primary member's payment method; itemized receipt
- [ ] Member removal: primary member can remove a member; removed member's data stays with them

---

### TICKET-6.2: Employer B2B — Pilot (3 Employers)
**Story Points**: 8  
**Dependencies**: EPIC-5

**Description:**
Sign 3 employer pilot agreements. Employer pays $200–$400/employee/year for Personal Care tier for their workforce. Build employer admin dashboard: roster management, aggregate utilization reporting (no individual PHI), billing.

**Acceptance Criteria:**
- [ ] Employer onboarding: signed BAA (employer as Business Associate), roster upload (CSV or SSO)
- [ ] Employee invitation flow: employer sends invite to employee email; employee creates account
- [ ] Employer admin dashboard: roster, seats used/available, aggregate utilization (% with health score, % who had GP interaction)
- [ ] Individual employee data: NOT visible to employer (only aggregate, de-identified)
- [ ] Billing: employer invoiced monthly via Stripe; net-30 terms
- [ ] 3 pilot employers signed with ≥100 employees total
- [ ] Employee onboarding rate target: 40% of invited employees activate within 30 days

---

### TICKET-6.3: GoodRx eRx Integration
**Story Points**: 5  
**Dependencies**: EPIC-4 (GP Rx authorization), EPIC-5

**Description:**
When a GP approves an Rx, route electronically to the pharmacy with the best GoodRx price for the patient's location. Patient sees price comparison before pharmacy is selected. Integration via Surescripts (eRx transmission) with GoodRx for price data.

**Acceptance Criteria:**
- [ ] On GP Rx approval: GoodRx API returns prices at nearby pharmacies for the prescribed medication
- [ ] Patient sees top 3 pharmacy options with GoodRx price + distance
- [ ] Patient selects pharmacy → Rx transmitted electronically via Surescripts or DrFirst
- [ ] Pharmacy confirmation sent to patient (push notification + in-app)
- [ ] Rx pickup reminder sent 1 day before estimated ready time
- [ ] Refill reminder: 7 days before estimated supply runs out (based on day supply on Rx)
- [ ] Non-controlled Rx only (controlled Rx blocked with explanation)
- [ ] GoodRx referral fee tracked per filled Rx (monetization stream)

**Technical Notes:**
- Surescripts requires DEA EPCS certification for electronic prescribing; start certification process in M9
- GoodRx Publisher API for price data; affiliate program for referral fees

---

### TICKET-6.4: Lab Order Routing (LabCorp + Quest)
**Story Points**: 5  
**Dependencies**: EPIC-4 (GP lab order creation)

**Description:**
When a GP creates a lab order, route it to LabCorp or Quest (patient's choice). Patient receives order on their phone, walks into any lab location, gets results. Results auto-ingested into HealthTrack when available.

**Acceptance Criteria:**
- [ ] GP creates lab order in Provider AI OS → order transmitted to selected lab network
- [ ] Patient receives lab order via push + in-app: "Your order is ready. Find a lab location."
- [ ] Patient selects LabCorp or Quest; nearest locations shown on map
- [ ] Lab results ingested automatically via HL7 FHIR R4 result feed (or LabCorp/Quest API)
- [ ] Results trigger AI analysis and update health graph
- [ ] Patient notified when results are available
- [ ] GP notified of result; critical values (flagged by lab) surface immediately in GP dashboard
- [ ] Lab referral fee tracked (monetization stream)

---

### TICKET-6.5: 50-State Telemedicine Coverage
**Story Points**: 5  
**Dependencies**: EPIC-5

**Description:**
Expand GP coverage to all 50 US states. Requires GPs to hold multi-state IMLC licenses (or individual state licenses). Build GP license matrix: which GP can see which patients based on patient's state.

**Acceptance Criteria:**
- [ ] License matrix in GP profile: GP lists all state licenses
- [ ] Patient-GP matching: patient can only be assigned to GP licensed in patient's state
- [ ] IMLC participation: at least 4 of 10 GPs hold IMLC compact licenses (covering 40 states)
- [ ] Unlicensed state handling: if patient moves to state where their GP isn't licensed, system flags and offers GP reassignment
- [ ] State coverage dashboard (internal): which states have at least 1 licensed GP
- [ ] All 50 states covered (at least 1 GP licensed per state) by end of M12

---

### TICKET-6.6: Android MVP (conditional)
**Story Points**: 13  
**Dependencies**: Android decision (Open Decision from PRD)  
**Status**: Conditional — only if Android decision is made by M10

**Description:**
Build an Android version of the Personal app using Google Health Connect for health data (equivalent to HealthKit). Prioritize the Access tier use case (Medicaid/SNAP users skew Android).

**Acceptance Criteria:**
- [ ] Android app: health score, proactive alerts, AI chat, clinical reports
- [ ] Google Health Connect: read permissions for steps, heart rate, sleep, SpO2, HRV (available types)
- [ ] Background sync: WorkManager + Health Connect background reading
- [ ] Feature parity with iOS Free + Plus tiers
- [ ] App submitted to Google Play Store
- [ ] Note: HealthKit-specific types (HRV, SpO2 background, mindfulness) may not be available on Android — document gaps

**Technical Notes:**
- Health Connect API is Google's unified health data API (Android 9+)
- Background reading requires Health Connect permission + WorkManager
- Not all 16 HealthTrack types are available on Android; graceful degradation required

---

### TICKET-6.7: BAA Procurement — Anthropic + Cloudflare
**Story Points**: 3  
**Dependencies**: None (can start any time; complete before commercial launch)

**Description:**
Obtain signed BAAs from Anthropic (covering Claude models in the AI synthesis pipeline) and Cloudflare (covering Workers, KV, R2, AI Gateway). Both require paid/enterprise plans.

**Acceptance Criteria:**
- [ ] Anthropic enterprise agreement signed; BAA executed
- [ ] Cloudflare BAA executed (Cloudflare Business or Enterprise plan)
- [ ] Both BAAs stored in secure legal document repository
- [ ] ENVIRONMENTS.md updated with BAA effective dates
- [ ] AI worker and gateway deployment configs annotated with BAA coverage

---

### TICKET-6.8: BAA Procurement — Fly.io + Supabase + Doxy.me
**Story Points**: 2  
**Dependencies**: None (can start any time; complete before commercial launch)

**Description:**
Obtain signed BAAs from Fly.io (hosting Rust Core), Supabase (PostgreSQL PHI store), and Doxy.me (HIPAA-compliant video). All three offer BAAs on paid plans.

**Acceptance Criteria:**
- [ ] Fly.io BAA executed (Fly.io Scale plan required)
- [ ] Supabase BAA executed (confirm signed, not just HIPAA plan active)
- [ ] Doxy.me BAA executed (required for HIPAA video visits with real patients)
- [ ] All BAAs stored in secure legal document repository
- [ ] ENVIRONMENTS.md updated with all BAA statuses

---

### TICKET-6.9: HIPAA Compliance Audit + Privacy Policy
**Story Points**: 3  
**Dependencies**: TICKET-6.7, TICKET-6.8

**Description:**
Before commercial launch: complete HIPAA Security Rule technical safeguards checklist, publish a HIPAA-compliant privacy policy and Notice of Privacy Practices (NPP), and confirm all PHI data flows are covered by executed BAAs.

**Acceptance Criteria:**
- [ ] HIPAA Security Rule checklist completed (access controls, audit logs, encryption at rest + in transit, breach notification procedure)
- [ ] Privacy policy published at `personal.health/privacy` (or equivalent)
- [ ] Notice of Privacy Practices (NPP) in app and on website
- [ ] All PHI data flows mapped and confirmed covered by BAAs (no uncovered vendor)
- [ ] SECURITY.md updated with full compliance status
- [ ] External HIPAA counsel review completed (or internal sign-off if counsel unavailable)

---

## Definition of Done

- [ ] Family plan live and accepting multi-member enrollments
- [ ] 3 employer pilots active with ≥100 employees
- [ ] eRx routing live (GoodRx integration)
- [ ] Lab order routing live (LabCorp + Quest)
- [ ] All 50 states covered by at least 1 licensed GP
- [ ] All BAAs signed (Anthropic, Cloudflare, Fly.io, Supabase, Doxy.me)
- [ ] HIPAA compliance audit complete; privacy policy and NPP published
- [ ] MRR ≥ $150K (all tiers combined)
- [ ] Path to $5M ARR visible (500 Care + B2B + referral revenue)

## Notes

- Employer B2B (TICKET-6.2) is a high-value monetization unlock. A 100-employee deal at $300/employee = $30K MRR from one contract. Start employer sales motion in M9 in parallel with EPIC-5 build.
- Android (TICKET-6.6) is explicitly conditional. Make the Android decision by M10 — the 13-story-point estimate means it can't be added late without delaying other M11-M12 work.
- EPCS certification for Surescripts (TICKET-6.3) takes 2–3 months. Start in M9 to have it ready for M11.
