# AGENTS.md: autonomous build instructions

You are a coding agent. Your job: build **imsg-windows**, River's Windows app for sending and receiving iPhone messages (iMessage + SMS, including group chats), and keep working autonomously until every milestone below passes its acceptance tests. Do not stop at a partial result.

## What you are building

A Windows desktop app that talks to a **BlueBubbles server** running on River's MacBook Pro. The server owns all Apple integration (it reads the macOS Messages database and sends through Messages). Your app is a client of its documented API:

- REST API + webhooks: https://docs.bluebubbles.app/server/developer-guides/rest-api-and-webhooks
- Realtime events over Socket.IO: `new-message`, `updated-message`, `typing-indicator`
- Auth: every request carries the server password as the `guid` query parameter
- Reference third-party client showing the endpoint set: https://github.com/mdunlap/WearBubbles

Full background and why the architecture is this way: see the sister repo `imsg-windows-research`.

## Non-negotiable product requirements

1. Send and receive iMessage + SMS, including group chats
2. Works from any network (server is reachable via its Cloudflare URL), no same-Wi-Fi requirement
3. Stays logged in forever: server URL + password entered (or QR-scanned) once, stored securely, auto-reconnect with backoff
4. Chat list with groups, message history with paging, attachments, tapbacks
5. Windows toast notifications for incoming messages when the app is unfocused

## Tech stack

Flutter desktop (Windows target). Reasons: the official BlueBubbles app is Flutter, so its open source code is a working reference for every API call and UI pattern; one codebase could later target other platforms. If you hit a hard blocker in Flutter, Tauri (Rust + webview) is the approved fallback. Store the server password with platform secure storage (flutter_secure_storage). Use socket_io_client for realtime.

## Operating rules

- Work milestone by milestone, in order. After each milestone, run its acceptance tests, then check off its boxes in this file and commit.
- Never mark a box done unless its acceptance test actually passed. No aspirational checkmarks.
- If a step needs a human (scanning the server QR, entering the password, pairing a test chat), do everything else first, then write exactly what you need in BLOCKED.md and continue with any unblocked work.
- Keep a running PROGRESS.md: one line per work session (date, what changed, test results).
- Manual end-to-end checks that need the real server go in E2E.md with steps and expected results, so a human can run them.

## Milestones and acceptance tests

### M0: Server up (human-assisted)
- [ ] BlueBubbles server installed on the MacBook, Messages signed in, Full Disk Access granted, strong server password set
- [ ] Cloudflare connection enabled in the server; note the server URL
- [ ] MacBook set to never sleep while on power
- Acceptance: from the Windows PC on a different network, `GET {server_url}/api/v1/ping?guid={password}` returns success

### M1: Connect and stay connected
- [ ] Settings screen: server URL + password, stored in secure storage
- [ ] QR import: parse a QR code containing the server URL and password (JSON or query-string form)
- [ ] On launch, connect without asking again; on failure, retry with exponential backoff and show status
- Acceptance: enter credentials once, fully quit the app, relaunch: chat list loads with no prompt. Kill the network for 30 s, restore it: the app reconnects on its own

### M2: Chat list
- [ ] `POST /api/v1/chat/query` with contacts resolved, sorted by most recent message
- [ ] Group chats show participant names and a group indicator
- Acceptance: the list matches the server's own chat list, including at least one real group chat

### M3: History
- [ ] `POST /api/v1/message/query` with pagination (older pages on scroll-up)
- [ ] Image attachments render inline via `GET /api/v1/attachment/{guid}/download`
- Acceptance: scroll up loads older messages; a photo from the iPhone thread displays in the app

### M4: Send
- [ ] `POST /api/v1/message/text` into an existing chat; new-chat creation by contact/number
- [ ] Optimistic UI with error state and retry
- Acceptance: a message sent from the app appears on the iPhone's Messages app and in the app thread; SMS fallback works to a green-bubble contact

### M5: Realtime
- [ ] Socket.IO: `new-message`, `updated-message`, `typing-indicator`
- [ ] Windows toast notification when unfocused; click focuses the chat
- Acceptance: a message sent from the iPhone appears in the app within 2 seconds; with the app unfocused, a toast appears

### M6: Groups and reactions
- [ ] Group thread view with sender labels; send to a group
- [ ] Tapback reactions via `POST /api/v1/message/react`
- Acceptance: hold a real 3+ person group conversation from the PC alone; a tapback sent from the app shows on the iPhone

### M7: Ship it
- [ ] Dark theme, unread badges, per-chat mute
- [ ] Launch on Windows startup (optional toggle)
- [ ] Packaged installer (MSIX or equivalent)
- Acceptance: clean install on a fresh Windows profile pairs by QR and reaches M5 behavior in under 5 minutes

## Definition of done

Every box above checked, all acceptance tests passing, PROGRESS.md and E2E.md current, and a tagged v1.0.0 release with the installer attached.
