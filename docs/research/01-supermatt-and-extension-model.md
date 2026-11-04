# SuperMatt and the Claude Code extension model: what a product-team add-on can build on

Research date: 2026-11-04. Sources: the installed SuperMatt plugin (v0.22.0) at `/root/.claude/plugins/synced/.../eb9bad6a-.../` (cited below as `SM/`). Shallow clones taken 2026-11-04: `svyatov/supermatt` @ `58dabac` (v0.23.0), `mattpocock/skills` @ `4ae2317` (v1.3.2), `obra/superpowers` @ `22f0b46` (v7.0.2), `EveryInc/compound-engineering-plugin` @ `799ac9a` (plugin v3.35.3), and `agentskills/agentskills` @ `83021e2`. Claude Code docs were fetched as Markdown from `code.claude.com/docs/en/*.md` on the same day.

Labels: **[F]** is a fact with a citation. **[I]** is an inference. **[R]** is a recommendation.

---

## 1. SuperMatt's lifecycle and the artifacts each phase produces

### 1.1 The main flow (idea → ship)

[F] `SM/README.md` and `SM/skills/ask-supermatt/SKILL.md` describe one main flow plus several on-ramps:

| Phase | Skill | Artifact, format, location |
|---|---|---|
| Sharpen the idea | `grill-with-docs` (the `grilling` and `domain-modeling` skills combined) | Decisions stay in the conversation. Domain terms go to `GLOSSARY.md`, or `GLOSSARY-MAP.md` plus one glossary per context. Hard-to-reverse decisions become ADRs in `docs/adr/NNNN-*.md` (`SM/skills/setup-supermatt-skills/domain.md`). |
| Optional detour | `handoff` → `prototype` → `handoff` | Throwaway code. A **logic** prototype is a single HTML state-machine demo; a **UI** prototype is several variants on one route, switched by a URL parameter. The prototype is committed to a `prototype/<name>` branch, and the issue that asked the question gets a pointer to it (`SM/skills/prototype/SKILL.md` rule 6). |
| Spec | `to-spec` | A Markdown spec with fixed sections: **Problem Statement, Solution, User Stories** ("As an <actor>, I want…, so that…", described as "a LONG, numbered list"), **Implementation Decisions** (modules, interfaces, contracts; no file paths), **Testing Decisions** (seams agreed with the user), **Out of Scope, Further Notes**. A fresh-context subagent reviews the draft before it is published. It is published to the issue tracker with the `ready-for-agent` label. On a local tracker the spec lives at `.scratch/<feature>/spec.md` (`SM/skills/to-spec/SKILL.md`, `issue-tracker-local.md`). |
| Tickets | `to-tickets` | Tracer-bullet vertical slices, each with its **blocking edges**. Locally: `.scratch/<feature>/issues/NN-<slug>.md`, with `Status:`, `Blocked by:`, **What to build**, **Seams** and acceptance checkboxes. On GitHub or GitLab: one issue per ticket using the tracker's native blocking links, with **Parent / What to build / Acceptance criteria / Seams** sections (`SM/skills/to-tickets/SKILL.md`). |
| Build | `implement` (one ticket per session) or `implement-spec` (parallel worktrees, one integration branch) | Small commits that reference the issue. Each build runs `tdd`, then `qa` in a fresh subagent, then `code-review` on three axes (Standards, Spec, Adversarial) with P0–P3 findings. Issues stay open until the merge (`SM/skills/implement/SKILL.md`). |
| Ship | `fix-findings`, `refactor`, `ship-pr` (`babysit-pr` and `recover-ci` were added upstream in v0.23.0) | A PR body shaped by `pr`, CI repair, squash merge, and closed issues. |
| Learn | `retro` | Ranked suggestions for changes to the agent's environment. |

[F] There are four on-ramps. `ideate` writes `docs/ideation/YYYY-MM-DD-<topic>.md`. `triage` moves incoming issues through 7 canonical labels (`bug`, `enhancement`, `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`). `diagnosing-bugs` handles hard bugs. `wayfinder` charts a "map" issue labelled `wayfinder:map` whose child **decision tickets** are typed `research`/`prototype`/`grilling`/`task`; locally this is `.scratch/<effort>/map.md` plus a `decisions/` folder. The map hands off to `to-spec`. `orchestrate` runs implement → ship unattended in Herdr panes (`SM/skills/orchestrate/SKILL.md`).

