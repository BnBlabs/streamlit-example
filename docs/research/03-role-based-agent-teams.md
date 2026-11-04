# 03: Role-based agent teams for a virtual product team

Research date: 2026-11-04.

**Read directly** (the strongest sources). Shallow clones in `/tmp/claude-0/research-clones/`, taken 2026-11-04:

- `FoundationAgents/MetaGPT` @ `11cdf46` (2026-01-21)
- `OpenBMB/ChatDev` `main` @ `464a681` (2026-10-30), plus the `chatdev1.0` branch
- `FSoft-AI4Code/AgileCoder`, `Md-Ashraful-Pramanik/MapCoder` and `multi-agent-systems-failure-taxonomy/MAST`
- `wshobson/agents` @ `7e2fb55` (2026-10-30)
- `VoltAgent/awesome-claude-code-subagents` @ `ed74d11` (2026-11-02)
- `contains-studio/agents` @ `a5a480c` (2025-07-28)
- `anthropics/skills` @ `ca45fc4`
- `anthropics/knowledge-work-plugins` @ `d900840` (2026-11-03)
- `deanpeters/Product-Manager-Skills` @ `cfcb697` (v0.84)
- `phuryn/pm-skills` @ `2a12295`
- `bmad-code-org/BMAD-METHOD` @ `022be93`

I also fetched pages from `anthropic.com/engineering` and `code.claude.com/docs` on the same day.

**Access limit.** This session's network policy blocked arxiv.org, openreview.net, iclr.cc, mlanthology.org and cognition.ai, so I could not open the papers or Cognition's essay. Claims from those sources come from web-search summaries of the pages and carry the label **[S]**. Check them against the PDFs before quoting numbers externally. To close the gap, allow those hosts and re-run the check.

**Labels:**

- **[F]** Fact, read directly from a cited primary source.
- **[S]** Fact taken from a search-engine summary of the cited source. Lower confidence.
- **[I]** Inference.
- **[R]** Recommendation.

---

## 1. "Software company" multi-agent systems: what they claim and what the evidence supports

### 1.1 MetaGPT: `Code = SOP(Team)`

[F] The README states the core philosophy as "`Code = SOP(Team)`... We materialize SOP and apply it to teams composed of LLMs." It describes the team as "product managers / architects / project managers / engineers" running "the entire process of a software company along with carefully orchestrated SOPs" (`MetaGPT/README.md` l.43–44).

[F] Each role is a short profile, and its output is a typed document schema rather than free chat:

- **ProductManager ("Alice").** Goal: "Create a Product Requirement Document or market research/competitive product research" (`metagpt/roles/product_manager.py` l.32–35). Its PRD schema has these fields: `Original Requirements, Product Goals, User Stories, Competitive Analysis, Competitive Quadrant Chart, Requirement Analysis, Requirement Pool, UI Design draft, Anything UNCLEAR` (`metagpt/actions/write_prd_an.py` l.190–203).
- **Architect.** Produces `Implementation approach, File list, Data structures and interfaces, Program call flow, Anything UNCLEAR` (`design_api_an.py`).
- **ProjectManager ("Eve").** Its job is to "break down tasks according to PRD/technical design, generate a task list, and analyze task dependencies" (`project_manager.py` l.24–30). Its outputs are packages, logic analysis, task list, API spec and shared knowledge (`project_management_an.py`).

[S] The paper was an ICLR 2024 oral. It reports 85.9% and 87.7% Pass@1 on HumanEval and MBPP, and executability of 3.75 out of 4 on its own SoftwareDev set. In the paper's ablation, adding roles beyond a lone Engineer improved both executability and the number of human revisions needed. Search summaries disagree about the exact Table 1 and Table 3 cells, including whether "2.25" is ChatDev's executability or MetaGPT's revision cost without feedback. Do not quote those cells until they are checked in the PDF (sources: [arXiv 2308.00352](https://arxiv.org/abs/2308.00352), [ICLR slides](https://iclr.cc/media/iclr-2024/Slides/18491.pdf)).

[F] **The MetaGPT codebase has itself moved away from the fixed SOP pipeline.**

- `RoleZero.use_fixed_sop` now defaults to `False` (`metagpt/roles/di/role_zero.py` l.98). Roles therefore act in a tool-using ReAct loop instead of a fixed action sequence.
- The default `generate_repo` team is `TeamLeader, ProductManager, Architect, Engineer2, DataAnalyst`. `ProjectManager()` and `QaEngineer` are commented out (`metagpt/software_company.py` l.44–62).

