# BenchSaurus

A free, single-file lab management app for ordering supplies, tracking inventory, running a shared events calendar with a journal-club paper library, keeping up with recurring lab jobs, and posting lab-wide notices — with a lighthearted dinosaur theme and over 50 hidden easter eggs.

No build step, no server to maintain. It's one HTML file that talks to a free Supabase database, and you can host it anywhere that serves static files.

![Dashboard screenshot](docs/screenshot-dashboard.png)

## Features

- **Orders** — request supplies, track status from request to received, flag urgent items
- **Inventory** — supplies, freezers, expiry dates, low-stock alerts, reorder in one click
- **Reports** — spending by week, month, vendor, or person
- **Events** — a shared calendar for seminars, talks, and journal club, with a **Literature Library** that automatically collects every journal club paper link in one place for later reading
- **Lab Jobs** — recurring chores (autoclave, waste pickup, freezer checks) with who's responsible and when they're due
- **Your day** — a personal dashboard panel showing your assigned jobs, your upcoming talks, your open requests, and a personal to-do list
- **Notifications** — lab-wide alerts (order digest, urgent flags, backup reminders, lab notices) go out by email *or* Slack, whichever your lab prefers
- **Activity feed** — a running log of everything that changed
- **Guided tour** — a short walkthrough for every tab, shown automatically the first time someone signs in and replayable anytime from the top bar or Settings
- **Works offline-first** — runs in local-only mode with no setup; add a free Supabase project when you're ready to share data across the lab

## Getting started

### 1. Set up your database (5 minutes)

1. Create a free project at [supabase.com](https://supabase.com).
2. In the Supabase SQL editor, run:

   ```sql
   create table if not exists items (
     id text primary key,
     kind text not null,
     data jsonb not null default '{}'::jsonb
   );
   alter table items enable row level security;
   create policy "lab members read" on items for select to authenticated using (true);
   create policy "lab members write" on items for insert to authenticated with check (true);
   create policy "lab members edit" on items for update to authenticated using (true) with check (true);
   create policy "lab members delete" on items for delete to authenticated using (true);
   alter publication supabase_realtime add table items;
   ```

3. Under **Authentication → Providers → Email**, turn off "Allow new users to sign up." Then add your lab members individually under **Authentication → Users** (this keeps sign-in restricted to people you invite).
4. Under **Project Settings → API**, copy the **Project URL** and the **anon (publishable) key**.
5. Open `index.html` in a text editor, and near the top of the `<script>` section paste your two values into:

   ```js
   const SUPA_URL = "";   // paste your Project URL here
   const SUPA_KEY = "";   // paste your anon/publishable key here
   ```

Until you do this, the app runs fine in **local-only mode** — one browser, nothing shared — so you can try it out before setting anything up.

### 2. Host the file

Any static host works. A few free options:

- **Cloudflare Pages** — drag-and-drop deploy, must be named `index.html`
- **Netlify** — same idea, drag-and-drop or connect a repo
- **GitHub Pages** — serve straight from this repo

### 3. Make it yours

Once it's running, open **Settings** to add your lab's name, members, vendors, storage locations, and notification preferences — no code editing required for day-to-day use.

### 4. Personalize the code (optional)

Everything above is doable from the app itself. A handful of things live in the HTML file instead, because they're either security-sensitive or need to exist before anyone has signed in yet. All are optional — skip this if the defaults are fine.

**Who can reset the app.** Near the top of the `<script>` section, search for `TRUSTED_ADMIN_EMAILS`:

```js
const TRUSTED_ADMIN_EMAILS = [
  // add your own email here, e.g. 'you@your-lab.edu'
];
```

Anyone whose Supabase sign-in email is in this list (or who is currently set as "Lab manager email" in Settings) can clear all the data or wipe the app back to a blank slate. Add your own email, and anyone else you want to always have that power regardless of who's lab manager later. This list isn't shown anywhere in the app, so it's safe to leave a name in here even after that person moves on.

**Starting point for dropdowns.** The app starts with no vendors, orders, inventory, or events, since every lab's setup is different (not everyone orders through the same kind of portal, for instance). The only things pre-filled are a handful of dropdown *options* — order categories, storage locations, lab member roles, and event types — which come from the `seed()` function (search for `function seed()`). Edit the lists there before you deploy if you'd rather start with your own categories, or just edit them from Settings once the app is running; either way works.

**Colors.** Near the top of the `<style>` section, under `:root`, is the full color palette as named variables (`--green`, `--coral`, `--sun`, `--sky`, `--plum`, and so on). Swap any hex value and it updates everywhere that color is used, no need to hunt through the rest of the file.

**Browser tab title.** Search for `<title>BenchSaurus · Lab Manager</title>` near the top of the file and change it to whatever you'd like the tab to say.

**Tab icon (favicon).** There isn't one by default. To add a quick emoji favicon, paste this into `<head>` (right under the `<title>` line works well) and swap the emoji for whichever one you like:

```html
<link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>🦕</text></svg>">
```

## Getting oriented: the guided tour

The first time anyone signs in, BenchSaurus walks through every tab automatically: what it's for, and the handful of buttons worth knowing about on each one. It only runs once per browser.

After that, anyone can replay it:

- Click **Take a tour** in the top bar to replay the current tab's walkthrough.
- Or go to **Settings → Help & tours** to replay any single tab, or run the whole thing again from the start.

This is handy when a new person joins the lab, since they get the same orientation without anyone having to explain it to them.

## Notifications: email or Slack

Under **Settings → Lab notifications**, choose how lab-wide alerts (the order digest, urgent flags, backup reminders, and lab notices) go out:

- **Email** — opens a draft in your default mail app, Outlook on the web, or Gmail on the web, depending on what you pick
- **Slack** — posts directly to a channel via an [incoming webhook](https://api.slack.com/messaging/webhooks). Create one at api.slack.com/apps, pick the channel, and paste the webhook URL into Settings.

Personal order-status alerts (opting in to hear about just your own requests) always go by email, since a Slack webhook can only post to one channel, not message an individual person.

## Data and privacy

Your lab's data lives in your own Supabase project — this repository doesn't include or transmit any data. Only people you explicitly add under Supabase's Authentication settings can sign in.

## Contributing

Issues and pull requests are welcome. This is a side project maintained as time allows, so response times may vary.

## License

MIT License — see [LICENSE](./LICENSE) for details.

Created by Maria Dima.
