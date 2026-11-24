# DucBox privacy notice

Last updated: 2026-11-24 Â· Applies to DucBox 0.7.1

DucBox is a local-first Chrome extension, a native Windows host, and an optional
desktop app. This notice describes only behavior that can be checked in this
repository. Every storage
location and network endpoint below names the code that implements it.

## Summary

- DucBox has no developer-operated server, account system, analytics, telemetry,
  advertising, or remote configuration.
- Your data stays on this machine unless **you** connect a third-party service
  (Google, an AI provider, a self-hosted relay) or open a platform page.
- Secrets are encrypted with Windows DPAPI, or with a passphrase you supply for
  backups, shares, and Tier-2 credentials.

## What is stored locally

### Vault database

`apps/host/src/config/paths.ts` resolves the vault root to `%LOCALAPPDATA%\DucBox`
(or `DUCBOX_HOME` when set). `vault.db` is SQLite. Its tables
(`apps/host/src/db/schema.ts`) hold:

- `accounts`, `assets`, `entities` â€” account and page/channel metadata you enter.
- `sessions`, `secrets` â€” captured cookies; cookie values are DPAPI ciphertext.
- `credentials` â€” optional Tier-2 login/password/TOTP entries (scrypt +
  AES-256-GCM, passphrase never stored).
- `vault_items` â€” provider API keys and agent secrets (values DPAPI ciphertext;
  the UI lists metadata only).
- `tool_accounts` â€” AI-tool account names/plans imported read-only from your own
  `cockpit-cli`; no tokens are copied.
- `calendar_accounts`, `calendar_events`, `reminders`, `goals`, `goal_tasks` â€”
  calendar, reminder, and goal data, including any Google data you sync.
- `projects`, `project_accounts`, `project_processes` â€” local project metadata
  under the scan root you configure.
- `ideas`, `notebook_sources`, `notebook_insights`, `profiles`,
  `profile_versions`, `runbooks`, `account_checks` â€” local workspace content.
- `agents`, `grants`, `approval_requests`, `runtime_sessions` â€” agent tokens,
  scopes, and approvals.
- `audit`, `settings` â€” a hash-chained audit log and your preferences.
- `day_columns`, `day_event_layout` â€” day-board layout.

### Other files under the vault root

- `profiles/` â€” browser profiles used for account sessions.
- `logs/host.log` â€” timestamped operational messages: startup arguments, request
  method and id, and error text (`apps/host/src/log.ts`,
  `apps/host/src/modes/native.ts`). Secret values are not written by design.
- `ideas/` â€” screenshot/idea files.
- `host/com.ducbox.host.json`, `extension-id.txt` â€” native-messaging manifest and
  the extension id.
- `host/serve.json` â€” the desktop host's loopback port and per-launch token;
  created on start and deleted on exit.

### Browser extension storage

`chrome.storage.local` stores small UI state (last attention marker, notified
reminders). `chrome.storage.session` stores short-lived UI state such as the
autosave suggestion and its suppression window. Neither holds vault secrets.

### Files outside the vault root

- Backups: encrypted `.ducbox` files written to the path you choose
  (scrypt + AES-256-GCM, `apps/host/src/vault/portable.ts`).
- Share packages: encrypted JSON written to the path you choose.
- Cookie exports: text files at the path you choose, written `0600` where the OS
  honours it.
- Codex switch: writes `%USERPROFILE%\.codex\auth.json` and keeps the previous
  file under `%LOCALAPPDATA%\DucBox\backups\codex`
  (`apps/host/src/core/tool-switch.ts`). Nothing is written until you press the
  switch action.

## Network connections

DucBox makes no background calls to the developer. Connections happen only in
these cases:

The desktop app also runs a local-only bridge: `ducbox-host --serve` listens on
`127.0.0.1`, writes a per-launch `host/serve.json` (port, token, pid), and the
extension's native host proxies to it. The token is readable only by the same
Windows user, and the server never leaves loopback
(`apps/host/src/modes/serve.ts`, `apps/desktop/src-tauri/src/main.rs`).

1. **Local browser control.** The host talks to Chrome over the DevTools
   protocol on `127.0.0.1` (`apps/host/src/chrome/cdp.ts`,
   `apps/host/src/chrome/launcher.ts`) to open the platform page you requested.
2. **Platform pages you open** â€” Facebook, Instagram, Threads, TikTok,
   YouTube/Google, Reddit, Pinterest, X, LinkedIn â€” load in your browser under
   their own policies so you can capture or use your own session.
3. **Google (optional).** If you connect Google, DucBox calls
   `accounts.google.com` for OAuth and the Google Calendar/Gmail APIs for the
   scopes you grant (`apps/extension/lib/google-oauth.ts`,
   `apps/host/src/core/google-calendar.ts`). Tokens are stored as DPAPI secrets.
4. **AI provider (optional).** If you configure an AI provider and use an AI
   action, the host sends the prompt to the base URL you configured (defaults:
   `https://api.openai.com/v1`, `http://localhost:11434/v1`) with the API key you
   stored (`apps/host/src/core/assistant-ai.ts`). Without a key, the assistant
   uses local rules only.
5. **Self-hosted relay (optional).** If you configure a relay URL, packages are
   sent only when you run a relay push/pull against it
   (`apps/host/src/relay/client.ts`). The developer does not operate a relay.
6. **Google Translate link.** The â€œDá»‹châ€ action opens a Google Translate URL in
   your browser only when you click it (`apps/extension/lib/idea-actions.ts`).

## What is not collected

No analytics, crash reporting, advertising identifiers, device fingerprints,
contact lists, or usage telemetry are sent anywhere. There is no remote config,
no A/B service, and no developer cloud account.

## Retention and deletion

- Uninstall the host and delete `%LOCALAPPDATA%\DucBox` to remove the vault,
  provider keys, tool-account records, profiles, logs, and backups.
- If you used the Codex switch, restore or delete `%USERPROFILE%\.codex\auth.json`
  separately; DucBox only writes it on your action.
- Remove the extension from `chrome://extensions` to delete its extension
  storage.
- Backups, shares, and exports live where you saved them; delete them there.

## Contact

Use the support contact published with the store listing.

## Code references

`apps/host/src/config/paths.ts`, `apps/host/src/db/schema.ts`,
`apps/host/src/vault/` (DPAPI, Tier-2, portable envelopes),
`apps/host/src/core/` (vault items, tool accounts, tool switch, backup, sharing,
cookie access, assistant AI, Google calendar), `apps/host/src/relay/`,
`apps/host/src/chrome/`, `apps/extension/lib/` (OAuth, storage helpers).
