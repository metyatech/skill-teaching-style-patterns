<!-- markdownlint-disable MD025 -->
# Tool Rules (compose-agentsmd)

- **Session gate**: before responding to ANY user message, run `compose-agentsmd` from the project root. AGENTS.md contains the rules you operate under; stale rules cause rule violations. If you discover you skipped this step mid-session, stop, run it immediately, re-read the diff, and adjust your behavior before continuing.
- `compose-agentsmd` intentionally regenerates `AGENTS.md`; any resulting `AGENTS.md` diff is expected and must not be treated as an unexpected external change.
- If `compose-agentsmd` is not available, install it via npm: `npm install -g compose-agentsmd`.
- To update shared/global rules, use `compose-agentsmd edit-rules` to locate the writable rules workspace, make changes only in that workspace, then run `compose-agentsmd apply-rules` (do not manually clone or edit the rules source repo outside this workflow).
- If you find an existing clone of the rules source repo elsewhere, do not assume it is the correct rules workspace; always treat `compose-agentsmd edit-rules` output as the source of truth.
- `compose-agentsmd apply-rules` pushes the rules workspace when `source` is GitHub (if the workspace is clean), then regenerates `AGENTS.md` with refreshed rules.
- Do not edit `AGENTS.md` directly; update the source rules and regenerate.
- `tools/tool-rules.md` is the shared rule source for all repositories that use compose-agentsmd.
- Before applying any rule updates, present the planned changes first with an ANSI-colored diff-style preview, ask for explicit approval, then make the edits.
- These tool rules live in tools/tool-rules.md in the compose-agentsmd repository; do not duplicate them in other rule modules.

Source: github:metyatech/agent-rules@HEAD/rules/global/command-execution.md

# Command execution

Platform-aware execution procedures live in the
`command-execution` skill.

## General execution rules

- Prefer repository-standard scripts and commands (from
  `package.json`, `Makefile`, README) over ad hoc invocations.
  The agent MUST NOT add wrappers, redirections, or pipes
  unless the user explicitly asks.
- Before proposing a fix for a reported command failure,
  reproduce the same command (or the closest equivalent) and
  observe the same failure.
- Treat any nonzero exit code as a failure unless the specific
  exit code is explicitly documented AND the agent's code
  explicitly checks for that documented value. The agent MUST
  NOT map unknown nonzero exits to benign states.
- The agent MUST NOT assume agent platform capabilities beyond
  what is available; fail explicitly when a required capability
  is unavailable.
- When expected tools are missing or configuration has changed,
  verify MCP connectivity, fix or report connection failures,
  and do not proceed with work that depends on the missing tool.
- When diagnosing a third-party tool failure, first check the
  latest stable release; if it still reproduces, record the
  verified limitation and use a deterministic workaround.

## Git and identity flows

- Avoid interactive git prompts (pass `--no-edit` or set
  `GIT_EDITOR=true`).
- When no branch is specified, work on the current branch.
  Direct commits to `main`/`master` are permitted in
  user-controlled repositories.
- For federated identity flows (Google, Apple, Microsoft,
  GitHub) where an automation-launched browser is blocked or
  degraded, hand off only the IdP step to a real browser
  session and resume automation after the redirect. The agent
  MUST NOT attempt to bypass provider anti-automation or
  embedded-browser restrictions.

## Privilege elevation

- When elevated privileges are required, use `sudo` directly.
  The agent MUST NOT launch a separate elevated shell such as
  `Start-Process -Verb RunAs`. Fall back to "Run as
  Administrator" only when `sudo` is unavailable.

## Windows and PowerShell environment

- This is a Windows/PowerShell environment. The agent MUST NOT
  invoke Unix-only commands directly; run PowerShell scripts
  via `pwsh` or `powershell -File`.