[F] The productivity skills relevant to product work are `grill-me`, `to-questionnaire` (writes an async questionnaire for a stakeholder), `what-would-you-do`, and `jury`. `jury` is a 3- or 5-seat blind-vote panel with fixed *lenses*: Pre-mortem, First principles, Cost, Outsider, Evidence (`SM/skills/jury/SKILL.md`).

### 1.2 Product work vs engineering work

[I] Only `grill-with-docs`, `ideate`, `wayfinder` and `to-questionnaire` are product-shaped, along with `jury`, the UI branch of `prototype`, and the Problem/Solution/User Stories half of the spec. `wayfinder` describes itself as "domain-agnostic: engineering work, course content" (`SM/skills/wayfinder/SKILL.md`). Everything from `to-tickets` onward is engineering.

### 1.3 Gaps for non-engineering roles

[F] SuperMatt ships **no `agents/` directory, no hooks, no output styles and no commands**. It is 39 skills in a flat `skills/` tree (`SM/` file listing; `SM/AGENTS.md` line 1). It has no notion of a role or persona. The closest things are the jury lenses and the "actor" in user stories.

[I] What each role lacks:

- **PM:** no PRD, goals, success metrics or non-goals section (the spec template has Out of Scope but no metrics), no prioritization or roadmap, no competitive analysis.
- **Customer persona:** no persona artifact, interview synthesis, or "voice of customer" reviewer. `to-questionnaire` is the nearest hook.
- **PMM:** nothing for positioning, messaging, launch or release notes. CE's `ce-strategy` is closer (section 5).
- **TPM:** no milestones, RAID or risk log, status reporting, or cross-team dependencies. Ticket blocking edges and `orchestrate` cover execution only.
- **SDM:** no estimation, capacity planning, staffing, or release-readiness gate beyond the "ready to ship" rule in `implement`.

---

## 2. Per-repo configuration and extension points

[F] `setup-supermatt-skills` writes the following (`SM/skills/setup-supermatt-skills/SKILL.md`; README lines 71–79):

- An `## Agent skills` block in whichever of `CLAUDE.md` or `AGENTS.md` already exists.
- `docs/agents/issue-tracker.md`: GitHub, GitLab, local Markdown, or **"Other"**. For "Other" the file is written from scratch "under the same headings as the GitHub template", covering create, read, list, comment, labels, close, blocking, and the wayfinding operations.
- `docs/agents/triage-labels.md`: maps each canonical role to your label string.
- `docs/agents/domain.md`: single-context or multi-context glossary layout.
- `docs/agents/cross-model.json`: `{mode: auto|host-only|claude|codex|cursor|omp, claude:{model,effort}, codex:{…}, cursor:{model}, omp:{…}}`. The schema is strict: "Reject unknown or missing keys", and an invalid file falls back to `host-only` (`SM/references/harnesses.md`).

[F] Across SKILL.md files these paths are referenced as follows: `issue-tracker.md` 19×, `cross-model.json` 11×, `domain.md` 7×, `triage-labels.md` 7× (grep over `SM/skills/*/SKILL.md`).

[F] **No extension points are documented.** There is no hook, template-override, reviewer-registration or phase-registration mechanism in the README, `AGENTS.md`, `CONTRIBUTING.md` or `references/`. In particular, the spec template is hard-coded inline in `to-spec/SKILL.md` inside `<spec-template>` tags.

[I] Several mechanisms work as extension points even though they are not documented as such:

1. **The "Other" tracker is a prose adapter.** Any tracker works if `issue-tracker.md` describes its operations.
2. **Repo-documented standards.** `code-review`'s Standards axis applies "whatever the repo documents" on top of a fixed smell baseline (`SM/skills/code-review/STANDARDS.md`). Role-specific rules written into repo docs therefore get enforced.
3. **`GLOSSARY.md` and ADRs** are read by nearly every skill.
4. **Skill composition by name.** A skill says "load and follow the `<name>` skill through the host's skill mechanism". However, "Only model-invoked skills can be dependencies" (`SM/.agents/invocation.md`).
5. **Docs prose.** Skills read `docs/agents/*.md` as instructions, so extra prose there reaches them. That is fragile, and re-running setup diffs these files against their templates.

`cross-model.json` cannot carry extra keys.

