# 02: Spec-driven development frameworks

**Question.** Which existing spec-driven development (SDD) framework should the user's "virtual software product team" build on? The team is a set of AI agents (Product Manager, TPM, Customer persona, SDM, PMM, and maybe more) that takes work through spec, design, prototype and full implementation. The user wants a Kiro-like spec flow, a spec format that can be swapped out, and customizations that survive upstream updates.

**Researched:** 2026-11-04.

**Access notes:**

- **Repositories read directly.** Shallow clones of each repo's default branch, taken on 2026-11-04. The commit is pinned in each link:
  - `github/spec-kit` @ `4aedb5b48f`
  - `bmad-code-org/BMAD-METHOD`: `main` @ `022be9314e` (the v7 preview) and branch `V6.12` @ `bab9082c3e` (stable)
  - `Fission-AI/OpenSpec` @ `3e4d5f0769`
  - `buildermethods/agent-os` @ `e892e61d32`
  - `eyaltoledano/claude-task-master` @ `c0c98d367c`
  - `gotalab/cc-sdd` @ `e2a0c671ae`
  - `Pimzino/spec-workflow-mcp` @ `d38e82eaa8`
  - `kirodotdev/Kiro` @ `bfe7ff30d9` (issue tracker and README only)
  - `kirodotdev/powers` @ `3a39f45582`
  - SuperMatt v0.22.0, read from the local plugin install.
- **Versions and dates** come from the npm registry (`registry.npmjs.org`) and PyPI, read on 2026-11-04.
- **Star counts** were read off the github.com repo pages on 2026-11-04. They are rounded figures as GitHub displays them. The GitHub REST API was not enabled in this session.
- **Not reachable from this sandbox:** `kiro.dev`, `tessl.io` and `docs.aws.amazon.com`. Claims about Kiro docs and Tessl come from search-engine extracts of those pages, so they are medium confidence. The sibling report [04-portability-and-aws-work-use.md](04-portability-and-aws-work-use.md) §2.2 has more Kiro feature detail gathered the same way.

**How each claim is labelled:** **[Fact]** means a cited primary source states it. **[Search]** means it comes from a search-engine extract of an official page. **[Inference]** is my reasoning from the facts. **[Rec]** is a recommendation.

---

## 1. Kiro (AWS)

