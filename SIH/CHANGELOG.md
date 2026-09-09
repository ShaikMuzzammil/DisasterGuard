# Changelog — DisasterGuard Command Centre Fix

## Branch: `fix/command-centre`
## Date: September 10, 2026

## Primary fix (P1 from audit)

### `command_centre1.html` and `command_centre2.html` — line 918

**Before:**
```js
if (sosRes.status === 'fulfilled' && Array.isArray(sosRes.value?.sos_requests)) {
  sos = sosRes.value.sos_requests.map(mapBackendSOS);
}
```

**After:**
```js
// Backend GET /api/sos returns { sos: [ ... ] } (see backend/routes/sos.js).
// Use the real `sos` field instead of the non-existent `sos_requests` key,
// otherwise the Command Centre silently falls back to demo data on initial load.
if (sosRes.status === 'fulfilled' && Array.isArray(sosRes.value?.sos)) {
  sos = sosRes.value.sos.map(mapBackendSOS);
}
```

**Why:** The backend route `GET /api/sos` (defined in `backend/routes/sos.js`
line 163) returns:
```json
{ "sos": [ { ...sosRecord, persons: [...] } ] }
```
The frontend was reading `sosRes.value?.sos_requests`, which is `undefined`
because that key does not exist in the response. As a result, the
`Array.isArray(...)` check returned `false`, the `sos` array was never
populated from the backend, and the page silently fell back to the demo
SOS data baked into the HTML until a live `NEW_SOS` Socket.IO event arrived.

**Impact:** After the fix, the Command Centre displays real backend SOS
records on initial page load, including any created since the last visit.

## Secondary fix (requested by user)

### `aboutus.html` — lines 265, 267

**Before:**
```html
<img ... alt="Shaik Mozzamil" />
<h3 ...>Shaik Mozzamil</h3>
```

**After:**
```html
<img ... alt="Shaik Muzzammil" />
<h3 ...>Shaik Muzzammil</h3>
```

**Why:** Name spelling correction requested by the project team member.

## Verification performed

### 1. JS syntax validation (node --check on inline scripts)
- `command_centre1.html` — block 1 (88553 chars) PASS, block 2 (101 chars) PASS
- `command_centre2.html` — block 1 (88553 chars) PASS, block 2 (101 chars) PASS
- `aboutus.html` — block 1 (4651 chars) PASS

### 2. Backend integration tests (Node 22, 53/53 PASS)
```
Summary: 53/53 Stage 4 tests passed.
🎉 ALL STAGE 4 END-TO-END INTEGRATION TESTS PASSED!
```

Critical assertions from `verify_stage4.js`:
- "STRICT DECOUPLING: Publishing risk did NOT automatically create any emergency broadcast alert" — PASS
- "Mission completion blocked when persons remain unaccounted (400 Bad Request)" — PASS
- "Warehouse stock atomic decrement verified" — PASS
- "Double mission assignment rejected with 409 Conflict" — PASS

### 3. Command Centre focused smoke test (18/18 PASS)

See `TEST_RESULTS.md` for the full output. Highlights:
- `GET /api/sos` returns `{ "sos": [...] }` (NOT `sos_requests`) — PASS
- Newly-submitted SOS appears in subsequent `GET /api/sos` calls — PASS
- Risk publish returns `riskId` (status 201) — PASS
- Emergency alert publish returns `alertId` (status 201) — PASS
- Emergency alert deactivate sets `is_active = 0` (status 200) — PASS
- Risk publish did NOT auto-publish an alert (human operator retains control) — PASS

## Files NOT modified (per project rules)

- `backend/**` — unchanged
- `citizen1.html` — unchanged
- `citizen2.html` — unchanged
- `rescue.html` — unchanged
- `login.html` — unchanged
- `signup.html` — unchanged
- `main-1.html`, `features.html`, `howitworks.html`, `contactus.html` — unchanged
- `api.js` — unchanged (the deployment guide mentions an optional `API_BASE`
  tweak for production, but that is a deployment-time concern, not a code change
  in this PR)
- `package.json` and `package-lock.json` — unchanged

## Git diff stat
```
 aboutus.html         | 4 ++--
 command_centre1.html | 7 +++++--
 command_centre2.html | 7 +++++--
 3 files changed, 12 insertions(+), 6 deletions(-)
```

## Known lower-priority issue (NOT fixed — out of scope)

**`removeAssignedTeam()` (line 1323 in both `command_centre1.html` and
`command_centre2.html`):** Currently modifies only frontend in-memory state.
There is no backend route to persist team unassignment. This is the
audit's P2 issue.

The fix would require adding a new backend route (e.g.,
`DELETE /api/missions/:id` or `POST /api/sos/:id/unassign`) that:
1. Sets the SOS `status` back to `PENDING`
2. Sets `assigned_team_id` to `NULL`
3. Sets the team's `status` back to `AVAILABLE`
4. Emits `SOS_UPDATED` and `TEAM_UPDATED` Socket.IO events

This is a backend change and is explicitly out of scope for this
Command Centre frontend fix. Reported for follow-up.

## Things deliberately NOT done

- ❌ Did not change the backend to return `sos_requests` instead of `sos`.
  The audit explicitly says: "Do not change the backend just to accommodate the frontend."
- ❌ Did not introduce Gemini API or any external AI API.
- ❌ Did not add a new database, framework, or architecture.
- ❌ Did not add new dependencies to `package.json`.
- ❌ Did not redesign the UI.
- ❌ Did not push to `master`. The fix lives on `fix/command-centre` only.