- In PowerShell, the backslash `\` is a literal character.
  Avoid shadowing PowerShell automatic variables; prefer
  single-quoted strings. Use `;` for sequential command
  chaining; the agent MUST NOT use `&&` or `||` as control-flow
  operators.
- In headless Windows/PowerShell flows, launch every
  non-interactive console child process headlessly. The agent
  MUST NOT use the `&` call operator from a windowless parent
  to spawn such children.
- For destructive PowerShell file operations, verify the final
  absolute target path first, normalize file attributes when
  needed, and prefer explicit PowerShell or .NET deletion APIs
  over alias-driven shell deletion.
- Use explicit `agent-browser` session names on Windows. If the
  default session bind fails, retry with a different name.
  Close all task-owned agent-browser sessions before concluding.

Source: github:metyatech/agent-rules@HEAD/rules/global/engineering-standards.md

# Engineering and design standards

## Tooling and dependencies

- Prefer official, well-maintained, latest-stable tools and
  dependencies. Prefer OSS or free-tier services; when using a
  paid or proprietary service, call out the tradeoff in
  documentation or commit message.
- Before designing or building a system, verify whether the
  whole system or any decomposed subsystem can be satisfied by
  an existing official or well-maintained system. Use the
  existing system by default; build custom logic only for
  verified gaps.

## System design

- Designs MUST be compositional; dependency direction MUST be
  clean (high-level depends on low-level abstractions).
  Control flow MUST be shallow. Naming MUST be
  intention-revealing. Change points MUST be centralized in
  configuration or constants.
- Keep code, docs, tests, and configuration DRY. Fix root
  causes; remove obsolete code in the same change set; repair
  broken tools at the source.
- Failure paths MUST tear down resources and MUST NOT leave
  partial state. Verify cleanup runs on every error branch.

## Runtime and async behavior

- The agent MUST NOT block async APIs or use synchronous I/O
  where responsiveness is expected.
- Prefer push-, event-, or signal-driven synchronization over
  periodic polling. Polling MAY be used only when no reliable
  authoritative event path exists OR the user explicitly
  requests it; document why and bound cadence and retry
  behavior in code.

## API surfaces

- Avoid external command execution; prefer native SDKs.
- Prefer stable public APIs; isolate and document any
  unavoidable use of internal or unstable APIs.
- Externalize large embedded strings, templates, and rule data
  into resource files.
- The agent MUST NOT commit build artifacts. Keep artifact
  directory names and `.gitignore` entries aligned.

## Linters, formatters, and static analysis

- Every code repository MUST have exactly one formatter and
  one linter (or static analyzer) per primary language. Tool
  versions MUST be pinned via lock files or manifests.
- The agent MUST NOT disable lint rules globally. Suppressions
  MUST be narrow, justified inline, and time-bounded.
- When editing a file managed by a formatter, run the
  formatter immediately before performing replace operations.

## CI enforcement

- CI MUST run formatting checks and linting on every pull
  request and require these for merge. CI MUST treat warnings
  as errors.

## Dependency and security scanning

- A repository with GitHub Actions MUST configure Dependabot
  version updates for every applicable package ecosystem and
  for `github-actions`, unless the repository has no external
  dependency or update surface.
- A repository MUST enable dependency vulnerability scanning,
  secret scanning, and CodeQL for every supported language.
- A web UI project MUST enforce automated visual accessibility
  checks in CI.

## Environment portability

- The agent MUST NOT introduce machine-specific environments.
  Paths MUST be relative; configuration MUST be explicit.
- Lifecycle hooks (install, build, test) MUST succeed on a
  clean machine. Invoke developer tools via `npm exec` or the
  equivalent project-managed runner. Regenerate and commit
  lock files in the same change set when the manifest changes.
- Agent-owned temporary files MUST live under the OS temporary
  directory unless the user explicitly approves otherwise.

## Tool integration

- Design tools and services for agent-compatibility via
  standard interfaces. CLI conventions live in the `cli-design`
  skill.

## Post-change deployment verification

Detailed deployment detection and verification procedures live
in the `post-deploy` skill.

- After modifying code, determine whether deployment steps
  beyond commit and push are needed before concluding.
- If the affected repository powers a globally linked package
  (an npm package whose global install path is a symlink to
  the local working tree), rebuild the package and verify the
  global binary is functional before reporting completion.
- If the affected repository powers a running service, daemon,
  or scheduled task, rebuild, restart, and verify the running
  instance with deterministic evidence. The agent MUST NOT
  claim completion until the running instance reflects the
  changes.
- Verify teardown of every agent-owned temporary resource
  (services, daemons, browser sessions, temporary clones,
  patch files) before concluding. If cleanup fails, fix the
  harness or the cleanup path. The agent MUST NOT leave
  residue.

Source: github:metyatech/agent-rules@HEAD/rules/global/gui-standards.md

# GUI design and verification

Detailed flow design and verification procedures live in the
`guided-gui-design` and `quality-workflow` skills.

## Design rules

- Design GUIs around the user's real workflow moment, next
  action, and result.
- The agent MUST NOT use persistent explanatory prose to
  compensate for an unclear UI.
- For Web GUIs with user-entered state that would be costly to
  recreate, preserve draft state across reloads, accidental
  tab/browser closes, and restarts by default. When draft
  persistence would be unsafe, provide an explicit equally-safe
  recovery path; the agent MUST NOT silently discard the draft.
- Optimize GUIs for first-use clarity (a new user can identify
  the primary path and the next action without prior training):
  use ordinary task language, avoid irrelevant internal jargon,
  keep precise operator/domain terms when they are part of the
  user's real work, make the current selection/source/result
  obvious at all times.
- For human/AI systems, keep operator surfaces, backend
  capabilities, and product surfaces distinct. The agent MUST
  NOT expose operator-only or backend-only paths as product GUI
  controls unless the requester explicitly asks.
- Follow established expectations for common controls (info
  icons, disclosure toggles, close buttons, tabs, row
  selection); deviate only when the UX gain clearly outweighs
  the surprise cost.
- The agent MUST NOT introduce horizontal scrolling in primary
  UI without explicit justification.
- In interactive selection flows, make the current item, choice
  set, and destination explicit at a glance.

## Verification rules

- For GUI work, before reporting completion the agent MUST
  complete: a first-use walkthrough on the claimed environments,
  screenshot-based review of every changed view, layout and
  state visibility checks for required content and controls,
  and a whole-screen plausibility pass.
- If the primary flow or next action is not immediately
  understandable in the first-use walkthrough, treat the work
  as unfinished and iterate until first-use clarity is achieved.

Source: github:metyatech/agent-rules@HEAD/rules/global/identity-and-scope.md

# Identity and scope

## Definitions

- **User** — the human operator. Name: "metyatech".
- **User-controlled repository** — a repository owned by
  `metyatech` on GitHub, or any repository where `metyatech`
  holds authoritative write permission (verify with
  `gh repo view --json owner,viewerPermission`).

## User identity

- The user controls the GitHub user/org `metyatech`, the npm
  scope `@metyatech`, and all repositories under
  `github.com/metyatech/*`.
- Treat any external reference to `metyatech` as
  user-controlled unless direct evidence contradicts it.
- Resolve identity uncertainty with the `gh` CLI before acting.

## Authority on user-controlled repositories

- Inside a user-controlled repository, the agent MAY perform
  end-to-end operations: issues, pull requests, pushes to
  default branches, releases, package publishes, and repository
  administration.
- For account-wide instructions, treat every user-controlled
  repository as in scope. Repository creation, splitting, and
  deletion are permitted within that scope.
- When publishing, cloning, adding submodules, or splitting
  repositories, default to placement under `metyatech` ownership
  unless the user specifies a different owner.

## Authority on third-party repositories

- The agent MUST obtain an explicit per-repository request
  before any write action on a third-party repository, even
  when push access exists.

## Resolving target ambiguity

- When a user instruction names a target ambiguously, verify
  the intended canonical repository from the request's purpose
  before writing.
- Keep uncertain work isolated on a separate branch or worktree
  until the canonical target is confirmed.

Source: github:metyatech/agent-rules@HEAD/rules/global/persistent-tracking.md

# Persistent task and thread tracking

## Definitions

- **Actionable task** — a discrete piece of work to complete
  (`task-tracker`, "what to do").
- **Thread** — a single discussion topic or design decision
  (`thread-inbox`, "what was discussed").
- **Persistent stage** — on-disk task stages: `pending`,
  `in-progress`, `committed`, `released`, `done`. `pushed` is
  derived from upstream reachability of the `committed` event.
- **Thread status** — `active`, `waiting` (auto-set by
  `--from user`), `needs-reply`, `review`, `resolved`.
- The agent MUST NOT create threads for tasks already tracked
  by `task-tracker`.

## Installation

- If not installed, install via
  `npm install -g @metyatech/task-tracker` and
  `npm install -g @metyatech/thread-inbox`.

## Storage

- `.tasks.jsonl` MUST be committed to version control. The
  agent MUST NOT add it to `.gitignore`.
- Store `.threads.jsonl` in the workspace root by passing
  `--dir <workspace-root>` to every `thread-inbox` invocation.
  The agent MUST NOT commit `.threads.jsonl`; add it to
  `.gitignore` explicitly.

## Session-start check

- At the start of any session that may involve state-changing
  work, run `task-tracker check`.
- At the start of every session, run
  `thread-inbox inbox --dir <workspace-root>` and
  `thread-inbox list --status waiting --dir <workspace-root>`.
  Report findings before starting new work.

## When to record

- When an actionable task emerges, immediately record it with
  `task-tracker add`. Treat `task-tracker` as the authoritative
  cross-session tracker; session-scoped task tools MUST NOT
  replace it.
- A thread MUST capture discussion topics, design decisions,
  and multi-session context to remember across sessions.
- Add a `--from user` thread message for any substantive user
  interaction (decisions, preferences, directions, questions,
  feedback, approvals); status auto-sets to `waiting`. Err on
  recording rather than omitting.
- Add a `--from ai` thread message for informational updates.
  Use `--status needs-reply` when asking the user a question
  and `--status review` when reporting completion for review.
- Record the user's actual words verbatim. Threads MUST read
  as a conversation transcript, not meeting minutes.

## Stage transitions and lifecycle

- When a task reaches `committed`, run
  `task-tracker update <id> --stage committed` immediately
  before `git add` and `git commit` so `.tasks.jsonl` is
  included in that closing commit. The agent MUST NOT create
  tracker-only follow-up commits to record `pushed`.
- When reporting a task as complete, state the lifecycle stage
  explicitly. The agent MUST NOT claim "done" while downstream
  stages remain incomplete.
- Resolve threads when the topic is fully addressed or the
  decision is implemented and recorded in rules. Periodically
  purge resolved threads.
- If a thread captures a persistent behavioral preference,
  encode it as a rule per `rule-system` and resolve the thread.

Source: github:metyatech/agent-rules@HEAD/rules/global/posture-and-delivery.md

# Agent posture, approval, and delivery

## Definitions

- **Direct mode** — invoked by the human user.
- **Delegated mode** — invoked by another agent (the
  delegator).
- **In-scope work** — work that follows logically from the
  requester's request without expanding their stated goal.
- **Delivery chain** — the ordered follow-on actions that
  complete a user request end-to-end: implementation, testing,
  runtime verification, deployment or release when applicable,
  documentation updates, and follow-on defect cleanup.
- **Terminal state** — the strongest justified completion
  state given the agent's authority. Normally pushed/released
  for user-controlled repos; an open pull request for
  third-party repos.
- **Irreducible blocker** — a condition the agent cannot
  resolve deterministically without user input.

## Default posture

- Optimize for minimal human effort; default to automation
  over manual steps. Drive work from the desired outcome and
  pick the highest-quality safe path. Prefer correctness,
  safety, robustness, verifiability, and maintainability over
  speed.
- Before acting, identify the exact user-visible effect the
  user expects and the real system surface that causally
  produces it. The agent MUST NOT substitute a proxy action
  (logging, recording, mirroring, hinting, staging) for the
  authoritative state change.
- The agent MUST NOT introduce backward-compatibility shims,
  legacy aliases, or temporary fallbacks unless the requester
  explicitly asks. Remove discovered legacy paths at the
  source in the same change set.
- Distinguish (1) instructions about how the agent should
  operate, (2) background context about the user's workflow or
  environment, (3) actual requirements for the artifact. The
  agent MUST NOT turn (1) or (2) into product features, scope,
  or UI requirements unless the user explicitly asks.

## Resolving uncertainty

- Resolve any uncertainty that can be settled deterministically
  through inspection, testing, or other reliable checks.
- Escalate to the user only when remaining uncertainty depends
  on user-only judgment (intent, preference, priority, risk
  tolerance). The agent MUST NOT fill user-only gaps with its
  own default.
- Make scope, risk, cost, and irreversibility decisions
  explicit when they materially affect the outcome. Infer
  intent beyond literal wording when justified by context;
  state the inference and propose the matching next step.

## Approval

- In a user-controlled repository for in-scope work, the
  user's request itself is plan approval. Proceed with
  implementation, testing, commits, pushes, releases, and
  deploys without re-asking. A blanket directive ("fix
  everything", "audit all repos") covers all in-scope
  follow-up.
- For user-owned publishable packages, an explicit "commit and
  push" or "complete this fix" approves the release/publish
  chain when release is the normal completion path.
- In delegated mode, the act of delegation itself is plan
  approval. The delegated agent MUST NOT re-request human
  approval; if scope must expand, fail back to the delegator.
- The agent MUST request explicit approval before any of:
  destructive or hard-to-reverse actions (force push, history
  rewrite, data deletion, repository deletion); third-party
  account side effects (billing, permissions, OAuth grants);
  scope expansion beyond the user's stated request; any action
  whose impact the agent cannot bound from inspection alone.

## Natural delivery chain

- After any user instruction, infer and execute the delivery
  chain end-to-end until the strongest justified terminal
  state is reached or an irreducible blocker remains.
- For user-owned publishable packages, when the user asks to
  commit and push or finalize a fix, treat release and publish
  as in-scope unless the user explicitly opts out.

## Focus discipline

- Stay focused on the current task until terminal state or
  irreducible blocker, unless the user explicitly switches.
- The agent MUST NOT pause at intermediate milestones, treat
  partial satisfaction as completion, or context-switch to
  other in-progress tasks merely because they are visible.
- When delegated agents are running and no other meaningful
  in-scope local work exists, wait for completion or the next
  material state change rather than polling without progress.

## PR review feedback

- The agent MUST NOT remain idle on a failing pull request
  when a known automated fix exists. Apply the fix, re-verify,
  commit, and push.
- Handling PR review feedback is always pre-approved. Run the
  full review loop until no actionable feedback remains or an
  irreducible blocker requires user input. Procedures live in
  the `pr-review-workflow` skill.

Source: github:metyatech/agent-rules@HEAD/rules/global/quality-and-verification.md

# Quality and verification

GUI verification lives in `gui-standards`. Procedural detail
lives in the `quality-workflow` skill.

## Definitions

- **State-changing work** — any change that modifies repository
  contents, deployed services, published artifacts, persisted
  data, or external accounts.
- **Acceptance criterion (AC)** — a binary, testable statement
  of a required outcome.
- **Critical-system journey** — an end-to-end user path through
  authentication, billing, authorization, persistence, or any
  other critical system whose failure carries direct user harm.

## Non-negotiable gates

For any state-changing work or any claim of "done", "fixed",
"working", or "passing":

1. Before starting, list AC as binary, testable statements.
2. Before each git commit, run the repository's full
   verification suite and observe success.
3. With each AC, define the evidence (concrete commands,
   outputs, or manual steps) that will demonstrate satisfaction.
4. For code or runtime changes, add automated tests. A bug fix
   MUST include a regression test.
5. Run the repository-standard verify command. If none exists,
   add one in the same change set.
6. Maintain AC and evidence internally; surface them only when
   the user requests.

## Failure handling

- The agent MUST NOT swallow errors. Fail fast with explicit
  context. Validate configuration and external inputs at system
  boundaries.
- CI and commit hooks MUST enforce full verification. Code
  fixes MUST follow a failing-test-to-passing-test loop.
- On a failing security audit, attempt the documented automated
  fix (`npm audit fix` or equivalent). If the fix succeeds and
  verification passes, commit and push.

## Runtime verification

- For user-facing apps, multi-client systems, multi-environment
  systems, or persisted-state systems, define the claimed
  environment matrix up front and verify every claimed primary
  path and state before concluding. Report evidence per claimed
  client, environment, and path. Any path not directly verified
  MUST stay unclaimed.
- For a critical-system journey, completion REQUIRES live or
  production-like end-to-end verification; unit, integration,
  build, and health checks alone are insufficient.
- Runtime verification MUST cover interruption, retry, reload,
  invalid-input, and stale-state behavior in addition to the
  happy path.
- If an intended environment cannot be exercised, stop short
  of completion, state the exact gap, and leave that
  environment unclaimed.

## Bug handling

- On every user-reported bug, identify the earliest
  deterministic gate that should have caught it and add or
  strengthen that gate in the same change set. A fix without a
  new catching gate is incomplete.
- When fixing a bug, the agent MUST identify the broader
  failure pattern that made the bug possible and extend the
  change so same-pattern failures are prevented by
  construction, by a shared invariant boundary, or by a
  generalized gate. A fix that only patches the observed
  instance is incomplete unless the remaining pattern is
  irreducible and explicitly reported.
- The agent MUST NOT claim bug-free behavior. Report scope,
  evidence, and residual risk.

Source: github:metyatech/agent-rules@HEAD/rules/global/release-and-publication.md

# Release and publication

Detailed release procedures live in the `release-publish` skill.

## Packaging hygiene

- Every published artifact MUST include a LICENSE file with
  `metyatech` as the copyright holder.
- The agent MUST NOT ship build artifacts, test artifacts, or
  local configuration files. Verify a clean environment can
  install and use the product via the README's documented
  steps.
- Define a SemVer policy and document what counts as a
  breaking change.
- Keep the package version and the Git tag consistent.
- Run dependency security checks before every release.

## Trusted publishing

- For public npm packages published from GitHub Actions, use
  npm trusted publishing (OIDC) instead of long-lived npm
  tokens whenever the package's npm registry supports it. The
  publish workflow MUST run on a Node and npm runtime version
  that satisfies trusted-publishing requirements.

## GitHub repository metadata

- For every public repository, set the GitHub Description,
  Topics, and Homepage fields. Topics MUST be assigned from the
  standard set defined in the `release-publish` skill.

## Verification of published artifacts

- Before reporting a publishable-package change as complete,
  verify the full delivery chain end-to-end: commit → push →
  version bump → release → publish → install verify.
- Verify that the published package resolves and runs correctly
  from a clean install before reporting completion.

## New repository bootstrap

- When work is intentionally isolated into a new user-owned
  repository because that repository is the canonical home for
  the work, bootstrap it as a real Git repository, create the
  GitHub remote, and push the initial history before reporting
  completion. Skip this only when the user explicitly opts out.

Source: github:metyatech/agent-rules@HEAD/rules/global/rule-system.md

# Rule and skill system

## Compliance vocabulary

The keywords MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT,
SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this
AGENTS.md and every included rule module carry the meanings of
RFC 2119 and RFC 8174:

- **MUST**, **REQUIRED**, **SHALL** — absolute requirement.
- **MUST NOT**, **SHALL NOT** — absolute prohibition.
- **SHOULD**, **RECOMMENDED** — strong recommendation; deviate
  only with a stated reason and weighed alternatives.
- **SHOULD NOT** — strong recommendation against; same
  deviation rule.
- **MAY**, **OPTIONAL** — permitted but not required.

When a rule omits a keyword, treat the rule as MUST.

## Composition

- The agent MUST NOT edit `AGENTS.md` directly. AGENTS.md is
  the composed output of rule modules and a per-repository
  ruleset, and MUST be self-contained at the repository root.
- A new repository MUST include `agent-ruleset.json`, MUST
  compose AGENTS.md, and MUST satisfy the required global
  standards before reporting the repository as complete.
- compose-agentsmd workflow procedures (session gate,
  pre-commit hook ordering, staging diffs, edit-rules /
  apply-rules) live in the compose-agentsmd repository's
  `tools/tool-rules.md`.

## Authoring rules

- Rules MUST be MECE: each obligation appears in exactly one
  module. Cross-reference rather than duplicate.
- Each rule MUST be atomic (one bullet, one testable
  obligation), use the imperative mood with an explicit
  compliance keyword (or implied MUST), and be testable as a
  yes/no check without outside context.
- Rules MUST NOT contain hedges such as "ideally", "where
  appropriate", "reasonable", or "perhaps". Replace hedges with
  explicit conditions.
- Placement: `rules/global/` for any-workspace rules;
  `rules/domains/<domain>/` only for opt-in domains;
  `agent-rules-local/` only when no other repository will need
  it.
- When updating a rule, encode the underlying general
  principle, not the incident-specific surface example. Trace
  each update back to the reasoning error that permitted the
  original mistake.
- Persistent user instructions about future agent behavior MUST
  be encoded in the appropriate rule module in the same change
  set unless the user explicitly scopes the instruction to the
  current task.
- Treat rule/skill gaps, redundancy, misplacement, and
  recurring review feedback as systemic; fix the underlying
  issue and similar instances in the same change set.
- In delegated mode, the agent MUST NOT modify rules directly;
  report rule-gap suggestions to the delegator.
- Session memory resets between sessions. Persistent
  behavioral knowledge MUST live in rules; rules are the
  source of truth.

## Authoring skills

- A skill MUST follow the Agent Skills open standard
  (agentskills.io/specification). A `SKILL.md` frontmatter
  MUST contain only `name` and `description`. The `name` MUST
  be lowercase alphanumeric with hyphens, at most 64
  characters. The `description` MUST explain trigger
  conditions.
- The body of `SKILL.md` MUST be platform-agnostic; intent-level
  wording is required and platform-specific tool names MUST
  NOT appear. Platform-specific examples MUST live in
  `README.md`.
- `SKILL.md` and `README.md` SHOULD be written in English.
  Reference content (lookup tables, language-locked UI labels)
  MAY use the user's natural language when keeping the original
  preserves direct linkage to user-facing artifacts.
- Skill instructions MUST be concise and action-oriented.
  Normative claims SHOULD use the compliance vocabulary.
- A skill MUST NOT duplicate AGENTS.md global rules; reference
  the rule module by name instead.

## Skill packaging and updates

- Each skill MUST live in its own repository with `SKILL.md`
  at the repository root. User-managed installable skills MUST
  live in public `metyatech/skill-<name>` repositories. Each
  skill repository MUST include a LICENSE file (MIT preferred)
  and MUST default to public visibility.
- The GitHub `metyatech/skill-*` repository is the canonical
  source of truth. Installed copies under
  `~/.agents/skills/<name>` are derived mirrors.
- The agent MUST NOT edit installed skill copies. Edit the
  source repository, push, and re-install via
  `npx skills add metyatech/skill-<name> --yes --global`.

Source: github:metyatech/agent-rules@HEAD/rules/global/sub-agent-delegation.md

# Sub-agent delegation and dispatch

## Definitions

- **Delegator** — the agent or human that spawns another agent.
- **Delegated agent** — the agent spawned by a delegator.
  Operates in delegated mode (defined in `posture-and-delivery`).
- **Restricted operation** — an operation a delegated agent
  MUST NOT perform without an explicit per-call delegation:
  modifying rules or rulesets, merging or closing pull requests,
  creating or deleting repositories, releasing or deploying,
  force-pushing, or rewriting published history.
- **Tier** — task difficulty/cost classification: Free, Light,
  Standard, Heavy, Large Context.
- **Effort** — model reasoning intensity (`low`, `medium`,
  `high`, `xhigh`, `max`); not every model supports every level.

## Tier classification

- **Free** — trivial lookups; Copilot 0x models only.
- **Light** — mechanical transforms, formatting, simple edits.
- **Standard** — general implementation, code review,
  multi-file changes.
- **Heavy** — architecture, safety-critical code, complex
  multi-step reasoning.
- **Large Context** — > 200k input tokens; prefer 1M-context.

## Spawning a delegated agent

- The delegating prompt MUST state delegated mode and the
  approval state, include acceptance criteria, verification
  requirements, and task context. The delegator MUST NOT assume
  the delegated agent has access to the calling conversation.
- The delegating prompt MUST NOT recite rules the delegated
  agent will already read from AGENTS.md.
- Two or more agents MAY write in the same repository only
  when each uses an isolated checkout, worktree, or branch AND
  one integration owner serializes merge or rebase back into
  the canonical branch. Otherwise, run them sequentially.

## Delegated-agent obligations

- Respond in English; report verification evidence concisely.
- The delegated agent MUST NOT modify rules directly; report
  rule-gap suggestions in the result for delegator review.
- Inherit the delegator's repository scope; MUST NOT expand it.
  If unable to operate within scope, fail explicitly back.
- If the delegated agent reports a read-only or no-write
  constraint, run a minimal reversible OS-temp probe and
  report the exact failure verbatim.
- A delegated agent MUST NOT perform a restricted operation
  without an explicit per-call delegation.

## Dispatch tooling

- Sub-agents MUST be launched via `agents-mcp` (the metyatech
  MCP server). The agent MUST NOT use platform built-in
  subagent spawners.
- Before spawning, run `ai-quota` to verify quota across all
  candidate agents. If `ai-quota` is unavailable or fails,
  report and MUST NOT spawn.
- When spawning a sub-agent, explicitly specify both `model`
  and `effort` from the model inventory in the
  `sub-agent-dispatch` skill. The agent MUST NOT rely on
  default model selection.
- When spawning an implementation sub-agent, set
  `mode: 'edit'`. Default `mode: 'plan'` is read-only.
- After spawning, return to the user immediately and use
  background or non-blocking monitoring; the agent MUST NOT
  block waiting for completion.
- Decision framework, model inventory tables, routing logic,
  prompt templates, quota fallback logic, and platform-specific
  procedures live in the `sub-agent-dispatch` skill.

## Orchestrator model selection

- When spawning the manager orchestrator role, default to
  `claude-sonnet-4-6` with `medium` effort.
- Escalate to `claude-opus-4-6` with `medium` effort when
  strict rule compliance is required. Research
  (arXiv:2505.11423) shows higher effort degrades
  instruction-following on multi-constraint rule sets.
- Use `high`, `xhigh`, or `max` effort only for complex
  reasoning tasks. The agent MUST NOT use elevated effort to
  improve rule compliance.

## Verification of sub-agent results

- The agent MUST NOT trust a completion claim without evidence.
  Every implementation sub-agent MUST return: restated AC, AC
  → evidence mapping (`PASS`/`FAIL`/`NOT RUN`), files changed,
  assumptions and uncertainties, risks and rollback notes.
- After implementation, run repo-standard verify commands for
  objective evidence.
- If verification fails, cannot run, or the task is Heavy tier
  or release/production, spawn a separate reviewer sub-agent
  (never the same instance) and require explicit `PASS`/`FAIL`.
  The reviewer MUST receive the original AC and spec.
- The agent MUST NOT adopt a result as done unless reviewer
  status is `PASS`. EXCEPTION: Standard tier with passing
  verify AND clear AC evidence MAY skip the reviewer.

## Cost, execution, and lifecycle

- Use the minimum reasoning effort that reliably produces
  correct output. Prefer newer-generation models at lower
  effort over older models at maximum effort. The agent MUST
  NOT use Heavy-tier models for Light or Standard tier tasks.
- The agent MUST NOT rapidly switch or respawn sub-agents for
  the same task while one is actively running without errors.
- After a team completes, shut down all team agents and clean
  up resources. If a sub-agent fails, the agent MUST NOT
  silently swallow the failure: retry, adjust, or escalate.
- If a delegated task fails repeatedly because of quota limits
  (HTTP 429), update the task's stage in `task-tracker` so the
  work resumes from the last successful stage.

Source: github:metyatech/agent-rules@HEAD/rules/global/writing-and-documentation.md

# Writing and documentation

## User responses

- Respond to the user in Japanese unless the user explicitly
  requests another language.
- Report commit and push activity only when the turn changed
  files.
- In direct mode, emit the Windows `SystemSounds.Asterisk`
  sound after completing a response. In delegated mode, the
  agent MUST NOT emit any sound. A manager agent MUST emit the
  sound at most once for the overall task.
- When delivering a new tool, feature, or artifact, explain
  what it is, how to use it, and its key capabilities.
- Keep progress reporting short and user-centric. Prefer long
  uninterrupted execution blocks; explain internals only when
  the user requests them.
- The agent MUST NOT include command transcripts in normal user
  reports unless the user explicitly requests them.
- At the end of a session or task, report any lingering
  unresolved technical friction or environment issues.

## Developer-facing writing

- Write developer documentation, code comments, commit
  messages, pull-request descriptions, and rule modules in
  English.

## README and documentation

- Every repository MUST include a README.md that covers:
  purpose, supported environments, setup/usage/development
  commands, required environment variables and configuration,
  release and deployment steps when applicable, and links to
  standard companion documents.
- For any change, assess documentation impact and update every
  affected doc in the same change set so documentation matches
  behavior. Omit no-op documentation notes from normal
  completion reports.
- For CLI tools, document every parameter with a description
  and at least one example invocation, plus at least one
  end-to-end example command.
- The agent MUST NOT include user-specific local paths, fixed
  workspace directories, drive letters, or personal data in
  documentation examples; prefer repository-relative paths and
  placeholders.

## Markdown linking

- When a Markdown document links to another local file, the
  link path MUST be relative to the Markdown file's location.