**What it is [Fact].** Kiro describes itself as "an agentic AI development platform that helps developers and teams plan, build, and review software". It runs as an IDE, a CLI, Web, Mobile and "Crew". "One unified agent harness" powers every surface, so "specs, steering, permissions, hooks, MCP servers, and custom agents can follow your project across workflows." The product source is closed; the `kirodotdev/Kiro` repository is only the "public issue and feedback tracker". ([Kiro README](https://github.com/kirodotdev/Kiro/blob/bfe7ff30d9/README.md), updated 2026-08-27.) Kiro is an Amazon product. When Kiro became generally available on 2025-11-18, Kiro CLI replaced Amazon Q Developer CLI, and the `q` commands remained as aliases **[Search, secondary: [Classmethod](https://dev.classmethod.jp/articles/kiro-ga-announcement-kiro-cli/)]**.

**Workflow phases [Search].**

- **Phases.** A spec runs Requirements → Design → Tasks → execution. Each phase writes a file under `.kiro/specs/<name>/` ([specs](https://kiro.dev/docs/specs/), [CLI v3 specs](https://kiro.dev/docs/cli/v3/specs/)).
- **Feature spec variants.** Feature specs come in two variants, Requirements-First and Design-First ([requirements-first](https://kiro.dev/docs/specs/feature-specs/requirements-first/)).
- **Bugfix specs.** These write `bugfix.md`, which records current behavior, expected behavior and "unchanged behavior" ([specs](https://kiro.dev/docs/specs/)).
- **Quick Spec.** Generates all three files in one pass with no approval gates ([quick-spec](https://kiro.dev/docs/specs/quick-spec/)).
- **Cross-surface.** In the CLI, `/spec new` offers feature, bugfix or Quick Spec. Specs are shared across surfaces, so a spec started in the CLI can be finished in the IDE.
- **Analysis and verification [Fact, README].** Kiro adds "Requirements Analysis" (to find contradictions, ambiguities and gaps) and "correctness with property-based testing".

**Artifacts [Search].**

- **`requirements.md`.** User stories whose acceptance criteria use EARS notation: "WHEN … THE SYSTEM SHALL …" ([concepts](https://kiro.dev/docs/specs/concepts)).
- **`design.md` and `tasks.md`.** `tasks.md` holds "discrete, trackable tasks". I could not read Kiro's own `design.md` headings on kiro.dev.
- **Community copies of the design format.** Kiro-compatible tools use **Overview / Architecture / Components and Interfaces / Data Models / Error Handling / Testing Strategy** ([spec-workflow-mcp `design-template.md`](https://github.com/Pimzino/spec-workflow-mcp/blob/d38e82eaa8/src/markdown/templates/design-template.md); third-party skill pages). **[Inference]** Treat these as the de-facto format, not an official one.

**Steering, hooks, skills, custom agents and Powers [Search unless noted].**

- **Steering.** Markdown files in `.kiro/steering/` with inclusion modes `always` (the default), `fileMatch` (`fileMatchPattern`), `manual` and `auto`. `AGENTS.md` is always included ([steering](https://kiro.dev/docs/steering/)).
- **Hooks.** JSON files in `.kiro/hooks/`. Triggers include Prompt Submit, Agent Stop, Pre/Post Tool Use, File Create/Save/Delete and Manual ([hook types](https://kiro.dev/docs/hooks/types)).
- **Agent Skills.** `.kiro/skills/` and `~/.kiro/skills/` ([skills](https://kiro.dev/docs/skills/)).
- **Custom agents.** `.kiro/agents/` and `~/.kiro/agents/`. Custom agents do not load skills unless you list them in `resources` as `skill://…` URIs.
- **MCP and the README feature list [Fact].** The README lists custom agents, Steering, Agent Skills, Hooks, Powers and MCP as features.
- **Powers [Fact].** Kiro's own `power-builder` power says Powers are now built to "Agent Plugins … an open, vendor-neutral specification (v1.0.0)", with `plugin.json`, `skills/<name>/SKILL.md` and `mcp.json`. It also says the legacy `POWER.md` + `steering/` format can be migrated ([agent-plugins-reference](https://github.com/kirodotdev/powers/blob/3a39f45582/power-builder/skills/agent-plugins-reference/SKILL.md), [migrate-plugin](https://github.com/kirodotdev/powers/blob/3a39f45582/power-builder/skills/migrate-plugin/SKILL.md)).

**Role model.** Kiro has no product-team personas. **[Inference]** You would build roles as custom agents plus skills.

**Customization and upgrades.**

- **What is user-owned [Inference].** Steering, skills, agents, hooks and Powers are all files the user owns under `.kiro/`, so product upgrades do not overwrite them.
- **What I could not find.** Any documented way to replace the spec *format* itself, such as a template override for `requirements.md`. Kiro's spec format looks fixed by the product.

**Hosts.** Kiro IDE, CLI and Web only. Other frameworks target Kiro, not the other way round. **License:** proprietary. Kiro CLI moved from Apache-2.0 (Q CLI) to the "AWS Intellectual Property License" **[Search, secondary: Classmethod]**. **Maturity:** GA since 2025-11. The CLI specs page was labelled "CLI 3.0 (Early Access)" and updated 2026-06-17 **[Search]**.

---

## 2. GitHub Spec Kit (`github/spec-kit`)

**Maturity [Fact].**

- **Version.** `specify-cli` 1.3.0, released 2026-11-03 ([CHANGELOG](https://github.com/github/spec-kit/blob/4aedb5b48f/CHANGELOG.md); PyPI upload 2026-11-03).
- **License.** MIT, © GitHub, Inc.
- **Activity.** About 147k stars. Releases ship every few days (1.2.3 to 1.3.0 between 2026-10-22 and 2026-11-03).

**Phases [Fact].**

- **Minimal flow.** `/speckit-constitution` (once), `/speckit-specify`, `/speckit-plan`, `/speckit-tasks`, `/speckit-implement`, `/speckit-converge`.
- **Full path.** Adds the quality gates `/speckit-clarify`, `/speckit-checklist` and `/speckit-analyze` ([quickstart](https://github.com/github/spec-kit/blob/4aedb5b48f/docs/quickstart.md), [README](https://github.com/github/spec-kit/blob/4aedb5b48f/README.md)).
- **`converge`.** Assesses the code against spec, plan and tasks, then "append[s] any remaining unbuilt work as new tasks" ([converge.md](https://github.com/github/spec-kit/blob/4aedb5b48f/templates/commands/converge.md)).
- **Bundled workflows.** `speckit` is the full cycle with review gates after spec and plan. `bugfix` handles bugs. `assess` covers "Idea intake, research, definition, shaping, and decision" ([workflows overview](https://github.com/github/spec-kit/blob/4aedb5b48f/docs/workflows/overview.md)).

**Templates (headings quoted) [Fact]** ([templates/](https://github.com/github/spec-kit/tree/4aedb5b48f/templates)):

- **`spec-template.md`.** "Feature Specification: [FEATURE NAME]" → "User Scenarios & Testing *(mandatory)*" (user stories with priorities P1–P3, each with "Independent Test" and Given/When/Then "Acceptance Scenarios") → "Edge Cases" → "Requirements *(mandatory)*" ("Functional Requirements", `FR-001: System MUST …`, `[NEEDS CLARIFICATION: …]` markers; "Key Entities") → "Success Criteria *(mandatory)*" → "Assumptions".
- **`plan-template.md`.** "Implementation Plan" → "Summary", "Technical Context", "Constitution Check", "Project Structure", "Complexity Tracking".
- **`tasks-template.md`.** Format `[ID] [P?] [Story] Description`. Phases run "Setup", "Foundational", one phase per user story ("🎯 MVP"), then "Polish & Cross-Cutting Concerns", followed by "Dependencies & Execution Order" and "Implementation Strategy".
- **`constitution-template.md`.** "Core Principles", "Governance".

**Customization model (the strongest found) [Fact].**

- **Presets.** "Stackable, priority-ordered customizations of Spec Kit templates, commands, and scripts … without forking core files."
  - **Resolution order.** Each file is resolved project overrides (`.specify/templates/overrides/`) > presets > extensions > core.
  - **Merge strategies.** `replace`, `prepend`, `append` and `wrap`.
  - **Inspection.** `specify preset resolve spec-template` shows which layer won.
  - Source: [presets overview](https://github.com/github/spec-kit/blob/4aedb5b48f/docs/presets/overview.md).
- **Extensions.** These add commands and hooks. Hook events include `before_specify`/`after_specify`, `before_plan`/`after_plan`, `before_tasks`/`after_tasks`, `before_implement`/`after_implement` and `before_analyze`/`after_analyze` ([extensions API](https://github.com/github/spec-kit/blob/4aedb5b48f/docs/extensions/api.md), [overview](https://github.com/github/spec-kit/blob/4aedb5b48f/docs/extensions/overview.md)).
- **Workflow overlays.** These "extend or override an installed workflow without editing the installed `workflow.yml`", which "keeps local customizations safe across … upgrades" ([overlays](https://github.com/github/spec-kit/blob/4aedb5b48f/docs/workflows/overlays.md)).
- **Upgrades.** Generated files are hash-tracked. "Modified files … are preserved so your customizations are not lost" ([integrations CLI](https://github.com/github/spec-kit/blob/4aedb5b48f/docs/integrations/cli.md), [upgrade](https://github.com/github/spec-kit/blob/4aedb5b48f/docs/upgrade.md)).
- **Community catalog** (as of 2026-11-04: 186 extensions, 42 presets; [extensions catalog](https://github.com/github/spec-kit/blob/4aedb5b48f/extensions/catalog.community.json)). Entries relevant here:
  - `ears`: "Author, lint, and convert requirements using EARS"
  - `product`: "Generates PRFAQ, Lean PRD, stakeholder summaries, and technical designs"
  - `product-forge`: "research → product-spec → plan → tasks → implement → verify → test → release-readiness"
  - `openspec`: OpenSpec's propose→archive flow
  - `squad`: "Bootstrap … a Squad agent team"
  - `critique`: "product strategy and engineering risk perspectives"
  - `team-assign`, plus several Jira bridges
  - All community entries are third party and unaudited.

**Roles [Fact/Inference].** The core has no personas. The `model-routing-governance` preset maps "provider-neutral Spec Kit roles" to runner profiles, but these are execution roles, not product roles. **[Inference]** Product personas would ship as an extension.

**Hosts [Fact].** The integration catalog has 43 keys ([catalog.json](https://github.com/github/spec-kit/blob/4aedb5b48f/integrations/catalog.json)):

- **Covered:** `claude` (installs `.claude/skills/speckit-*/SKILL.md`), `codex`, `copilot`, `cursor-agent`, `kiro-cli` (prompts under `.kiro/prompts/`; Kiro CLI does not substitute `$ARGUMENTS`) ([kiro-cli.md](https://github.com/github/spec-kit/blob/4aedb5b48f/docs/integrations/kiro-cli.md)), `gemini` and others.
- **Missing:** There is no Kiro IDE key and no `amazon-q` key. The CHANGELOG records an earlier "Amazon Q Developer CLI Integration". **[Inference]** `kiro-cli` has taken its place.

---

## 3. BMAD-METHOD (`bmad-code-org/BMAD-METHOD`)

**Maturity [Fact].**

- **Two release lines** ([README](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/README.md)):
  - Stable is **v6.12**: npm `bmad-method` 6.12.4, released 2026-10-14, "frozen aside from bug fixes".
  - **V7** is in "preview alpha" on `main`, with a "v7 release ETA [of] mid-November 2026".
- **License.** MIT, but "BMad" and "BMAD-METHOD" are trademarks ([TRADEMARK.md](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/TRADEMARK.md)).
- **Stars.** About 55k.

**Phases [Fact].**

- **Delivery loop.** Clarify → Plan → Build and verify → Learn and adjust, with "right-sized process": small changes go straight to build.
- **Planning skills.** `bmad-product-brief`, `bmad-prfaq`, `bmad-prd`, `bmad-ux` and `bmad-architecture`, then the Product Owner turns a PRD into tickets and epics.
- **Build skills.** `bmad-build`, `bmad-build-auto`, `bmad-code-review` and `bmad-retrospective` ([skills-and-agents](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/docs/reference/skills-and-agents.md); v6.12 layout `src/bmm-skills/{plan,ship}` on [V6.12](https://github.com/bmad-code-org/BMAD-METHOD/tree/bab9082c3e/src/bmm-skills)).

**Agent roster [Fact].**

- **v7 `main`.** "Six named agents" ([skills-and-agents](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/docs/reference/skills-and-agents.md)):
  - Analyst (Mary): brainstorm, market/domain/technical research, competitive teardown, user-voice research, product brief, PRFAQ challenge
  - Product Manager (John): PRD
  - Product Owner (Sofia): spec into tickets, slicing, refining, board, replan
  - Architect (Winston)
  - Developer (Amelia): build, QA tests, code review, retro
  - UX Designer (Sally)
  - The Technical Writer (Paige) is "on hiatus".
- **v6.12 branch.** Has five agent folders; there is no PO agent ([V6.12 agents](https://github.com/bmad-code-org/BMAD-METHOD/tree/bab9082c3e/src/bmm-skills/agents)).
- **Roles that were merged away.** v6.3.0 (2026-04-09) folded the Scrum Master (Bob), QA (Quinn) and quick-flow dev (Barry) into the Developer ([V6.12 CHANGELOG](https://github.com/bmad-code-org/BMAD-METHOD/blob/bab9082c3e/CHANGELOG.md)). **[Inference]** The classic "Analyst/PM/Architect/UX/PO/SM/Dev/QA" roster describes v4 through early v6. It is not current.

**Templates (headings quoted) [Fact].**

- **PRD.** [`prd-template.md`](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/skills/bmad-prd/assets/prd-template.md):
  - Spine: "0. Document Purpose", "1. Vision", "2. Target User" (with "Jobs To Be Done" and "Key User Journeys"), "3. Glossary", "4. Features" (`FR-1: {Short capability name}`), "5. Non-Goals (Explicit)", "6. MVP Scope", "7. Success Metrics", "8. Open Questions", "9. Assumptions Index".
  - An "Adapt-In Menu" adds clusters for consumer, enterprise, regulated, developer-product and embedded work.
- **PR/FAQ.** [`prfaq-template.md`](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/skills/bmad-prfaq/assets/prfaq-template.md): "{Headline}", "How It Works", "Getting Started", "Customer FAQ", "Internal FAQ", "The Verdict".
- **Architecture.** [`spine-template.md`](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/skills/bmad-architecture/assets/spine-template.md): "Design Paradigm", "Invariants & Rules" (`AD-1`), "Consistency Conventions", "Stack", "Structural Seed", "Capability → Architecture Map", "Deferred".

**Customization (well designed and update-safe) [Fact]** ([customize-bmad](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/docs/customize/customize-bmad.md)):

- **Override files.** Every customizable skill ships a `customize.toml` schema. You "never edit it; updates overwrite it". Instead you add sparse overrides:
  - `_bmad/custom/<skill>.toml` (team, committed)
  - `_bmad/custom/<skill>.user.toml` (personal, gitignored)
- **Merge rules.** Scalars are overridden, tables deep-merge, arrays of tables keyed by `code`/`id` merge by key, and other arrays append. There is "No removal".
- **What agents expose.** `role`, `identity`, `communication_style`, `principles`, `persistent_facts` (including `file:` globs), `activation_steps_prepend/append` and `[[agent.menu]]` items.
- **What workflows expose.** The same, plus `on_complete` and workflow-specific fields such as `prd_template` and `validation_checklist_template`. So **the PRD template itself is swappable** ([adopt-across-a-team](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/docs/customize/adopt-bmad-across-a-team.md), [bmad-prd/customize.toml](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/skills/bmad-prd/customize.toml)).
- **Limit on identity.** `agent.name` and `agent.title` are read-only. To get a differently named agent, "copy the skill folder, rename it, and ship it as a custom skill".
- **Roster and modules.** A central `_bmad/custom/config.toml` can rebrand roster descriptors or add "fictional" party members. Modules (Test Architect, Creative Intelligence Suite, Game Dev Studio) install through `npx skills add` plus `bmad setup` ([add-modules](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/docs/customize/add-modules.md)). The v4-era name "expansion packs" now survives only in old CHANGELOG entries.

**Party mode [Fact].** `bmad-party-mode` "puts your installed BMad agents in one conversation, in character". It has run modes `auto`, `session`, `subagent` and `agent-team`, custom casts (`party_members`, `party_groups`), append-only party memory, and shipped rooms ("Code Review Crew", "Anti-Consensus Club") ([run-multi-agent-discussions](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/docs/customize/run-multi-agent-discussions.md); V6.12 CHANGELOG #2479/#2530).

**Churn risk [Fact].** The Unreleased (v7) changelog lists many breaking changes: Sofia replacing `bmad-spec`/`bmad-ticket`/`bmad-correct-course`, removal of `planning_artifacts`, removal of web bundles, renamed TOML keys. `bmad setup` migrates renamed override files but "never changes a value inside a file" ([CHANGELOG](https://github.com/bmad-code-org/BMAD-METHOD/blob/022be9314e/CHANGELOG.md)).

**Hosts [Fact].**

- **v6.12 installer.** Has about 50 platform codes, including `claude-code` (`.claude/skills`), `codex`, `cursor`, `github-copilot` and `kiro` (`.kiro/skills`) ([platform-codes.yaml](https://github.com/bmad-code-org/BMAD-METHOD/blob/bab9082c3e/tools/installer/ide/platform-codes.yaml)).
- **v7.** Installs through the Skills CLI or the Claude Code and Codex plugin marketplaces.
- **Amazon Q.** No Amazon Q target.

---

## 4. OpenSpec (`Fission-AI/OpenSpec`)

**Maturity [Fact].** npm `@fission-ai/openspec` 1.16.0, released 2026-10-30. MIT. About 74k stars. Commits landed the day of research ([CHANGELOG](https://github.com/Fission-AI/OpenSpec/blob/3e4d5f0769/CHANGELOG.md)).

**Phases [Fact].**

- **Core commands.** `/opsx:propose`, `/opsx:explore`, `/opsx:apply`, `/opsx:update`, `/opsx:sync` (merge delta specs into main specs) and `/opsx:archive`.
- **Expanded commands.** `new`, `continue`, `ff`, `verify` and `bulk-archive` ([commands.md](https://github.com/Fission-AI/OpenSpec/blob/3e4d5f0769/docs/commands.md)).
- **The change model.** OpenSpec centres on *changes* (proposals carrying delta specs) that are archived into living `openspec/specs/`.

**Templates [Fact]** (default schema `spec-driven`: proposal → specs → design → tasks; [schema.yaml](https://github.com/Fission-AI/OpenSpec/blob/3e4d5f0769/schemas/spec-driven/schema.yaml)):

- **`proposal.md`.** "Why", "What Changes", "Capabilities" ("New Capabilities", "Modified Capabilities"), "Impact".
- **`spec.md`.** "Spec Delta", "Purpose", "ADDED Requirements", then "Requirement: …" and "Scenario: …" with **WHEN**/**THEN** bullets. `openspec validate` warns when a scenario has no WHEN/GIVEN or THEN step.
- **`design.md`.** "Context", "Goals / Non-Goals", "Decisions", "Risks / Trade-offs".
- **`tasks.md`.** Numbered task groups.

**Customization: closest to a native spec-format adapter [Fact]** ([customization.md](https://github.com/Fission-AI/OpenSpec/blob/3e4d5f0769/docs/customization.md)):

- **Project config.** `openspec/config.yaml` sets a default `schema`, project `context` and per-artifact `rules`.
- **Custom schemas.** These live in `openspec/schemas/<name>/` (`schema.yaml` + `templates/`). They define their own artifacts (`id`, `generates`, `template`, `instruction`, `requires`), so the artifact graph itself can be swapped.
- **Schema resolution.** CLI flag > change metadata > project config > default. User-level schemas live in `~/.local/share/openspec/schemas/`.
- **Community schemas** are copied into the project, not package-managed. Examples:
  - `nanopm`: "PM-first workflow … audit → strategy → roadmap → PRD"
  - `epic-driven`
  - `anvil`: adds review and test-plan artifacts
  - `intent-driven`: adds ADRs
- **Upgrades.** `openspec update` regenerates tool instruction files. Custom schemas live in user-owned paths. **[Inference]** They survive updates, but a forked schema will not pick up upstream changes to `spec-driven` automatically.

**Roles.** None. A schema's `instruction` text can carry a persona. **[Inference]**

**Hosts [Fact].** 41 tools, including Claude Code (`.claude/skills/openspec-*` plus `/opsx:` commands), Codex (skills only, `$openspec-*`), Cursor, GitHub Copilot (including the cloud coding agent), Kiro (`.kiro/skills/`, `.kiro/prompts/`) and Amazon Q Developer (`.amazonq/prompts`, `@opsx-<id>`) ([supported-tools.md](https://github.com/Fission-AI/OpenSpec/blob/3e4d5f0769/docs/supported-tools.md)).

---

## 5. Agent OS (`buildermethods/agent-os`)

**Maturity [Fact].** v3.0, released 2026-01-20. Last commit 2026-10-07 (unreleased fixes only). MIT, © CasJam Media LLC. About 5.6k stars ([CHANGELOG](https://github.com/buildermethods/agent-os/blob/e892e61d32/CHANGELOG.md)).

**Scope after v3 [Fact].** v3 "refocuses the framework on … establishing and injecting standards".

- **What it hands off.** Spec writing goes to host **Plan Mode**, and "Implementation/orchestration phases [are] retired".
- **Commands.** `/discover-standards`, `/inject-standards`, `/index-standards`, `/plan-product` and `/shape-spec`.
- **Product docs.** `plan-product` writes `agent-os/product/mission.md`, `roadmap.md` and `tech-stack.md`.
- **Spec folder.** `shape-spec` must run in plan mode. It saves `agent-os/specs/{YYYY-MM-DD-HHMM-slug}/` with `plan.md`, `shape.md`, `standards.md`, `references.md` and `visuals/` ([shape-spec.md](https://github.com/buildermethods/agent-os/blob/e892e61d32/commands/agent-os/shape-spec.md), [plan-product.md](https://github.com/buildermethods/agent-os/blob/e892e61d32/commands/agent-os/plan-product.md)).

**Customization [Fact].** Profiles can inherit from one another (`config.yml` `inherits_from`), and `sync-to-profile.sh` syncs project standards back to a base profile. **Hosts:** "Claude Code, Cursor, Antigravity, and other AI tools"; the installer writes `.claude/commands/agent-os/`. **Roles:** none. **[Inference]** This is a standards-injection layer, not a spec lifecycle. It is useful as a pattern for steering, not as a base.

---

## 6. Taskmaster (`eyaltoledano/claude-task-master`)

**Maturity [Fact].** npm `task-master-ai` 0.43.1, released 2026-03-31. Last repo commit 2026-04-23. About 28k stars. The license is **"MIT WITH Commons-Clause"** (not OSI open source). The docs now sit under Hamster (`tryhamster.com`) ([package.json](https://github.com/eyaltoledano/claude-task-master/blob/c0c98d367c/package.json), [README](https://github.com/eyaltoledano/claude-task-master/blob/c0c98d367c/README.md)).

**Model [Fact].** Taskmaster is a PRD-to-task-graph engine (CLI plus MCP server), not a spec-authoring flow.

- **Commands.** `task-master parse-prd`, `list`, `next`, `show`, `research`, `move` across tags.
- **PRD templates.** "Overview / Core Features / User Experience / Technical Architecture / Development Roadmap / Logical Dependency Chain / Risks and Mitigations / Appendix", and an "RPG (Repository Planning Graph)" PRD with "Capability Tree", "Module Definitions", "Dependency Chain" ([templates](https://github.com/eyaltoledano/claude-task-master/tree/c0c98d367c/.taskmaster/templates)).
- **Rule profiles.** amp, claude, cline, codex, cursor, gemini, kiro, opencode, kilo, roo, trae, vscode, windsurf and zed ([profiles.js](https://github.com/eyaltoledano/claude-task-master/blob/c0c98d367c/src/constants/profiles.js)).
- **Roles.** None.

**[Inference]** Activity has slowed since April 2026, and the license limits commercial redistribution. It is at most a task-graph component.

---

## 7. Tessl

**[Search, medium-low confidence]** Tessl has two products: a spec-driven development framework that connects to agents over MCP, and a registry. The registry began as a "Spec Registry" (open beta 2025-09-16, per a third-party review) and has been repositioned as a registry of versioned skills/"tiles", e.g. `tessl install tessl-labs/spec-driven-development` ([registry tile](https://tessl.io/registry/tessl-labs/spec-driven-development); [third-party review](https://codemyspec.com/blog/tessl-review)). The review describes the spec-as-source generator as closed beta.

**[Fact]** npm `@tessl/cli` 0.119.0 was released 2026-10-30 under license "SEE LICENSE.md", meaning proprietary ([npm](https://registry.npmjs.org/@tessl/cli)). I could not read tessl.io directly. **[Inference]** It is not a fit for an update-safe, open base.

---

## 8. Kiro-flow ports and related tools

### cc-sdd (`gotalab/cc-sdd`)

**[Fact]** cc-sdd 3.1.0, released 2026-09-23. MIT. About 3.8k stars ([README](https://github.com/gotalab/cc-sdd/blob/e2a0c671ae/README.md)).

- **Kiro compatibility.** It is "Kiro-inspired". "Existing Kiro specs remain compatible and portable."
- **Flow.** `/kiro-discovery` (writes `brief.md`, and `roadmap.md` when needed) → `kiro-spec-init` → `kiro-spec-requirements` → `kiro-spec-design` → `kiro-spec-tasks` → `kiro-impl`. `kiro-impl` runs per-task TDD subagents with an independent reviewer. `kiro-spec-batch` handles multi-spec work, and `kiro-steering` covers steering.
- **Install.** 17 Agent Skills across 8 hosts. Claude Code and Codex are stable; Cursor, Copilot, Devin, OpenCode, Gemini CLI and Antigravity are beta. There is no Kiro install target, because the output *is* `.kiro/`.
- **Requirements template.** "Requirements Document", "Introduction", "Boundary Context (Optional)", "Requirement N", "**Objective:** As a {{ROLE}}…", "Acceptance Criteria" using all five EARS patterns ("When … the [system] shall", "If … then", "While …", "Where …", "The [system] shall") ([requirements.md](https://github.com/gotalab/cc-sdd/blob/e2a0c671ae/tools/cc-sdd/templates/shared/settings/templates/specs/requirements.md)).
- **Customization.** Edit `.kiro/settings/templates/` and `rules/`. The README suggests "PRD-style requirements" as a use case. On re-install, the installer asks per file group whether to overwrite, append or keep ([migration guide](https://github.com/gotalab/cc-sdd/blob/e2a0c671ae/docs/guides/migration-guide.md)). **[Inference]** That is weaker than layered overrides: customizations survive only if you choose "keep" each time.

### spec-workflow-mcp (`Pimzino/spec-workflow-mcp`)

**[Fact]** An MCP server plus web dashboard and VS Code extension. License **GPL-3.0**. About 4.3k stars ([README](https://github.com/Pimzino/spec-workflow-mcp/blob/d38e82eaa8/README.md)).

- **Version conflict.** The repo CHANGELOG lists 2.2.7 (2026-05-04), but npm `latest` is 2.2.5 (2026-03-02).
- **Flow.** A Kiro-like Requirements → Design → Tasks → Implementation flow, plus steering docs (`product`, `tech`, `structure`) and an approval workflow.
- **Templates.** `requirements-template.md` uses "WHEN [event] THEN [system] SHALL [response]".
- **Upgrade behavior.** Bundled templates in `.spec-workflow/templates/` are "Always overwrite[n]", while `.spec-workflow/user-templates/` is checked first ([workspace-initializer.ts](https://github.com/Pimzino/spec-workflow-mcp/blob/d38e82eaa8/src/core/workspace-initializer.ts), [steering-guide.ts](https://github.com/Pimzino/spec-workflow-mcp/blob/d38e82eaa8/src/tools/steering-guide.ts)). This is a clean, update-safe two-layer override.
- **Hosts.** Any MCP client: Claude Code, Cursor, Codex, Cline and others.

### SuperMatt `to-spec` (the user's current toolkit)

**[Fact]** SuperMatt v0.22.0 is MIT. `to-spec` synthesizes the conversation into a spec and *publishes it to the issue tracker* with label `ready-for-agent`. Its headings are "Problem Statement", "Solution", "User Stories" ("As an <actor>, I want a <feature>, so that <benefit>"), "Implementation Decisions", "Testing Decisions", "Out of Scope" and "Further Notes" (local plugin `skills/to-spec/SKILL.md`).

**[Inference]** Its target is a tracker issue, not a file tree. That is a different storage target any adapter must handle.

---

## 9. Comparison

| | Kiro | Spec Kit | BMAD | OpenSpec | Agent OS | Taskmaster | cc-sdd | spec-workflow-mcp |
|---|---|---|---|---|---|---|---|---|
| **Artifact format** | requirements.md (EARS) / design.md / tasks.md; bugfix.md | constitution + spec.md (user stories, FR-###, Given/When/Then) / plan.md / tasks.md | brief, PRFAQ, PRD (FR-n), UX, architecture spine, tickets/epics | proposal / delta specs (Requirement + WHEN/THEN) / design / tasks | mission/roadmap/tech-stack; plan.md + shape.md | PRD → tasks.json graph | Kiro format (5 EARS patterns) + brief/roadmap | Kiro-like (WHEN/THEN/SHALL) + steering |
| **Format pluggability** | None documented | **High**: presets replace/wrap templates *and* commands; layered resolution | Medium: per-workflow template keys (`prd_template`, `brief_template`) | **Highest**: custom schema defines the artifact DAG | Low | Low (PRD in, fixed task schema) | Medium: edit `.kiro/settings/templates` | Medium: `user-templates/` |
| **Roles beyond engineering** | None | None in core; community `product` (PRFAQ), `product-forge`, `critique`, `squad` | **Analyst, PM, PO, UX, Architect, Dev**; PRFAQ; party mode; custom party members | None (community `nanopm` schema) | None | None | None | None |
| **Update-safe customization** | User files under `.kiro/` (format fixed) | **Yes**: overrides, presets, extensions, overlays; hash-tracked files | **Yes**: sparse `_bmad/custom/*.toml`, team and user layers | Yes for config and schemas; forks drift | Profiles + sync | n/a | Partial (installer prompt) | Yes (two-layer) |
| **Claude Code** | No | Yes (skills) | Yes (skills/plugin) | Yes | Yes | Yes (MCP) | Yes (stable) | Yes (MCP) |
| **Kiro** | Native | kiro-cli only | Yes (`.kiro/skills`) | Yes (skills + prompts) | Not listed | Yes (rules profile) | Writes `.kiro/` format | MCP |
| **Cursor / Codex / Copilot** | No | Yes / Yes / Yes | Yes / Yes / Yes | Yes / Yes / Yes | Cursor | Yes / Yes / VS Code | beta / stable / beta | Yes / Yes / – |
| **Amazon Q** | Superseded by Kiro CLI | No (removed) | No | Yes | Not listed | Q CLI via MCP | No | Any MCP client |
| **License** | Proprietary (AWS) | MIT | MIT + trademark | MIT | MIT | MIT + Commons Clause | MIT | GPL-3.0 |
| **Version / date** | GA 2025-11 | 1.3.0 / 2026-11-03 | 6.12.4 / 2026-10-14 (v7 alpha) | 1.16.0 / 2026-10-30 | 3.0 / 2026-01-20 | 0.43.1 / 2026-03-31 | 3.1.0 / 2026-09-23 | 2.2.5 npm / 2026-03-02 |
| **Stars (approx.)** | n/a | 147k | 55k | 74k | 5.6k | 28k | 3.8k | 4.3k |

---

## 10. Conflicts, gaps and open questions

- **Kiro docs were unreadable here.** I could not confirm Kiro's official `design.md`/`tasks.md` headings or whether any spec-template override exists. If Kiro has added template customization, the conclusion that Kiro is a format target, not a base, weakens. Re-check https://kiro.dev/docs/specs/ once `kiro.dev` is allowed.
- **BMAD is mid-transition.** Roster, file layout and TOML keys differ between v6.12 and v7. Anything built on BMAD now should pin a version and expect a migration around the v7 GA (ETA mid-November 2026, per the README).
- **Star counts are approximate**, read off rendered GitHub pages, not the API.
- **spec-workflow-mcp** has a version mismatch between its repo and npm (see §8).
- **The spec-kit community catalog is unaudited.** The relevant extensions (`ears`, `product`, `squad`) are third party; the catalog says "Maintainers … do not review, audit, endorse".

---

## Implications

### Which framework fits best as a base

**[Inference]** No framework covers all three of the user's needs: (a) pluggable format, (b) non-engineering roles, (c) update-safe add-ons.

- **Spec Kit** is strongest on (a) and (c). It is GitHub-backed and releases every few days. Its extension model (commands + `before_*/after_*` hooks), presets (template and command replacement with `wrap`/`append`) and workflow overlays (gated, resumable YAML pipelines) are the only documented, package-managed way to add phases and roles without forking.
- **OpenSpec** has the cleanest *format* abstraction, a declarative artifact DAG in `schema.yaml`. Its community schemas are copy-in, though, and it has no hook or role layer.
- **BMAD** is strongest on (b): six personas, PRFAQ, party mode, `customize.toml` team/user overrides with documented merge rules. It is mid-way through a breaking v6 → v7 transition, and new agent identities require shipping your own skill folders anyway.
- **Kiro** is a format and host target, not a base: it is closed and its format looks fixed. **cc-sdd** is the best evidence that the Kiro format ports cleanly to Claude Code.

**[Rec]**

1. **Base on Spec Kit.** Pin the CLI version, and install the team as an extension ("product-team") plus one preset per spec format:
   - **Phases as workflow steps.** Model spec → design → prototype → implement as a Spec Kit workflow with review gates. Prototype is not a core phase, so add it as a workflow step or extension command. Customize the workflow through overlays.
   - **Role reviews as hooks.** PM, Customer persona and PMM reviews attach to `after_specify`. SDM and TPM attach to `after_plan`/`after_tasks`, for staffing, sequencing and risk.
2. **Borrow from BMAD rather than build on it.** Take its persona schema (role / identity / communication_style / principles / persistent_facts / menu) and its sparse team-plus-user override layering for your role definitions. Take party mode as the pattern for multi-role review. Revisit BMAD as a base after v7 reaches GA.
3. **Use OpenSpec's `schema.yaml` as the design reference** for the adapter interface below. That keeps an OpenSpec back end possible later.
4. **Keep role definitions host-neutral.** Write them as Agent Skills, so the same files work in Claude Code and in Kiro's `.kiro/skills`. This matches the portability findings in report 04.

### What a pluggable "spec format adapter" must cover

**[Rec]** Define a canonical intermediate model, and have each format implement parse and emit against it:

1. **Artifact graph.**
   - Artifact `id`, output path/glob, template, authoring instruction, `requires` dependencies (OpenSpec-style), and which canonical slot it fills (`intent`, `requirements`, `design`, `tasks`).
   - Optional non-engineering slots: `prfaq`, `positioning`, `launch-plan`.
2. **Requirement grammar and linting.** Encode each format's acceptance-criteria rules and validate them:
   - Kiro and cc-sdd: the 5 EARS patterns ("WHEN/IF/WHILE/WHERE … SHALL").
   - Spec Kit: `FR-###: System MUST` + Given/When/Then.
   - OpenSpec: `Requirement:` / `Scenario:` WHEN/THEN.
   - BMAD: `FR-n` under Features.
   - SuperMatt: numbered user stories.
3. **IDs and traceability.** A stable ID scheme, plus links from requirement → design decision (`AD-n`) → task. Include each format's task annotations: `[P]`/`[Story]`, `_Boundary:_`/`_Depends:_`, and status markers such as `[ ]`, `[x]` and `[~]`.
4. **Change semantics.** Whole-document specs (Kiro, Spec Kit) versus delta specs (`ADDED`/`MODIFIED` plus archive and sync in OpenSpec).
5. **Storage target.** File tree (`.kiro/specs/<f>/`, `specs/<n>/`, `openspec/changes/<c>/`) versus issue tracker (SuperMatt `to-spec` publishes an issue).
6. **Gates and ownership.** Which phase transitions need approval, and which role authors or reviews each artifact. For example: Customer persona reviews requirements, PMM owns the PR/FAQ, TPM owns tasks and dependencies.
7. **Lossless round-trip or declared loss.** Converting between formats can drop information: an EARS requirement has no "success metric", and a PR/FAQ has no tasks. The adapter should report what was lost and never drop it silently.
8. **Override layering.** Format packs must be overridable the way Spec Kit presets and BMAD `_bmad/custom` are, so team tweaks survive upstream updates.

**Formats the first set of adapters should cover:**

| Format | What defines it |
|---|---|
| Kiro EARS | requirements/design/tasks |
| Spec Kit | spec/plan/tasks + constitution |
| BMAD | PRD + architecture spine |
| Amazon PR/FAQ | Headline, Customer FAQ, Internal FAQ; BMAD's `prfaq-template.md` is a ready reference |
| OpenSpec | proposal/delta spec |
| SuperMatt `to-spec` | Problem, Solution, User Stories, Implementation Decisions, Testing Decisions, Out of Scope; published to the issue tracker |
