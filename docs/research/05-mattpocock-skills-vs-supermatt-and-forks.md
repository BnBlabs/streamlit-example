# mattpocock/skills vs SuperMatt and other derivatives: what to build a product-team add-on on

Research date: 2026-11-04. This follows `01-supermatt-and-extension-model.md` (cited as **R01**) and `04-portability-and-aws-work-use.md` (cited as **R04**).

**Sources**

| Short name | Source |
|---|---|
| `UP/` | `mattpocock/skills`, history fetched in full with `--filter=blob:none`. `origin/main` = `4ae2317` (2026-10-31). Tag `v1.3.2` = `d1478fc` (2026-10-29). Main is 3 commits ahead of the tag, and all 3 touch only `in-progress/` and `implement-spec`. |
| `SM/` | `svyatov/supermatt` @ `58dabac` (v0.23.0, 2026-10-28). |
| Installed SuperMatt | v0.22.0. It has 39 skills, which is v0.23.0 minus `babysit-pr` and `recover-ci`. |
| `VS/` | `vercel-labs/skills` @ `9197d91` (CLI v1.7.6, 2026-11-03). |
| `CC/` | Claude Code docs snapshot. |
| GitHub data | From `gh api search/*`. Per-repo REST endpoints were refused by the session proxy, so star counts and dates come from search results. |

Labels: **[F]** fact with citation, **[I]** inference, **[R]** recommendation.

**Corrections to R01**

1. **[F]** Upstream renamed `CONTEXT.md` to `GLOSSARY.md` itself, in `d80fa0f` (2026-08-15, merged as `e484a80` on 2026-09-24). SuperMatt's "Renamed: `CONTEXT.md` is `GLOSSARY.md`" row (`SM/README.md` Scope table) is relative to its fork point. It is not a divergence from current upstream.
2. **[F]** SuperMatt v0.23.0 has 41 skills: 27 user-invoked and 14 model-invoked. `recover-ci` is a new model-invoked skill.
3. **[F]** The official Claude marketplace does **not** serve v1.3.2 (see section 5).

---

## 1. Skill inventory side by side

### 1.1 Inventory

The **U** column marks user-invoked skills (`disable-model-invocation: true`). The frontmatter of every `SKILL.md` was grepped.

| Upstream v1.3.2 (bucket) | U | SuperMatt v0.23.0 | U | Notes |
|---|---|---|---|---|
| ask-matt (eng) | U | ask-supermatt | U | Router; renamed |
| code-review (eng) | | code-review | | 2 axes upstream vs 3 axes plus external peer (section 1.2) |
| codebase-design (eng) | | codebase-design | | SM adds `RED-FLAGS.md` and `STATE-CONTRACTS.md` |
| diagnosing-bugs (eng) | | diagnosing-bugs | | SM adds `BUG-CLASSES.md` and `PERFORMANCE.md`; missing upstream's forced-mutation proof (section 1.3) |
| domain-modeling (eng) | | domain-modeling | | Near-identical (+5/−3 lines) |
| grill-with-docs (eng) | U | grill-with-docs | U | Wording only |
| implement (eng) | U | implement | U | 17 → 48 lines; heavy functional change |
| implement-spec (eng) | U | implement-spec | U | Reworked (+32/−21) |
| improve-codebase-architecture (eng) | U | improve-architecture | U | Renamed and reworked |
| pr (eng) | | pr | | Cosmetic (±17) |
| prototype (eng) | | prototype | | Small functional additions |
| research (eng) | | research | | Small |
| retro (eng) | U | retro | U | SM adds transcript scripts |
| setup-matt-pocock-skills (eng) | U | setup-supermatt-skills | U | SM adds `cross-model.json` |
| tdd (eng) | | tdd | | SM adds a mutation check, characterization tests and scripts |
| to-spec (eng) | U | to-spec | U | SM adds fresh-context review (section 1.2) |
| to-tickets (eng) | U | to-tickets | U | SM adds Seams, `## Parent` and direct-blocker rules |
| triage (eng) | U | triage | U | SM adds delegated-decision mode |
| wayfinder (eng) | U | wayfinder | U | SM adds coordinator claims |
| wizard (eng) | | (none) | | Not in SM |
| grill-me (prod) | U | grill-me | U | Wording |
| grilling (prod) | | grilling | | Behavioral change (section 1.2) |
| handoff (prod) | U | handoff | U | Small |
| teach (prod) | U | teach | U | Small; `GLOSSARY-FORMAT.md` renamed `TERMS-FORMAT.md` |
| to-questionnaire (prod) | U | to-questionnaire | U | Small |
| wait-what (prod) | U | wait-what | U | Wording |
| writing-for-agents (prod) | | writing-for-agents | | Near-identical |
| *in-progress, not shipped in the plugin:* chief-of-staff, claude-handoff, loop-me, setup-ts-deep-modules, software-factory, writing-beats, writing-fragments, writing-shape | all U | (none) | | SM "dropped … the rest of its in-progress set" |
| *misc (frozen):* git-guardrails-claude-code, migrate-to-shoehorn, scaffold-exercises, setup-pre-commit | | (none) | | |
| | | **SM-only:** ideate, orchestrate, refactor, improve-tests, improve-file-structure, commit, ship-pr, babysit-pr, fix-findings, what-would-you-do, jury | U | |
| | | **SM-only:** qa, recover-ci, dependency-vetting | | |
| | | resolving-merge-conflicts | | Upstream removed it in `daa01d8` (2026-09-24); SM kept it |

