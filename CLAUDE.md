# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Communication Style

Be direct and concise. Provide technical explanations only when requested or when a logic error occurs.

---

## Git & GitHub

Commits are authorized — proceed without asking. Group related logic into single commits using Conventional Commits:

- `feat:` new feature
- `fix:` bug fix
- `refactor:` restructuring without behavior change
- `docs:` documentation only

Do not commit minor incremental changes or typos individually — wait until there is a meaningful logical unit of work.

Commit message format:
- Use the imperative mood: `feat: add login screen`, not `Added login screen`
- Be specific: `fix: clear AsyncStorage on JWT expiry` beats `fix: bug`
- One-line subject is enough for small changes; add a blank line + body for anything non-obvious

```bash
git add <specific files>
git commit -m "type: subject"
git push
```

Never use `git add .` blindly — exclude `.env`, `node_modules`, and build artifacts.

---

## Execution Guardrails

If a task or command fails twice in a row, stop immediately. Explain why it likely failed and wait for input before attempting a third time. Do not loop on broken approaches.

---

## Code Quality

Prioritize clean, modular code. Every new script must include a brief comment block at the top:

```
// What this file does (1–2 sentences)
// Dependencies: list packages/modules this file relies on
```

---

## Environment Management

Use a `.env` file for all sensitive data (API keys, credentials, connection strings). Ensure `.env` is listed in `.gitignore` before the first commit in any project. Never hardcode secrets in source files.

---

## Progress

Update this section at the end of every session. Record what was completed, what is in progress, and the immediate next steps.

_(No sessions recorded yet.)_

---

## Repository Layout

This is a personal code workspace with three independent projects:

| Folder | What it is |
|--------|-----------|
| `flappy-bird/` | Standalone browser game — single `index.html`, no build step |
| `IntelliJCS1027/` | CS coursework (assignments, labs) — Java via IntelliJ |
| `wfnproject/` | Full-stack mobile dating/matching app (React Native + Node.js) |

---

## wfnproject

The main active project. Two sub-packages that must be run independently.

### WFN-Project-Server (Node.js / Express)

**Setup** — requires a `.env` file in the server root:
```
MONGODB_URI=<connection string from MongoDB Atlas "Main Cluster">
PORT=3000
```
MongoDB credentials: username = lowercase name, password = `123`.

**Run:**
```bash
cd wfnproject/WFN-Project-Server
npm install
node server.js
```

**Architecture:**
- `server.js` — HTTP server + Socket.io for real-time messaging. Creates rooms keyed by `[senderId, recipientId].sort().join('-')`.
- `src/app.js` — Express app. Active routes: `/api/users`, `/api/chat`, `/api/messages-matches`. (Message and match routes are commented out.)
- `src/config/db.js` — MongoDB connection via native driver (`MongoClient`). Note: `Users.js` model also calls `mongoose.connect` directly — there are two separate connection paths (native driver + Mongoose).
- `src/Models/Users/Users.js` — User document references four sub-documents via ObjectId refs: `BasicUserInfo`, `UserProfile`, `UserConnectedAccounts`, `UserPreferences`.

### WFN-Project-Client (React Native / Expo)

**Run:**
```bash
cd wfnproject/WFN-Project-Client
npm install
npx expo start --dev-client   # or: npm start
```
For Android/iOS: `npm run android` / `npm run ios`. For web: `npm run web`.

**Architecture:**
- `App.js` — Root. Wraps everything in `AuthProvider`. Renders `<Tabs>` when `userToken` exists, otherwise `<AuthStackNavigator>`.
- `navigation/AuthContext.js` — JWT token stored in AsyncStorage. `signIn(token)` / `signOut()`. Auto-login on app start is currently commented out.
- `navigation/AuthenticationNav.js` — Unauthenticated stack: `LoginScreen → MainInfo → BasicUserInfo → ProfileInfo → Preferences`.
- `navigation/tabs.js` — Authenticated bottom tabs: Home, Messages (matches+messages), Profile, Help.
- `screens/` — One file per screen; API calls use `axios`.
- `components/` — Reusable form inputs (picker, slider, text input, date picker, gradient button).

**API base URL** is configured inside individual screens (no central config file).
