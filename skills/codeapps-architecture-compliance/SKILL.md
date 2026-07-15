---
name: codeapps-architecture-compliance
description: Use when a repository must be made compliant with Power Apps Code Apps architecture, when preparing an app for Dataverse or Power Apps deployment, when the user asks to scan, audit, or fix a Code App, or in the first session after this skill is installed in a project.
license: MIT
compatibility: Power Apps Code Apps (Vite + React/TS + @microsoft/power-apps).
metadata:
  author: manfred-siew
  version: "1.0"
---

# CodeApps Architecture Compliance

## Overview

Bring any app in the current repository into compliance with the Power Apps Code Apps architecture (the CodeSpec starter conventions: Vite + React + strict TypeScript + `@microsoft/power-apps`), so it can connect to Dataverse and deploy to Power Apps.

Core rule: **npm packages that bundle into `dist/` are allowed; external runtime services are not.** All data flows through Dataverse or Power Platform connectors. The Power Apps host owns authentication. Localhost testing (`npm run dev`) must keep working at all times.

## When to Use

- First session in a repository after installing this skill → propose the scan
- "Make this app deployable to Power Apps" / "hook this up to Dataverse"
- "Scan / audit / check my app's architecture"
- Before running `pac code push`

Not for: Canvas Apps YAML, Power Pages, or model-driven app customization.

## Workflow

Every scan produces the three Required Outputs in Steps 2–4. A scan that skips one is incomplete.

**Step 0 — Propose the scan.** Tell the user: "I'll run a full architecture compliance scan of this repository — read-only, no code changes — and produce a compliance report, a `Dataverse Required Tables.md` file, and a project `AGENTS.md`." Wait for agreement before Step 1.

**Step 1 — Scan (read-only).** Check every rule in [references/architecture-checklist.md](references/architecture-checklist.md). Cover at minimum:
- `package.json` dependencies and scripts
- `index.html` external `<script>` / `<link>` tags
- Grep source for `fetch(`, `axios`, `XMLHttpRequest`, `WebSocket`, `EventSource`, hardcoded `https://` origins
- Auth patterns: login endpoints, tokens in `localStorage`/`sessionStorage`, auth SDKs
- Backend-service SDKs (firebase, supabase, aws-amplify, etc.) and secrets in source
- `power.config.json`, `vite.config.ts` plugins, `src/generated/` services, TypeScript strictness

**Step 2 — Required Output 1: Compliance Report.** Present in chat as a table: `# | Severity | File:line | Rule | Finding | Proposed fix`. Severities: **Blocker** (breaks deploy or violates the architecture), **Major** (works locally but non-compliant), **Advisory**. Also list the rules that passed, so the user sees coverage, not just failures.

**Step 3 — Required Output 2: `Dataverse Required Tables.md`.** Derive entities from the front end the user has built — TypeScript interfaces/types, mock data, form fields, table columns — and write `Dataverse Required Tables.md` at the repo root using [references/dataverse-tables-format.md](references/dataverse-tables-format.md). This file is written on **every** scan, even when the data model is small or partially inferred; mark uncertain columns with `(verify)`. The user pastes it into Copilot in the Power Apps maker portal to create the tables.

**Step 4 — Required Output 3: project `AGENTS.md`.** If the repo has no `AGENTS.md`, create one from [references/AGENTS-template.md](references/AGENTS-template.md). If one exists, add only the missing sections (spec-driven workflow, skill routing, architecture rules). This is the file every agent reads on each session so spec-driven development and skill routing happen automatically.

**Step 5 — Remediate (only after the user approves the plan).** Fix Blockers first. Route work through the specialist skills:

| Task | Skill |
|---|---|
| Overall design / "does this fit the Code Apps model" | codeapps-architect |
| Project setup, vite config, PowerProvider, `pac code init` | codeapps-app-scaffolder |
| Dataverse tables, generated services, CRUD, queries | codeapps-dataverse-specialist |
| Replacing external APIs with Power Platform connectors | codeapps-connector-integrator |
| Config values, secrets → environment variables | codeapps-env-vars-specialist |
| Read-only Dataverse inspection during analysis | codeapp-dataverse-query |
| Solutions, pipelines, Dev→Test→Prod | codeapps-alm-engineer |
| Build + `pac code push` | codeapp-deploy |

**Step 6 — Verify localhost, then host.** `npm install && npm run dev` must serve the app locally; existing tests (`vitest`, Playwright) must still run. Only then use `pac code run` for host-emulated testing and `pac code push` to deploy.

## Quick Reference — top violations

| Violation | Compliant replacement |
|---|---|
| `fetch`/`axios` to an external API | Dataverse generated service or Power Platform connector |
| CDN `<script>`/`<link>` in `index.html` | npm package bundled by Vite; self-hosted fonts |
| Custom login, JWT in `localStorage` | Delete — the Power Apps host authenticates users |
| firebase / supabase / amplify SDKs | Dataverse + connectors |
| API keys or secrets in source | Environment variables (codeapps-env-vars-specialist) |
| Hand-written `power.config.json` | Generate with `pac code init` |

## Common Mistakes

| Mistake | Reality |
|---|---|
| "The diff is the documentation" — skipping the markdown outputs | `Dataverse Required Tables.md` and `AGENTS.md` are required deliverables of every scan, not optional reports. |
| Fixing code before presenting the report | The user approves the remediation plan first. Scan → report → approve → fix. |
| Removing npm packages to satisfy "no external dependencies" | Bundled packages are compliant. Only external **runtime services** (APIs, CDNs, auth, third-party backends) are violations. |
| Breaking `npm run dev` while remediating | Localhost testing is a compliance requirement — verify it after every remediation batch. |
| Inventing Dataverse schema from nothing | Derive tables from what the front end actually uses; mark inferred columns `(verify)`. |
