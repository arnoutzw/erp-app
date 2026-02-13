# SSO Integration — ForgeERP

Firebase Google SSO authentication for ForgeERP.
All auth state is client-side; credentials are stored in `localStorage`, never committed to source control.

---

## Architecture Overview

```
┌────────────────────┐      ┌────────────────────────┐
│   Login Screen     │      │   Firebase Console      │
│  (2-step wizard)   │─────▶│  Google Auth Provider   │
│                    │      │  Firestore DB           │
└────────┬───────────┘      └────────────┬────────────┘
         │                               │
         │  signInWithPopup()            │ onAuthStateChanged()
         ▼                               ▼
┌────────────────────┐      ┌────────────────────────┐
│   _firebaseAuth    │◀────▶│   _currentUser          │
│  (Auth instance)   │      │  (Google user object)   │
└────────┬───────────┘      └────────────┬────────────┘
         │                               │
         ▼                               ▼
┌────────────────────┐      ┌────────────────────────┐
│  Firestore Sync    │      │  Sidebar user display   │
│  kanban-boards/    │      │  + Settings gear        │
│  lab-inventory/    │      │  + Sign-out button      │
└────────────────────┘      └────────────────────────┘
```

## SDK

Firebase JS SDK 9.23.0 (compat layer) loaded via CDN:

```html
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
```

---

## Login Flow (2-Step)

### Step 1 — Firebase Configuration

The user provides their Firebase project credentials. Two input methods:

| Method | Description |
|--------|-------------|
| **Paste snippet** | Paste the full `const firebaseConfig = { ... }` from the Firebase Console. Auto-parsed with regex. |
| **Manual entry** | Fill individual fields: `apiKey` (required, must start with `AIza`), `projectId` (required), plus optional `authDomain`, `storageBucket`, `messagingSenderId`, `appId`. |

`authDomain` and `storageBucket` auto-fill from `projectId` (e.g. `my-project.firebaseapp.com`).

On submit the config is validated and saved to `localStorage` under key `forgeERP_firebaseConfig`:

```js
function saveFirebaseConfig(cfg) {
  localStorage.setItem(FIREBASE_CONFIG_KEY, JSON.stringify(cfg));
}
```

If a config already exists in storage, step 1 is skipped — the user goes straight to step 2.

A **"Skip — use ERP without sync"** button lets the user bypass auth entirely and use the app offline-only.

### Step 2 — Google Sign-In

Triggers a Google OAuth popup via Firebase Auth:

```js
const provider = new firebase.auth.GoogleAuthProvider();
await _firebaseAuth.signInWithPopup(provider);
```

On success, `onAuthStateChanged` fires and the app transitions to the main shell. On failure, an error banner is shown (except for `auth/popup-closed-by-user` which is silently ignored).

---

## Auth State Listener

Registered once during `_doFirebaseInit()`:

```js
_firebaseAuth.onAuthStateChanged(user => {
  _currentUser = user;
  if (user) {
    // Authenticated
    firestoreDoc = firebase.firestore().collection('kanban-boards').doc('shared');
    scrumboardConnected = true;
    showApp();
    updateSidebarUser();
    if (DB.scrumboardSync.autoSync) startRealtimeListener();
  } else {
    // Signed out
    firestoreDoc = null;
    scrumboardConnected = false;
    stopRealtimeListener();
    updateSidebarUser();
  }
});
```

**When authenticated:**
- `_currentUser` is set (Google user object with `email`, `displayName`, `photoURL`, `uid`)
- Firestore references are initialized (`kanban-boards/shared`)
- The main app shell is displayed
- Sidebar shows user avatar/name + settings gear + sign-out button
- Real-time listener starts if `autoSync` is enabled

**When unauthenticated:**
- All Firestore references are cleared
- Real-time listener is stopped
- Sidebar shows "Not signed in" with a sign-in button

---

## Global State Variables

