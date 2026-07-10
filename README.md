# 28th Annual Buck Lake Fishing Derby — Setup

A single-file app (`index.html`) — deploy it the same way you deployed the World Cup pool app.

## 1. Firebase project
- Use a new Firebase project (or a new "app" inside your existing one — up to you).
- Enable **Authentication → Google sign-in**.
- Enable **Realtime Database**, start in test mode for now (lock down rules before sharing widely — see below).
- Copy your web app config into the `firebaseConfig` object near the top of the `<script>` block.

## 2. Set yourself as admin
- Edit `ADMIN_EMAILS = ["yourname@gmail.com"]` to your actual email so you see the Admin tab.

## 3. Odds & live scores — fully automatic, no manual entry
- Sign up free at https://the-odds-api.com (no card required, 500 requests/month) and paste the key into `ODDS_API_KEY`.
- Odds fetch **automatically on login and every 5 minutes** while the app is open — no button to press. **Important**: only the admin's browser actually calls the odds API — every other signed-in friend's browser was originally polling independently too, which multiplies quota usage by however many people have the app open at once (easily blows through the free tier's 500/month in under an hour with just a few friends online). Now only the admin fetches, and syncs the results to Firebase for everyone else to pick up for free. Practical implication: **keep the admin's phone with the app open** during game times for odds/scores to keep updating for the whole group — if it's closed, updates pause until reopened.
- **Live scores fetch on the same schedule** for World Cup, MLB, and UFC, and show up right on each event card (🔴 LIVE or FINAL, with the score once it's reported) — so people can see how things stand without leaving the app. Admin also gets a compact "Live scores" reference list when settling, so there's no need to check another site to remember who won.
- Scores are used for two things: display, and (new) **automatic settlement of game/moneyline bets** — the instant a tracked game's score comes back `completed: true`, its bets settle and pay out with no admin action. This is on by default (toggle in Admin if you'd rather review every result yourself first).
- **Prop bets and fish catches are never auto-settled** — there's no automated way to know a fight ended by submission in round 2, or that a photo really shows an 18" bass. Those always need a tap from you. Admin gives you a fast path for both (dropdown of real prop options, photo shown right next to Approve/Reject) but "zero manual action" isn't honestly possible for either.
- **Game bets and prop bets settle through two separate Admin controls** — settling a game's moneyline result never touches prop bets on that same event, even if they share an event ID. This matters if you ever let a parlay mix a moneyline pick and a prop pick from the same game: each leg only resolves through its own settlement action (automatic for the game leg, manual for the prop leg), and the parlay only pays out once every leg has been individually settled.
- For World Cup, MLB (Jays/Yankees/Dodgers only), and UFC, odds refresh also **auto-discovers real fixtures**, replacing "TBD" placeholders as matchups are confirmed, and **recomputes the UFC method/round prop odds** live from the fetched moneyline.
- **How bet odds get locked**: the number you see updates live in the UI, and the bet slip shows the **combined odds building up as you add each pick**— but the payout is only calculated on whatever odds are live the instant you tap **Place Bet**. If the line moved since you added the pick, the app updates your slip and asks you to confirm again before it places anything (same as a real sportsbook).
- Sports/markets outside the free API's coverage still need the Admin "Add/edit event" box — there's no way around that without a paid odds provider.

## 4. Players: invite list & custom names
- Admin tab → "Invited players" lets you add specific Gmail addresses. Leave it empty and anyone with a Google account can join; add at least one email and the pool becomes invite-only (uninvited emails see a "not invited" screen instead of the app).
- Everyone picks their **own display name** the first time they sign in (a small prompt pops up, pre-filled with their Google name) and can change it anytime by tapping their name in the header.

## 5. New UI behavior
- **Swipe** left/right anywhere in the main content to move between Leaderboard → Derby → My Bets → Game Bets → Prop Bets (Admin is nav-tap only, not in the swipe order).
- The header, credits, and nav bar stay **frozen at the top** while you scroll; the day tabs (Thu/Fri/Sat/Sun) and Derby sub-tabs stick just below them.

## 6. Buck Lake Fish Derby (separate leaderboard from the betting pool)
- New "🎣 Derby" tab with three sub-views: **Derby Board** (leaderboard), **Log a Catch**, and **My Catches**.
- Scoring combines size and species: **points per fish = length in inches + a type bonus (Walleye +5, Bass +3, Pike +1)**. E.g. an 18" Bass = 21 pts; a 24" Walleye = 29 pts.
- **Verification required**: logging a catch requires a photo (camera or upload) and a length in inches. It sits as "Pending" and earns zero points until an admin approves it in Admin → "Verify Catches" (photo + claimed type/length shown side-by-side with Approve/Reject buttons). Rejected catches can include an optional reason shown to the angler.
- Photos are compressed client-side (resized, JPEG ~70% quality) and stored directly in the Realtime Database as part of the catch record — no Firebase Storage setup needed. This keeps things simple for a weekend derby; if your group logs a lot of catches with large photos, consider moving to Firebase Storage instead (happy to help wire that up if it comes up).

## 7. Fill in real matchups
The seed data in `EVENTS` has placeholders (`TBD`) for games/matches not yet confirmed:
- World Cup QF opponents (locking in as Round of 16 finishes)
- Wimbledon semifinal and final matchups (Jul 9–10 semis, Jul 11 women's final, Jul 12 men's final)
- Exact Jays/Yankees games that weekend

UFC 329's full card (5 early prelims, 4 prelims, 5 main card fights) is already seeded with real fighter names and generic placeholder odds — swap in real moneylines and fight-specific method/round odds as they're posted closer to fight night. Each fight already has an auto-generated "method & round" prop market (KO/TKO or Submission by round, or Decision, for each fighter) so your friends can chase bigger payouts than the plain moneyline.

Two ways to update any of this:
- Edit the `EVENTS` object directly in the code and redeploy, **or**
- Use the Admin tab's "Add/edit event" box (paste JSON) — this writes to Firebase directly (`eventsOverride`), so it updates instantly for every signed-in friend without a redeploy. This is the better option once the app is live and your friends are using it.

## 5. Deploy
```
firebase init hosting   # if not already done for this project
firebase deploy
```

## 6. Firebase security rules (recommended before sharing)
Right now anyone signed in can write anywhere. At minimum:
```json
{
  "rules": {
    "users": {
      "$uid": {
        ".read": "auth != null",
        ".write": "auth != null && (auth.uid === $uid || root.child('allowedAdmins').child(auth.uid).exists())"
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
    },
    "settledEvents": {
      ".read": "auth != null",
      ".write": "auth != null"
    },
    "settledProps": {
      ".read": "auth != null",
      ".write": "auth != null"
    }
  }
}
```
This stops people from editing each other's credit totals or catch records directly. It does **not** stop someone from calling `settleEvent()` or `verifyCatch()` from the console — for a friend-group pool that's probably fine, but if you want it airtight, both should move to a Cloud Function later (and the `users` write rule above is a placeholder — proper admin-gated writes need a real admin list stored server-side, not just client-side `ADMIN_EMAILS`).

`settledEvents` and `settledProps` are what make the ✅/❌ result badges show up on the actual pick buttons once something's settled — they record which outcome/option won so every user's browser (and a fresh page load) shows the same result, not just the settling admin's own session.

## How the mechanics work
- **Credits**: everyone starts at 100 — one single cumulative balance for the whole weekend, not reset per day. Stake is deducted the moment a bet is placed (escrowed); payout = stake × decimal odds, credited back on settlement if the pick wins. Winnings from Thursday's games are just as spendable on Sunday's as the original 100 were.
- **Parlays**: combined odds = product of each leg's decimal odds, shown live as you build the slip. All legs must resolve before a parlay pays (one leg loses → whole parlay loses).
- **Cutoffs**: any event past its `time` is grayed out / unclickable in the UI, and settlement/placement double-checks the clock server-side-ish (client clock — good enough for a friend pool, not casino-grade).
- **Leaderboard** (landing page): one combined ranking — **Total = (Derby points × 80%) + (Betting credits × 20%)**. Shows each broken out in its own column so it's clear where someone's rank is coming from, sorted by Total. Note this weights the raw numbers directly rather than normalizing each to a percentage of the group's max — if betting credits swing wildly higher than derby points (e.g. a big parlay win), it'll still pull real weight even at 20%, just less than an unweighted sum would. Adjust the `DERBY_WEIGHT`/`BETTING_WEIGHT` constants in `renderLeaderboard()` if you want different balance.
- **Derby tab → Derby Board**: a second, derby-only view (Fish/Length/Total-points-from-fish columns) for anyone who just wants to see the fishing standings on their own, separate from the combined Leaderboard.
- **Derby points**: from *approved* catches only — points per fish = length (inches) + type bonus (Walleye +5 / Bass +3 / Pike +1). Pending and rejected catches don't count.
