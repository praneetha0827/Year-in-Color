# Year in Color — setup guide

This is a real, standalone web app for two people to log daily colors + notes and
see each other's data live. It uses **Firebase Firestore** as the backend (free
tier is plenty for this), so there's no server to run — just a static file and
a Firebase project.

Total setup time: ~10 minutes.

## 1. Create a Firebase project (free)

1. Go to https://console.firebase.google.com and sign in with a Google account.
2. Click **Add project**, give it any name (e.g. "year-in-color"), finish the wizard.
3. In the left sidebar, click the **</>** (web app) icon to register a web app.
   Give it a nickname, click **Register app**. Don't bother with Firebase Hosting
   in this step — you can skip it.
4. You'll now see a `firebaseConfig` object with `apiKey`, `authDomain`, etc.
   Copy the whole thing.

## 2. Enable Firestore

1. In the left sidebar, go to **Build → Firestore Database**.
2. Click **Create database**, choose any region close to you, start in
   **test mode** (we'll tighten this in step 4).

## 3. Paste your config into the app

1. Open `index.html` in a text editor.
2. Near the top of the `<script type="module">` block, find:
   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     ...
   };
   ```
3. Replace it with the config you copied in step 1.
4. Save the file.

## 4. Lock down the security rules

By default "test mode" allows anyone to read/write anything for 30 days, then
locks everything out. Set a permanent rule scoped to this app instead:

1. In Firestore, go to the **Rules** tab.
2. Replace the contents with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /trackers/{coupleId}/{document=**} {
         allow read, write: if true;
       }
     }
   }
   ```
3. Click **Publish**.

This means anyone who has your specific tracker link can read/write it — like a
shared Google Doc link. There's no login step, so don't post the link publicly;
just share it directly with your partner.

## 5. Put the file online

Pick whichever is easiest for you:

**Easiest — Netlify Drop (no account strictly required, instant link):**
- Go to https://app.netlify.com/drop
- Drag `index.html` onto the page
- You'll get a live URL in seconds (e.g. `https://random-name.netlify.app`)

**Also easy — GitHub Pages:**
- Create a new GitHub repo, upload `index.html` (rename it if you like)
- Repo Settings → Pages → deploy from the branch → you'll get a URL

**Also works — Hugging Face Spaces (Static):**
- Go to https://huggingface.co/new-space
- Give it a name, and for **SDK** choose **Static** (not Gradio/Streamlit/Docker)
- Once created, upload your file through the Space's **Files** tab and name it
  exactly `index.html` at the root of the repo — Spaces auto-serves it
- Your live app appears at `https://huggingface.co/spaces/YOUR_USERNAME/YOUR_SPACE_NAME`
  (there's also a direct embed URL shown on the Space page)
- Note: Spaces are public by default — anyone who finds the Space *can* open the
  app, but without your specific `?c=...` invite link they can't see or touch
  your actual tracker data (they'd just get a fresh empty one). If you want the
  Space itself hidden from search/browsing too, you can mark it **Private**, but
  then your partner would need a Hugging Face account and be added as a
  collaborator to view it — more setup than it's usually worth for this.

**If you already use Firebase Hosting:**
- `npm install -g firebase-tools`
- `firebase login`
- `firebase init hosting` (point the public directory at this folder)
- `firebase deploy`

## 6. Start using it

1. Open your live URL. It'll ask for both your names, then show you a
   shareable link (with a `?c=...` code in it — that code is what ties both
   of you to the same shared tracker).
2. Copy that link and send it to your partner.
3. They open it on their own device, pick their name, and you're both logging
   into the same space — each can see the other's colors, streaks, and notes.
4. The "copy invite link" button at the top of the app gives you that link
   again any time.

## Notes

- Each device remembers who it belongs to (via `localStorage`), so you won't
  be asked "who are you?" every time — only once per device/browser.
- If you ever want a second couple/tracker (e.g. testing), just visit the URL
  without the `?c=...` part and it'll generate a fresh one.
- Firebase's free "Spark" tier gives 50k document reads and 20k writes a day,
  which is far more than two people logging once a day will ever use.

## 7. Install it as an app on your phone

The app now includes everything needed to be installed like a real app —
its own icon, a full-screen window with no browser address bar, and basic
offline support. There's nothing extra to set up: once you upload the new
files (`manifest.json`, `service-worker.js`, the icon PNGs, and the updated
`index.html`) alongside each other in the same place your site is hosted,
this works automatically.

**On iPhone (Safari):**
1. Open your tracker link in Safari (not Chrome — iOS only supports this
   through Safari).
2. Tap the **Share** icon (square with an arrow) at the bottom.
3. Scroll down and tap **Add to Home Screen**.
4. It'll appear on your home screen with the app's icon, and open full-screen
   with no browser bar when tapped.

**On Android (Chrome):**
1. Open your tracker link in Chrome.
2. Tap the **⋮** menu (top right) → **Add to Home screen** / **Install app**
   (Chrome sometimes shows this as a banner automatically too).
3. Confirm — it'll install like a normal app icon.

Do this separately on each of your phones, using each person's own invite
link the first time (so the right shared tracker gets remembered on that
device going forward, even when launched from the home screen icon instead
of a browser tab).

## Running a small trial with other couples

You don't need any code changes to let 2–3 couples try this. Every tracker is
identified by its own random code (the `?c=...` in the link), and all of a
couple's data lives under that code in the database — completely separate
from any other couple's. So:

- Each couple you invite should go through the **"Start a new shared
  tracker"** flow themselves (from the same live URL you're already using) —
  this gives *them* their own private code and data, not yours.
- Don't hand out your own `?c=...` link to anyone but your partner — that
  exact link is what grants access to your specific data.
- Since there's no login system yet (see the security rule in step 4), access
  is based purely on knowing the exact link — similar to an unlisted Google
  Doc. Fine for a small trial among people you trust; not something to post
  publicly.
- All couples share the same free Firebase project for now. The free tier
  comfortably covers a handful of couples logging daily — you'd only need to
  think about upgrading if this grows well beyond a small trial.

This is a good way to validate the idea before investing in a "real" version
with individual accounts, proper sign-in, and stricter per-couple security —
which would be the natural next step if people actually want to keep using
it.