[F] SuperMatt splits into 26 **user-invoked** skills (`disable-model-invocation: true`) and 13 **model-invoked** ones:

- **User-invoked:** ask-supermatt, commit, fix-findings, grill-me, grill-with-docs, handoff, ideate, implement, implement-spec, improve-*, jury, orchestrate, refactor, retro, setup, ship-pr, teach, to-questionnaire, to-spec, to-tickets, triage, wait-what, wayfinder, what-would-you-do.
- **Model-invoked:** code-review, codebase-design, dependency-vetting, diagnosing-bugs, domain-modeling, grilling, pr, prototype, qa, research, resolving-merge-conflicts, tdd, writing-for-agents.

The split was taken by grepping frontmatter in `SM/skills/*/SKILL.md`. It matters for add-ons (see section 4).

---

## 3. mattpocock/skills and how SuperMatt diverges

[F] **Structure.** The repo has buckets `skills/engineering/` (20), `productivity/` (7), `in-progress/` (8, e.g. `chief-of-staff`, `software-factory`), `misc/` (frozen) and `deprecated/`. The `plugin.json` (`mattpocock-skills` v1.3.2) lists 27 shipped skills explicitly. Releases are managed with Changesets (`.changeset/`). It is installable as `mattpocock-skills@claude-plugins-official`, through the Codex, Copilot and Gemini CLIs, or via `npx skills` (README).

[F] **Release cadence** (tag commit dates): v1.0.0/1.0.1 2026-06-17, v1.1.0 07-08, v1.2.0/1.2.2 08-05, v1.2.3 08-06, v1.3.0 09-29, v1.3.1 10-04, v1.3.2 10-29. That is roughly monthly minors with patch bursts.

[F] **License.** MIT, © 2026 Matt Pocock.

[F] **Philosophy** (README, SCOPE.md):

- "Approaches like GSD, BMAD, and Spec-Kit try to help by owning the process… they take away your control."
- The skills are "small, easy to adapt, and composable… Hack around with them. Make them your own."
- SCOPE.md: config options, harness-specific branches and per-person tweaks "belong in your own `CLAUDE.md` / `AGENTS.md`, or in a fork".
- `.out-of-scope/new-skills.md`: new-skill proposals are closed. "If you can compose the behaviour from existing skills, it doesn't get a new skill… that composition belongs in your own prompts or your own skills. You're very welcome to publish a skill yourself."

[F] **Divergence.** SuperMatt was "forked from mattpocock/skills at `c55ee46`" with "selected v1.3.1 updates" (`SM/README.md` lines 28 and 132). It:

- **Renames** `ask-matt`→`ask-supermatt`, `improve-codebase-architecture`→`improve-architecture`, `setup-matt-pocock-skills`→`setup-supermatt-skills`, `CONTEXT.md`→`GLOSSARY.md`.
- **Flattens** the buckets into `skills/<name>`, because Codex rejects nested skills.
- **Drops** `misc/` and most of `in-progress/`.
- **Adds** `ideate`, `qa`, `orchestrate`, `refactor`, `improve-tests`, `improve-file-structure`, `commit`, `ship-pr`, `fix-findings`, `what-would-you-do`, `jury`, `dependency-vetting`, plus `babysit-pr` and `recover-ci` in v0.23.0.
- **Strengthens** existing skills: a third review axis, a fresh-context spec check, mutation checks in TDD, and peer review across four hosts.
- **Borrows** from CE (`code-review`, `diagnosing-bugs`, `to-spec`, `ideate`) and Superpowers (`tdd`, `diagnosing-bugs`), with MIT `NOTICE` files kept per skill.

[F] **Upstream tracking is manual.** Changelog entries say "Imported from mattpocock/skills" (0.2.0) and "adapts upstream v1.3.1's whole-spec workflow" (0.18.0). No sync script or upstream remote is described in `scripts/`, `AGENTS.md` or `CONTRIBUTING.md`.

[I] The upstream v1.3.2 changes (2026-10-29), such as `code-review` reading `CODING_STANDARDS.md`, do not appear in SuperMatt's Unreleased notes. SuperMatt therefore tracks upstream selectively and with lag.

[F] **SuperMatt cadence.** v0.1.0 shipped 2026-09-23 and v0.22.0 shipped 2026-10-17: about 30 releases in 25 days. v0.23.0 followed on 2026-10-18. The upstream `AGENTS.md` now says the version is bumped "only [on] an explicit release request" while ordinary fixes accumulate under Unreleased. The installed copy here is 0.22.0, one release behind; see auto-update defaults in section 4.

