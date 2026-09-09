# DisasterGuard — Two-Domain Free Deployment Guide
## Frontend on Vercel + Backend on Render (separate domains)

> **Goal:** Deploy the entire DisasterGuard platform for $0 using two free services:
> - **Vercel** hosts the **frontend** (all 13 HTML files + `api.js`) at one URL (e.g., `https://disasterguard.vercel.app`)
> - **Render** hosts the **backend** (Node + Express + SQLite + Socket.IO) at a separate URL (e.g., `https://disasterguard-backend.onrender.com`)
>
> Total cost: **$0**. No credit card required.
> Total time: ~30 minutes.
> Result: A live, end-to-end DisasterGuard platform you can demo to anyone.

---

## Why two separate domains?

The DisasterGuard backend uses **`better-sqlite3`** — a native SQLite library that needs a persistent disk file. Vercel's serverless functions are stateless and ephemeral (no disk), so the backend **cannot run on Vercel**. The frontend (HTML+JS+CSS) is just static files, so Vercel is perfect for it.

This means:
- Vercel serves your login page, citizen portal, rescue portal, command centre, etc.
- Render serves your API routes and Socket.IO real-time events.
- The frontend talks to the backend cross-domain using fetch + WebSockets.

This is a standard, production-safe pattern. The backend already uses `app.use(cors())` (allowing all origins) and Socket.IO is configured with `cors: { origin: '*' }`, so cross-domain communication works out of the box.

---

## Prerequisites (5 minutes)

You need **3 free accounts**:

| Service | Sign up URL | What it's for |
| --- | --- | --- |
| GitHub | <https://github.com/signup> | Host your code + auto-deploy trigger |
| Vercel | <https://vercel.com/signup> | Host the frontend (sign in with GitHub) |
| Render | <https://render.com/signup> | Host the backend (sign in with GitHub) |

