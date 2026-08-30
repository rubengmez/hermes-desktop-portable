# Hermes Desktop Portable

> [!WARNING]
> **These are community-built binaries of the original open-source project,
> not official Hermes releases.**
>
> This repo is **not affiliated with Nous Research**. It does **not** own,
> modify, or distribute the Hermes source code. It only **compiles** the
> upstream [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
> repository (MIT-licensed) into ready-to-download Windows binaries. For the
> official app and docs, go to <https://hermes-agent.nousresearch.com>.

**Portable, Windows build of Hermes Desktop** compiled as a **lite client** — GUI
only, with **no local Hermes Agent** installed. It connects to a remote gateway
(`hermes serve`) on another machine, so your desktop stays clean while the agent
lives wherever your gateway runs.

> **Rebuilt every week** from the latest `NousResearch/hermes-agent@main`, so each
> release stays current with upstream.

---

## Download

Grab the latest from **[Releases](https://github.com/rubengmez/hermes-desktop-portable/releases)**:

- `Hermes-desktop-portable-win-x64.zip` — extract anywhere, run `Hermes.exe`.
  No installer, no local agent, nothing to install.

> ⚠️ **Unsigned build.** This binary is **not code-signed** (free builds can't be).
> On first run Windows SmartScreen may warn — choose **More info → Run anyway**.
> Only install binaries you understand and trust; verify the source on the
> upstream repository if you're unsure.

---

## How it works

This is a **build automation** repo. It doesn't fork or patch Hermes — it runs
[GitHub Actions](.github/workflows/hermes-desktop-windows.yml) that:

1. **Checks out the official upstream** `NousResearch/hermes-agent@main` (not a fork).
2. Installs the Node/JS workspace dependencies (`npm ci`).
3. **Compiles only the desktop GUI** (`npm run pack` in `apps/desktop`) as a
   *lite client* — the agent runtime is intentionally excluded.
4. **Publishes** the result as a versioned GitHub Release + a 30-day Actions artifact.

So each release is exactly the upstream Hermes Desktop source, built fresh on the
latest `main`, with nothing added and nothing removed except the local agent
(effectively making it a "client-only" build).

**Triggers:** manual (`Run workflow`), every Sunday 06:00 UTC, or a push of a `v*` tag.

---

## Connect to a remote gateway

The backend is a `hermes serve` process on another machine (VPS / home server /
tailnet). See the [Hermes docs on remote backends](https://hermes-agent.nousresearch.com/docs/user-guide/desktop#connecting-to-a-remote-backend).

### Option A — in the app

Open **Settings → Gateways → Remote gateway**, enter your backend URL
(`http://host:9119`) and sign in.

### Option B — environment variables

```powershell
$env:HERMES_DESKTOP_IGNORE_EXISTING = "1"   # don't probe/install a local agent
$env:HERMES_DESKTOP_REMOTE_URL    = "https://your-gateway:9119"
.\Hermes.exe
```

On the server, run the backend (bind to a non-loopback address so the auth gate
engages):

```bash
# in ~/.hermes/.env on the server
HERMES_DASHBOARD_BASIC_AUTH_USERNAME=admin
HERMES_DASHBOARD_BASIC_AUTH_PASSWORD=choose-a-strong-password
HERMES_DASHBOARD_BASIC_AUTH_SECRET="$(openssl rand -base64 32)"

hermes serve --host 0.0.0.0 --port 9119
```

Keep that process running (systemd/tmux). Beyond a trusted LAN, put it behind VPN
(Tailscale) or use the OAuth / Nous Portal provider.

---

## Updating

Because this is an unsigned, no-agent build, it **can't self-update in-app**: the
Hermes Desktop updater targets the official Nous release feed, not third-party
builds. To update:

1. Download the newest zip from **Releases**.
2. Replace your `win-unpacked` folder / `Hermes.exe`.
3. Done. (You can also trigger a fresh build any time via **Actions → Run workflow**.)

---

## Legal & attribution

- **Hermes Agent** is MIT-licensed — upstream: [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) · docs: <https://hermes-agent.nousresearch.com>.
- **"Hermes"** and the **Hermes logo** are assets of their respective owners.
- This repo's own files (workflow, README, LICENSE) are MIT. The produced binary
  is the upstream MIT-licensed Hermes build, **recompiled, not modified**.
- Report Hermes bugs upstream, not here — this repo only automates the build.