Totals:

- **[F]** **Upstream:** 27 shipped skills (16 user-invoked, 11 model-invoked), listed explicitly in `UP/.claude-plugin/plugin.json`.
- **[F]** **SuperMatt:** 41 skills (27 user-invoked, 14 model-invoked).

**Names you asked about**

- **[F]** `to-prd` and `to-issues` no longer exist. Commit `386d4ff` (2026-07-02, "unify planning skills into /to-spec + /to-tickets") renamed `to-prd` to `to-spec` (78% similarity). Older `.kiro/skills/to-prd` copies found on GitHub (section 4) are pre-July snapshots.
- **[F]** **`software-factory`** (in-progress, user-invoked) is a *glossary*: "the language for designing [a software factory]", with terms Task, Trigger, Run, Runner, Ticket tracker, Orchestrator and others. It also has reference files: `EXAMPLE-TASKS.md`, `RUNNERS.md`, `TICKET-TRACKERS.md`, `GIT-HOSTS.md` and `OBSERVABILITY-PLATFORMS.md` (`UP/skills/in-progress/software-factory/SKILL.md`, added in `4ae2317`).
- **[F]** **`chief-of-staff`** (in-progress, user-invoked) is a long-running coordinator. It works through background subagents and keeps a dated, commit-cited wiki. Its "Procedures" step runs other skills **by reading their files**: "write a spec with `../to-spec/SKILL.md`, break it into tickets with `../to-tickets/SKILL.md` … implement them with `../implement-spec/SKILL.md`" (lines 53–54).
- **[I]** This is upstream's own workaround for the rule that user-invoked skills can't be chained, which section 2 covers. It is the closest upstream analogue to an orchestrating "virtual team lead".
- **[F]** In-progress skills install only through `npx skills@latest add mattpocock/skills --skill=<name>` (`UP/skills/in-progress/README.md`).

### 1.2 SKILL.md diffs: what SuperMatt actually changes

Method: `diff -u UP/skills/<bucket>/<name>/SKILL.md SM/skills/<name>/SKILL.md`. Added and removed line counts per skill: code-review +104/−24, implement +25/−7 (the file grew from 17 to 48 lines), to-tickets +22/−12, grilling +19/−7, to-spec +17/−6, prototype +4/−3, grill-with-docs +4/−2.

**grill-with-docs: cosmetic**

- **[F]** Upstream: "Call the Skill tool twice, for "grilling" and "domain-modeling"."
- **[F]** SuperMatt: "Load and follow the `grilling` and `domain-modeling` skills, each through the host's skill mechanism."
- **[F]** SuperMatt also adds `argument-hint` and `license: MIT`. Behavior is the same.

**grilling: functional, and against upstream policy**

