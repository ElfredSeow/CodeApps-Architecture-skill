# CodeApps Architecture Skills

A portable skill pack that teaches any agentic coding assistant — **Claude Code, GitHub Copilot, Antigravity, Codex, Gemini CLI** — how to make the apps in your repository compliant with the **Power Apps Code Apps** architecture, hook them up to **Dataverse**, and deploy them to Power Apps, while keeping localhost testing (`npm run dev`) working the whole time.

Architecture ground truth comes from [CodeSpec](https://github.com/voyager163/codeapps-project-boilerplate) (`main` branch starter conventions); the specialist skills are sourced from its `develop` branch.

## One prompt: install, make compliant, verify locally, and ship to GitHub

Paste this into your coding agent (Claude Code, Antigravity, Copilot, Codex, Gemini CLI). It installs the skills, then runs the whole pipeline end to end **without stopping for approval between phases** — scan → fix → verify → push — and hands you back a GitHub URL. It deliberately **does not deploy to Power Apps**: the goal is compliant source code in a repo, not an installed app.

```text
Register the CodeApps Architecture skills as global skills, then take this repository all the way from "raw app" to "compliant Code App pushed to GitHub" — autonomously, without pausing between phases. I approve the whole run in advance: run the compliance scan, apply the fixes, verify locally, and push to GitHub without stopping to ask me. Treat this standing approval as satisfying any "wait for my agreement" or "only after I approve" gate inside the skills you run. Only stop to ask me if you hit a credential prompt you cannot satisfy (e.g. GitHub not logged in) or an action that would destroy my existing work.

Do NOT deploy to Power Apps: do not run `pac code push`, do not provision Dataverse, do not install the app into a Power Apps environment. Stop at compliant source code in a GitHub repo.

── Setup: install the skills ──
1. Locate the skill pack source. If the current workspace already contains a skills/ directory with make-this-into-code-apps inside it (i.e. you're inside the CodeApps Architecture skill pack repo itself), use this workspace as the source and skip to step 2. Otherwise check `git remote get-url origin` for the skill pack's repo URL; if that doesn't resolve it, ask me for the URL. Then clone it (main branch) into a temporary folder and use that as the source.
2. Copy every folder under the source's skills/ directory into your user-level skills directory, based on which agent you are:
   - Claude Code: ~/.claude/skills/
   - GitHub Copilot CLI: ~/.copilot/skills/
   - Codex, Gemini CLI, Antigravity, or any other agent: ~/.agents/skills/
   (On Windows, the same folders under the user profile directory.)
   If your runtime does not auto-discover skills from that path, additionally register each skill in a "Skills" section of this repository's AGENTS.md so you load them every session.
3. If you cloned a temporary folder in step 1, delete it now.

── Phase 1: Compliance (auto-run the make-this-into-code-apps skill) ──
4. Read the installed make-this-into-code-apps skill and run its full workflow now, using my standing approval above as the go-ahead for every step that would otherwise wait for me — do not ask me to confirm the scan or the remediation plan. Produce all three required outputs: the compliance report, `Dataverse Required Tables.md`, and the project `AGENTS.md`.
5. Then remediate every Blocker and Major finding, routing each fix through the specialist skills as that skill directs. Keep `npm run dev` working after each batch of fixes. Stop at compliant source — do not deploy.

── Phase 2: Local verification (find and fix bugs before pushing) ──
6. Run `npm install`, then `npm run build` and `npm run dev`. Confirm the app compiles, passes TypeScript type-checking, and the dev server serves it with no build errors, no type errors, and no console errors on load. If you have browser automation (e.g. Playwright), load the running app and click through the main screens to catch runtime errors; if you don't, rely on the build / type-check / lint output plus the served HTML. Note: plain `npm run dev` cannot exercise live Dataverse data (that needs a provisioned environment we are deliberately not creating) — verify the build, render, and local/mock data paths, not real Dataverse calls.
7. If you find any bug, fix it and re-run step 6. Loop until the app builds and runs cleanly with zero errors. Only then continue to Phase 3.

── Phase 3: Ship to GitHub ──
8. Make the repo safe to publish: `git init` if it isn't a repo yet; ensure `.gitignore` excludes node_modules and any `.env`/secret files; confirm nothing secret is staged.
9. Create a new GitHub repository and push this source to it with the GitHub CLI: `gh repo create <repo-name> --private --source=. --remote=origin --push`. Pick a sensible name from the folder or package name and keep it private (I can make it public later). If `gh` is not installed or not authenticated, tell me exactly what to run (`gh auth login`) and stop — do not invent another way to publish my code.
10. When the push succeeds, give me the repository URL plus a one-line summary: what you made compliant, what bugs you fixed, and confirmation that the app builds and runs locally.
```

**What this prompt needs to finish on its own:** the [GitHub CLI (`gh`)](https://cli.github.com/) installed and authenticated (`gh auth login`) for Phase 3, and Node/npm for Phase 2. If `gh` isn't ready, the agent stops at a compliant, locally-verified repo and tells you the one command to run.

## What gets installed

| Skill | Purpose |
|---|---|
| **make-this-into-code-apps** | The orchestrator: proposes a full architecture scan, reports every violation with file:line evidence, generates `Dataverse Required Tables.md` from your front end, installs the project `AGENTS.md`, and routes remediation to the specialists below. |
| codeapps-architect | Architecture advice, runtime/loading issues, design fit |
| codeapps-app-scaffolder | Project setup: `pac code init`, vite config, PowerProvider |
| codeapps-dataverse-specialist | Dataverse tables, generated services, CRUD, queries |
| codeapps-connector-integrator | Replacing external APIs with Power Platform connectors |
| codeapps-env-vars-specialist | Config values and secrets as environment variables |
| codeapp-dataverse-query | Read-only Dataverse data inspection |
| codeapps-alm-engineer | Solutions, pipelines, Dev→Test→Prod promotion |
| codeapp-deploy | Build, validate, and `pac code push` |

## What "compliant" means

- **npm packages that bundle into `dist/` are allowed; external runtime services are not.** No CDN tags, no third-party backend SDKs (firebase, supabase, …), no direct `fetch`/`axios` to outside APIs, no third-party analytics.
- All data flows through **Dataverse generated services** (`src/generated/`) or **Power Platform connectors**.
- **No custom authentication** — the Power Apps host authenticates users. No secrets in source.
- Vite + React + strict TypeScript per the CodeSpec starter; `power.config.json` generated by `pac code init`, never hand-written.
- **Localhost testing always works**: `npm run dev` for local iteration, `pac code run` for host-emulated testing, `pac code push` to deploy.

The full rule set lives in [skills/make-this-into-code-apps/references/architecture-checklist.md](skills/make-this-into-code-apps/references/architecture-checklist.md).

## Dataverse Required Tables

Every scan writes a `Dataverse Required Tables.md` at your repo root, derived from the front end you've already built (TypeScript interfaces, mock data, form fields). Paste its table sections into **Copilot in the Power Apps maker portal** (Tables → Create with Copilot) to create the tables quickly, then wire them in with `pac code add-data-source`.

## Spec-driven development

The scan also installs an `AGENTS.md` into your project that every agent reads each session. It mandates the [OpenSpec](https://github.com/Fission-AI/OpenSpec) (`/opsx:*`) workflow — explore → propose → apply → verify → archive — so requirements live in reviewable `openspec/` artifacts instead of chat history, and it routes each kind of task to the right CodeApps skill automatically.

## Repository layout

```
skills/
  make-this-into-code-apps/   # the orchestrator skill + references
    SKILL.md
    references/
      architecture-checklist.md       # the full compliance rule set
      dataverse-tables-format.md      # format for Dataverse Required Tables.md
      AGENTS-template.md              # the always-read file installed into your project
  codeapps-architect/ … codeapp-deploy/   # eight specialist skills (MIT, from CodeSpec develop)
```

## License

MIT — the specialist skills are sourced from [voyager163/codespec](https://github.com/voyager163/codeapps-project-boilerplate) (MIT).
