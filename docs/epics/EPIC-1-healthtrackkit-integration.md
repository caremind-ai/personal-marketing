# EPIC-1: HealthTrackKit Integration + BAA Procurement

**Priority**: P0 (Critical)  
**Phase**: 1 — Foundation  
**Timeline**: M1–M2 (June–July 2026)

## Overview

Integrate the existing HealthTrackKit SPM into the PreventHealth iOS app and validate the full data pipeline end-to-end. Simultaneously procure BAAs from all PHI-handling vendors. Nothing else can ship until both are complete — the BAAs are a legal blocker and the sync is the product's data foundation.

## User Stories

As a user, I want my Apple Watch and iPhone health data to sync automatically so that Personal can monitor my health without me manually entering anything.

As a physician (future), I want to trust that all patient data was collected with proper consent and is stored in a HIPAA-compliant system before I interact with any patient.

## Child Tickets

### TICKET-1.1: BAA Procurement — Anthropic
**Story Points**: 3  
**Dependencies**: None

**Description:**
Obtain a signed BAA from Anthropic covering use of Claude models in the AI synthesis pipeline. Required before any real user PHI flows through the AI worker. Anthropic BAA requires an enterprise agreement.

**Acceptance Criteria:**
- [ ] Anthropic enterprise agreement signed
- [ ] BAA executed and stored in secure legal document repository
- [ ] AI worker deployment config updated to note BAA effective date
- [ ] ENVIRONMENTS.md updated with BAA status

**Technical Notes:**
- Contact: Anthropic enterprise sales
- Current AI worker uses Claude Sonnet 4.6 via CF service binding

---

### TICKET-1.2: BAA Procurement — Cloudflare
**Story Points**: 2  
**Dependencies**: None

**Description:**
Obtain a signed BAA from Cloudflare covering Workers, KV, R2, and AI Gateway. Required before PHI flows through the gateway or AI worker.

**Acceptance Criteria:**
- [ ] Cloudflare BAA executed (available via Cloudflare enterprise plan)
- [ ] BAA stored in secure legal document repository
- [ ] ENVIRONMENTS.md updated with BAA status

---

### TICKET-1.3: BAA Procurement — Fly.io + Supabase
**Story Points**: 2  
**Dependencies**: None

**Description:**
Obtain signed BAAs from Fly.io (hosting Rust Core) and Supabase (PostgreSQL with PHI). Both offer HIPAA BAAs on paid plans.

**Acceptance Criteria:**
- [ ] Fly.io BAA executed (available on Fly.io Scale plan)
- [ ] Supabase BAA executed (HIPAA plan already active — confirm BAA is signed, not just plan)
- [ ] BAAs stored in secure legal document repository
- [ ] ENVIRONMENTS.md updated with all BAA statuses

---

### TICKET-1.4: HealthTrackKit SPM Integration — PreventHealth iOS
**Story Points**: 5  
**Dependencies**: TICKET-1.1, TICKET-1.2, TICKET-1.3 (BAAs before any PHI)

**Description:**
Add HealthTrackKit as an SPM dependency in the PreventHealth iOS app. Wire up permissions request, incremental sync (on `sceneDidBecomeActive`), and historical sync (on first login). Patient ID comes from auth token's `healthTrackPatientId` field.

**Acceptance Criteria:**
- [ ] SPM dependency added: `https://github.com/caremind-ai/healthtrack`, product `HealthTrackKit`
- [ ] HealthKit capability enabled in Xcode project entitlements
- [ ] On login: `syncer.requestPermissions()` called; user sees permission dialog for all 16 types
- [ ] On `sceneDidBecomeActive`: `syncer.syncIncremental()` runs silently in background
- [ ] On first login: `syncer.syncHistorical()` runs and uploads up to 6 months of history
- [ ] Anchors persisted in Keychain; no duplicate uploads on retry
- [ ] Failed syncs do not advance anchor (data loss prevention)
- [ ] Console log emitted on sync completion: type count, samples uploaded, errors

**Technical Notes:**
- Pattern documented in `caremind-ai/healthtrack` handoff doc
- HealthTrackKit already built with `HKAnchoredObjectQuery` per type; just needs wiring
- Patient ID: parse from JWT `healthTrackPatientId` claim

---

### TICKET-1.5: End-to-End Pipeline Smoke Test
**Story Points**: 3  
**Dependencies**: TICKET-1.4

**Description:**
Write and run a documented smoke test proving the full pipeline works: iOS HealthKit sync → FHIR observation in DB → GraphQL query returns it → AI synthesis returns a non-error insight. Required before beta opens to any external users.

**Acceptance Criteria:**
- [ ] Sync a known heart rate observation from a test device
- [ ] Observation appears in `observations` GraphQL query with correct LOINC code (8867-4)
- [ ] `insights` GraphQL query returns AI-generated insight (non-error) referencing the observation
- [ ] GraphQL subscription (`observationAdded`) fires when a new sync arrives
- [ ] Test documented in `docs/smoke-test.md` with steps and expected output
- [ ] Runs clean against production gateway + AI worker (not just local)

---

### TICKET-1.6: Waitlist Email Capture
**Story Points**: 2  
**Dependencies**: None (parallel with BAAs)

**Description:**
Add email capture to `personal.g-a-l-a-c-t-i-c.com` marketing site. Store signups in Loops.so (no PHI, no HealthKit data). Target: 5,000 waitlist signups before beta opens in M3.

**Acceptance Criteria:**
- [ ] Email input form added to marketing site hero section
- [ ] Successful submission shows confirmation ("You're on the list")
- [ ] Emails stored in Loops.so audience (or equivalent)
- [ ] Duplicate submissions handled gracefully (no error, silent dedup)
- [ ] No PHI collected (email only)
- [ ] GDPR-compliant: checkbox + link to privacy policy (even if policy is placeholder)

**Technical Notes:**
- Loops.so has a simple API: `POST https://app.loops.so/api/v1/contacts/create`
- No backend needed — can call from frontend with public API key

---

## Definition of Done

- [ ] All 4 BAAs signed and filed
- [ ] HealthTrackKit integrated and syncing in TestFlight build
- [ ] Smoke test passing against production infrastructure
- [ ] Waitlist form live and capturing emails
- [ ] No PHI in any system without a signed BAA

## Notes

- TICKET-1.1 through 1.3 (BAAs) are on the critical path for everything else. Start immediately.
- BAA negotiations can take 2–4 weeks; don't let them block engineering work on TICKET-1.4.
- The smoke test (TICKET-1.5) gates the M3 beta — no external users without it passing.