- **[F]** SuperMatt caps a round at "at most four frontier questions". It adds a "smallest option: do nothing, reuse what exists, or defer" rule.
- **[F]** It adds a **delegated mode**, which only `triage` may select, where the agent records its own decisions instead of asking.
- **[F]** It routes questions through native tools: "**Claude Code:** `AskUserQuestion`", plus Codex, Cursor and OMP equivalents.
- **[F]** Upstream explicitly rejected both the question cap and the native question tool. See `UP/.out-of-scope/question-limits.md` and `native-question-tool.md` line 9: "I've tried the `AskUserQuestion` UI and don't want it in these skills". Upstream says to put that preference in your own `CLAUDE.md`.
- **[F]** SuperMatt also drops upstream's v1.3.2 line "Word each question so "yes" accepts your recommended answer" (`95249b0`).

**to-spec: functional**

- **[F]** SuperMatt reads `docs/agents/issue-tracker.md` and `triage-labels.md` explicitly, and stops if either is missing.
- **[F]** It accepts a `wayfinder` map reference.
- **[F]** It does not publish straight away. Its new **step 4** checks the draft in a fresh context: "Spawn one sub-agent with the draft … Report: (a) places where the spec contradicts itself … (b) implementation decisions that the current codebase cannot support … (c) user stories that no implementation or testing decision covers …". Findings that need a decision go to the user before the spec is published.
- **[F]** It adds contract and fixture rules for independently deployed provider/consumer pairs, and a conditional `## Migration map` section.
- **[F]** Testing Decisions gain "The seams under test, as agreed with the user in step 2".
- **[F]** The spec template headings are otherwise unchanged.

**to-tickets: functional**

- **[F]** Exploration moves into a fresh read-only subagent.
- **[F]** Only *direct* blockers are listed.
- **[F]** Every child gets a `## Parent` reference and a **Seams** section, and the source spec loses its `ready-for-agent` label.
- **[F]** The skill ends by telling the user to run `/implement` on a frontier ticket.
- **[F]** Upstream's v1.3.2 change "make each ticket its sub-issue" (`cffab50`) is **absent** from SuperMatt.

**implement: heavy functional change**

- **[F]** Upstream's whole body is 6 instructions: fetch the ticket, "Call the Skill tool with "tdd"…", typecheck regularly, "call the Skill tool with "code-review"", commit.
- **[F]** SuperMatt adds:
  - frontier-ticket selection over a parent spec;
  - criterion-conflict escalation;
  - PR-branch handling;
  - a recorded base commit;
  - a full gate before each slice commit;
  - **`qa` in a fresh subagent**, then `code-review`;
  - "Fix every verified finding … at every severity";
  - leaving issues open until merge through `/ship-pr`;
  - refactor and characterization-test rules;
  - follow-up review rounds.

**prototype: small functional change**

- **[F]** SuperMatt adds a guarded `localStorage` probe with JSON export/import for file-URL demos.
- **[F]** It names the branch `prototype/<name>` and pushes it.
- **[F]** It points to the decision ticket, or to the handoff when there is no issue yet.

**code-review: largest change**

- **[F]** Upstream has two axes, Standards and Spec, run as parallel sub-agents with an inline Fowler smell baseline.
- **[F]** SuperMatt adds a third axis, **Adversarial** (`ADVERSARIAL.md`), and moves the baseline into `STANDARDS.md` with added refactor and test checks.
- **[F]** It writes the diff, including untracked files, to `$DIR/diff.patch`.
- **[F]** It runs a host reader and an optional **external reader** per axis, chosen through `docs/agents/cross-model.json`.
- **[F]** A cross-reader validator (`VALIDATOR.md`) and P0–P3 severities complete the change.
- **[F]** SuperMatt lacks upstream's v1.3.2 rules. Upstream's text is "When `CODING_STANDARDS.md` or `CONTRIBUTING.md` exists, it must be on the list" plus "Start every sub-agent with a fresh context window" (`3da8c01`, `7739eae`). SuperMatt has only the weaker "such as `CODING_STANDARDS.md` or `CONTRIBUTING.md`".

**Overall characterization [I]**

