# Build instructions for Claude Code

This repo has a working single-page workout tracker (index.html, manifest.json, sw.js,
icons). It's a push/pull/legs lift logger with a rest timer. It currently saves everything
to the browser's localStorage. Turn it into a self-hosted web app on this home-lab server,
with all workout data stored on the server.

## Environment
Inspect this machine first (OS, Docker/compose, existing reverse proxy, which VPN is in
use: Tailscale / WireGuard / OpenVPN) and confirm with me before choosing a stack. I reach
the server only over my VPN or LAN.

## Requirements
1. **Local/VPN only.** Must never be reachable from the public internet. Bind only to the
   LAN/VPN interface (or 127.0.0.1 behind a reverse proxy). No port forwarding, no public
   tunnels. Single user, no accounts. Optional shared access token via env var, off by default.
2. **Server-side storage.** Small backend (Python FastAPI or Node, minimal dependencies)
   with SQLite in a mounted volume/data dir. REST API for:
   - completed sessions: list (newest first), create, delete
   - the in-progress workout draft: get/put, so a workout started on one device shows on
     another and a refresh never loses sets
   - export all data as JSON / import JSON. Import must accept the app's existing backup
     format {app, v, history:[...]} and merge by session id without duplicates.
3. **Keep the frontend as-is.** Same look, program, timer, and behavior. Only replace the
   localStorage persistence layer (LS / loadHistory / saveSession / deleteSession /
   saveDraft in index.html) with API calls. Keep the existing data shapes (session objects
   with id, date, day, variant, unit, minutes, exercises[]).
4. **Works when the VPN drops at the gym.** Keep a localStorage cache plus an outbox queue.
   Writes go to the server when reachable, otherwise queue locally and sync automatically
   when the connection returns. Show a small sync indicator (synced / offline, N pending).
   Server wins on conflicts for completed sessions; draft is last-write-wins by timestamp.
5. **Mobile first.** Must work well in iPhone Safari and as an "Add to Home Screen" app.
   Keep the manifest, icons, and service worker. The service worker caches the app shell
   only, never API responses. Bump its cache name when the frontend changes.
6. **HTTPS.** Service workers and screen wake lock need a secure context, so plain http over
   an IP won't fully work on iPhone. Pick the simplest HTTPS option for this setup (e.g.
   `tailscale serve` / Tailscale certs if Tailscale is used, or Caddy with an internal CA,
   with steps to trust the root cert on an iPhone). Explain the choice.
7. **Deploy.** Dockerfile + docker-compose.yml (or a systemd unit if no Docker), with a
   restart policy and a healthcheck. Nightly SQLite backup keeping the last 14 copies in a
   folder I can point other backups at.

## Process
- Look at the existing files and this machine, then show me a short plan (stack, file
  layout, how HTTPS and VPN-only binding will work) before writing code.
- Test the API with a quick script and check the app at a phone-sized viewport.
- Commit your work to this repo as you go.
- Finish with a README: start/stop, the URL to open on my phone, adding it to the Home
  Screen, where data and backups live, and how to import my old backup JSON.
