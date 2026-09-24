# PrepStack

Interview prep, sprint by sprint. A new visitor lands on a marketing page, answers a 3-step quiz (experience
level, time budget, how many questions — 75 to 700), and gets a curriculum sized to that: 3 to 12 sprints
pulled from a 666-question bank, always front-loaded with the full must-do interview core. Sprints are
self-paced — sprint 1 starts the day you finish onboarding, and each later sprint starts the day after you
finish the one before it, so falling behind shifts the rest of the plan instead of piling up overdue flags
on sprints you haven't reached yet. On top of that: a spaced-repetition revision sheet, a timed "contest" of
unseen questions unlocked after each sprint, and real company tags (see below) on questions that have
actually been asked at Google, Amazon, and 55+ others.

React + Vite, deployed as a static site. Progress is saved in browser localStorage by default (Profile →
Settings has Export/Import for manual backup) — no account needed. Optional free email login via Supabase
adds cross-device sync; see below.

System Design and Low-Level Design tracks are planned as future additions alongside the DSA track.

## Run locally
    npm install
    npm run dev

## Deploy to GitHub Pages
1. Create a GitHub repo and push this folder to the `main` branch.
2. Repo → Settings → Pages → Source: **GitHub Actions**.
3. Every push to `main` builds and deploys (`.github/workflows/deploy.yml`).
Site URL: `https://<username>.github.io/<repo>/`

## Edit the question list
Edit `scripts/questions.txt` (one question per line: `Title|E/M/H|flags|slug-override`), and the topic
grouping in `scripts/build-data.mjs`. `npm run build` (or `npm run data`) regenerates
`src/data/questions.json` — personalized plans (see above) are derived from this master list at runtime
(`src/curriculum.js`), so editing it changes every tier, not just the full one.
Flags: `*` must-do (revision sheet), `$` LeetCode premium, `+` extra from GfG/classics.
If a LeetCode link 404s, put the correct slug in the 4th column.

## Company tags
Questions show which real companies they've been asked at (e.g. "Google +43" on a question row), with a
dedicated Companies page per company. This comes from
[snehasishroy/leetcode-companywise-interview-questions](https://github.com/snehasishroy/leetcode-companywise-interview-questions),
a public scrape of LeetCode's own company-tag feature (snapshot: 12 Jul 2026) — not a fabricated list. Raw
`all.csv` files for ~60 well-known companies are checked into `scripts/companies-raw/<slug>/all.csv`;
`scripts/build-companies.mjs` matches them against our own question bank by LeetCode slug and writes
`src/data/companies.json` (this only tags existing questions — it never adds new ones or changes sprint
counts). Coverage varies a lot by company since most of that source only publishes a partial list per
company.

To refresh the data or add a company: pull that company's latest `all.csv` from the source repo into
`scripts/companies-raw/<slug>/all.csv`, add it to the `COMPANIES` map at the top of
`scripts/build-companies.mjs`, and run `npm run data`.

## Optional: email login + sync across devices with Supabase

By default PrepStack needs no backend and no account — everything lives in your browser's localStorage.
If you want to log in and pick up your progress on another device, you can wire up a free Supabase project.
This is entirely optional; skip this section and nothing changes.

1. Go to [supabase.com/dashboard](https://supabase.com/dashboard) and create a free account + new project
   (pick any name/region/password — you won't need the database password for this).
2. Once the project is ready: **Project Settings → API**. Copy the **Project URL** and the **anon public**
   key (not the `service_role` key — that one must never be shipped to a browser).
3. In this repo, copy `.env.example` to `.env.local`:
       cp .env.example .env.local
   and paste the two values in:
       VITE_SUPABASE_URL=https://xxxxxxxx.supabase.co
       VITE_SUPABASE_ANON_KEY=eyJ...
4. In the Supabase dashboard, open **SQL Editor → New query**, paste the contents of
   [`supabase/schema.sql`](supabase/schema.sql), and run it. This creates a `progress` table with Row Level
   Security so each account can only ever read or write its own row.
5. By default, Supabase requires email confirmation before a new account can sign in. For your own personal
   use you can turn this off (**Authentication → Providers → Email → Confirm email**) so sign-up logs you in
   immediately; otherwise check your inbox after signing up.
6. `npm install` (pulls in `@supabase/supabase-js`) and restart `npm run dev`. Profile now shows an
   "Account & sync" card with email sign-up/log-in.

### Deploying with sync enabled
Add `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` as **repository secrets** (Repo → Settings → Secrets
and variables → Actions), then reference them as `env:` in the build step of `.github/workflows/deploy.yml`.
The anon key is safe to expose in a public build — it can only do what the RLS policies above allow
(read/write the signed-in user's own row).

Without the two env vars set, the app silently skips all of the above and behaves exactly like the
localStorage-only version — nothing breaks if you never do this.

### Google/GitHub login (not currently in the UI)
The Profile page only shows email sign-up/log-in right now. The Supabase client is already configured for
it (PKCE flow in `src/supabase.js`, `auth.signInWithOAuth` in `src/store.jsx`), so adding provider buttons
back later is just UI work, not a rework — it needs a real one-time setup in Google Cloud Console and/or
GitHub's OAuth Apps settings plus enabling the provider in Supabase, which is why it's parked for now.

## Adding a different backend later
All local persistence is in `src/store.jsx` (`storage.load/save`); the optional Supabase sync sits alongside
it (`loadCloud`/`saveCloud` in the same file, `src/supabase.js` for the client). Swap either layer for API
calls to your own server (Node/Express or Spring Boot) if you outgrow Supabase.
