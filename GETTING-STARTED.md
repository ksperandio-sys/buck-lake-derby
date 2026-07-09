# Buck Lake Fishing Derby — Complete Guide (Phone Only)

This guide assumes you've never used GitHub or Firebase before. Every step is spelled out. Do Part 1 once, before Thursday. Part 2 is what you'll actually use during the weekend. Part 3 is for whenever you want to change something later.

A quick note on words used below:
- **"Repo"** (repository) = a folder on GitHub that holds your website's files.
- **"Commit"** = GitHub's word for "save this change." Every time you edit a file on GitHub and save it, that's a commit.
- **"Deploy"** = making the website live/updated for everyone. With this setup, saving (committing) a change *is* the deploy — there's no separate button.

---

## PART 1 — One-time setup

### Step A: Create your Firebase project
1. Open your phone's browser, go to **console.firebase.google.com**
2. Sign in with your Google account if it asks.
3. Tap **"Create a project"** (or **"Add project"**).
4. Type a name — anything works, e.g. `buck-lake-derby`. Tap **Continue**.
5. It may ask about Google Analytics — you can turn this **off**, it's not needed. Tap **Create project**.
6. Wait for the spinner to finish (~20-30 seconds), then tap **Continue**.

### Step B: Turn on "Sign in with Google" for your app
1. On the left side, find and tap **Build**, then tap **Authentication**.
   (If you don't see "Build," look for a menu icon — three lines — to open the sidebar.)
2. Tap **Get started**.
3. You'll see a list of sign-in methods. Tap **Google**.
4. Tap the toggle switch to turn it **Enable**.
5. It'll ask for a support email — pick your own email from the dropdown.
6. Tap **Save**.

### Step C: Create the database
1. On the left sidebar, tap **Build**, then **Realtime Database**.
2. Tap **Create Database**.
3. Pick any location shown (doesn't matter which). Tap **Next**.
4. Choose **"Start in test mode"**. Tap **Enable**.
   (This means the app can read/write freely for now. We'll tighten this in Step G.)

### Step D: Get your app's "config" — a block of code Firebase gives you
1. Tap the **gear icon** ⚙️ near the top left, then tap **Project settings**.
2. Scroll down until you see **"Your apps"**.
3. Tap the icon that looks like `</>` (this means "web app").
4. Type any nickname, e.g. `derby-web`. Leave other boxes unchecked. Tap **Register app**.
5. You'll now see a gray box of code containing something like this:
   ```
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "buck-lake-derby.firebaseapp.com",
     databaseURL: "https://buck-lake-derby-default-rtdb.firebaseio.com",
     projectId: "buck-lake-derby",
     storageBucket: "buck-lake-derby.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abc123"
   };
   ```
6. **Leave this screen open in one browser tab** — you'll copy these values into your file in Step F. You can also just take a screenshot of it as a backup.
7. Tap **Continue to console** to finish.

### Step E: Put your website file on GitHub
1. Open a new browser tab, go to **github.com**
2. Tap **Sign up** if you don't have an account (it's free), or **Sign in** if you do.
3. Once signed in, tap the **"+"** icon near the top right, then tap **"New repository"**.
4. Give it a name, e.g. `buck-lake-derby`. Make sure **Public** is selected (not Private).
5. Tap **Create repository**.
6. On the new (empty) repo page, tap **"uploading an existing file"** (or "Add file" → "Upload files").
7. Tap to choose a file, and select the **`index.html`** file from the zip I gave you (unzip it first if your phone hasn't already).
8. Scroll down, tap **Commit changes** (this saves the upload).

### Step F: Paste your Firebase config into the file, right on GitHub
1. Still on your repo page, tap on **index.html** to open it.
2. Tap the **pencil icon** ✏️ (usually top right of the file view) to edit it.
3. Use your browser's search-in-page feature (or just scroll) to find this block near the top:
   ```
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT.firebaseapp.com",
     databaseURL: "https://YOUR_PROJECT-default-rtdb.firebaseio.com",
     projectId: "YOUR_PROJECT",
     storageBucket: "YOUR_PROJECT.appspot.com",
     messagingSenderId: "YOUR_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   ```
4. Go back to your Firebase tab from Step D, and carefully copy each value over — replace `"YOUR_API_KEY"` with your real `apiKey` value (keep the quote marks), and so on for all 7 lines.
5. Scroll to the bottom of the GitHub edit screen, tap **Commit changes**.

Your admin email is already set in the file (`karlosperandio@gmail.com`), so nothing to do there.

### Step G: Turn your repo into a live website
1. On your repo page, tap **Settings** (near the top, may be under a "..." menu on small screens).
2. On the left, tap **Pages**.
3. Under "Build and deployment," set **Source** to **Deploy from a branch**.
4. Set **Branch** to **main**, folder **/ (root)**. Tap **Save**.
5. Wait about a minute, then refresh this same Settings → Pages screen. A green box will appear with your live link, something like:
   `https://karlosperandio.github.io/buck-lake-derby/`
6. **Save this link** — this is what you'll share with friends and use yourself all weekend.

### Step H: Tell Firebase to trust your new website address
1. Go back to your Firebase tab: **Authentication → Settings** (top tab) **→ Authorized domains**.
2. Tap **Add domain**.
3. Type just the domain part of your link from Step G — for example `karlosperandio.github.io` (no `https://`, no trailing slash or folder name).
4. Tap **Add**.

### Step I: Lock the database down (2 minutes, do this before sharing the link widely)
1. Firebase tab → **Realtime Database → Rules** (tab near the top).
2. Delete what's there and paste this in:
   ```json
   {
     "rules": {
       "users": {
         "$uid": {
           ".read": "auth != null",
           ".write": "auth != null && auth.uid === $uid"
         }
       },
       "bets": {
         "$uid": {
           ".read": "auth != null",
           ".write": "auth != null && auth.uid === $uid"
         }
       },
       "catches": {
         "$uid": {
           ".read": "auth != null",
           ".write": "auth != null && auth.uid === $uid"
         }
       },
       "eventsOverride": {
         ".read": "auth != null",
         ".write": "auth != null"
       },
       "allowedEmails": {
         ".read": "auth != null",
         ".write": "auth != null"
       }
     }
   }
   ```
3. Tap **Publish**.

### Step J: Test everything
1. Open your live link from Step G in a fresh browser tab.
2. Tap to sign in with Google, using `karlosperandio@gmail.com`.
3. You should land on the Leaderboard and see an **Admin** tab along the top. If you see it, setup worked. 🎉

---

## PART 2 — Using it during the derby

**To invite a friend:**
Open the app → **Admin** tab → find "Invited players" → type their Gmail address → tap **Add**. They can sign in within seconds — nothing to update or re-upload.

**To fix a game once the real matchup is known** (e.g. World Cup opponents, exact Blue Jays game):
Admin tab → "Add/edit event" section → pick the day → type the sport name exactly as shown elsewhere in the app (e.g. `World Cup`) → paste in details using this pattern:
```
{"id":"wc_qf1","title":"Quarterfinal 1","teams":["France","Brazil"],"time":"2026-07-09T16:00:00-04:00","odds":{"home":-140,"away":120,"draw":250},"props":[]}
```
Use the same `"id"` as the placeholder it's replacing so it updates instead of duplicating.

**Game bets (moneyline) settle themselves** once a final score comes back — nothing for you to do. If you'd rather review results yourself before payouts happen, Admin tab → uncheck "Auto-settle game bets."

**Prop bets (method/round, over/under, etc.) always need one tap** — there's no data feed that reports these, so a person has to know the actual outcome. Admin tab → "Settle a prop bet" → type or tap the Prop ID → pick the option that actually happened from the dropdown → **Settle Prop & Pay Out**.

**Fish catches always need one tap too** — Admin tab → "Verify Catches" → look at the photo/length → **Approve** or **Reject**.

None of the above require touching GitHub — they all happen live, instantly, right in the app.

---

## PART 3 — Making a code change later (e.g. if something needs fixing)

1. Go to your GitHub repo in your phone's browser.
2. Tap **index.html**.
3. Tap the pencil ✏️ icon to edit.
4. Make your change.
5. Scroll down, tap **Commit changes**.
6. Wait ~30-60 seconds, then refresh your live link — the change is now live.

That's the entire update process — editing and committing on GitHub *is* the deploy. If you're not sure what to change, send me what you want fixed and I'll write out the exact text to paste in.
