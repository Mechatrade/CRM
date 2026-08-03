# Mechatrade CRM — setup

Three files, no command line, no build step:

| File | What it is |
|---|---|
| `index.html` | The whole app |
| `firestore.rules` | Security rules you paste into the Firebase console |
| `README.md` | This guide |

---

## 1. Create the Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) → **Create a project**
2. Name it `mechatrade-crm` → **Continue**
3. Turn Google Analytics **off** → **Create project** → wait ~30s → **Continue**

## 2. Register the web app

1. On the project dashboard, click the **`</>`** (Web) icon
2. Nickname: `Mechatrade Web`. Do **not** tick Firebase Hosting — you're using GitHub Pages
3. **Register app** → copy the whole `firebaseConfig = { … }` object
4. **Continue to console**

## 3. Paste the config into `index.html`

Open `index.html` in any text editor, press Ctrl+F, search `REPLACE_ME`. Replace this block with what Firebase gave you:

```javascript
const firebaseConfig = {
  apiKey: "REPLACE_ME",
  authDomain: "REPLACE_ME.firebaseapp.com",
  projectId: "REPLACE_ME",
  storageBucket: "REPLACE_ME.appspot.com",
  messagingSenderId: "REPLACE_ME",
  appId: "REPLACE_ME"
};
```

These keys are safe in public code. The Firestore rules are the actual lock, not the keys.

## 4. Turn on Email/Password sign-in

**Build → Authentication → Get started → Email/Password → Enable → Save**

## 5. Create the database

**Build → Firestore Database → Create database** → location **`asia-southeast1` (Singapore)** → **Production mode** → Create.
The location can't be changed later.

## 6. Paste the security rules

**Firestore Database → Rules** tab → delete everything there → paste the entire contents of `firestore.rules` → **Publish**.

## 7. Create the first administrator by hand

Only an admin can add people, so the first one is made manually.

**a. Create the sign-in:**
Authentication → **Users** tab → **Add user** → enter your email and a password → Add user.
Click the new row and copy the **User UID** (a long string).

**b. Create the profile:**
Firestore Database → **Data** tab → **Start collection** → Collection ID: `users` → Next.
Document ID: **paste the User UID** (do not click auto-ID). Add these fields:

| Field | Type | Value |
|---|---|---|
| `displayName` | string | Your name |
| `email` | string | Your email |
| `role` | string | `admin` |
| `department` | string | `Admin` |
| `active` | boolean | `true` |

Save. That's the only manual record you'll ever create — everyone else gets added from the Team screen.

## 8. Put it on GitHub Pages

1. [github.com/new](https://github.com/new) → name it `mechatrade-crm` → Create repository
2. **uploading an existing file** → drag in `index.html` (and the other two files if you want them stored) → Commit
3. Repo **Settings → Pages** → Source: **Deploy from a branch** → Branch `main`, folder `/ (root)` → Save
4. Wait a minute. Your URL is `https://YOURNAME.github.io/mechatrade-crm/`

## 9. Authorise that domain in Firebase — don't skip this

Firebase blocks sign-in from unknown domains.

**Authentication → Settings → Authorized domains → Add domain** → enter `YOURNAME.github.io`

Now open your Pages URL and sign in as the admin.

## 10. Add your team

**Team → Add person.** Fill in name, work email, a temporary password, role, and monthly target. The account is created in the background — you stay signed in. Give the person their temporary password; they can request a new one via **Reset password**.

---

## How permissions actually work

| Tier | Roles | Sees |
|---|---|---|
| Staff | Sales Person, BD Officer, Marketing Officer | Only records they own, plus leads they sourced |
| Manager | Sales Officer, BD Manager, Marketing Manager | Every record in their department |
| Admin | Administrator | Everything, plus Team and Settings |

This is enforced in the security rules, not just hidden in the interface — a Sales Person's browser can't retrieve another rep's deals even if someone edits the page.

**Marketing → Sales handoff:** a Marketing Officer or Manager creates a lead and assigns it to a Sales or BD person. Ownership moves, but the creator keeps read access, so campaign reporting still works after handoff. When that lead converts, the resulting deal carries `sourcedBy` back to whoever sourced it — that's how the Campaign performance report attributes revenue.

## Daily use

- **Leads** — capture, work, then **Convert**, which creates the account, contact, and open deal in one step.
- **Pipeline** — drag a card between stages. The left edge shows how long it's been sitting: green under 7 days, amber 7–21, red beyond. Dropping onto **Lost** asks for a reason, which feeds the loss report.
- **Activities** — every call, meeting, and follow-up. Overdue ones surface on the dashboard.
- **Reports** — funnel, win rate, sales cycle, revenue by source, campaign ROI, activity volume.
- **Settings** — currency, the ageing thresholds, and every dropdown list.

## Changing things later

Almost everything is in labelled blocks at the top of the `<script>` in `index.html`:

- **Add a field to any record** — add one line to `SCHEMAS`. Lists, forms, detail panels, and search all pick it up. `table:true` puts it in the table.
- **Rename or reorder pipeline stages** — edit `STAGES`. Existing deals keep their old stage name until moved.
- **Change who sees what** — the `tier` values in `ROLES`. Sales Officer ships as `manager` (sees all of Sales); change it to `staff` if it should be own-records-only. Change it in `firestore.rules` too — the `isManager()` list — or the rules will block what the interface allows.
- **Rename the app** — search for `Mechatrade`.

## If something doesn't work

- **"Missing or insufficient permissions"** — the rules weren't published, or the signed-in user has no `users` document.
- **Sign-in fails on the live site but works locally** — step 9, authorised domains.
- **A screen is empty for one person but not another** — that's the scoping doing its job. Check their role in Team.
- **Firebase asks you to create an index** — the app avoids composite indexes, but if a link ever appears, clicking it and waiting a minute is the fix.