[I] The project that coined `Code = SOP(Team)` now ships a lead agent and fewer, more capable roles. The fixed role chain is no longer its default.

### 1.2 ChatDev 1.0 (chat chain) and 2.0 (DevAll)

[F] ChatDev 1.0, now on the `chatdev1.0` branch, is a "Virtual Software Company" with these recruited roles: CEO, Counselor, CHRO, CPO, CTO, Programmer, Code Reviewer, Software Test Engineer and Chief Creative Officer (`CompanyConfig/Default/ChatChainConfig.json`).

- The chain is `DemandAnalysis → LanguageChoose → Coding → CodeCompleteAll(×10) → CodeReview(×3) → Test(×3) → EnvironmentDoc → Manual`.
- Each phase is a two-agent dialogue. For example, DemandAnalysis is CPO ← CEO, and CodeReviewComment is Code Reviewer ← Programmer (`PhaseConfig.json`).
- A "Human" config adds a `HumanAgentInteraction` phase after code review: "you can play as a reviewer and ask the programmer agent to modify the code" (`wiki.md` l.402–406). This is the only place a real customer enters the loop.

[F] ChatDev 2.0 (DevAll), released Jan 7, 2026, drops the software-company framing. It describes itself as "a Zero-Code Multi-Agent Platform for 'Developing Everything'", and the classic company has moved to a maintenance branch (`ChatDev/README.md`). The team's later work centers on learned orchestration: the "puppeteer" paper (NeurIPS 2025) uses "a learnable central orchestrator optimized with reinforcement learning" that "dynamically activates and sequences agents".

