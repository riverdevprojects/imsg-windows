# imsg-windows

River's own Windows app for iPhone messages: send and receive iMessage + SMS (including group chats) from the PC, from any network, paired once by QR code and logged in for good.

## How it works

The app is a client of a BlueBubbles server running on River's MacBook Pro. The Mac owns the Apple side (it reads and sends through macOS Messages); this app talks to the server's REST + Socket.IO API over a Cloudflare URL, so it works on any Wi-Fi and stays signed in.

```
iPhone  <== iMessage/SMS ==>  MacBook (BlueBubbles server)  <== Cloudflare ==>  this app
```

Background and the alternatives that were rejected (pure Bluetooth bridge, Phone Link): see the `imsg-windows-research` repo.

## Build

This repo is designed to be built by an autonomous coding agent. Point your agent at **AGENTS.md** and let it run: it contains the tech stack, milestones, checklists, and acceptance tests through v1.0.0.

Prerequisites before M1 can be tested: the BlueBubbles server installed on the Mac (https://bluebubbles.app/install/) with its Cloudflare URL and password noted.
