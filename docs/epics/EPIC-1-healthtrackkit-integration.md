# EPIC-1: HealthTrackKit Integration + Consumer Foundation

**Priority**: P0 (Critical)  
**Phase**: 1 — Foundation  
**Timeline**: M1–M2 (June–July 2026)

## Overview

Integrate the existing HealthTrackKit SPM into the Personal iOS app and validate the full data pipeline end-to-end. Build the waitlist capture for pre-launch email collection. This epic establishes the data foundation everything else is built on.

## User Stories

As a user, I want my Apple Watch and iPhone health data to sync automatically so that Personal can monitor my health without me manually entering anything.

As a product team, I want a smoke test that proves the full HealthKit → FHIR → AI pipeline works before we open beta to anyone.

## Child Tickets

### TICKET-1.1: HealthTrackKit SPM Integration — Personal iOS
**Story Points**: 5  
**Dependencies**: None

**Description:**
Add HealthTrackKit as an SPM dependency in the Personal iOS app. Wire up permissions request, incremental sync (on `sceneDidBecomeActive`), and historical sync (on first login). Patient ID comes from auth token's `healthTrackPatientId` field.

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

### TICKET-1.2: End-to-End Pipeline Smoke Test
**Story Points**: 3  
**Dependencies**: TICKET-1.1

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

### TICKET-1.3: Waitlist Email Capture
**Story Points**: 2  
**Dependencies**: None

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

- [ ] HealthTrackKit integrated and syncing in TestFlight build
- [ ] Smoke test passing against production infrastructure
- [ ] Waitlist form live and capturing emails

## Notes

- The smoke test (TICKET-1.2) gates the M3 beta — no external users without it passing.
- BAA procurement and HIPAA compliance sign-off are deferred to EPIC-6 (pre-commercialization). See TICKET-6.7–6.9 for details. Development uses synthetic/test data until BAAs are in place.
