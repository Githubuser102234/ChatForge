# ChatNow — Global Chat

This repository contains a simple global chat app using Firebase Authentication and **Realtime Database**.
Files included:
- `index.html` — single-file demo with authentication, messaging, and admin controls.
- `firebase-rules.json` — Realtime Database rules to apply in your Firebase console.

## Features implemented
- Email/password sign up & sign in.
- Sending messages saved under `/messages`.
- Admin controls:
  - Delete any message.
  - Toggle **lock** (when locked, only admins can send messages).
  - Toggle **disable** (when disabled, only admins can read & write).
  - Ban/unban users (stored under `/bans/{uid}`).
  - Promote to admin via Realtime Database or the helper `window.__promoteToAdmin(uid)` (run from browser console).
- Admin verification is done two ways:
  1. Add admin UIDs to the `KNOWN_ADMIN_UIDS` array in `index.html` (quick local fallback).
  2. Add admin entries in Realtime Database under `/admins/{uid}` with value `true`. Example:
     ```
     {
       "admins": {
         "UID_FROM_FIREBASE": true
       }
     }
     ```

## Recommended Realtime Database structure
- `/messages/{messageId}`: { uid, userName, text, ts, isAdmin }
- `/admins/{uid}`: true
- `/bans/{uid}`: { by: adminUid, expires: epochMillis }
- `/config`: { locked: boolean, disabled: boolean }

## Security / Rules
Apply the rules in `firebase-rules.json` in your Realtime Database rules tab. They enforce:
- Only authenticated users can read messages (unless `disabled` is true).
- When `disabled` is true, only admins can read/write.
- When `locked` is true, only admins can write messages.
- Only admins may delete messages, modify `/config`, modify `/admins`, or write `/bans`.

**Important**: initial admin entries must be created from the Firebase Console (or using server-side privileged SDK) — until an admin exists, you can add UIDs to the local `KNOWN_ADMIN_UIDS` array to let specific accounts use admin UI.

## How to deploy
1. Copy `index.html` into a GitHub Pages repo (or host on static hosting).
2. Open Firebase Console -> Realtime Database -> Rules, and paste `firebase-rules.json`.
3. In Realtime Database -> Data, create initial `/admins/{uid}: true` for each admin UID (find UID under Authentication users).
4. Optionally, edit `KNOWN_ADMIN_UIDS` in `index.html` to add admin UIDs as a JS fallback.

## Notes & security considerations
- Never rely only on client-side checks for admin status — rules enforce server-side checks too.
- For production use consider creating admin accounts via server-side (Admin SDK) and use Firebase Custom Claims for more robust role checks.
- For added security, store fewer sensitive operations on client and use Cloud Functions with authenticated checks for moderation tasks.