- About 20% of SuperMatt's changes are renames and cosmetic edits: names, `license:`, `argument-hint`, and multi-host invocation strings repeated in every skill.
- About 80% are real behavior:
  - independent-context checks (spec review, QA, adversarial review);
  - explicit artifact handoffs (Seams, `## Parent`, `prototype/<name>`);
  - a ship tail (ship-pr, babysit-pr, recover-ci).
- Nearly all of that value is **post-spec engineering rigor**. None of it adds PM, PMM, customer, TPM or SDM capability.

### 1.3 Sync lag

- **[F]** The fork point `c55ee46` is dated 2026-09-18. There are **78 upstream commits** between it and v1.3.2.
- **[F]** These v1.3.2 fixes are verifiably missing from `SM/` (grep for the fix text):
  - tdd's "what each proposed seam catches and misses" (`3f59913`);
  - to-tickets sub-issues (`cffab50`);
  - code-review fresh-context and standards-file rule (`3da8c01`, `7739eae`);
  - diagnosing-bugs "prove the mutation landed" (`f3fc563`);
  - grilling's "yes accepts" wording (`95249b0`).
- **[F]** SuperMatt's changelog describes its imports one by one ("Imported from mattpocock/skills", "adapts upstream v1.3.1's whole-spec workflow"). It has no sync tooling.

---

## 2. `disable-model-invocation: true` and what an orchestrator can chain

**Upstream, user-invoked (16 shipped plus all 8 in-progress):** ask-matt, grill-with-docs, implement, implement-spec, improve-codebase-architecture, retro, setup-matt-pocock-skills, to-spec, to-tickets, triage, wayfinder, grill-me, handoff, teach, to-questionnaire, wait-what. In-progress: chief-of-staff, claude-handoff, loop-me, setup-ts-deep-modules, software-factory, writing-beats, writing-fragments, writing-shape.

**Upstream, model-invoked (11 shipped):** code-review, codebase-design, diagnosing-bugs, domain-modeling, pr, prototype, research, tdd, wizard, grilling, writing-for-agents. The 4 misc skills are also model-invoked.

**SuperMatt, user-invoked (27):** the same 16 under SuperMatt names, plus ideate, orchestrate, refactor, improve-tests, improve-file-structure, commit, ship-pr, babysit-pr, fix-findings, what-would-you-do and jury.

**SuperMatt, model-invoked (14):** code-review, codebase-design, dependency-vetting, diagnosing-bugs, domain-modeling, grilling, pr, prototype, qa, recover-ci, research, resolving-merge-conflicts, tdd, writing-for-agents.

**The rule is the same in both repos**

- **[F]** Upstream: "A user-invoked skill may invoke model-invoked skills, but it can never reach another user-invoked skill" (`UP/.agents/invocation.md:8`).
- **[F]** SuperMatt: "Only model-invoked skills can normally be dependencies" (`SM/.agents/invocation.md:20`). Its one exception is `orchestrate`, which types commands into Herdr worker panes.
- **[F]** Claude Code enforces the rule: "If Claude tries anyway, Claude Code blocks the call and instructs it not to reproduce the deploy steps another way" (`CC/skills.md`).

**What this means for an add-on [I]**

- In Claude Code, role agents and skills can call `grilling`, `domain-modeling`, `prototype`, `research`, `code-review` and `tdd` from either repo, plus `qa` from SuperMatt.
- They **cannot** invoke `to-spec`, `to-tickets`, `implement`, `grill-with-docs` or `wayfinder` in either repo. SuperMatt does not loosen this.
- Upstream's `chief-of-staff` gets around the rule by reading sibling `SKILL.md` files by path. That works only when skills sit side by side, as with `npx skills` installs, not across plugin caches. It also runs against the spirit of Claude Code's block message.
- **[F]** Kiro does not document `disable-model-invocation` (R04 §2). The same skills may therefore auto-trigger, and be chainable, in Kiro.

---

## 3. Adapters

### 3.1 SuperMatt

SuperMatt has three kinds of adapter.

**(1) Host adapters**

