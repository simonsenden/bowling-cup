# 🎳 The Ultimate Bowling Cup

A mobile web app for one bowling tournament night: 8 players, 2 lanes, 3 group games and 1 final.
It syncs live across everyone's phones. After each game, one person per lane types in the 4 scores. The app works out the rest: duels, points, standings, drinks, the final seeding and the ceremony.

It's a single `index.html` with no build step and no npm. It's hosted for free on **GitHub Pages**, and **Firebase Realtime Database** (free Spark plan) does the syncing.

Setup takes about 10 minutes. You don't need any coding experience.

---

## 1. Create a Firebase project and copy the config

1. Go to <https://console.firebase.google.com> and sign in with a Google account.
2. Click **Create a project** (or **Add project**), give it a name such as `bowling-cup` and click **Continue**. You can switch Google Analytics **off**. Then click **Create project**.
3. On the project overview page, click the **Web** icon `</>` ("Add app").
4. Enter a nickname such as `bowling-web`. **Don't** tick "Firebase Hosting". Click **Register app**.
5. Firebase now shows a code block containing `const firebaseConfig = { ... }`. Leave that tab open. You'll copy it in step 2.6, after the database exists.

## 2. Create the Realtime Database and paste the rules

1. In the left menu, open **Build → Realtime Database** and click **Create Database**.
2. For location, choose **Belgium (europe-west1)**.
3. Choose **Start in locked mode** and click **Enable**.
4. Open the **Rules** tab, delete everything there and paste this in:

```json
{
  "rules": {
    "rooms": {
      "$room": {
        ".read": true,
        ".write": true,
        "games": {
          "$g": {
            "$lane": {
              "$player": {
                ".validate": "newData.isNumber() && newData.val() >= 0 && newData.val() <= 300"
              }
            }
          }
        }
      }
    },
    "$other": { ".read": false, ".write": false }
  }
}
```

5. Click **Publish**.
6. Go to **Project settings** (the ⚙️ gear next to "Project Overview"), scroll down to **Your apps** and select **Config**. Copy the object, which looks like this:

```js
{
  apiKey: "AIza...",
  authDomain: "bowling-cup-xxxx.firebaseapp.com",
  databaseURL: "https://bowling-cup-xxxx-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "bowling-cup-xxxx",
  storageBucket: "bowling-cup-xxxx.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abc123"
}
```

> ⚠️ **Make sure `databaseURL` is in there.** If you registered the web app before you created the database, the config can be missing that line. In that case, copy the URL shown at the top of the **Realtime Database → Data** tab (it starts with `https://` and ends with `firebasedatabase.app`) and add it yourself.

7. Open `index.html` in a text editor (Notepad, TextEdit, VS Code, or GitHub's own editor all work). Near the top of the `<script>`, find this block:

```js
// >>> PASTE YOUR FIREBASE CONFIG HERE <<<
const FIREBASE_CONFIG = {
  apiKey: "PASTE_YOUR_API_KEY",
  ...
};
```

Replace the values with yours and keep the `const FIREBASE_CONFIG =` part. Save the file.

If the config is missing or incomplete, the app shows a friendly "Almost ready" screen instead of breaking.

## 3. Create a GitHub repo and upload the files

1. Go to <https://github.com/new>.
2. Name the repo, for example `bowling-cup`, and set it to **Public**. (GitHub Pages is free for public repos.)
3. Click **Create repository**.
4. On the new repo page, click **uploading an existing file** and drag in `index.html` and `README.md`. Then click **Commit changes**.

## 4. Turn on GitHub Pages

1. In the repo, go to **Settings → Pages**.
2. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
3. Set **Branch** to **main** and the folder to **/ (root)**. Click **Save**.
4. Wait about 1 minute and refresh. A message appears: "Your site is live at `https://<your-user>.github.io/bowling-cup/`".

## 5. Play

1. Open the Pages URL on your phone and tap **Create tournament**. Enter your name, level and a **4-digit Admin PIN**. You are the Admin.
2. You get a room code such as `BOWL-7K2` and an invite link:
   `https://<your-user>.github.io/bowling-cup/?room=BOWL-7K2`
   Send it to the group with **Copy invite link** or **Share**, or let people scan the **QR code** on your screen.
3. Once all 8 players have joined, tap **Draw teams**. Every phone shows the draw at the same time.
4. After each game, one person per lane taps **Submit scores** and types in the 4 numbers from the alley screen. That's all the input needed.

**Admin tips**
- Tap the 🔒 in the top-right corner and enter the PIN to unlock Admin on any phone. It stays unlocked until you close the tab.
- From there you can remove a player (before the draw), edit any score (open "Earlier games" / "All scores"), switch on the **Sprint Final (5 frames)**, settle sudden-death ties and **Reset tournament**. Resetting keeps the players and clears the draw and all scores.
- **Scoring self-test:** add `?test=1` to the URL to check the duel, bonus and tiebreak math.

---

## Is it OK that my Firebase config is in a public repo?

**Yes.** The Firebase web config (`apiKey` and the rest) is not a secret. It only tells the app which project to talk to, and every Firebase web app ships it to the browser. **The security rules protect your data.** The rules above allow reading and writing only inside `rooms/` and reject any score outside 0–300. Everything else in the database is locked.

This is a party app, so keep in mind:
- Anyone who has a room code can read and write that room. Only share the link with your group.
- The Admin PIN is stored as a SHA-256 hash, never in plain text. It keeps honest friends from pressing the wrong buttons, but it isn't bank-grade security.
- To clean up after the night, delete the room under **Realtime Database → Data → rooms**.

## Troubleshooting

| Problem | Fix |
|---|---|
| "Almost ready" screen | The config isn't pasted correctly, or `databaseURL` is missing (see step 2.6). |
| "Can't reach the server" | Check your internet connection. Also check that the config values have no typos. |
| "Could not read the room" | The rules weren't published. Repeat step 2.4–2.5. |
| The pill says "Reconnecting…" | The phone lost signal. Scores you submit are kept on the phone and sync once the connection is back. |
| Old version after an update | GitHub Pages can cache for a few minutes. Wait, then reload. |
| Admin PIN check fails on a local file | The PIN check needs `https://`. Use the GitHub Pages link, not a file opened from your computer. |
