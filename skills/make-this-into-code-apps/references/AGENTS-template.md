# AGENTS.md — Power Apps Code Apps project

<!-- Installed by make-this-into-code-apps. Every AI agent working in this
repository reads this file at the start of every session and follows it. -->

This repository is a **Power Apps Code App** (Vite + React + strict TypeScript + `@microsoft/power-apps`). It deploys to Power Apps and uses Dataverse / Power Platform connectors for all data.

## 1. Spec-driven development (always)

Before implementing any meaningful change, use the OpenSpec workflow — do not code straight from chat:

- Explore/propose first: `/opsx:explore`, `/opsx:new`, `/opsx:ff`, or `/opsx:propose`
- Keep requirements in `openspec/` artifacts, not only in conversation history
- Implement with `/opsx:apply`, verify with `/opsx:verify`, archive with `/opsx:archive`

If OpenSpec is not initialized in this repo, run `/opsx:onboard` (or ask the user to install OpenSpec) before large changes.

## 2. Skill routing (always)

When a task matches a row below, load and follow that skill before acting:

| Task | Skill |
|---|---|
| Architecture questions, runtime/loading issues, design fit | codeapps-architect |
| New project setup, vite config, PowerProvider, `pac code init` | codeapps-app-scaffolder |
| Dataverse tables, generated services, CRUD, queries | codeapps-dataverse-specialist |
| External API → Power Platform connector integration | codeapps-connector-integrator |
| Config values and secrets → environment variables | codeapps-env-vars-specialist |
| Read-only Dataverse data inspection | codeapp-dataverse-query |
| Solutions, pipelines, environment promotion | codeapps-alm-engineer |
| Build and deploy (`pac code push`) | codeapp-deploy |
| Compliance scan / audit / pre-deploy check | make-this-into-code-apps |

## 3. Architecture rules (non-negotiable)

- All runtime data access goes through generated services in `src/generated/` or Power Platform connectors — never direct `fetch`/`axios` to external APIs
- No external runtime services: no CDN tags in `index.html`, no third-party backend SDKs (firebase, supabase, …), no third-party analytics
- No custom authentication — the Power Apps host authenticates users
- No secrets in source; use environment variables
- npm packages that bundle into `dist/` are fine
- `power.config.json` is CLI-generated (`pac code init`) — never hand-edit
- Keep TypeScript strict; fix type errors before completing a task

## 4. Verification (before claiming done)

- `npm run dev` serves the app on localhost — this must always keep working
- `npm run build` and tests pass; lint is clean
- Host-emulated test with `pac code run` before any `pac code push`
