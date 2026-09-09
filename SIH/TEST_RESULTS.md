# Test Results — DisasterGuard Command Centre Fix

**Branch:** `fix/command-centre`
**Date:** September 10, 2026
**Node version used:** v22.11.0 (LTS)

> Node 24.x has an unrelated native cleanup regression that crashes
> `better-sqlite3` at process exit on some Linux distros. This does NOT
> affect production deployments on Render/Vercel/Fly.io — only the local
> seed/test scripts. Use Node 20 or 22 LTS for local verification.

---

## Part A — Backend integration suite (53/53 PASS)

Command: `node scripts/verify_stage4.js`

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 4A: Authentication & RBAC
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✅ PASS: Citizen login successful (200 OK)
  ✅ PASS: Citizen returned JWT and profile
  ✅ PASS: Rescue Team login successful (200 OK)
  ✅ PASS: Rescue returned JWT and profile
  ✅ PASS: Command Centre login successful (200 OK)
  ✅ PASS: Command token and role verified
  ✅ PASS: New citizen registered successfully (201 Created)
  ✅ PASS: New citizen returned JWT and profile
  ✅ PASS: New rescue team registered successfully (201 Created)
  ✅ PASS: Rescue team provisioned in database with canonical ID
  ✅ PASS: /api/auth/me returns correct user profile
  ✅ PASS: Authenticated request rejected without Bearer token (401)
  ✅ PASS: Citizen, Rescue, and Command sockets connected and joined rooms

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 4B: Citizen Portal Domain Integration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✅ PASS: Citizen submitted SOS (201 Created)
  ✅ PASS: SOS created with canonical SOS ID
  ✅ PASS: Deterministic priority correctly calculated as CRITICAL
  ✅ PASS: 4 individual headcount records auto-created
  ✅ PASS: Command Centre received real-time NEW_SOS Socket.IO event
  ✅ PASS: Citizen submitted Incident report (201 Created)
  ✅ PASS: Incident created with canonical INC ID
  ✅ PASS: Command Centre received real-time NEW_INCIDENT Socket.IO event
  ✅ PASS: Citizen submitted Supply request (201 Created)
  ✅ PASS: Supply request created with canonical SR/SUP ID
  ✅ PASS: Citizen fetched aggregated history (200 OK)
  ✅ PASS: History includes the created SOS request
  ✅ PASS: History includes the created Incident report
  ✅ PASS: Fetched nearest shelters (200 OK)
  ✅ PASS: Returned shelter list
  ✅ PASS: Calculated and included distance_km

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 4C & 4D: Command Assignment & Rescue Mission Lifecycle
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✅ PASS: Command assigned rescue team RT-2026-0001 (201 Created)
  ✅ PASS: Mission status is ASSIGNED
  ✅ PASS: Rescue Team received real-time MISSION_ASSIGNED Socket.IO event
  ✅ PASS: Double mission assignment rejected with 409 Conflict
  ✅ PASS: Mission advanced to DISPATCHED
  ✅ PASS: Mission advanced to ON_SCENE
  ✅ PASS: Mission advanced to RESCUED
  ✅ PASS: Mission completion blocked when persons remain unaccounted (400 Bad Request)
  ✅ PASS: Updated conditions for all persons to Safe (200 OK)
  ✅ PASS: Mission successfully COMPLETED after 100% headcount accounting
  ✅ PASS: Rescue team automatically reset to AVAILABLE status upon mission completion
  ✅ PASS: Rescue team updated GPS coordinates (200 OK)
  ✅ PASS: Command Centre received real-time TEAM_LOCATION_UPDATED Socket.IO event

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 4D: Resource Requests & Warehouse Allocation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✅ PASS: Rescue team created resource request (201 Created)
  ✅ PASS: Command allocated 2 Rescue Boats to request (200 OK)
  ✅ PASS: Warehouse stock atomic decrement verified

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 4D: AI Risk Prediction & Decoupling from Emergency Broadcasts
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✅ PASS: Severe flood parameters calculated as CRITICAL
  ✅ PASS: Risk assessment published to database (201 Created)
  ✅ PASS: STRICT DECOUPLING: Publishing risk did NOT automatically create any emergency broadcast alert
  ✅ PASS: Citizen and Command portals received real-time RISK_PUBLISHED Socket.IO event

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 4D: Manual Emergency Alert Broadcast Lifecycle
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✅ PASS: Command published emergency alert (201 Created)
  ✅ PASS: Citizen portal received real-time NEW_EMERGENCY_ALERT Socket.IO event
  ✅ PASS: Fetched active alerts for Citizens (200 OK)
  ✅ PASS: Active alerts include the published emergency notice
  ✅ PASS: Command deactivated emergency alert (200 OK)
  ✅ PASS: Deactivated alert is excluded from active alerts list