[S] The ChatDev paper (ACL 2024) reports that removing role prompts gave the largest ablation drop. Quality fell from 0.3953 to 0.2212, and the column that appears to be executability fell from 0.88 to 0.58. Its qualitative examples: a programmer told to "prefer GUI design" writes GUI code, and without a "careful reviewer" role, review feedback stays high-level ([ACL Anthology 2024.acl-long.810](https://aclanthology.org/2024.acl-long.810.pdf)).

[I] This evidence supports *specific instructions per phase*, such as "prefer GUI" or "hunt for bugs". It does not show that those instructions need separate agents. A single agent given the same instruction was not tested.

### 1.3 AgileCoder and MapCoder

[F] AgileCoder's README says that "instead of assigning fixed roles to agents, AgileCoder mimics real-world software development by creating a backlog of tasks and dividing the development process into sprints". On its own ProjectDev set, human-rated executability was ChatDev 32.79, MetaGPT 7.73 and AgileCoder 57.79. AgileCoder used about 5× ChatDev's tokens (36,818 vs 7,440) and about 3.7× its time (444 s vs 120 s) (`AgileCoder/README.md`).

[I] The MetaGPT score here sits far below its self-reported 3.75/4. Each team's own benchmark favors its own system, so treat all of these "company" benchmarks as weak evidence.

[F] MapCoder (ACL 2024) uses four *process-stage* agents (Retrieval, Planning, Coding, Debugging), not job titles. It reports HumanEval 93.9% and MBPP 83.1%. The README now says "We will no longer maintain this repository" (`MapCoder/README.md`).

[I] The strongest coding results come from roles defined by *pipeline function* (plan, code, debug, verify), not by organizational title.

### 1.4 Contrary evidence and critiques

[F] **MAST ("Why Do Multi-Agent LLM Systems Fail?")** defines 14 failure modes in 3 categories (`MAST/taxonomy_definitions_examples/definitions.txt`). The taxonomy figure in the repository (`assets/taxonomy_v11_cropped-1.png`) gives the shares:

- **Poor specification: 37.17%.** Disobey task specification 15.2%, step repetition 11.5%, unaware of termination conditions 6.54%, disobey role specification 1.57%.
- **Inter-agent misalignment: 31.41%.** Reasoning-action mismatch 7.59%, information withholding 6.02%, conversation reset 5.50%, task derailment 5.50%, ignored other agent's input 4.71%, fail to ask for clarification 2.09%.
- **Task verification: 31.41%.** Incorrect verification 13.61%, no or incomplete verification 9.16%, premature termination 8.64%.

[S] The paper was accepted at NeurIPS 2025 (inter-annotator κ = 0.88) and argues that failures "stem from system design issues". Improving ChatDev's role prompts and topology helped (+9.4% success, from a baseline near 25 to 33%) but did not fix it ([arXiv 2503.13657](https://arxiv.org/abs/2503.13657)).

[I] Role confusion is rare, at 1.57%. The big losses are vague specs, missing stop conditions, information lost at handoffs, and weak verification. More job-title agents add handoffs, which are a main failure surface.

[S] **Google Research, "Towards a Science of Scaling Agent Systems"** (arXiv 2512.08296, Dec 2025; blog Jan 28, 2026) evaluated 180 configurations:

- On sequential tasks, every multi-agent variant lowered performance by 39–70%.
- Independent agents amplified errors 17.2× and centralized (orchestrated) ones 4.4×.
- Parallelizable tasks gained, with Finance-Agent up about 81% ([Google Research blog](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/)).

[S] **Wang et al., "Rethinking the Bounds of LLM Reasoning: Are Multi-Agent Discussions the Key?"** (ACL 2024) found that a single agent with a strong prompt nearly matches the best multi-agent discussion. With demonstrations in the prompt, the single agent can beat discussion ([arXiv 2402.18272](https://arxiv.org/abs/2402.18272)).

[S] **Zheng et al., "When 'A Helpful Assistant' Is Not Really Helpful"** (Findings of EMNLP 2024) tested 162 personas on 2,410 factual questions across 4 LLMs. Adding personas to system prompts "does not improve model performance", and the per-question effect is "largely random" ([ACL Anthology 2024.findings-emnlp.888](https://aclanthology.org/2024.findings-emnlp.888/)).

[S] **Cognition, "Don't Build Multi-Agents"** (Walden Yan, June 12, 2025) gives two principles: share full context and traces, and "actions carry implicit decisions, and conflicting decisions carry bad results". Parallel writers on a shared artifact therefore break. A follow-up, "Multi-Agents: What's Actually Working" (April 22, 2026), reportedly allows narrower collaboration where extra agents contribute intelligence "while writes remain single-threaded" ([cognition.ai](https://cognition.ai/blog/dont-build-multi-agents); I could not open it).

[F] **Anthropic, "How we built our multi-agent research system"** (June 13, 2025) reports these results:

- A lead agent (Opus 4) with Sonnet 4 subagents "outperformed single-agent Claude Opus 4 by 90.2%" on an internal research eval.
- Multi-agent systems use about 15× the tokens of chat.
- "token usage by itself explains 80% of the variance".
- Poor fits are domains "that require all agents to share the same context", and "most coding tasks involve fewer truly parallelizable tasks than research".
- Each subagent needs "an objective, an output format, guidance on the tools and sources to use, and clear task boundaries" ([anthropic.com](https://www.anthropic.com/engineering/multi-agent-research-system)).

[F] **Anthropic, "Building effective agents"** (Dec 19, 2024) says: "finding the simplest solution possible, and only increasing complexity when needed". It describes an evaluator-optimizer loop "when we have clear evaluation criteria", and a code-review voting pattern where "several different prompts review and flag the code" ([anthropic.com](https://www.anthropic.com/engineering/building-effective-agents)).

[F] **Claude Code best practices**, fetched 2026-11-04:

- "Handing the same task to a subagent doesn't make the answer any better. A fresh reviewer does help, because it hasn't seen your reasoning, which is a different thing from delegating."
- Subagents pay off for "a reviewer that hasn't seen how the code was written" and "a reviewer that can read but not edit". A read-only `tools: Read, Grep, Glob` limit "is enforced by Claude Code rather than requested in a prompt".
- A warning about reviewers: "A reviewer that's asked to find problems will always find *something*." The page advises scoping reviews, for example "skip style" ([code.claude.com/docs/en/best-practices](https://code.claude.com/docs/en/best-practices)).

The subagents page says that each subagent "runs in its own context window with a custom system prompt, specific tool access, and independent permissions" ([code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents)).

[F] **Anthropic, "Equipping agents for the real world with Agent Skills"** (Oct 16, 2025) presents skills as an alternative to per-role agents: "Instead of building fragmented, custom-designed agents for each use case", agents are specialized with composable skills loaded by progressive disclosure ([anthropic.com](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)).

### 1.5 Verdict on "separate agents vs lenses"

[I] The evidence converges on three points:

- **(a)** Role *instructions* matter. ChatDev's ablation shows this.
- **(b)** Role *personas* add little by themselves (Zheng et al.).
- **(c)** Separate agents earn their cost in two situations:
  - parallel, independent work (Anthropic research system; Google scaling study);
  - independent review from a fresh context that has not seen the author's reasoning (Claude Code docs).

  They cost accuracy on sequential, shared-artifact work: the Google 39–70% drop, MAST's handoff failures, and Cognition's implicit-decision conflicts.

The spec → design → prototype → implementation lifecycle is mostly sequential, with each artifact feeding the next. So most roles should be **artifact templates plus review lenses** run by one lead, with separate contexts reserved for reviewers, the simulated customer, and parallel build slices.

---

## 2. Claude Code subagent and skill collections with product or business roles

| Collection | Roles found | How defined | Quality notes |
|---|---|---|---|
| **VoltAgent/awesome-claude-code-subagents** (MIT, 176 agent files) | `08-business-product/`: product-manager, project-manager, scrum-master, business-analyst, ux-researcher, customer-success-manager, legal-advisor, content-marketer, and others. `04-quality-security/`: architect-reviewer, qa-expert, security-auditor, compliance-auditor | [F] PM: "senior product manager with expertise in building successful products that delight users"; `model: haiku`; long keyword bullet lists; a checklist with "User satisfaction > 80% achieved"; a JSON "context manager" query to an agent that does not exist in the file | [F] The sample completion message invents results: "Revenue impact +$4.2M with 2.3x user growth. NPS improved from 32 to 58" (`product-manager.md` l.225). [I] This primes the model to report invented metrics. Keyword lists are broad, with no artifact contract or gate. Low quality as role prompts. |
| **contains-studio/agents** (no LICENSE file; last commit 2025-07-28) | product/: sprint-prioritizer, feedback-synthesizer, trend-researcher; project-management/: project-shipper, studio-producer, experiment-tracker; design/: ux-researcher; marketing/: growth and social roles | [F] Built around a "6-day sprint" studio. Personas such as "You are a master launch orchestrator…"; `PROACTIVELY use this agent…` descriptions | [I] Persona-heavy and opinionated toward an app studio. No PRD/PR-FAQ artifacts. Licensing is unclear for reuse. |
| **wshobson/agents** (MIT, 202 agent files, 92 plugins) | No PM, PMM, TPM or customer agent. It has `startup-business-analyst` (skills), `business-analytics`, `content-marketing`, `hr-legal-compliance`, `comprehensive-review` (architect-review, security-auditor, code-reviewer) | [F] `agent-teams/agents/team-reviewer.md` is *one* reviewer "focused on one assigned review dimension (security, performance, architecture, testing, or accessibility)". `multi-reviewer-patterns` adds dedup and severity-merge rules ("Conflicting severity — Use the higher severity rating") | [I] Engineering-strong. The single reviewer agent parameterized by lens is the most reusable pattern found. |
| **anthropics/knowledge-work-plugins** (Apache-2.0) | `product-management` v1.2.0: skills write-spec, roadmap-update, stakeholder-update, synthesize-research, competitive-brief, metrics-review, sprint-planning, product-brainstorming. `marketing`: campaign-plan, brand-review, competitive-brief ("positioning and messaging comparison"), content-creation (including press-release structure) | [F] **Skills only. No agents or personas.** The `write-spec` PRD has Problem Statement, Goals, Non-Goals, User Stories, Requirements P0/P1/P2 with acceptance criteria, leading and lagging Success Metrics, Open Questions "tagged with who needs to answer (engineering, design, legal, data)", and Timeline. `synthesize-research` asks for "Evidence… with source attribution" and Confidence High/Medium/Low, and notes "A quote is evidence, not a finding" | [I] The best-structured PM artifacts found. Anthropic packages the PM role as a set of skills, not as an agent. |
| **anthropics/skills** | `doc-coauthoring`, `internal-comms` | [F] `doc-coauthoring` Stage 3 "Reader Testing" invokes "a sub-agent with just the document content and the question" to "catch blind spots" | [I] This is a direct template for a fresh-context spec reviewer. |
| **deanpeters/Product-Manager-Skills** (v0.84; **CC BY-NC-SA 4.0**) | 77 skills, including `press-release` (Working Backwards), `prd-development`, `proto-persona`, `positioning-statement` (Geoffrey Moore), `battle-card-builder`, `discovery-interview-prep`, `voice-of-customer-miner`, `stakeholder-mapping` | [F] `press-release`: "This is not a marketing artifact for launch day—it's a planning tool". `proto-persona`: "This is not a validated persona… a structured placeholder" | [I] Strong pedagogy. The non-commercial, share-alike license limits copying into a work tool, so reference it rather than vendor it. |
| **phuryn/pm-skills** (MIT; 69 skills, 9 plugins) | discovery, strategy, execution (`create-prd`, `strategy-red-team`, `pre-mortem`), go-to-market (`gtm-strategy`, `positioning-ideas`, `beachhead-segment`, `/plan-launch`, battlecard), research (`user-personas`, `interview-script`, `summarize-interview`) | [F] `strategy-red-team`: "Self-refute, don't fabricate… a red-team that manufactures doubt is as useless as one that rubber-stamps". `user-personas`: "Ground all insights in actual data; avoid assumptions" | [I] Good review-lens material, permissively licensed. |
| **BMAD-METHOD** (MIT) | Six named persona agents: Analyst "Mary", PM "John", PO "Sofia", Architect "Winston", Dev "Amelia", UX "Sally". Plus `bmad-prfaq` and `bmad-party-mode` | [F] Agents are skills that load a persona with a menu of codes, for example the PM's `PRD`. `bmad-prfaq` acts as "a relentless but constructive product coach", requires claims to be "verified against current real-world data", and supports `--headless`. `party-mode` runs a round-table and can "build an AI focus-group" | [I] The most complete role roster, and it is a single-session persona switch, not a parallel team. |

[F] **No collection defines a TPM, an SDM, a PMM or a simulated "Customer" as an agent with a grounded evidence pack.**

- "project-manager" agents (VoltAgent; MetaGPT's ProjectManager) do task breakdown, scheduling and risk work.
- PMM appears only as marketing-content skills.
- The closest thing to a customer is BMAD's party-mode "AI focus-group" and the persona *documents* in the PM packs.

---

## 3. Simulated customers and synthetic user research

**Evidence that it can work, when grounded:**

- [S] **Park et al. 2024, "Generative Agent Simulations of 1,000 People"** (arXiv 2411.10109). Agents built from **2-hour qualitative interviews** with 1,052 real people reproduced their General Social Survey answers "85% as accurately as participants replicate their own answers two weeks later". Interview-grounded agents reduced accuracy bias across racial and ideological groups compared with agents given only demographics ([Stanford HAI](https://hai.stanford.edu/policy/simulating-human-behavior-with-ai-agents)).
- [S] **Brand, Israeli & Ngwe, "Using GPT for Market Research"** (HBS WP 23-062; MSI 2025 update). LLM willingness-to-pay estimates were "realistic and comparable to estimates from human studies", with a downward-sloping demand curve. The model showed a first-option bias, and new-feature WTP needed fine-tuning ([HBS PDF](https://www.hbs.edu/ris/Publication%20Files/23-062_47458a4a-8be2-4b53-a3a3-9b5af20578b1.pdf)).
- [S] **Elicitron** (Autodesk Research, arXiv 2404.16045, JCISE 2024). Simulated users walking through product scenarios surfaced more latent needs than empathic lead-user interviews. These are the authors' own simulations, with no independent replication.
- [S] **UXAgent** (CHI EA 2025, Northeastern, Notre Dame and Amazon; arXiv 2502.12561 and 2504.09407). Persona-driven LLM agents drive a real browser to *pilot-test a usability study design* before running it with humans. UX researchers valued it but "expressed concerns".

**Evidence on the limits:**

- [S] **Bisbee et al. 2024, "Synthetic Replacements for Human Survey Data? The Perils of LLMs"** (*Political Analysis* 32(4), doi:10.1017/pan.2024.5). Mean responses tracked ANES, but synthetic data had "less variation" and regression coefficients "often differ significantly". The authors raise "serious concerns about the quality, reliability, and reproducibility".
- [S] **Wang, Morgenstern & Dickerson** (arXiv 2402.01908, 3,200 human participants, 16 identities, 4 LLMs). LLMs "misportray and flatten" identity groups and can essentialize them. Prompting with identity-coded names and raising temperature only partly mitigate this. I could not confirm the journal venue.
- [S] **Nielsen Norman Group, "Synthetic Users: If, When, and How to Use AI-Generated Research"** (Rosala & Moran, 2024). They reran three real studies with ChatGPT and the Synthetic Users product. Synthetic responses were "too shallow", more agreeable and more favorable than real users. The verdict: "user research needs real users". Acceptable use is drafting proto-personas or hypotheses that are labeled as such.

**How to ground a Customer agent:**

- [I] The positive results share one feature: the simulation is **conditioned on real data from the people it stands for**, such as interview transcripts (Park) or market data and conjoint designs (Brand). The negative results come from demographic-only prompts.
- [I] The documented failure directions are agreeableness, low variance, flattening and first-option bias. Every one of them makes a synthetic customer *approve* a concept. A synthetic "yes" is therefore nearly worthless, and a synthetic "no" or "confused" is the useful signal.

[R] Ground the Customer agent this way:

1. **Use an evidence pack, not a persona blurb.** Give it verbatim interview notes, support tickets, sales-call notes and usage data per segment, each with a source ID. Prefer the `synthesize-research` output format, which carries evidence plus confidence.
2. **Require citations.** Every reaction cites evidence IDs, or is tagged `[ungrounded]`.
3. **Run several personas per segment, sampled separately.** Include a skeptic and a non-adopter, to counter flattening and low variance.
4. **Give the agent jobs it does well:**
   - answer the PR/FAQ's *external* FAQ in the customer's words;
   - say what it would do instead (the status quo or competitor);
   - walk through a prototype UXAgent-style and report confusion points.
5. **Never use it to validate demand.** Output is labeled "hypotheses to test with real customers", and the gate it feeds requires a human sign-off or real-customer evidence.

---

## 4. Product marketing patterns and PR/FAQ generators

[F] PMM content found in the collections:

- **Positioning.** Geoffrey Moore's positioning-statement template in `deanpeters/positioning-statement`; `positioning-ideas`, `value-prop-statements` and `beachhead-segment` in phuryn (the last cites "Crossing the Chasm").
- **Messaging and competition.** `competitive-brief` ("positioning and messaging comparison") and `brand-review` against "messaging pillars" in Anthropic's marketing plugin.
- **Launch.** `/plan-launch` and `gtm-strategy` in phuryn; `project-shipper` in contains-studio.

[F] PR/FAQ generators:

- `deanpeters/press-release`: the press release only. It says the FAQ is part of the method but does not template it.
- `bmad-prfaq`: the full PR + customer FAQ + internal FAQ, ending in "A complete PRFAQ document + a distillate for… a PRD", with a `--headless` mode.
- Anthropic's `content-creation`: a *launch-day* press-release structure, which is a different artifact from the working-backwards PR.

[I] At Amazon the PR/FAQ is owned by the product owner (PM), not by marketing (section 5). In a virtual team, PMM's natural role on the PR/FAQ is a **review lens**: is the headline newsworthy, is the customer named, is the category clear, would the quote be credible? PMM then **authors** the downstream positioning, messaging and launch plan once the spec is stable.

---

## 5. Amazon practices the role prompts should match

All items here are **[S]**: Amazon-owned pages (aboutamazon.com, amazon.jobs, aws.amazon.com) were found by search but not opened.

- **Working Backwards PR/FAQ.**
  - Teams write a press release for the finished product before building, plus a customer (external) FAQ and an internal FAQ covering cost, feasibility and risk.
  - Secondary accounts of Bryar & Carr's *Working Backwards* (2021) say the PR is under one page and the FAQ five pages or fewer.
  - Ten or more drafts and five or more senior reviews before funding is common.
  - The process was used for most major initiatives since about 2004; AWS services such as S3 and EC2 went through it.
  - The Leadership Principles begin: "Customer Obsession: Leaders start with the customer and work backwards" ([amazon.jobs/en/principles](https://www.amazon.jobs/en/principles)).
- **Six-page narratives.** Bezos's 2017 shareholder letter describes meetings built around "narratively structured six-page memos" read silently at the start. Great memos "are written and re-written, shared with colleagues… set aside for a couple of days, and then edited again". PowerPoint was dropped by a June 9, 2004 email.
- **Bar Raiser.**
  - aboutamazon.com describes Bar Raisers as volunteer interviewers acting as "objective third-party" advisers and stewards of the 16 Leadership Principles, with more than 3,600 of them.
  - They drive the hiring debrief, assess long-term fit beyond the immediate role, and are not paid extra ([aboutamazon.com/news/workplace/amazon-bar-raiser](https://aboutamazon.com/news/workplace/amazon-bar-raiser)).
  - [I] Outside hiring, "bar raiser" is used informally for an independent, cross-team quality reviewer. That is the useful analogue here: a reviewer from outside the team, with a strong voice.
- **Two-pizza teams and single-threaded owners.** Teams small enough to feed with two pizzas own a product's full lifecycle and customer experience. A single-threaded leader is dedicated to one initiative ([AWS Executive Insights](https://aws.amazon.com/executive-insights/content/amazon-two-pizza-team/); [AWS Well-Architected DevOps Guidance OA.STD.6](https://docs.aws.amazon.com/wellarchitected/latest/devops-guidance/oa.std.6-provide-teams-ownership-of-the-entire-value-stream-for-their-product.html)).
- **SDM (Software Development Manager).** Postings vary by team. Common threads:
  - hiring, coaching and "career management and development of direct reports";
  - "solving deep technical problems";
  - planning "complex and highly-technical projects";
  - one posting asks for 3+ years managing engineering teams and 7+ years of engineering.

  [I] The SDM owns the team's delivery, operational health (on-call, operational excellence) and estimates. The SDM is not the spec author.
- **TPM (Technical Program Manager).** Postings mention:
  - "lead the technical planning, coordination, architecture, and implementation of multiple programs";
  - "assemble project teams, assign responsibilities, identify resource gaps, and develop work back plans";
  - translating business requirements into technical specs;
  - negotiating priorities across organizations;
  - managing launches.

  [I] The TPM owns the *cross-team* plan, dependencies, risks and launch readiness, and is technical enough to challenge designs.
- **Product Manager – Technical and PMM.** PMM postings center on "own go-to-market", branding and messaging, and leading go-to-market "from concept through launch". No posting found mentions PR/FAQ explicitly.

  [I] Public sources consistently attribute the PR/FAQ to the product leader or PM, so PMM is a reviewer of it and the author of the GTM work that follows.

---

## Implications

### A. Design principles, with the evidence for each

1. **Roles are contracts, not personas.** [R] Each role is an artifact template, a checklist and a gate. A "You are a senior X" blurb is not a role. Evidence: Zheng et al. (personas ≈ no gain); ChatDev ablation (specific instructions matter); the quality of the Anthropic `write-spec` skill against VoltAgent's keyword lists.
2. **Keep one lead and one writer per artifact.** [R] The lead session (TPM/orchestrator) runs the lifecycle, and authoring roles load as skills in that session or in one build subagent per slice. Evidence:
   - the Google study (sequential tasks: −39 to −70%; orchestration contains error amplification);
   - Cognition (single-threaded writes);
   - MetaGPT's own move to `TeamLeader` with `use_fixed_sop=False`;
   - ChatDev 2.0's pivot to orchestration.
3. **Use separate contexts only for independence or parallelism.** [R] The Customer agent and reviewers run in fresh, read-only subagents, and build slices run in parallel worktrees. Evidence: Claude Code docs ("A fresh reviewer does help… delegating doesn't"); the Anthropic research system (parallel breadth); `doc-coauthoring` reader testing.
4. **Engineer against MAST.** [R] Every gate has:
   - explicit inputs and outputs (against "disobey task spec", 15.2%);
   - a termination condition and a round cap (6.54% + 11.5% step repetition);
   - handoffs as files, not chat (information withholding 6.02%, conversation reset 5.50%);
   - a verifier that is *not* the author (verification failures 31.41%).
5. **Scope reviewers to stop invented findings.** [R] Each lens gets a severity rubric and an explicit "out of scope" list ("skip style"). A clean bill is an allowed outcome (Claude Code docs; phuryn `strategy-red-team`: "Self-refute, don't fabricate").
6. **Never let the synthetic customer approve anything.** [R] Ground it in an evidence pack and treat its output as hypotheses. A human or real-customer gate follows (Bisbee; Wang et al.; NN/g; Park).

### B. Recommended roster: form, inputs, outputs, gates

| Role | Form | Inputs | Outputs | Gate it owns or joins | Evidence |
|---|---|---|---|---|---|
| **TPM / Program Manager** | **Lead agent** (main session). Orchestrator only; does not author content | Idea, evidence pack, all artifacts | Plan with phases, dependencies and owners; RAID log; gate checklist status; work-back plan; launch-readiness checklist | Runs every gate; owns **G5 Launch readiness** | Amazon TPM postings; Anthropic orchestrator-worker; Google: centralized 4.4× vs 17.2× error amplification |
| **Product Manager** | **Skill set used by the lead** (PR/FAQ, PRD, user stories) | Evidence pack, Customer-agent reactions, constraints | **PR/FAQ** (PR under 1 page, external and internal FAQ); **PRD** (Anthropic `write-spec` shape: goals, non-goals, P0–P2 with acceptance criteria, metrics, open questions with owners) | Authors **G1 PR/FAQ** and **G2 Spec** | Amazon Working Backwards; Anthropic PM plugin is skills-only |
| **Customer** | **Separate read-only subagent**, one instance per persona or segment | Evidence pack (verbatim notes, tickets, data with IDs); the artifact under test | External-FAQ answers in the customer's voice; "what I'd do instead"; prototype walkthrough with confusion points; every claim cited or tagged `[ungrounded]` | Reviewer at G1 and **G4 Prototype**. **Advisory only**: a human or real-customer check is required to pass | Park (interview grounding); UXAgent; Elicitron; limits: Bisbee, Wang, NN/g |
| **Bar Raiser** | **Separate fresh-context reviewer**, read-only | The artifact plus its gate checklist only, without the author's chat | Pass/fail against the checklist; top 3 blocking issues; "what a skeptical VP asks" | Required reviewer at G1, G2 and G3 | Claude Code "fresh reviewer"; Amazon Bar Raiser as independent steward; MAST verification failures |
| **SDM** | **Review lens** at spec and design; **template** for the delivery plan | PRD, design | Feasibility and estimate (T-shirt sizes), staffing and slicing, operational-readiness items (on-call, alarms, runbooks), "what we will not do" | Reviewer at G2 and G3; co-signs G5 | Amazon SDM postings (delivery, ops, team) |
| **Architect / Principal Engineer** | **Skill** to author design docs and ADRs; **lens** at spec review | PRD | Design doc (MetaGPT-like: approach, interfaces, data, call flow, open questions); ADRs | Authors **G3 Design** | MetaGPT Architect schema; wshobson architect-review |
| **PMM** | **Lens** on the PR/FAQ; **skill** for positioning, messaging and launch | PR/FAQ, PRD, competitive brief | Positioning statement (Moore), messaging pillars, launch plan or GTM brief, competitive battlecard | Reviewer at G1 (headline and category); authors **G5 GTM** | Amazon PMM postings ("own go-to-market"); deanpeters, phuryn and Anthropic marketing skills |
| **UX Designer** | **Skill** (flows, wireframes or prototype brief) plus a **lens** (usability, accessibility) | PRD, Customer reactions | User flows, prototype spec, accessibility checklist | Authors the prototype brief at G4; reviewer at G2 | BMAD UX agent; wshobson accessibility dimension |
| **Builder (SDE)** | **Subagent per vertical slice** (worktree), single writer | Approved design plus tickets | Code, tests, PR | **G6 Implementation** | Cognition (single writer per artifact); Anthropic (parallel only where independent) |
| **QA** | **Lens** at spec (testability of acceptance criteria) plus a **fresh-context verifier** after build | PRD acceptance criteria, build | Test plan; verification report against the spec | Reviewer at G2; verifier at G6 | MAST 3.2/3.3 (13.61% incorrect verification); MetaGPT QA role |
| **Security / Privacy** | **Lens** triggered by data or auth changes | Design, diff | Threat model notes, blocking findings | Conditional at G3 and G6 | wshobson team-reviewer security dimension |
| **Legal / Compliance** | **Lens, triggered only** (PII, payments, regulated claims) with a mandatory human escalation | PR/FAQ claims, PRD | Flagged claims and requirements for human counsel | Conditional at G1 and G2 | VoltAgent legal-advisor and compliance-auditor exist, but legal advice from an LLM needs human review [I] |
| **Data / Analytics** | **Lens** | PRD success metrics | Metric definitions, instrumentation plan, guardrail metrics | Reviewer at G2 | Anthropic `write-spec` leading/lagging metrics; `metrics-review` |
| Solutions Architect | **Optional lens** for B2B and enterprise products | PR/FAQ internal FAQ | Integration and deployment objections from the field | Optional at G1 | [I] Same role as the Customer agent, but voices integrator constraints |

### C. Lifecycle gates

| Gate | Artifact | Reviewers |
|---|---|---|
| **G0 Evidence** | Evidence pack | Exists, with source IDs |
| **G1 PR/FAQ** | PM drafts | Customer agent(s), PMM lens, Bar Raiser, and a **human sign-off** |
| **G2 Spec (PRD)** | PM drafts | SDM, Architect, QA, Data and UX lenses, triggered Security and Legal lenses, Bar Raiser |
| **G3 Design** | Architect drafts | SDM, Security, Bar Raiser; TPM updates the dependency plan |
| **G4 Prototype** | Builder builds; UX writes the brief | Customer-agent walkthrough plus a human check |
| **G5 Launch / GTM** | PMM drafts | TPM readiness checklist; SDM operational readiness |
| **G6 Implementation** | Builder slices | QA verifier, code-review lenses |

Every gate has a round cap (for example, 2 review cycles) and an explicit "approve / approve-with-notes / block" outcome, written to a file.

### D. Open questions that could change this recommendation

- Neither MetaGPT nor ChatDev compares a role-chain against the *same* prompts run as one agent. A small internal eval would settle it for this product, for example the same PRD made both ways and scored by blind human review.
- How much real customer data will exist per segment? With none, the Customer agent should drop to "proto-persona" mode and G1 should require a real interview.
- The MetaGPT, ChatDev and MAST numbers marked [S] need checking against the PDFs once arxiv.org and aclanthology.org are reachable.
