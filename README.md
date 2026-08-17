# Baliston Health — Content & Marketing Planner (Supabase build)

Two people, one live planner. Edits appear on the other person's screen within a second
or two, without anyone passing files around.

Still a single HTML file, still no build step. The only addition is a Supabase project,
which is free at this scale.

---

## What you need

A Supabase account (free tier). Nothing else — no server, no Node, no deploy pipeline.

The free tier covers this comfortably: the whole planner is under 100 rows and a few
hundred KB. You are nowhere near any limit.

**One caveat worth knowing now:** free Supabase projects pause after about a week of
inactivity. You'd both need to be away from the planner for a full week for that to
happen, and it takes one click in the dashboard to wake it. If that bothers you, the paid
tier removes it.

---

## Setup — about 5 minutes, once

### 1. Create the project

[supabase.com](https://supabase.com) → New project. Pick the region closest to you
(Frankfurt or Paris for France).

### 2. Set the two people who are allowed in

Open `index.html` and find this at the top of the script:

```js
const SUPABASE_URL = "";
const SUPABASE_ANON_KEY = "";
const ALLOWED_EMAILS = ["", ""];
```

Fill in all three. URL and anon key come from Dashboard → **Settings → API**.
`ALLOWED_EMAILS` is your address and your assistant's — these go into the security policy,
so anyone else who signs in gets an empty planner and can write nothing.

> **Use the anon public key, never `service_role`.** The anon key is designed for public
> client code and is safe to commit. The service_role key bypasses all security — if it
> reaches a browser, anyone viewing source controls your database.

### 3. Let the planner create its own table

Open the planner and sign in. It checks whether the table exists, and if it doesn't, shows
you a screen with the exact SQL, a **Copy** button and a link straight to your project's
SQL Editor. Paste, press Run, click "check again". Done — you'll never see that screen
again.

**Why isn't this automatic?** Creating tables requires the service_role key, and putting
that in a public page would hand over your whole database. This is the one manual step,
and it exists precisely so the rest is safe.

### 4. Turn on email sign-in

Dashboard → **Authentication → Providers → Email**. Enable it.

Under **Authentication → URL Configuration**, add your planner's URL to **Redirect URLs**.
Sign-in links won't work without this.

### 5. Publish

Upload `index.html` and `.nojekyll` to a repo, then Settings → Pages → deploy from `main`
/ root.

---

## What your assistant has to know

Nothing about GitHub, Supabase, or any of the above.

She opens the link, types her email, clicks the link that arrives, and starts working.
That's the whole thing. No account to create, no password, no install.

---

## How the syncing behaves

**Live, not instant-by-keystroke.** Changes are written about half a second after you
stop typing, then pushed to the other person. In practice it feels immediate.

**Row-level, so you don't overwrite each other.** Each task, content piece and KOL is its
own database row. You editing Monday's carousel and Narindra editing Thursday's KOL
research touch different rows and never collide. Only rows that actually changed get
written.

**The exception:** if you both edit *the same field of the same task* within a second of
each other, the later write wins and the earlier one is lost silently. With two people
this is rare, but it isn't impossible, so don't treat it as impossible.

**Presence.** The circles in the header show who else is in the planner right now. Yours
is the filled one.

**Which week you're looking at is personal.** You can be reviewing last week while
Narindra works on this week. That's deliberate — only the data is shared, not the view.

**If the connection drops**, the chip reads "Offline" and your work is held in the
browser. It syncs when you're back. Don't close the tab while it says that.

---

## Falling back

The Supabase build still runs without a database. Choose "Skip — use this browser only"
on the setup screen and it behaves like the standalone version: saves locally,
export/import to move work between people. Useful if the project is paused or you're
somewhere without a connection.

Export and Import still work in cloud mode too. **Import overwrites the shared database
for both of you** — it's a restore, not a merge.

---

## Backups

Cloud storage is not a backup. Export `data.json` from Review & archive at the end of each
month and commit it to the repo. It costs ten seconds and covers you against a bad import,
a mistaken bulk delete, or a project you stop paying for.

---

## If something doesn't work

**Sign-in link goes nowhere.** The planner's URL isn't in Redirect URLs (step 4). Add it.

**Signed in but the planner is empty.** Your email isn't in the allowlist policy. Check
step 3 for typos — the addresses must match exactly.

**"Could not load the Supabase library."** The page can't reach jsdelivr or esm.sh.
Usually a corporate network or an extension blocking CDNs.

**Edits don't appear for the other person.** Realtime wasn't enabled. Re-run the
`alter publication` line from step 2.

**Everything worked, now nothing loads.** Free project paused after a week idle. Open the
dashboard and resume it.

---

## Where the code is

| Look for | What it does |
|---|---|
| `SUPABASE_URL` / `SUPABASE_ANON_KEY` | Connection settings |
| `collectRows()` | Turns the planner into database rows |
| `applyRow()` | Turns a database row back into planner state |
| `flush()` | Works out what changed and writes only that |
| `subscribe()` | Live updates and presence |
| `const WS` | The six workstreams and their targets |
| `function seed()` | What a brand-new empty database gets filled with |
