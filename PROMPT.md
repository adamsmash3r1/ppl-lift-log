# Build instructions for Claude Code

This folder holds a working single-page workout tracker (index.html, manifest.json,
sw.js, icons). It's a push/pull/legs lift logger with a rest timer. It currently saves
everything to the browser's localStorage. Turn it into a self-hosted web app on my home
lab, with all workout data stored on this server.

## My environment
Inspect this machine and figure these out yourself, then confirm them with me:
- OS, whether Docker / docker compose is installed
- How I reach it remotely (Tailscale / WireGuard / OpenVPN). I only access it over the VPN or LAN.
- Any existing reverse proxy (Caddy / Nginx Proxy Manager / Traefik) and ports already in use
- A free port to use

## Requirements
1. **Local/VPN only.** It must never be reachable from the public internet. Bind only to
   the LAN/VPN interface (or 127.0.0.1 behind my reverse proxy). No port forwarding, no
   public tunnels. Single user (me), so no accounts needed. Add an optional shared
   access token via env var, off by default.
2. **Server-side storage.** Small backend (Python FastAPI or Node, your choice, keep
   dependencies minimal) with SQLite in a mounted volume. REST API for:
   - completed sessions: list (newest first), create, delete
   - the in-progress workout draft: get/put, so I can start on my phone and see it on
     another device, and a refresh never loses sets
   - export all data as JSON / import JSON. Import must accept the app's existing backup
     format {app, v, history:[...]} and merge by session id without duplicates.
3. **Keep the frontend as-is.** Same look, program, timer, and behavior. Only replace
   the localStorage persistence layer with API calls. Keep the existing data shapes
   (session objects with id, date, day, variant, unit, minutes, exercises[]).
4. **Works when the VPN drops at the gym.** Keep a localStorage cache and an outbox
   queue. Writes go to the server when it's reachable, otherwise queue locally and sync
   automatically when the connection returns. Show a small sync indicator (synced /
   offline, N pending). The server wins on conflicts for completed sessions; the draft
   uses last-write-wins by timestamp.
5. **Mobile first.** Must work well in iPhone Safari and as an "Add to Home Screen" app.
   Keep the manifest, icons, and service worker. The service worker should cache the app
   shell only, never API responses.
6. **HTTPS.** Service workers and screen wake lock need a secure context, so plain http
   over an IP won't fully work on iPhone. Pick the simplest HTTPS option for my setup
   (e.g. `tailscale serve` / Tailscale certs if I use Tailscale, or Caddy with an
   internal CA, giving me steps to trust the root cert on my iPhone). Explain the choice.
7. **Deploy.** Dockerfile + docker-compose.yml (or a systemd unit if no Docker), with a
   restart policy and a healthcheck. Add a nightly SQLite backup that keeps the last 14
   copies in a folder I can point my existing backups at.

## Process
- First look at the existing files and this machine, then show me a short plan
  (stack, file layout, how HTTPS and VPN-only binding will work) before writing code.
- Test the API with a quick script, and check that the app loads at a phone-sized
  viewport.
- Finish with a README: how to start/stop, the URL to open on my phone, how to add it to
  the Home Screen, where the data and backups live, and how to import my old backup JSON.
- This repo is public on GitHub. Never commit data, backups, tokens, or .env files; add
  a .gitignore for them.
