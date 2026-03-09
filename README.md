# 🥊 Muay Thai Transformation Tracker

A 12-week interactive training, nutrition, and habit tracking system connected to Firebase Firestore. Built as a single HTML file — no build tools, no frameworks, just open and go.

---

## What It Tracks

- **Training sessions** — Monday Muay Thai, Wednesday Conditioning + Muay Thai, Saturday long session + optional light session, Sunday recovery. Each with checklists, effort scores, and session notes.
- **Daily habits** — Water intake, portion control, snack discipline, movement, sleep, and overall discipline in a weekly grid.
- **Nutrition discipline** — Daily food checklist and reflection (triggers, control, improvements).
- **Water intake** — Visual glass-by-glass tracker (each glass = 250ml).
- **Weekly reviews** — Weight, waist, progress photos, discipline/energy/recovery scores (1–10), and written reflections.
- **12-week progress** — Overall completion chart, weight trend, and challenge progress bar.

---

## Setup Guide

### Step 1 — Create a Firebase Project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project**
3. Give it a name (e.g. `muaythai-tracker`)
4. Google Analytics is optional — you can disable it
5. Click **Create project** and wait for it to finish

### Step 2 — Create the Firestore Database

1. In your Firebase project, go to **Build → Firestore Database** in the left sidebar
2. Click **Create database**
3. **Choose "Start in test mode"** — this allows reads and writes without authentication (fine for personal use)
4. Select a Firestore location close to you (e.g. `us-central1` or `southamerica-east1` for LATAM)
5. Click **Enable**

> ⚠️ If you accidentally created the database in **production mode**, go to **Firestore → Rules** and replace the rules with:
>
> ```
> rules_version = '2';
> service cloud.firestore {
>   match /databases/{database}/documents {
>     match /{document=**} {
>       allow read, write: if true;
>     }
>   }
> }
> ```
>
> Then click **Publish**.

### Step 3 — Register a Web App

1. In your Firebase project, click the **gear icon → Project settings**
2. Scroll down to **Your apps** and click the web icon (`</>`)
3. Give it a nickname (e.g. `tracker-web`)
4. You do NOT need Firebase Hosting — leave that unchecked
5. Click **Register app**
6. Firebase will show you a config object that looks like this:

```json
{
  "apiKey": "AIzaSy...",
  "authDomain": "your-project.firebaseapp.com",
  "projectId": "your-project-id",
  "storageBucket": "your-project.appspot.com",
  "messagingSenderId": "123456789",
  "appId": "1:123456789:web:abc123"
}
```

7. Copy that entire object

### Step 4 — Open the Tracker

1. Open `muaythai-tracker.html` in your browser
2. Paste the Firebase config object into the text area
3. Click **CONNECT & START**
4. If the connection is successful, you'll see the dashboard with a green **Firebase** indicator in the top bar

That's it. Your data now syncs to Firestore in real time.

---

## Alternative: Local Mode

If you don't want to use Firebase, click **"Use Local Storage Instead"** on the setup screen. All data will be saved in your browser's local storage. Note that this data is tied to your browser and will be lost if you clear browser data.

---

## Firestore Data Structure

The tracker stores data in these collections:

| Collection  | Document ID                         | Description                                              |
| ----------- | ----------------------------------- | -------------------------------------------------------- |
| `meta`      | `connectionTest`                    | Connection verification                                  |
| `weekFocus` | `w1`, `w2`, ... `w12`               | Weekly focus checklist state                             |
| `notes`     | `w1`, `w2`, ... `w12`               | Weekly dashboard notes                                   |
| `training`  | `w1_monday`, `w3_saturday_am`, etc. | Session checklists and notes                             |
| `habits`    | `w1`, `w2`, ... `w12`               | Weekly habit grid (day × habit)                          |
| `water`     | `2026-03-09`, `2026-03-10`, etc.    | Daily water glass count                                  |
| `nutrition` | `2026-03-09`, `2026-03-10`, etc.    | Daily food checklist and reflection                      |
| `reviews`   | `w1`, `w2`, ... `w12`               | Weekly review (weight, scores, reflections)              |
| `photos`    | `w1`, `w2`, ... `w12`               | Progress photos as compressed base64 (front, side, back) |

Photos are compressed client-side to 800px max width at 70% JPEG quality (~100-150KB each) and stored directly in Firestore as base64 data URLs. No Firebase Storage needed. At 3 photos × 12 weeks ≈ 5MB total — well within Firestore's 1GB free tier.

---

## Security Notes

Test mode rules expire after **30 days** by default. When they expire, writes will start failing. To fix this:

1. Go to **Firestore → Rules**
2. Either extend the expiration date or replace the rules with open access (see Step 2 above)
3. For long-term use, consider adding Firebase Authentication and restricting rules to your user ID:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null
                         && request.auth.uid == "YOUR_USER_ID";
    }
  }
}
```

---

## Tech Stack

- Vanilla HTML/CSS/JS — no build step
- Firebase Firestore SDK v10 (loaded via CDN) — photos stored as compressed base64
- Fonts: Bebas Neue + Barlow (Google Fonts)
- Single file, runs from anywhere
- Zero cost on Firebase Spark (free) plan

---

## Troubleshooting

| Problem                                     | Fix                                                                                                               |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| "Firestore rules are blocking writes"       | Create database in test mode or update rules (see Step 2)                                                         |
| Photos not saving                           | Photos are compressed and stored in Firestore — check if Firestore write rules are open                           |
| Data not persisting after refresh           | Check the top bar for Firebase/Local indicator. If it says Local, your Firebase config may have been cleared      |
| Red toast on every save                     | Open browser console (F12) to see the full error                                                                  |
| Test mode expired                           | Update Firestore rules to extend or remove the expiration                                                         |
| App shows setup screen after it was working | Firebase config was cleared. Paste it again                                                                       |
| Photos look low quality                     | Images are compressed to keep Firestore usage low. 800px wide at 70% JPEG quality is enough for progress tracking |