Optional: install Node.js 20 or 22 LTS locally for testing first
(<https://nodejs.org/>). Not strictly required if you trust the deploy pipeline.

---

## Step 1 — Push your code to GitHub (5 minutes)

1. Unzip `DisasterGuard-fix-command-centre.zip` to a folder on your computer, e.g., `~/disasterguard`.
2. Inside that folder you should see:

   ```
   SIH/
   ├── README.md
   ├── CHANGELOG.md
   ├── TEST_RESULTS.md
   ├── DEPLOYMENT_GUIDE.md            ← this file
   ├── git-diff.patch
   ├── .gitignore
   ├── api.js
   ├── login.html
   ├── signup.html
   ├── main-1.html
   ├── features.html
   ├── howitworks.html
   ├── aboutus.html                   ← name spelling corrected
   ├── contactus.html
   ├── citizen1.html
   ├── citizen2.html
   ├── rescue.html
   ├── command_centre1.html           ← SOS fix applied
   ├── command_centre2.html            ← SOS fix applied
   └── backend/
       ├── server.js
       ├── package.json
       ├── database/
       ├── middleware/
       ├── routes/
       ├── utils/
       └── scripts/
   ```

3. Create a new GitHub repository:
   - Go to <https://github.com/new>
   - Repository name: `disasterguard-sih`
   - Set to **Public** (Vercel and Render need to read it; private requires paid plans)
   - **Do not** initialize with README/license (you already have files)
   - Click **Create repository**

4. From your terminal, inside the `SIH/` folder:

   ```bash
   cd SIH
   git init
   git add .
   git commit -m "DisasterGuard SIH 2026 — initial commit (with Command Centre SOS fix)"
   git branch -M master
   git remote add origin https://github.com/<YOUR_GITHUB_USERNAME>/disasterguard-sih.git
   git push -u origin master
   ```

   Replace `<YOUR_GITHUB_USERNAME>` with your actual GitHub username.

5. Refresh your GitHub repo page — you should see all the HTML files + `backend/` folder.

---

## Step 2 — Deploy the BACKEND on Render (15 minutes)

### 2.1 Create the Web Service

1. Go to <https://dashboard.render.com/> and click **New +** → **Web Service**.
2. Click **Build and connect a new repository** → authorize Render to access your GitHub → select your `disasterguard-sih` repo.
3. Fill the form:

   | Field | Value |
   | --- | --- |
   | **Name** | `disasterguard-backend` (this becomes your backend subdomain) |
   | **Project** | (leave default or create one called `DisasterGuard`) |
   | **Language** | Node |
   | **Branch** | `master` |
   | **Region** | Choose the closest to you |
   | **Root Directory** | `backend` ← **IMPORTANT** |
   | **Build Command** | `npm install` |
   | **Start Command** | `npm start` |
   | **Instance Type** | Free |

4. Click **Advanced** to expand the advanced settings. You'll add environment variables and a persistent disk here.

### 2.2 Add Environment Variables

Scroll to **Environment Variables** and add the following:

| Key | Value | Notes |
| --- | --- | --- |
| `NODE_VERSION` | `22.11.0` | Pins Node version (avoids the Node 24 native crash). |
| `PORT` | `5000` | Backend listens here; Render maps it to port 443 automatically. |
| `DB_PATH` | `data/disasterguard.db` | Relative to `backend/`. Will live on the persistent disk. |
| `JWT_SECRET` | (generate with `openssl rand -hex 32` locally, paste the output) | Used to sign JWT tokens. **Keep this secret.** |
| `CORS_ORIGIN` | `https://disasterguard.vercel.app` | Your future Vercel URL. (You can edit this later if you pick a different Vercel name.) |

### 2.3 Add a Persistent Disk (CRITICAL — this is what makes SQLite work)

Scroll to **Disks** and click **Add Disk**:

| Field | Value |
| --- | --- |
| **Name** | `disasterguard-data` |
| **Mount Path** | `/opt/render/project/src/data` ← exactly this path |
| **Size** | 1 GB (free tier max) |

This is where SQLite will store `disasterguard.db`. Without this disk, the database
would be wiped on every deploy and the seed data would be lost on every restart.

### 2.4 Deploy

1. Click **Create Web Service** at the bottom.
2. Render will:
   - Clone your repo
   - `cd backend`
   - Run `npm install` (this will rebuild `better-sqlite3` against Render's Linux — takes 2–3 min)
   - Run `npm start` (boots Express on port 5000)
3. Watch the **Logs** tab. You should see:
   ```
   🛡️  DisasterGuard Backend Server is Running!
   📍 Port: http://localhost:5000
   📁 Database: SQLite at data/disasterguard.db
   ⚡ Real-Time: Socket.IO initialized
   [Seed] Seeding canonical DisasterGuard accounts, shelters, and resources...
   [Seed] Seeding completed successfully.
   ```
4. Once deployed, Render gives you a URL like:
   **`https://disasterguard-backend.onrender.com`**
   Write this URL down — you'll need it in Step 3.

### 2.5 Verify the backend is live

Open these URLs in your browser:

- <https://disasterguard-backend.onrender.com/api/health>
  → should return JSON with `"status":"healthy"` and `"seededUsers":8`
- <https://disasterguard-backend.onrender.com/api/sos>
  → should return JSON with `"sos":[...]` (empty array is OK — this is the field the frontend fix reads!)

If both work, your backend is live. 🎉

If `seededUsers` is `0`, the seed didn't run. Open the Render **Shell** tab and run:
```bash
cd backend
node database/seed.js
```
Then restart the service.

### 2.6 Render free tier — what to expect

- **First request after 15 min of inactivity takes ~30 seconds to wake up.** For a demo, just say "give it 30 seconds to warm up."
- The 1 GB disk is plenty — the seed + a year of SOS records will use <50 MB.
- Render auto-redeploys whenever you push to `master`.

---

## Step 3 — Edit `api.js` to point at your Render backend (2 minutes)

This is the **only** manual code change required for production.

1. In your local project (or directly on GitHub), open `api.js`.
2. Find **line 12**:

   ```js
   const API_BASE = 'http://localhost:5000';
   ```

3. Replace it with:

   ```js
   // Auto-detect: localhost for development, your Render URL for production.
   const API_BASE =
     (typeof window !== 'undefined' && window.location.hostname === 'localhost')
       ? 'http://localhost:5000'
       : 'https://disasterguard-backend.onrender.com';  // ← YOUR RENDER URL HERE
   ```

   Replace `disasterguard-backend.onrender.com` with your actual Render URL from Step 2.4.

4. Commit and push:

   ```bash
   git add api.js
   git commit -m "deploy: point api.js at Render backend URL for production"
   git push origin master
   ```

This keeps local dev working (when you run `npm start` on your laptop, the frontend still talks to `http://localhost:5000`) while production uses your Render URL.

---

## Step 4 — Deploy the FRONTEND on Vercel (5 minutes)

### 4.1 Create the Vercel project

1. Go to <https://vercel.com/> and click **Add New…** → **Project**.
2. Import your `disasterguard-sih` GitHub repository.
3. Configure:

   | Field | Value |
   | --- | --- |
   | **Framework Preset** | Other |
   | **Root Directory** | `./` (the repo root — this is where your HTML files are) |
   | **Build Command** | (leave empty — pure static site) |
   | **Output Directory** | `./` |
   | **Install Command** | (leave empty) |

4. Click **Deploy**.

### 4.2 Wait for deployment

Vercel will:
- Detect 13 HTML files + `api.js` at the repo root
- Serve them at `https://disasterguard-<your-vercel-name>.vercel.app`
- Auto-redeploy on every push to `master`

### 4.3 Verify the frontend is live

Visit your Vercel URL:

- `https://disasterguard-<your-name>.vercel.app/login.html`
- Log in as **Command Centre**:
  - Role: `command`
  - ID: `CC-2026-0001`
  - Password: `CC-2026-0001`
- You should land on the Command Centre dashboard with **NO console errors** (F12 → Console).

### 4.4 Verify the SOS fix is live

1. On the deployed Command Centre page, open DevTools (F12).
2. Go to **Network** tab.
3. Refresh the page.
4. Find the request to `https://disasterguard-backend.onrender.com/api/sos`.
5. Response body should be:
   ```json
   { "sos": [ ... ] }
   ```
   (NOT `sos_requests` — this is the bug fix in action!)
6. The SOS table on the dashboard should populate from real backend data.

### 4.5 Verify real-time Socket.IO works

1. Open **TWO browser windows** side by side.
2. Window A: Vercel frontend → log in as `citizen01@disasterguard.com` / `Citi@2026One`.
3. Window B: Vercel frontend → log in as `CC-2026-0001` / `CC-2026-0001` (Command Centre).
4. In Window A, submit a test SOS.
5. In Window B, the SOS table should update within ~1 second, **without refreshing the page** — that's Socket.IO delivering the `NEW_SOS` event across the two domains.

If this works, you have a fully live, real-time DisasterGuard platform. 🚀

---

## Step 5 — Quick checklist (verify everything works)

After both deploys, run through this checklist:

| # | Check | How to verify | Pass? |
| --- | --- | --- | --- |
| 1 | Backend health endpoint works | Visit `https://disasterguard-backend.onrender.com/api/health` — returns `"status":"healthy"`, `"seededUsers":8` | ☐ |
| 2 | Backend SOS endpoint returns the right field | Visit `https://disasterguard-backend.onrender.com/api/sos` — returns `{ "sos": [...] }` (NOT `sos_requests`) | ☐ |
| 3 | Frontend loads on Vercel | Visit `https://disasterguard-<name>.vercel.app/login.html` — login page renders | ☐ |
| 4 | Command Centre login works | Log in as `CC-2026-0001` / `CC-2026-0001` | ☐ |
| 5 | Real SOS data appears | Command Centre dashboard SOS table is not stuck on demo data | ☐ |
| 6 | No browser console errors | F12 → Console → no red errors | ☐ |
| 7 | New SOS appears in real time | Submit SOS as citizen → see it appear in Command Centre within 1 sec | ☐ |
| 8 | Mission assignment works | From Command Centre, assign a rescue team to an SOS | ☐ |
| 9 | Rescue portal receives mission | Log in as `RT-2026-0001` / `RT@2026-0001` → mission appears | ☐ |
| 10 | Risk publish does NOT auto-create alert | Run + publish a risk assessment → check Emergency Alerts tab → no new alert appears automatically | ☐ |

If all 10 pass, you're fully deployed. 🎉

---

## Architecture diagram (what you've built)

```
        ┌─────────────────────────────────────────────────────────┐
        │                    USER'S BROWSER                       │
        │  https://disasterguard.vercel.app                      │
        │                                                         │
        │   login.html  citizen1.html  rescue.html                │
        │   command_centre1.html  (etc.)                          │
        │                  ↕ loads                                │
        │                api.js                                   │
        │                  ↕ fetch() + io()  (cross-origin)       │
        └──────────────────┬──────────────────────────────────────┘
                           │
                           │ HTTPS + WebSocket
                           │
        ┌──────────────────▼──────────────────────────────────────┐
        │              RENDER (backend, separate domain)         │
        │  https://disasterguard-backend.onrender.com             │
        │                                                         │
        │   Express + Socket.IO + JWT                             │
        │   /api/auth  /api/sos  /api/missions  /api/shelters     │
        │   /api/incidents  /api/resources  /api/risk            │
        │   /api/alerts  ...                                      │
        │                  ↕ reads/writes                         │
        │         SQLite (disasterguard.db)                       │
        │         on a 1 GB persistent disk                       │
        └─────────────────────────────────────────────────────────┘
```

---

## What if I want to update the code later?

When you push a new commit to `master` on GitHub:
- **Render** auto-redeploys the backend (takes ~2 min)
- **Vercel** auto-redeploys the frontend (takes ~30 sec)

You don't need to manually redeploy anything.

---

## What about the fix branch (`fix/command-centre`)?

The fix is already merged into `master` in the ZIP I gave you. If you want to keep
the fix isolated on its own branch for code review:

```bash
git checkout -b fix/command-centre   # already done in the ZIP
git push -u origin fix/command-centre
```

Then on GitHub, open a Pull Request from `fix/command-centre` → `master`, review the
diff (only 3 files changed), and click **Merge** when satisfied.

---

## Troubleshooting

### "Failed to fetch" or CORS errors in the browser console

1. Verify `api.js` line 12 points at your Render URL (not `localhost`).
2. Verify the Render backend is awake — visit `/api/health` directly. First request after sleep takes ~30 sec.
3. Verify `CORS_ORIGIN` env var on Render allows your Vercel URL. (The default `app.use(cors())` allows all origins, so this is rarely an issue.)

### Socket.IO not updating in real time

1. Open browser DevTools → Console. You should see `[Socket.IO] Connected to backend with ID ...`. If you don't, the backend URL in `api.js` is wrong.
2. Some corporate firewalls block WebSockets. Try a different network (mobile hotspot).
3. Socket.IO automatically falls back to HTTP polling if WebSockets are blocked — slower but functional.

### SOS table is empty even after the fix

1. Verify `/api/sos` returns data by visiting the URL directly in your browser.
2. If it returns `{ "sos": [] }`, the DB is empty — submit a test SOS from the citizen portal first.
3. If it returns an error, check the Render logs.

### Login fails with HTTP 401

1. The seeded accounts weren't created. Open the Render **Shell** tab and run:
   ```bash
   cd backend
   node database/seed.js
   ```
2. Restart the service from the Render dashboard.

### Render backend takes 30 seconds to wake up

That's the free tier — it sleeps after 15 min of inactivity. For a hackathon demo,
just say "give it 30 seconds". For always-on, upgrade to Render Starter ($7/mo)
or use Fly.io's free tier (no sleep).

---

## Cost summary

| Service | Free tier used | Cost |
| --- | --- | --- |
| GitHub | Public repo, unlimited | $0 |
| Vercel | Hobby plan, 100 GB bandwidth | $0 |
| Render | Free Web Service + 1 GB disk | $0 |
| **Total** | | **$0 / month** |

No API keys needed. No external AI APIs. No new databases. The risk engine is a
deterministic weighted-score algorithm (not an LLM). Map tiles are OpenStreetMap
(no key). Notifications are in-app + Socket.IO (no SMS/email).

---

## You're done! 🎉

You now have a live, two-domain, fully-free DisasterGuard platform:
- **Frontend:** `https://disasterguard-<your-name>.vercel.app`
- **Backend:** `https://disasterguard-backend.onrender.com`

Share the Vercel URL with your SIH 2026 judges / teammates and log in with:
- **Command:** `CC-2026-0001` / `CC-2026-0001`
- **Rescue:** `RT-2026-0001` / `RT@2026-0001`
- **Citizen:** `citizen01@disasterguard.com` / `Citi@2026One`

---

**End of guide.** Questions? See `README.md` and `CHANGELOG.md` in the same ZIP.
