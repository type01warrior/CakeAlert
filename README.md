# 🎂 Cake Alert — Never Miss a Sweet Moment

A mobile-first Progressive Web App (PWA) for tracking birthdays, anniversaries, and other milestone events — with cloud sync, smart notifications, photo support, a calendar view, and full backup/restore.

Built by **Type01warrior**.

---

## ✨ Features

### 🏠 Home Tab
- Scrollable list of all upcoming events sorted by next occurrence
- Each event card shows name, type icon, photo (if added), days remaining, and date
- **Today's events** highlighted at the top
- Quick-tap to open full event detail
- Search / filter events by name or type
- FAB (floating action button) to add a new event

### 📅 Calendar Tab
- Monthly calendar view with event dots on relevant dates
- Tap a date to see all events on that day
- Navigate between months with prev/next arrows
- Swipe left/right to switch between Home and Calendar tabs

### ➕ Add / Edit Events
Each event stores:
- **Name** — person or occasion name
- **Type** — Birthday, Anniversary, or any custom type you create (with custom emoji icon)
- **Photo** — taken from camera or picked from gallery, with a built-in **image cropper**
- **Date** — day, month, and optional year (year-less events repeat every year automatically)
- **Categories** — tag events with custom category labels (e.g. Family, Work)
- **Mobile number** — optional, for quick WhatsApp / call access
- **Location** — optional city or address
- **Notes** — gift ideas, plans, anything else

### 🔔 Notifications
- **Daily reminder** — configurable time to receive a digest of today's and upcoming events
- **Pre-event alerts** — get notified 1 day and 7 days before an event
- **In-app notification centre** — a dedicated panel listing all recent alerts with clear-all support
- Native **Android push notifications** via the `AndroidNotif` JS bridge (scheduled alarms)
- Browser-based web notifications as fallback on non-Android platforms

### 🗂️ Custom Types & Categories
- Built-in event types: Birthday 🎂, Anniversary 💍, and more
- **Type Manager** — create, rename, or delete custom event types with any emoji icon
- **Category Manager** — create, rename, or delete custom categories with any emoji icon
- Categories displayed as filterable chips on the home screen

### ☁️ Cloud Sync (Firebase)
- **Real-time Firestore sync** — changes made on one device appear on all others instantly
- Sign in with **email/password** or **Google account**
- Manual "Sync now" button with live status indicator
- Built-in **Connection Test** and **Firestore security rules guide** inside Settings
- **Offline persistence** enabled (changes made offline sync when reconnected)

### 💾 Backup & Restore
- **XML backup** — export all events to a `.xml` file
- **Android backup bridge** — named backups saved directly to device storage via `AndroidBackup` JS bridge
- **Restore picker** — list saved backups, preview, rename, delete, or restore
- **Import from file** — pick an `.xml` backup from the device file system
- **Merge or Replace** modes when restoring (add new events only, or overwrite everything)

### 🎨 Appearance
- **Light / Dark mode** toggle, persisted in `localStorage`
- Clean, minimal design with a rose/cream color palette
- Smooth iOS-style slide-up modals with swipe-down-to-close gesture on all panels

### 📱 Mobile UX
- Safe-area insets for notched phones and iOS home bar
- Swipe-down gesture to close any bottom sheet
- Android hardware back button support via `window.cakeAlertBack()` JS bridge
- Tap-to-open, swipe-to-dismiss — no jarring page navigations

---

## 📁 File Structure

```
├── index.html    # Entire app — UI, logic, Firebase, notifications, backup
├── logo.png      # Splash screen logo (220×220, shown on load)
```

> The app is intentionally a **single HTML file** — no build step, no bundler. Drop it in a WebView or serve it from any static host.

---

## 🔐 Authentication

- Email/password **Sign In** and **Sign Up** tabs
- **Google Sign-In** (one-tap OAuth flow)
- **Forgot password** — sends a Firebase reset email
- **Continue without account** — local-only mode (data stored in `localStorage`, no cloud sync)
- Auto-skips auth screen if already signed in
- 6-second auth timeout cap so a slow network never hangs the splash screen

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML + Tailwind CSS (CDN) + Lucide Icons |
| Auth & Database | Firebase Authentication + Firestore (v10 compat SDK) |
| Image Cropping | Cropper.js v1.6.2 |
| Fonts | Google Fonts — Lexend |
| PWA | Web App Manifest, safe-area insets |
| Android | WebView with `AndroidNotif`, `AndroidBackup`, `AndroidCamera` JS bridges |

---

## 🚀 Getting Started

### Option A — Browser / Static Host
1. Copy `index.html` and `logo.png` to any static web server or open locally.
2. Replace the `firebaseConfig` block near the top of `index.html` with your own project credentials (see below).
3. Open `index.html` in a browser. Sign in or continue offline.

### Option B — Android WebView
Load `index.html` inside a WebView and expose the following JavaScript interfaces:

```kotlin
// Notification scheduling
webView.addJavascriptInterface(NotifBridge(), "AndroidNotif")
// AndroidNotif.schedule(jsonPayload)
// AndroidNotif.cancel(eventId)

// File-based backup/restore
webView.addJavascriptInterface(BackupBridge(), "AndroidBackup")
// AndroidBackup.save(xmlString, fileName) → filePath
// AndroidBackup.list()                    → JSON array of {name, path, date}
// AndroidBackup.read(filePath)            → xmlString
// AndroidBackup.deleteBackup(filePath)    → boolean
// AndroidBackup.downloadFile(content, filename)

// Camera / gallery
webView.addJavascriptInterface(CameraBridge(), "AndroidCamera")
// AndroidCamera.pickImage()  — triggers gallery picker

// Back-button handling
// Call: webView.evaluateJavascript("window.cakeAlertBack()", callback)
// Returns: true (JS handled it) | false (finish the Activity)
```

### Firebase Setup
Replace the `firebaseConfig` in `index.html` with your own project's values:

```javascript
const firebaseConfig = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT.firebaseapp.com",
  projectId:         "YOUR_PROJECT_ID",
  storageBucket:     "YOUR_PROJECT.firebasestorage.app",
  messagingSenderId: "YOUR_SENDER_ID",
  appId:             "YOUR_APP_ID"
};
```

Then publish the following **Firestore Security Rules** (Firebase Console → Firestore → Rules):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if
        request.auth != null &&
        request.auth.uid == userId;
    }
  }
}
```

---

## 📦 Firestore Data Structure

```
users/
  └── {uid}/
        └── events/
              └── {eventId}: {
                    name:       "Alice",
                    type:       "birthday",
                    day:        15,
                    month:      8,
                    year:       1995,        // optional
                    photo:      "data:image/…",
                    mobile:     "+91 98765 43210",
                    location:   "Mumbai",
                    notes:      "Loves chocolate cake",
                    categories: ["family"],
                    updatedAt:  <timestamp>
                  }
```

---

## 📤 Backup Format

Backups are `.xml` files with a simple structure:

```xml
<CakeAlertBackup version="1" exported="2025-08-15T10:30:00.000Z">
  <Events>
    <Event id="abc123" name="Alice" type="birthday" day="15" month="8" year="1995" .../>
    …
  </Events>
</CakeAlertBackup>
```

Import supports both **Replace** (overwrites all local events) and **Merge** (adds/updates without deleting existing events).

---

## 📄 License

Personal / open use. Please credit **Type01warrior** if you build on this.