- **[F]** SuperMatt ships one flat `skills/` tree with `.claude-plugin/`, `.cursor-plugin/plugin.json` and `.agents/plugins/marketplace.json` (Codex). OMP loads the Claude metadata "through its documented compatibility fallback".
- **[F]** Every skill also carries `agents/openai.yaml`, and **upstream does too**.
- **[F]** `references/harnesses.md` maps human syntax per host: `/supermatt:<name>`, `/<name>`, `$<name>`, `/skill:<name>`.
- **[F]** That file also maps model-side loading per host, for example "On OMP, read `skill://<name>`", plus capability fallbacks.

**(2) Peer-review adapters**

- **[F]** `docs/agents/cross-model.json` selects `auto | host-only | claude | codex | cursor | omp`, with model and effort per CLI and strict validation.
- **[F]** `code-review` and `jury` then shell out to another vendor's CLI as an independent reader. They use fixed argument vectors and confirmation prompts for Cursor and OMP (`SM/references/harnesses.md` "External peer resolver").

**(3) Issue-tracker adapters**

- **[F]** These are the same as upstream: GitHub, GitLab, local, or "Other" as prose in `docs/agents/issue-tracker.md`.
- **[F]** SuperMatt's skills *read* those files explicitly. Upstream's skills say "The issue tracker … should have been provided to you" and rely on the `## Agent skills` block that setup writes into `CLAUDE.md` or `AGENTS.md`.

### 3.2 What the adapters buy over upstream

- **[I]** Host adapters buy little. Upstream already ships official managed installs for Claude Code, Codex, Copilot CLI and VS Code, plus `npx skills` for 75+ other agents (`UP/README.md` install section; `UP/.agents/install-block.md`).
  - SuperMatt adds a native Cursor plugin and OMP.
  - Upstream reaches Cursor through `npx skills -a cursor`, and an unofficial OMP distribution exists (`xloouis/mattpocock-skills-unofficial-plugin`).
- **[I]** The peer-review adapter is SuperMatt's one genuinely unique capability: cross-vendor second opinions in review and jury. It costs a second CLI, its auth, and data egress to that vendor.
- **[F]** Upstream's equivalent setup is `setup-matt-pocock-skills`. It writes the `## Agent skills` block, `docs/agents/issue-tracker.md`, `docs/agents/triage-labels.md` (when `triage` is installed) and `docs/agents/domain.md`.
- **[F]** That setup offers to `git mv` a legacy `CONTEXT.md` to `GLOSSARY.md` (`UP/skills/engineering/setup-matt-pocock-skills/SKILL.md`, from `ed0de59`).
- **[F]** Its only missing piece relative to SuperMatt is `cross-model.json`.

### 3.3 Kiro

- **[F]** `grep -ri kiro` returns nothing in either repo.
- **[F]** Kiro support comes **only through `npx skills`**. Its agent table has an entry `'kiro-cli': { skillsDir: '.kiro/skills', globalSkillsDir: ~/.kiro/skills }` (`VS/src/agents.ts:476-483`), and `VS/docs/agents.md:47,80-86` documents it.
- **[F]** Kiro custom agents must add `"resources": ["skill://.kiro/skills/**/SKILL.md"]`.
- **[F]** The compatibility table says Kiro CLI does **not** support `allowed-tools`, and does support hooks.
- **[I]** SuperMatt's flat layout is also installable this way, through `npx skills add svyatov/supermatt -a kiro-cli`. Its host-specific prose assumes Claude, Codex, Cursor or OMP, though, and its peer resolver has no Kiro branch.

---

## 4. Other derivatives on GitHub

**Scale**

