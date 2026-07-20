# Monthly Finance Tracker

A single-file, no-backend personal finance and portfolio tracker. Everything runs client-side in `finance_tracker.html` — open it in any browser, no install or server required.

## Features

- **Monthly tab**: import a bank/credit card statement PDF (auto-parsed into expense line items, grouped by import batch), track multiple income sources by category, fixed monthly expenses (car payment, rent, utilities), and see what you saved each month. Month navigation via arrows or a picker.
- **Portfolio tab**: track Savings accounts and Stock/Investment accounts separately. Investment accounts support live price refresh (via a free Finnhub API key), a transaction log per holding (buys, with auto-calculated average cost basis and shares), gain/loss tracking, an asset allocation donut chart, point-in-time net worth snapshots, and a cash flow log for tracking transfers between accounts (e.g. bonus → savings → stock purchase).
- **Yearly Overview tab**: income vs. expenses chart, net worth trend, month-by-month breakdown.
- **Backup / Restore**: since all data lives in the browser's `localStorage` (which can be wiped by clearing browser data), use the "Backup Data (JSON)" button periodically and keep the file somewhere durable (cloud drive, this repo, etc.). "Restore from Backup" loads a previously saved file back in.
- **Cloud Sync (optional)**: connect a free Firebase project so the tracker's data stays current across every device/browser you open it in, instead of being stuck in one browser's local storage. See setup below.
- **CSV export**: per-month export of income, expenses, fixed costs, and account balances.

## Setting up Cloud Sync (Firebase)

This is optional — the app works fully offline with just `localStorage` and manual JSON backups. Cloud Sync adds automatic cross-device syncing via your own free Firebase project.

1. Go to [console.firebase.google.com](https://console.firebase.google.com/) and create a new project (the free Spark plan is enough for personal use).
2. In the project, go to **Build → Firestore Database**, click **Create database**, and start it in **test mode**.
3. Click the gear icon → **Project settings** → scroll to "Your apps" → click the **`</>`** (web) icon → register an app (you don't need Firebase Hosting) → copy the `firebaseConfig` object shown.
4. Open the app, expand the **Cloud Sync (Firebase)** card at the top, and paste that whole config object into the first box.
5. Make up a long, random string (20+ characters, mash your keyboard) and paste it into the second box. This acts as your access key since there's no login screen — **keep this string and your Firebase config out of anywhere public**, since anyone who has them can read or write your data.
6. In the Firebase console, go to Firestore's **Rules** tab and paste in (swapping in your random string):
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /financeTracker/YOUR-RANDOM-ID-HERE {
         allow read, write: if true;
       }
       match /{document=**} {
         allow read, write: if false;
       }
     }
   }
   ```
   Click **Publish**.
7. Back in the app, click **Connect**. Repeat steps 4–7 (same config, same random ID) on any other device you want kept in sync.

**Security note**: this setup intentionally skips login (no email/password) to keep things simple, at the cost of relying on the random ID as the only thing protecting your data. It's a reasonable tradeoff for a personal tool as long as you don't publish the file (with the config filled in) somewhere public. If you want real authentication instead, that's a follow-up enhancement — ask and it can be added.

## Usage

1. Download `finance_tracker.html` and open it directly in a browser (double-click, or `File > Open`).
2. All data is stored locally in that browser's storage — nothing is sent anywhere except:
   - Live stock quotes, if you set up a free [Finnhub](https://finnhub.io/register) API key on the Portfolio tab (your key and requests go directly from your browser to Finnhub; nothing passes through any other server).
3. Click **Backup Data (JSON)** regularly and keep the exported file somewhere safe — this is your only way to recover data if the browser's storage is ever cleared. Restoring is done via **Restore from Backup**.

## Notes for GitHub

- This is a static HTML file — no build step, no dependencies to install. It can be committed as-is.
- If you want a hosted, shareable version (e.g. via GitHub Pages), keep in mind each visitor's data lives only in *their* browser — it is not shared or synced between devices or people. Two people opening the same hosted page will each have their own independent, local data.
- Consider adding periodic JSON backups (from the "Backup Data" button) to this repo as well, so your financial history has a version-controlled home outside of browser storage.
