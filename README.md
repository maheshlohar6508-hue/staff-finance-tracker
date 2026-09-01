# Vahan Track — Vehicle HP Loan Collections App

Real backend: Node.js + Express + SQLite database, bcrypt password hashing,
JWT login sessions, photo uploads, and Excel/CSV bulk customer import.

## What's inside
- `server.js` — API server (auth, customers, calls/visits, staff admin, Excel import)
- `db.js` — SQLite schema + seeds one admin account on first run
- `public/index.html` — the app itself (mobile + desktop, single page)
- `customer-import-template.csv` — sample file to test bulk import
- `.env.example` — copy to `.env` and fill in

## 1. Run it on your own computer first (optional, to test)
```bash
cd vahan-track-server
npm install
cp .env.example .env
# open .env and set JWT_SECRET to a long random string
npm start
```
Open `http://localhost:3000`. Log in with **admin / admin123**, then
**immediately go to the Staff tab → Reset password** for the admin account.

## 2. Put it live on the internet (Railway — free tier, ~10 minutes)

SQLite needs a persistent disk, so use a host that offers one. Railway's free
trial supports this easily.

1. Go to https://railway.app and sign up (GitHub login is easiest).
2. Click **New Project → Deploy from GitHub repo** (push this folder to a new
   GitHub repo first — or use **Empty Project → Empty Service** and drag-drop
   this folder / use the Railway CLI `railway up` from inside the folder).
3. In the service **Settings → Variables**, add:
   - `JWT_SECRET` = a long random string (generate one: `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`)
4. In **Settings → Volumes**, add a volume and mount it at `/app/data` — then
   also add a variable `DB_PATH=/app/data/data.sqlite` and change the uploads
   folder similarly if you want photos to survive redeploys (see note below).
5. Railway auto-detects Node and runs `npm install && npm start`.
6. Once deployed, Railway gives you a public URL like
   `https://vahan-track-production.up.railway.app` — that's your live app,
   usable on any phone or computer browser.
7. Log in with `admin / admin123` and change the password right away.

**Alternative hosts:** Render.com (needs a paid persistent disk add-on for
SQLite to survive restarts), or Fly.io (has free persistent volumes, slightly
more setup via `fly.toml`). If you'd rather not manage a server at all, tell
me and I can adapt this to use a managed database (e.g. Supabase Postgres)
which stays reliable regardless of host.

## 3. Add your 3,500 customers via Excel
- Go to **Add → Excel / CSV madhun bulk add kara**.
- Your file's column headers should be (any order): `Name, Bike Model, Phone,
  Address, Loan Amount, Total Paid, Total Due, EMI Amount, Next Due Date`
  (date format `YYYY-MM-DD` works best).
- A ready sample is in `customer-import-template.csv` — open it, replace the
  rows with your real data, save, and upload.
- You can upload in batches (e.g. 500 at a time) if your sheet is large.

## Security built in
- Passwords are hashed with bcrypt — never stored in plain text.
- Login sessions use signed JWT tokens (12-hour expiry).
- Login endpoint is rate-limited against brute-force guessing.
- Only admins can add/remove staff or reset passwords.
- Security headers via Helmet; all database queries are parameterized
  (no SQL injection).

## What you should still do
- Change the default admin password immediately after first login.
- Set a strong, unique `JWT_SECRET` (never reuse the example value).
- If staff and admin are on different domains, restrict CORS in `server.js`
  instead of allowing all origins.
- Take regular backups of `data.sqlite` and the `uploads/` folder.