- **[F]** `mattpocock/skills` has **313,215 stars and 26,278 forks**, last pushed 2026-11-04. `svyatov/supermatt` has **20 stars and 6 forks**, last pushed 2026-10-28, and is not a GitHub fork.
- **[F]** Repository search for "mattpocock skills" returns 399 repos.
- **[F]** The most-starred true forks (`fork:only`, upstream's description) are small: utarn/engineer-skills (49), ninehills (47), Pipedrive-OSS (16, pushed 2026-10-14).

**Notable derivatives**

| Repo | Stars | Last push | What it adds |
|---|---|---|---|
| vinvcn/mattpocock-skills-zh-CN | 5,536 | 2026-10-28 | Simplified-Chinese translation. Each upstream sync is translated by Codex and reviewed by a human. Adds a DeepSeek Harness (dsh) plugin. |
| devcxl/mattpocock-skills-zh | 529 | 2026-11-03 | Chinese translation that claims a daily upstream sync |
| FeatherHunter/dsh-mattpocock-skills-deck | 286 | 2026-11-04 | DSH plugin bundling the 27 v1.3.2 skills |
| tt-a1i/matt-skills-with-to-goal | 193 | 2026-10-30 | Fork at "v1.3.2-to-goal.1". Adds `to-goal`, `spec-executor` and `execute-spec-in-fork`, which compress context into a verifiable goal contract and run it in a forked session that returns evidence. |
| alpcanaydin/kuzgun | 27 | 2026-11-03 | macOS kanban viewer for mattpocock tickets |
| CoreyLyn/skills | 7 | 2026-10-28 | `autopilot-spec`: runs upstream `implement-spec`, then acceptance, review, fix and auto-merge |
| whutlichao/orchestra-auto | 9 | 2026-11-02 | Orca supervisor that keeps upstream's chain (grill-with-docs → to-spec → to-tickets → implement → code-review) with per-step worktrees and gates |
| toomanyopenfiles/pi-matt-implement-flow | 4 | 2026-10-30 | Parallel `/implement` successor for the pi agent |
| hsyam/matt-skills-ui | 5 | 2026-10-07 | Claude Code plugin that shows specs, tickets and wayfinder maps as a graph |
| xiayuzizhuo666/pm-skills | 1 | 2026-09-03 | **PM-role port**: `prd-grill`, `product-glossary`, PDRs in `docs/pdr/`, `acceptance-criteria` ("the PM equivalent of TDD"), and `prd-review` (Standards + Intent) |
| xloouis/mattpocock-skills-unofficial-plugin | 1 | 2026-10-30 | OMP plugin |
| AppleCG/devflow | 9 | 2026-07-08 | Fuses superpowers, mattpocock and OpenSpec |

**Kiro usage**

- **[F]** Code search for `"mattpocock/skills" path:.kiro` returns 36 files. Example: `Udaymore741/Security_Boat_Round2_Task/.kiro/skills/{to-prd,to-issues,grilling,…}/SKILL.md`, an old snapshot.
- **[F]** Another example is `allenatallahopen-wq/agent-skills/.kiro/steering/skills.md`.
- **[I]** People drop upstream copies straight into `.kiro/skills/`, almost certainly through `npx skills`. No maintained Kiro-specific port exists.

**Marketplaces**

- **[F]** `"mattpocock-skills" filename:marketplace.json` returns 158 hits. Most are vendored copies (`…/sources/mattpocock-skills/`).
- **[F]** The canonical listing is `anthropics/claude-plugins-official`.

**Roles [I]**

- No derivative adds a multi-role product team.
- `pm-skills` is the only role-shaped port. It is tiny and a rewrite, not an add-on.
- `tt-a1i` and `CoreyLyn` show the composable pattern Matt endorses: separate skills that build on upstream's artifacts.

---

## 5. `npx skills` (vercel-labs/skills) and update channels

**How `npx skills` installs**

- **[F]** `npx skills@latest add mattpocock/skills -a <agent>` (`UP/README.md`). Project scope installs to `./<agent>/skills/`, and `-g` installs to `~/<agent>/skills/`.
- **[F]** The default method is a **symlink** from each agent directory to one canonical copy. `--copy` makes independent copies.
- **[F]** A test asserts `.kiro/skills/<name>` is a symlink to `.agents/skills/<name>` (`VS/src/add.test.ts:226-236`). **[I]** Check that Kiro follows symlinks, and use `--copy` if it doesn't.
- **[F]** Discovery walks `skills/` up to 3 levels deep, so upstream's bucketed layout works. It also reads `.claude-plugin/plugin.json` and `marketplace.json` (`VS/docs/creating-skills.md`).

**Updates**

- **[F]** `npx skills check` exits 1 when updates exist, so it can gate CI. `npx skills update [-p|-g] [names]` re-installs any skill whose upstream folder hash differs from the lock (`~/.agents/.skill-lock.json` or project `skills-lock.json`; `VS/src/update.ts:1033-1060`).
- **[I]** Updates overwrite local edits with no merge.
- **[F]** Upstream says to "re-run `add` to pick up new skills".

**Claude Code update channel**

- **[F]** The official marketplace entry is `{"source":"url","url":"https://github.com/mattpocock/skills.git","sha":"49dd158…"}`, fetched from `raw.githubusercontent.com/anthropics/claude-plugins-official/main/.claude-plugin/marketplace.json` on 2026-11-04.
- **[F]** `49dd158` (2026-10-09) carries `plugin.json` version **1.3.1**. It is 5 commits behind v1.3.2.
- **[F]** Upstream's own ADR says so: "the **sha is pinned**, so a release reaches installed users when that pin moves, not the moment we tag" (`UP/.agents/adr/0002-…md:40`).
- **[I]** "Auto-updates by default" therefore means auto-updates *to Anthropic's pin*, which lags by weeks.
- **[I]** A marketplace you control, listing `mattpocock/skills` with `autoUpdate` on, would track upstream releases sooner. You would own that cadence.

---

## 6. Upstream extensibility

**What upstream says about customization**

- **[F]** SCOPE.md:10: config options, harness-specific branches and per-person tweaks "belong in your own `CLAUDE.md` / `AGENTS.md`, or in a fork (skills.sh gives you an editable copy)".
- **[F]** `.out-of-scope/new-skills.md:9`: "that composition belongs in your own prompts or your own skills. You're very welcome to publish a skill yourself."
- **[F]** Pull requests are collaborator-only (CONTRIBUTING.md).
- **[F]** Rejected ideas: renaming skills to avoid name collisions (use `/mattpocock-skills:code-review`), more first-class trackers (use "Other"), native question tools (use `CLAUDE.md`), and question caps.

**Config surface**

- **[F]** The only config is the prose files written by setup: `docs/agents/issue-tracker.md`, `triage-labels.md` and `domain.md`, referenced from the `## Agent skills` block.
- **[F]** Skills also read `GLOSSARY.md` (named in 9 SKILL.md files) and ADRs under `docs/adr/`.
- **[F]** Skills also read `CODING_STANDARDS.md` (named in 3 files). code-review must include it, and `chief-of-staff` recommends it.
- **[F]** There is no machine-readable config, hook point or template override. The `to-spec` template is inline in `<spec-template>`.

**Augmenting without forking [I]**

These approaches work in both repos:

1. **Repo docs that skills already read.**
   - Role rules in `CODING_STANDARDS.md` get enforced by code-review's Standards axis.
   - Product vocabulary in `GLOSSARY.md` is used by to-spec and to-tickets.
   - Product decisions recorded as ADRs are respected "in the area you're touching".
   - Extra prose in the `## Agent skills` block of `CLAUDE.md` or `AGENTS.md` is read too. That block is the one Kiro also reads, through `AGENTS.md`.
2. **Your own skills** that consume upstream's artifacts: the spec issue or `.scratch/<feature>/spec.md`, and tickets with blocking edges.
3. **Claude Code only:** a `UserPromptExpansion` hook on `mattpocock-skills:to-spec` and similar commands, to inject role addenda (R01 §4).

---

## Recommendation

**[R] Use a hybrid: consume upstream mattpocock/skills unmodified as the base, build the product team as your own add-on, and port at most a few SuperMatt skills into the add-on with attribution. Do not base on SuperMatt.**

### Option 1: upstream directly (the base of the recommendation)

**(a) Staying updated: best.**

- In Claude Code, install `mattpocock-skills@claude-plugins-official`, which auto-updates to Anthropic's pin (it currently lags: v1.3.1 at `49dd158`).
- Alternatively, list `mattpocock/skills` in your own marketplace with `autoUpdate` on.
- Your add-on plugin declares `"dependencies":["mattpocock-skills"]`. A cross-marketplace dependency needs `allowCrossMarketplaceDependenciesOn: ["claude-plugins-official"]` in your marketplace, or the user installs it first (R01 §4).
- In Kiro, run `npx skills add mattpocock/skills -a kiro-cli`, then `npx skills check` in CI and `npx skills update`.

**(b) Kiro portability: good.**

- Upstream's prose is deliberately harness-neutral. It has no host branches, and grilling stays in plain chat.
- `npx skills` targets `.kiro/skills/` natively.
- **Caveat:** Kiro may ignore `disable-model-invocation`.

**(c) Functional value lost compared with SuperMatt:**

- the fresh-context spec review;
- the `qa` step and the Adversarial review axis;
- the ship tail;
- `jury` and `ideate`;
- cross-vendor peers.

None of these are product roles. Your PM, Customer and PMM reviewers can supply a better, role-specific version of the spec review.

### Option 2: SuperMatt as the base

**(a) Staying updated:** poor.

- Single maintainer, 20 stars.
- 78 upstream commits since the fork, with at least 5 v1.3.2 fixes missing.
- About 30 releases a month means churn, and it is not in the official marketplace, so there is no default auto-update.

**(b) Kiro portability:** weaker. Every flow skill embeds Claude, Codex, Cursor and OMP invocation strings, and `cross-model.json` has no Kiro peer.

**(c) Functional gains:** real, but engineering-only. The user-invoked chaining limit is unchanged, so it doesn't help orchestration.

**Divergence from upstream intent:** grilling's `AskUserQuestion` routing and four-question cap contradict decisions upstream recorded explicitly.

### Option 3: what to port from SuperMatt

**Prefer `jury`.** It is the most product-relevant SuperMatt skill: a blind 3- or 5-seat panel with Pre-mortem, First-principles, Cost, Outsider and Evidence lenses. It maps directly onto a PM/SDM decision forum. Port it as a renamed skill, for example `product-team:decision-jury`:

- strip the `cross-model.json` resolver and keep `host-only`;
- keep `SM/LICENSE` (MIT, © Matt Pocock and Leonid Svyatov) in a per-skill `NOTICE`;
- consider making it model-invoked so your role agents can call it.

**Optionally `ideate`** for PM discovery. It needs `docs/agents/domain.md` and `issue-tracker.md`, which upstream setup already writes.

**Port `qa` only if needed.** It is coupled to `../code-review/SKILL.md` and `references/scratch-dependencies.md`, and it shells out to `codex` or `claude -p` sandboxes. It is engineering, not product. If you want it, depend on it at runtime rather than copying.

**Naming and maintenance**

- Keep every ported skill under a name upstream doesn't use. Kiro's `.kiro/skills/` is flat with no namespacing, so a second `code-review` or `grilling` collides.
- Treat ports as your own code. Pin the source commit, here `58dabac`, in the NOTICE.

### Concrete shape of the add-on [R]

**One repo** containing:

- `skills/` (Agent Skills frontmatter only);
- `.claude-plugin/plugin.json` with a dependency on `mattpocock-skills`;
- `agents/` for the Claude Code role subagents;
- `kiro/agents/*.json` generated from the same persona bodies.

**Distribution**

- Claude Code: install the plugin.
- Kiro: `npx skills add <you>/product-team -a kiro-cli`, next to `npx skills add mattpocock/skills -a kiro-cli`.

**Role skills consume upstream's artifacts.** Examples: `prd-review` runs after `/to-spec`, `persona-interview` feeds `grilling`, and `release-readiness` reads ticket state. They put durable product rules into `GLOSSARY.md`, ADRs, `CODING_STANDARDS.md` and `AGENTS.md`, which upstream skills already honor.

**Hand off to the user for upstream's user-invoked flow skills** ("now run `/to-spec`"). Use a Claude Code `UserPromptExpansion` hook only as an optional enhancement. Watch `software-factory` and `chief-of-staff`: if they graduate, they become upstream's own orchestration layer to build on.

**Borrow from `xiayuzizhuo666/pm-skills`** (MIT): the PDR and acceptance-criteria ideas are useful prior art for the PM role. Copied text needs a NOTICE.