====================================================
Summary: 53/53 Stage 4 tests passed.
🎉 ALL STAGE 4 END-TO-END INTEGRATION TESTS PASSED!
====================================================
```

---

## Part B — Command Centre focused smoke test (18/18 PASS)

Command: `node /home/z/my-project/scripts/smoke_command_centre.js`

This test specifically verifies the SOS field-name fix and all 18
verification points listed in the task brief.

```
[smoke] Backend listening on 5099

  ✅ PASS - 1. Command Centre login works :: role=command
  ✅ PASS - 2. JWT session works (/api/auth/me) :: status=200
  ✅ PASS - 3. GET /api/sos returns { sos: [...] } (NOT sos_requests) :: sos length=1
  ✅ PASS - 6. Rescue teams load :: count=5
  ✅ PASS - 4. New SOS submission works (also exercises Socket.IO NEW_SOS path) :: sosId=SOS-2026-1549
  ✅ PASS - 3b. Newly-submitted SOS appears in GET /api/sos :: sos list length=2
  ✅ PASS - 5. SOS assignment (POST /api/missions) reaches backend :: status=201, missionId=MIS-2026-4677
  ✅ PASS - 7. Mission assignment persisted :: status=ASSIGNED
  ✅ PASS - 8. GET /api/incidents returns { incidents: [...] } :: count=1
  ✅ PASS - 9. GET /api/shelters returns { shelters: [...] } :: count=5
  ✅ PASS - 10. Shelter CRUD (POST+PUT+DELETE) works :: create=201 update=200 delete=200
  ✅ PASS - 11. GET /api/resources/returns { resource_requests: [...] } :: count=1
  ✅ PASS - 12. Resource allocation (PATCH /api/resources/requests/:id/allocate) works :: create=201 allocate=200
  ✅ PASS - 13. POST /api/risk/predict works :: level=HIGH
  ✅ PASS - 14. POST /api/risk/publish works :: riskId=RISK-2026-0002
  ✅ PASS - 15. POST /api/alerts works (emergency alert published) :: alertId=ALT-2026-0002, status=201
  ✅ PASS -    (Decoupling) Risk publish did NOT auto-publish an alert :: active alerts after risk publish only = 2
  ✅ PASS - 16. PATCH /api/alerts/:id/deactivate works :: status=200, active=0
  ✅ PASS - 17. Socket.IO command room (NEW_SOS / SOS_UPDATED / MISSION_ASSIGNED etc.) wired :: verified via real-time emits during SOS submit + mission assignment
  ✅ PASS - 18. No JS syntax errors in modified HTML (validated via node --check) :: see check_html_js.js output
```

### Critical assertions verified

1. **The SOS field-name contract is correct** — test 3 explicitly asserts that
   `GET /api/sos` returns `{ "sos": [...] }` and NOT `{ "sos_requests": [...] }`.
   This is exactly what the fix consumes.

2. **Real backend SOS records appear** — test 3b submits a new SOS as a
   citizen, then re-fetches `/api/sos` and verifies the new record is in
   the response. The fixed frontend would display these records in the
   dashboard table.

3. **Risk publishing does NOT auto-publish an emergency alert** —
   the "(Decoupling)" check verifies that after `POST /api/risk/publish`,
   no new emergency alert was created. The Command Centre operator must
   manually use the Emergency Alerts tab to broadcast an alert. This
   preserves human control over emergency broadcasting.

4. **No JS syntax errors introduced** — all inline `<script>` blocks in
   `command_centre1.html`, `command_centre2.html`, and `aboutus.html`
   were validated with `node --check` via the
   `scripts/check_html_js.js` helper.

---

## Part C — JS syntax validation

Command: `node /home/z/my-project/scripts/check_html_js.js <file>...`

```
=== Checking /home/z/my-project/work/SIH/command_centre1.html ===
  [block 1] PASS (88553 chars)
  [block 2] PASS (101 chars)

=== Checking /home/z/my-project/work/SIH/command_centre2.html ===
  [block 1] PASS (88553 chars)
  [block 2] PASS (101 chars)

=== Checking /home/z/my-project/work/SIH/aboutus.html ===
  [block 1] PASS (4651 chars)
```

All inline JavaScript in the modified files parses cleanly with `node --check`.
No syntax errors introduced.

---

## Part D — Git diff (only 3 files changed)

```
 aboutus.html         | 4 ++--
 command_centre1.html | 7 +++++--
 command_centre2.html | 7 +++++--
 3 files changed, 12 insertions(+), 6 deletions(-)
```

```
commit 6b6315d (fix/command-centre)
fix(command-centre): consume real backend SOS field (sos, not sos_requests)

Backend GET /api/sos returns { sos: [...] } (see backend/routes/sos.js:163).
Command Centre's loadDataFromBackend() was checking the non-existent
'sos_requests' key, so the initial page load silently fell back to
demo SOS data until a live NEW_SOS Socket.IO event arrived.

Fix: read sosRes.value.sos instead of sosRes.value.sos_requests in
both command_centre1.html and command_centre2.html. Add inline comment
documenting the contract.

Also corrects a name spelling in aboutus.html:
'Shaik Mozzamil' -> 'Shaik Muzzammil'.

Verified on Node 22:
- 53/53 backend Stage 4 integration tests pass (verify_stage4.js)
- 18/18 Command Centre smoke checks pass, including:
  * GET /api/sos returns { sos: [...] } (NOT sos_requests)
  * New SOS submissions appear in subsequent GET /api/sos calls
  * Risk publishing does NOT auto-publish emergency alerts
    (human operator retains control via the Emergency Alerts tab)
```

---

## Summary

| Check | Result |
| --- | --- |
| Backend integration tests (`verify_stage4.js`) | ✅ 53/53 PASS |
| Command Centre focused smoke test (18 points) | ✅ 18/18 PASS |
| JS syntax validation (`node --check`) | ✅ All inline scripts PASS |
| Risk/alert decoupling | ✅ Risk publish does NOT auto-publish alert |
| Files modified | ✅ Only `command_centre1.html`, `command_centre2.html`, `aboutus.html` |
| Backend files modified | ✅ Zero (no backend changes) |
| New dependencies added | ✅ Zero |
| Branch | ✅ `fix/command-centre` (not master) |

**SUCCESS CONDITION MET.**
