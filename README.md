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

**Portable, Windows build of Hermes Desktop** compiled as the upstream **lite
client** (`HERMES_DESKTOP_VARIANT=light`) — GUI only, with **no local Hermes
Agent**. It connects to a remote gateway (`hermes serve`) on another machine, so
your desktop stays clean while the agent lives wherever your gateway runs.

> **One release per upstream release.** The build runs only when
> `NousResearch/hermes-agent` publishes a version that is not published here yet
> — not on every scheduled run.

---

## Download

Grab the latest from **[Releases](https://github.com/rubengmez/hermes-desktop-portable/releases)**:

- `Hermes-desktop-portable-win-x64-<version>.zip` (e.g.
  `Hermes-desktop-portable-win-x64-0.22.5.zip`) — extract anywhere and run the
  `Hermes*.exe` inside (the lite variant names it `Hermes Light.exe`). No
  installer, no local agent, nothing to install.

The version in the file name is the upstream version that binary was compiled
from — check it before running, and compare it with **Help → About**.

> ⚠️ **Unsigned build.** This binary is **not code-signed** (free builds can't be).
> On first run Windows SmartScreen may warn — choose **More info → Run anyway**.
> Only install binaries you understand and trust; verify the source on the
> upstream repository if you're unsure.

---

## How it works

This is a **build automation** repo. It doesn't fork or patch Hermes — it runs
[GitHub Actions](.github/workflows/hermes-desktop-windows.yml) that:

1. **Detects** the latest published release of `NousResearch/hermes-agent` and the
   exact commit its tag points at.
2. **Skips everything** when this repo already publishes that release tag at that
   commit (so a daily schedule costs a two-second check, not a 40-minute build).
3. Otherwise **checks out that exact upstream release tag** (not a fork, not `main`).
4. Installs the Node/JS workspace dependencies (`npm ci`) and **compiles the
   desktop GUI** as the `light` variant — the local agent is not bundled.
5. **Stamps the real version** into the package
   (`-c.extraMetadata.version=<version>`), because both `package.json` files in
   the upstream tree carry a `0.0.0` placeholder: an unstamped build reports
   `0.0.0` in **Help → About**, and the release would look older than it is.
6. **Publishes** a GitHub Release tagged with the exact upstream tag, plus the
   zipped portable app.

Each release therefore corresponds 1:1 to an upstream Hermes release. Nothing is
added and nothing is removed except the local agent payload.

**Triggers:** a daily check (05:05 UTC — builds only on a new upstream version) or
manual `Run workflow` (which rebuilds only if the version is missing, unless you
tick `force`).

---

## Connect to a remote gateway

The backend is a `hermes serve` process on another machine (VPS / home server /
tailnet). See the [Hermes docs on remote backends](https://hermes-agent.nousresearch.com/docs/user-guide/desktop#connecting-to-a-remote-backend).

### Option A — in the app

Open **Settings → Gateways → Remote gateway**, enter your backend URL
(`http://host:9119`) and sign in.

### Option B — environment variables

```powershell
$env:HERMES_DESKTOP_REMOTE_URL = "https://your-gateway:9119"
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

## Which version does the app show?

**Help → About** shows the **client's** version first (the number stamped at build
time, i.e. the zip's version), then a separate card for the remote **backend**,
which can be a different release — the two are updated independently.

> Note for the previous (non-lite) builds: those were stamped `payload:
> bootstrap`, and for that payload the About panel prefers the *gateway's*
> version over the build's own. That is why a freshly downloaded zip could
> display an older number than the release it came from. The `light` variant
> resolves `displayVersion → baseVersion → package version → gateway`, so the
> client looks like the release it was built from.

---

## Updating

Because this is an unsigned, community build, it **can't self-update in-app**: the
Hermes Desktop updater targets the official Nous release feed, not third-party
builds. To update:

1. Download the newest zip from **Releases** (the file name carries the version).
2. Replace your `win-unpacked` folder.
3. Done.

---

## Legal & attribution

- **Hermes Agent** is MIT-licensed — upstream: [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) · docs: <https://hermes-agent.nousresearch.com>.
- **"Hermes"** and the **Hermes logo** are assets of their respective owners.
- This repo's own files (workflow, README, LICENSE) are MIT. The produced binary
  is the upstream MIT-licensed Hermes build, **recompiled, not modified** (the
  only build-time override is the version stamp).
- Report Hermes bugs upstream, not here — this repo only automates the build.
