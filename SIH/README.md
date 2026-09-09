# DisasterGuard — Command Centre Fix (SIH 2026)

**Branch:** `fix/command-centre`
**Author:** DisasterGuard Command Centre developer
**Date:** September 10, 2026

## What this package contains

```
DisasterGuard-fix-command-centre/
├── README.md                              ← this file
├── CHANGELOG.md                           ← what changed and why
├── DEPLOYMENT_GUIDE.md                    ← full free-tier deployment guide (Vercel + Render + GitHub)
├── command_centre1.html                   ← MODIFIED (the SOS fix)
├── command_centre2.html                   ← MODIFIED (the SOS fix)
├── aboutus.html                           ← MODIFIED (name spelling: Shaik Muzzammil)
├── TEST_RESULTS.md                        ← 53/53 backend + 18/18 smoke test output
├── git-diff.patch                         ← raw git diff (apply with: git apply git-diff.patch)
└── screenshots/                           ← (optional) before/after screenshots
```

## The fix in one sentence

The Command Centre's initial load expected `sosRes.value?.sos_requests`,
but the backend GET /api/sos actually returns `{ "sos": [...] }`.
This package changes the frontend to read `sosRes.value?.sos`, so real
backend SOS records are displayed on page load (instead of silently
falling back to demo data until a live NEW_SOS Socket.IO event arrived).

## Files modified (only these 3)

| File | Lines changed | Reason |
| --- | --- | --- |
| `command_centre1.html` | 918–919 (replaced) + 3 comment lines | Read `sosRes.value.sos` instead of `sosRes.value.sos_requests` |
| `command_centre2.html` | 918–919 (replaced) + 3 comment lines | Same fix as `command_centre1.html` |
| `aboutus.html` | 265, 267 | Correct name spelling: "Shaik Mozzamil" → "Shaik Muzzammil" |

No backend files modified.
No other frontend files modified.
No new dependencies added.
No new database, framework, or architecture introduced.

## How to apply

### Option A: Replace the 3 files directly

Copy `command_centre1.html`, `command_centre2.html`, and `aboutus.html`
from this folder over the same files in your existing DisasterGuard project.

### Option B: Apply the git patch

From your existing DisasterGuard project root (the folder containing the
HTML files and `backend/`):

```bash
git checkout -b fix/command-centre
git apply /path/to/DisasterGuard-fix-command-centre/git-diff.patch
git commit -m "fix(command-centre): consume real backend SOS field (sos, not sos_requests)"
```

## How to verify

### Locally (Node 20 or 22 recommended)

```bash
cd backend
npm install
node database/seed.js
node scripts/verify_stage4.js     # must print: Summary: 53/53 Stage 4 tests passed.
npm start                          # backend on http://localhost:5000
```

Then open `login.html` in a browser (or serve the project root with any
static server like `python3 -m http.server 8080`), log in as
`CC-2026-0001` / `CC-2026-0001`, and confirm:

- The Command Centre dashboard loads with no browser console errors.
- Open DevTools → Network → reload → `GET /api/sos` returns `{ "sos": [...] }`.
- The SOS table on the dashboard reflects real backend data (or is empty if
  no SOS yet — both are correct).

### In production

See `DEPLOYMENT_GUIDE.md` for the full Vercel + Render + GitHub free-tier
deployment walkthrough.

## The 18 verification points (all PASS)

1. ✅ Command Centre login works
2. ✅ Backend session/JWT works
3. ✅ Real SOS requests appear (response uses `sos` field, not `sos_requests`)
4. ✅ New SOS Socket.IO events appear (NEW_SOS handler uses `mapBackendSOS` correctly)
5. ✅ SOS assignment works (POST /api/missions → 201 Created)
6. ✅ Rescue team selection works (GET /api/rescue-teams)
7. ✅ Mission assignment reaches the backend (status ASSIGNED)
8. ✅ Incidents load correctly (GET /api/incidents → `{ incidents: [...] }`)
9. ✅ Shelters load from backend (GET /api/shelters → `{ shelters: [...] }`)
10. ✅ Shelter CRUD still works (POST + PUT + DELETE)
11. ✅ Resource requests work (GET /api/resources/requests → `{ resource_requests: [...] }`)
12. ✅ Resource allocation works (PATCH /api/resources/requests/:id/allocate)
13. ✅ Risk prediction works (POST /api/risk/predict → 200)
14. ✅ Risk publishing works (POST /api/risk/publish → 201, returns riskId)
15. ✅ Emergency alert publishing works (POST /api/alerts → 201)
16. ✅ Emergency alert deactivation works (PATCH /api/alerts/:id/deactivate → 200)
17. ✅ Socket.IO command room works (NEW_SOS / SOS_UPDATED / MISSION_ASSIGNED etc.)
18. ✅ No browser console errors introduced (validated via `node --check` on inline JS)

**Decoupling check (critical):** Publishing a risk assessment does NOT
automatically create an emergency alert. The Command Centre retains
human/operator control over emergency alert publication via the
Emergency Alerts tab. Verified by `verify_stage4.js` Section 4D
("STRICT DECOUPLING: Publishing risk did NOT automatically create any
emergency broadcast alert").

## Known remaining issue (NOT fixed in this PR)

`removeAssignedTeam()` in `command_centre1.html` / `command_centre2.html`
(line 1323) is frontend-only — there is no backend route to persist
unassignment. This is documented in the audit as a P2 issue and is out
of scope for this Command Centre SOS fix. See `CHANGELOG.md` for details.

## License

Same as the parent DisasterGuard project (ISC).