---

## 4. Claude Code extension mechanics for "consume upstream unmodified + add-on"

All citations below are `code.claude.com/docs/en/<page>`, fetched 2026-11-04.

### Plugins, dependencies, versions

- [F] **Dependencies exist** (`plugins/dependencies`, `plugins-reference#dependencies`).
  - `plugin.json` takes `"dependencies": ["name" | "name@marketplace" | {name, marketplace, version}]`. Installing or enabling a plugin installs or enables its dependencies.
  - A dependency-only plugin is a valid "bundle".
  - Version ranges (`~2.1.0`, `^2.0`) resolve against git tags named **`<plugin-name>--v<version>`**.
  - A plugin listed by relative path, with no matching tag, uses the marketplace's current copy, and the range is "checked when the plugin loads".
  - Cross-marketplace dependencies need `allowCrossMarketplaceDependenciesOn` in the *root* marketplace, unless the user already installed the dependency at the same scope.
- [I] SuperMatt tags releases `v<version>` (`SM/AGENTS.md`, release.sh) and lists itself with `"source": "./"`. A `{name:"supermatt", version:"^0.23"}` constraint would therefore fall back to a load-time check rather than tag resolution. An unconstrained dependency works.
- [F] **Pinning via your own marketplace.** A marketplace entry can point at `{"source":"github","repo":"svyatov/supermatt","ref":…,"sha":…}` (`plugins/marketplace-reference`). You can re-list the unmodified upstream plugin at a pinned commit in your own marketplace, alongside your add-on.
- [F] **Updates.** Setting `version` "pins the plugin to that version until you change it". Background auto-update is **on by default only for Anthropic official marketplaces** (e.g. `claude-plugins-official`) and **off for every other marketplace** unless toggled in `/plugin` or set with `autoUpdate` (`plugins/loading#which-marketplaces-and-plugins-auto-update`).
- [F] **Namespacing.** Plugin skills are `/plugin-name:skill-name`. "A plugin skill and a skill at any of the locations above: both load". `skillOverrides` (`"on"|"name-only"|"user-invocable-only"|"off"`) "is not [applied to] plugin skills. Manage those through `/plugin` instead" (`skills#override-skill-visibility-from-settings`). An add-on therefore **cannot shadow or hide an individual SuperMatt skill**. It can only add its own skills next to them.
- [F] **What a plugin can contain** (`plugins-reference#standard-layout`): `skills/`, `agents/`, `hooks/hooks.json`, `.mcp.json`, `output-styles/`, `workflows/`, `monitors/`, `bin/`, and `settings.json`. The plugin `settings.json` supports only the `agent` and `subagentStatusLine` keys, so a plugin *can* set a default main-thread agent. "A `CLAUDE.md` at the plugin root isn't loaded as context".

### Skills and the Agent Skills standard