```js
const FIREBASE_CONFIG_KEY = 'forgeERP_firebaseConfig';
let _firebaseAuth = null;     // firebase.auth() instance
let _currentUser = null;      // Current Google user object (or null)
let firestoreDoc = null;      // Firestore DocumentReference for kanban-boards/shared
let scrumboardListener = null; // Firestore onSnapshot unsubscribe function
let scrumboardConnected = false;
```

---

## Sign-Out

Two code paths:

| Function | Behavior |
|----------|----------|
| `logoutAndReset()` | Signs out, clears all auth state, returns to the login screen. Used by the sidebar sign-out button. |
| `signOutFirebase()` | Signs out and clears auth state but stays on the current screen. Used internally. |

```js
async function logoutAndReset() {
  stopRealtimeListener();
  await _firebaseAuth.signOut();
  _currentUser = null;
  firestoreDoc = null;
  scrumboardConnected = false;
  _firebaseAuth = null;
  showLoginScreen();
}
```

---

## Firestore Security Rules

Defined in `firestore.rules`. Deploy with:

```bash
firebase deploy --only firestore:rules
```

### Rule Summary

| Collection | Read | Write | Create/Delete |
|-----------|------|-------|---------------|
| `kanban-boards/{boardId}` | Authenticated | Authenticated + must have `projects` key + < 1 MB | Admin only (`admin@forgeerp.nl`) |
| `retro-presence/{docId}` | Authenticated | Own doc only (`auth.uid == docId`) | — |
| `retro-nicknames/{docId}` | Authenticated | Authenticated | — |
| `lab-inventory/{docId}` | (no explicit rule — denied by default) | — | — |
| Everything else | Denied | Denied | Denied |

> **Note:** `lab-inventory` currently has no explicit rule in `firestore.rules`. Inventory sync operations will fail unless you add an allow rule. Example:
> ```
> match /lab-inventory/{docId} {
>   allow read, write: if request.auth != null;
> }
> ```

---

## Authenticated User in the UI

When signed in, the sidebar bottom section renders:

```
┌──────────────────────────────────────┐
│  [avatar]  Display Name       ⚙  ⎋  │
│            user@email.com            │
└──────────────────────────────────────┘
```

- **Avatar**: Google profile photo, or generated initials on a green badge
- **⚙ (settings gear)**: Opens the Power User Settings panel (project deletion, DB export/import, factory reset, etc.)
- **⎋ (sign-out)**: Calls `logoutAndReset()`

When **not** signed in:

```
┌──────────────────────────────────────┐
│  [?]  Not signed in             ⎆   │
│       Sync disabled                  │
└──────────────────────────────────────┘
```

---

## Power User Settings (Auth-Gated)

The settings panel is only accessible when `_currentUser` is set. It provides:

| Action | Description |
|--------|-------------|
| Delete Project | Remove a project + its BOM entries + sync mappings from the local DB |
| Delete Inventory Item | Remove a single item from local inventory |
| Purge Firestore Inventory | Batch-delete all docs in `/lab-inventory` on Firestore |
| Export DB | Download full local DB as timestamped JSON |
| Import DB | Upload a JSON file to overwrite local data |
| Reset Firebase Config | Clear stored credentials + sign out |
| Factory Reset | Wipe all `localStorage` (DB + config) and reload |

---

## Firebase Config Reference

See `firebase-config.example.js` (not loaded by the app):

```js
const FIREBASE_CONFIG = {
  apiKey:            '',                          // Required
  authDomain:        '<project>.firebaseapp.com',
  projectId:         '<project>',                 // Required
  storageBucket:     '<project>.appspot.com',
  messagingSenderId: '',
  appId:             ''
};
```

Obtain these values from:
**Firebase Console** → Project Settings → General → Your apps

---

## Key Files

| File | Purpose |
|------|---------|
| `index.html` | All auth logic (login screen HTML + JS functions) |
| `firebase-config.example.js` | Reference config template (not loaded by the app) |
| `firestore.rules` | Firestore security rules |
