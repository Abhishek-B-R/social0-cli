# AGENTS.md — Social0 CLI

Guidance for AI agents (and humans) using the Social0 CLI.

> **Prefer the CLI when you have a shell.** Install [`social0`](https://www.npmjs.com/package/social0) (`npx social0` / `social0 login`) and run commands like `social0 accounts`, `social0 publish`, `social0 status`. Use [MCP](https://github.com/Abhishek-B-R/social0-mcp) only when the host has no reliable shell (Claude.ai / ChatGPT remote connectors, or stdio-only MCP hosts).
>
> Skill for ClawHub / OpenClaw (same content in both public repos): [`skills/social0/SKILL.md`](./skills/social0/SKILL.md) · also in [social0-mcp](https://github.com/Abhishek-B-R/social0-mcp/tree/main/skills/social0).

## What this is

- **Package:** [`social0`](https://www.npmjs.com/package/social0) (binary `social0`)
- **Auth:** `social0 login` with `sk_live_…`, or `SOCIAL0_API_KEY` for CI/headless
- **API:** thin client for `https://api.social0.app/v1`
- **Not included:** Completing platform OAuth in-chat (dashboard / `social0 accounts connect` opens the link)
- **Sibling tool:** MCP package [`@social0/mcp`](https://www.npmjs.com/package/@social0/mcp) — for connector-only hosts

## Setup checklist

1. User has Social0 account + connected platforms at https://social0.app/dashboard/connections  
2. API key created at https://social0.app/dashboard/api-keys (`sk_live_…`)  
3. `npm i -g social0` (or `npx social0 …`) then `social0 login`  
4. Verify with `social0 whoami` and `social0 accounts`

## Command reference

Use `--json` when you need machine-readable output.

### Auth & diagnostics

| Command | Notes |
|---------|--------|
| `social0 login` | Interactive masked prompt, or `echo "sk_live_…" \| social0 login` |
| `social0 logout` | Clear stored credentials |
| `social0 whoami` | User name, email, plan, masked key |
| `social0 doctor` | Auth + API connectivity |
| `social0 passphrase status\|set\|remove` | Optional local-file encryption when OS keychain is unavailable |

### Accounts

| Command | Notes |
|---------|--------|
| `social0 accounts` | Numeric IDs + platform + handle. **Call first** when accounts are ambiguous |
| `social0 accounts connect <platform>` | Opens dashboard connect flow |
| `social0 accounts disconnect <id>` | Disconnect by numeric ID |

Prefer **platform names** (`twitter`, `linkedin`) when one account per network; use numeric IDs when multiple accounts share a platform. IDs can reorder after reconnects — re-list if unsure.

### Posts & drafts

| Command | Notes |
|---------|--------|
| `social0 post create` | Interactive wizard, or `--content` / `--platform` |
| `social0 post list` / `show` / `edit` / `delete` | Prefer **full UUID** when short IDs fail |
| `social0 drafts` / `drafts publish` / `drafts schedule` / `drafts delete` | Draft workflow |
| `social0 post init` | Sample JSON template |

### Publish & schedule

| Command | Notes |
|---------|--------|
| `social0 publish --content "…" --platform twitter linkedin` | One-shot |
| `social0 publish <draft-id>` / `latest` | Publish existing draft |
| `social0 publish launch.md` | Markdown + front matter |
| `echo "…" \| social0 publish` | Stdin |
| `social0 schedule <id> --time "tomorrow 9am"` | Natural language times (uses config timezone) |
| `social0 status <tracking-id>` | Poll until terminal — **always** after multi-platform / video |

### Media

| Command | Notes |
|---------|--------|
| `social0 upload <file>` / `social0 upload ./images/*` | Returns media IDs for posts |

### Config

| Command | Notes |
|---------|--------|
| `social0 config` / `get` / `set` | e.g. `defaultTimezone`, `outputFormat` |
| `social0 link` | Project-local `.social0` |

---

## Quick agent examples

```bash
social0 accounts
social0 publish --content "Launching today!" --platform twitter linkedin
social0 upload ./hero.png
social0 schedule <draft-id> --time "tomorrow 9am"
social0 status <tracking-id>
```

---

## Platform names

Canonical: `linkedin`, `facebook`, `instagram`, `youtube`, `pinterest`, `tiktok`, `twitter_x`, `threads`, `bluesky`.

Aliases: `x`/`twitter` → `twitter_x`; `ig` → `instagram`; `fb` → `facebook`; `yt` → `youtube`; `li` → `linkedin`; `bsky` → `bluesky`; `pin` → `pinterest`.

---

## Recommended agent behavior

1. Prefer **CLI** whenever a shell is available; fall back to MCP tools only if not.  
2. Call `social0 accounts` before the first publish in a session.  
3. Prefer one-shot `social0 publish` / `schedule` for simple requests; use drafts when the user wants to edit first.  
4. After any publish, return `tracking_id` and poll with `social0 status` until terminal.  
5. On `partial`, summarize which platforms failed and why.  
6. Never invent account IDs or tracking IDs.  
7. Never claim the CLI can finish Instagram/Facebook OAuth — send users to the dashboard (or `accounts connect`).  
8. Confirm before live publish unless the user clearly says “post now” / “publish immediately”.  
9. Schedules: confirm timezone; CLI accepts natural language (`tomorrow 9am`).

---

## MCP fallback (no shell)

When the host only supports MCP connectors:

- Remote URL: `https://mcp.social0.app/mcp` (OAuth)
- Local: `npx -y @social0/mcp` with `SOCIAL0_API_KEY`
- Full MCP tool parameter tables: [social0-mcp AGENTS.md](https://github.com/Abhishek-B-R/social0-mcp/blob/main/AGENTS.md)

---

## Out of scope

- Completing platform OAuth inside the chat  
- Analytics / inbox / social listening  
- Dashboard-only features not on `/v1`

Use https://api.social0.app/docs for full REST schemas.

---

## Troubleshooting (humans)

### Not logged in / `401`

```bash
social0 login
# or CI:
export SOCIAL0_API_KEY=sk_live_…
```

Keys start with `sk_live_` (legacy `s0_live_` still works).

### `command not found: social0`

```bash
npm install -g social0
# or
npx social0 …
```

Needs [Node.js 20+](https://nodejs.org/).

### No connected account / ambiguous platform

Connect at https://social0.app/dashboard/connections. Re-run `social0 accounts` and pass numeric IDs when multiple accounts share a platform.

### One platform failed, others succeeded

Expected for multi-platform jobs. Poll `social0 status <tracking-id>`.

### Auth / connectivity looks wrong

```bash
social0 doctor
```

---

## Local development

```bash
git clone https://github.com/Abhishek-B-R/social0-cli.git
cd social0-cli
npm install
npm run build
npm run dev -- accounts
```