- [F] "Claude Code skills follow the Agent Skills open standard". The spec's portable frontmatter is `name`, `description`, `license`, `compatibility`, `metadata`, and `allowed-tools`. Claude Code-only extensions include `disable-model-invocation`, `user-invocable`, `context: fork`, `agent`, `model`, `effort`, `hooks` and `paths` (`skills`; `agentskills/agentskills/docs/specification.mdx`).
- [F] The spec repo lists 46 clients in `docs/snippets/clients.jsx`, among them Claude Code, Claude, ChatGPT & Codex, Cursor, GitHub Copilot, VS Code, Gemini CLI, OpenCode, Goose, Amp, Junie, Kiro, Roo Code, Factory, pi, OpenHands, Letta, TRAE and Tabnine. The spec code is Apache-2.0 and the docs are CC-BY-4.0. (agentskills.io itself was unreachable through this session's proxy; the GitHub source was used.)
- [F] **Key constraint.** With `disable-model-invocation: true`, "Claude can't invoke the skill on its own… If Claude tries anyway, Claude Code blocks the call". Such skills also "can't [be] preload[ed]" into subagents (`skills#control-who-invokes-a-skill`, `sub-agents#preload-skills-into-subagents`). A user writing `/name` mid-sentence grants permission for that message.
- [I] An add-on skill or agent **cannot programmatically chain** `supermatt:to-spec`, `to-tickets`, `implement`, `grill-with-docs` or `wayfinder`. It can call the 13 model-invoked ones, such as `supermatt:grilling`, `domain-modeling`, `prototype`, `research`, `qa`, `tdd` and `code-review`. For the flow skills, the add-on must either hand control back to the user ("now run `/supermatt:to-spec`") or emit artifacts in SuperMatt's formats itself. SuperMatt's own `orchestrate` gets around the restriction by typing commands into worker sessions as the user, through Herdr.

### Subagents (role definitions)

- [F] Subagents are defined in `.claude/agents/*.md` or a plugin's `agents/`. Plugin agents have the lowest priority, and a same-name project or user agent wins.
- [F] Frontmatter fields are `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills` (full content of up to 32 skills preloaded), `mcpServers`, `hooks`, `memory` (`user|project|local` → `.claude/agent-memory/<name>/`), `background`, `effort`, `isolation: worktree`, `color`, `initialPrompt`, and `omitClaudeMd`.
- [F] **Plugin subagents ignore `permissionMode`, `mcpServers`, `hooks` and `initialPrompt`.** Names can't contain `:`. Nesting is allowed up to 3 layers by default (`sub-agents`).

### Agent teams

- [F] Agent teams exist but are **experimental** and off by default; enable them with `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` (`agent-teams`).
  - A lead session and teammates share a task list and message each other directly.
  - Teammates can be spawned "using the <X> agent type", which reuses subagent definitions: their `tools`, `model` and body.
  - However, "Claude Code doesn't apply the definition's `skills` to a teammate".
  - Limits: one team per session, no nested teams, no resumption of in-process teammates, a fixed lead, and interactive sessions only (no `-p`).
- [I] Teams are a natural fit for a "PM / TPM / SDM / Customer debate". Because they are experimental and don't preload skills, role behaviour has to live in the agent body or in model-invoked skills.

### Hooks, output styles, CLAUDE.md

- [F] **`UserPromptExpansion`** fires "when a user-typed command expands into a prompt". It matches on `command_name`, receives `command_source: "plugin"`, and can return `additionalContext` or block. The docs' own example is a hook that "append[s] the team's review checklist" to a review skill (`hooks#userpromptexpansion`). `PreToolUse` with matcher `Skill` covers calls Claude makes itself. Plugins ship hooks in `hooks/hooks.json`.
- [I] This is the cleanest way to **augment an unmodified SuperMatt skill**. For example, when `/supermatt:to-spec` runs, inject "also include Success Metrics and a Positioning section; run the `pm-review` agent before publishing". The coupling is soft: it relies on prompt text, not on an API.
- [F] **Output styles** replace the system prompt's role and tone. Plugins can ship `output-styles/`, and `force-for-plugin: true` applies one automatically (`output-styles`). [I] This is too blunt for multiple roles in one session.
- [F] **CLAUDE.md layering:** managed policy → `~/.claude/CLAUDE.md` → `./CLAUDE.md` or `./.claude/CLAUDE.md` → `CLAUDE.local.md`. The files are *concatenated, not overridden*. Subdirectory files load on demand, `@path` imports work up to 4 hops deep, and path-scoped rules can go in `.claude/rules/`. `AGENTS.md` is also supported (`memory`). SuperMatt writes its config block into whichever of these files exists.

---

## 5. Superpowers and Compound Engineering

### Superpowers (`obra/superpowers`, v7.0.2, 2026-11-03, MIT © 2025 Jesse Vincent)

- [F] **Workflow** (README "The Basic Workflow"): brainstorming → using-git-worktrees → writing-plans (tasks of 2–5 minutes with exact paths and code) → subagent-driven-development or executing-plans → test-driven-development → requesting-code-review → finishing-a-development-branch. There are 15 skills in total.
- [F] **Artifacts:** `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` and `docs/superpowers/plans/YYYY-MM-DD-<feature>.md` (`skills/brainstorming/SKILL.md:246`, `skills/writing-plans/SKILL.md:16`).
- [F] **Roles:** only engineering sub-roles, defined as prompt templates inside skills: `implementer-prompt.md`, `task-reviewer-prompt.md` and `code-reviewer.md`. There are no product personas.
- [F] **Extension:**
  - A `SessionStart` hook bootstraps `using-superpowers` (`hooks/hooks.json`).
  - "User instructions (CLAUDE.md, AGENTS.md…) take precedence over skills" (`using-superpowers/SKILL.md:65`).
  - Personal skills live in `~/.claude/skills/`.
  - The project "[doesn't] generally accept contributions of new skills".

### Compound Engineering (`EveryInc/compound-engineering-plugin`, plugin.json v3.35.3, MIT © 2025 Every)

- [F] **Loop:** `ce-brainstorm` → `ce-plan` (one plan artifact that grows from requirements into an implementation plan, under `docs/plans/`) → `ce-work` → `ce-simplify-code` → `ce-code-review` → `ce-compound` (learnings in `docs/solutions/`). `lfg` runs the loop hands-off. There are 37 skills (README).
- [F] **Product coverage:** `ce-strategy` maintains `STRATEGY.md` (who it serves, metrics, tracks), which `ce-ideate`, `ce-brainstorm`, `ce-plan`, `ce-product-pulse` and `ce-dogfood` read. `ce-product-pulse` produces time-windowed product signal reports.
- [F] **Personas:**
  - 16 code-review personas, e.g. security, correctness, api-contract and data-migration (`skills/ce-code-review/references/personas/`).
  - 8 doc-review personas, including **product-lens**, **design-lens** and **scope-guardian**. Each is activated by signals in the document (`skills/ce-doc-review/references/persona-selection.md`).
  - These are "Specialist prompt assets… not an externally exposed plugin component" (`CONCEPTS.md`).
- [F] **Extension:**
  - Per-repo config lives in `.compound-engineering/config.yaml`, with `docs_root` to relocate artifacts.
  - **Compound Packs** (experimental) are declared folders of rule files with `title` and `applies_when` frontmatter. Brainstorm and plan "ground" in them, review enforces them, and every use is cited as `(pack: <id>, <path>)`. Packs can be local or ref-pinned git repos (`docs/guides/packs.md`).
- [I] Of the four projects, CE has the only documented, data-driven extension point, and its product-lens review and strategy doc are the nearest prior art for PM and PMM roles.

---

## 6. Licenses

| Project | License | Copyright | Redistribute or fork? |
|---|---|---|---|
| svyatov/supermatt | MIT (`SM/LICENSE`) | 2026 Matt Pocock; 2026 Leonid Svyatov | Yes, keeping both notices. Per-skill `NOTICE` files carry third-party MIT notices for CE, Superpowers and others, and must travel with the skills that have them. |
| mattpocock/skills | MIT | 2026 Matt Pocock | Yes. |
| obra/superpowers | MIT | 2025 Jesse Vincent | Yes. |
| EveryInc/compound-engineering-plugin | MIT | 2025 Every | Yes. |
| agentskills/agentskills | Apache-2.0 (code), CC-BY-4.0 (docs) | | Yes, with attribution and NOTICE rules. |
| Claude Code docs | Not an open license (not verified here) | | [I] Cite them; don't copy them. |

[F] SuperMatt's README also credits MIT-licensed `software-design`, `show-me`, awesome-copilot, `reducing-entropy` and ECC (`SM/THIRD-PARTY-NOTICES.md`).

---

## Implications for building role add-ons on top of SuperMatt

Constraints that shape every option ([F] unless marked):

1. No SuperMatt extension API exists, and its spec and ticket templates are inline in its SKILL.md files.
2. `skillOverrides` can't touch plugin skills, and plugin namespacing means you can add skills next to SuperMatt's but can't replace them.
3. The 26 user-invoked SuperMatt skills can't be called by your agents or skills, while the 13 model-invoked ones can.
4. Third-party marketplaces don't auto-update by default.
5. Upstream churn is high: about 30 releases in October 2026, plus repeated skill rewrites.
6. Upstream mattpocock explicitly tells people to put compositions "in your own skills" or a fork.

### Option A: hard fork of SuperMatt

Fork it, rename the plugin (e.g. `productmatt`), edit `to-spec`'s template, add role skills, and edit `ask-supermatt`.

- **Pros:** full control. You can change the spec template, insert phases, make flow skills model-invocable, and rewire the router.
- **Cons:** you lose updates. Each upstream release must be merged by hand into heavily edited prose. SuperMatt itself already lags its own upstream this way. MIT allows the fork.
- [R] Choose this only if you must change SuperMatt's core contracts, such as the spec format or the gates in `implement`.

### Option B: separate add-on plugin that depends on SuperMatt (recommended starting point)

You publish a marketplace containing two entries:

- `supermatt`, re-listed **unmodified** via `{"source":"github","repo":"svyatov/supermatt","sha":"<pinned>"}`.
- Your `product-team` plugin with `"dependencies":["supermatt"]`.

Both live in the same marketplace, so no cross-marketplace allowlist is needed. Your plugin contains:

- **`agents/`** with `product-manager.md`, `tpm.md`, `customer.md`, `sdm.md` and `pmm.md`. Each has a `tools` allowlist, a `model`, `memory: project`, and `skills:` preloading your role skills plus model-invoked SuperMatt skills (`supermatt:grilling`, `supermatt:domain-modeling`, `supermatt:research`, `supermatt:prototype`).
- **Role skills** (Agent Skills spec frontmatter only, so they port to Codex and Cursor). Examples: `prd-review`, `persona-interview`, `positioning-brief`, `program-plan`, `release-readiness`. They read and write SuperMatt's artifacts: the spec issue or `.scratch/<feature>/spec.md`, `GLOSSARY.md`, ADRs, and the `docs/agents/issue-tracker.md` operations.
- **`hooks/hooks.json`** with `UserPromptExpansion` matchers on `supermatt:to-spec`, `supermatt:to-tickets` and `supermatt:implement`. These inject role addenda as `additionalContext`, for example "before publishing, dispatch `product-team:customer` and `product-team:pmm` reviewers; add Success Metrics and Launch sections under Further Notes".
- **A router skill** (`ask-product-team`) mapping phases to roles. It tells the user when to type the next `/supermatt:*` command, because those flow skills can't be auto-invoked.

How each part handles upstream updates:

- **Pros:** SuperMatt stays byte-for-byte upstream. Updating means bumping one `sha` (or turning on auto-update), and the add-on versions independently. It also matches mattpocock's "compose in your own skills" guidance.
- **Cons:**
  1. The coupling is semantic: hooks and role skills depend on SuperMatt's file paths, section names and skill names. A rename (SuperMatt deprecates for one release before removing, per `AGENTS.md`) breaks you quietly, so pin and re-test on each bump.
  2. You can't change SuperMatt's spec template, only append to or post-process its output.
  3. The user still has to drive the flow skills.
  4. Version-range constraints don't resolve against SuperMatt's `v*` tags, so pin with `sha` in your marketplace instead.

### Option C: per-repo overlay, no plugin

Put `.claude/agents/*.md`, `.claude/skills/*`, `.claude/settings.json` hooks, CLAUDE.md sections and a GLOSSARY.md in each target repo, next to the installed SuperMatt plugin.

- **Pros:** zero packaging. Project agents outrank plugin agents, and it's easy to iterate.
- **Cons:** every repo gets its own copy and the copies drift. Better as a prototyping stage before Option B.
- [R] Use this to prototype roles in this repository, then promote them to Option B.

### Option D: "skills by name" only, with no hooks or dependencies

A standalone plugin whose skills tell the user or model to "load the `supermatt:<x>` skill", with the same runtime constraints as Option B.

- **Pros:** the simplest option.
- **Cons:** no install-time guarantee that SuperMatt is present, and no automatic augmentation of SuperMatt's phases.

### Further recommendations

- [R] Model role interaction with **subagents first** and treat **agent teams** as an optional experimental mode. Teams don't apply `skills:`, so put role behaviour in the agent body.
- [R] Store role artifacts next to SuperMatt's: `docs/product/` (PRD addenda, personas, positioning) and `docs/program/` (milestones, RAID). Record cross-cutting decisions as ADRs so SuperMatt's skills read them for free. Phrase role rules as repo-documented standards so `supermatt:code-review`'s Standards axis enforces them.
- [R] Borrow ideas from CE: its `product-lens`, `design-lens` and `scope-guardian` persona prompts, `STRATEGY.md`, and the Compound Packs `applies_when` pattern. All are MIT, and copied text needs a `NOTICE` the way SuperMatt does it. If PM and PMM roles end up dominating, it may be worth evaluating CE as the base instead.
- [R] If upstream changes become essential, for example a pluggable spec template or a model-invocable `to-spec`, propose them to `svyatov/supermatt` before forking. SuperMatt is a single-maintainer project (README: "Fixes go to the latest release only"). mattpocock/skills does not accept new skills.
