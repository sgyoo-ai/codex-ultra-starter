# Shared project work rules

These rules apply to the project containing this file and its working subdirectories. Do not automatically extend them to other projects or user-wide configuration.

## Scope, protection, and records

- Use the user's preferred language; default to English if no preference is established. Lead with results and reasons, and stay concise without omitting important constraints, evidence, or verification limits. Give brief updates during longer work.
- Follow the latest user request and already authorized scope. Do not expand review or design requests into implementation or deployment approval, or repeatedly request approval already given.
- Start by reading only the relevant portions of this project's README and work/environment documentation. Do not treat historical execution status as current evidence or import another project's records or approvals.
- Before changing anything, identify its role, inputs, outputs, callers, and relevant persistence/recovery paths. Prefer existing code and installed tools; do not make repeated speculative patches.
- Preserve user material, existing files, uncommitted and other workers' changes, and running services. Keep unrelated improvements separate and do not arbitrarily change other projects or global settings.
- Installation, spending, deployment, external transmission, device/app operation, and permission/network/audio changes must remain within the current request's authorization. Avoid broad process termination. Before a restart, check its target, impact, and existing authorization.
- Do not collect secrets, cookies, private conversations, full environments, Keychain contents, or protected permission databases for investigation.
- Add abstractions or extract functions only when they clearly improve readability, responsibility boundaries, reuse, or testability; avoid one-off helpers, wrappers, interfaces, and structures for speculative future needs without those benefits.
- Perform validation needed for the change. Avoid unnecessary large tests or model calls, and do not treat brief silence as a reason to fail or terminate a long-running task. Reuse valid verification evidence for unaffected code and conditions, while retaining necessary additional checks and existing requirements for independent review.
- Distinguish actual changes, checks, and unknowns. Writing code, building, installing, running, and user confirmation are separate stages. Record important decisions, completion status, blockers, and next actions in project documentation.
- Do not install memory automation, log collectors, schedulers, or resident services unless requested.

## Ultra role allocation

- Use Ultra role allocation by default in this project. Keep the main model and mode selected in the app; do not assume automatic detection of the current mode. Do not repeatedly ask or search to identify it, or add detectors, version-forcing patches, or model-cache changes.
- The main owns goals, constraints, overall context, user coordination, task allocation and dependencies, decision records, result integration, and final acceptance. It handles short, clear tasks and tightly coupled sequential work directly.
- List roles by general applicability and expected frequency of use. Select by task type, judgment required, and how firmly scope is defined. Treat the order as guidance, not mandatory stages or call quotas. Delegate only work that is independently worthwhile.

| Role | Model / effort | Selection criteria |
|---|---|---|
| implementer | `gpt-6.1-sol` / `max` | Ordinary detailed design, implementation-contract drafts, code analysis, changes, and necessary validation |
| gui_operator | `gpt-6.1-terra` / `max` | Screen, browser, and app interaction; visual interpretation; GUI reproduction and behavior checks |
| researcher | `gpt-6.1-sol` / `high` | Read-only research requiring search-scope decisions, code-path tracing, or synthesis of multiple sources |
| reviewer | `gpt-6.1-sol` / `max` | Independent checks of existing requirements, contracts, and defects |
| luna_worker | `gpt-6-luna` / `max` | Small implementation or repetitive changes with a fixed contract, owned files, and acceptance criteria |
| luna_researcher | `gpt-6-luna` / `max` | Bounded read-only extraction, comparison, organization, or summaries with specified sources, questions, and output |
| architect | `gpt-6-astra` / `xhigh` | Choosing or changing core architecture and consequential contracts; resolving serious design conflicts |
| deep_reasoner | `gpt-6-astra` / `max` | Examining assumptions, counterexamples, and contradictions in a specific hard question remaining after evidence is organized |

- Implementer may carry ordinary design through implementation. Use researcher for investigation-only results; when changes are authorized, implementer may continue the relevant analysis. Use luna_researcher for specified extraction/comparison and luna_worker for a small change under a fixed contract.
- Terra interprets screens and visual material and performs assigned GUI observation, interaction, reproduction, and behavior checks. Prefer suitable connectors, APIs, or CLIs. Hand code analysis, changes, or local validation back to the main for the appropriate Sol owner. The main schedules work on the same GUI or device.
- Assign architect early when a consequential new decision concerns persistence, durability, input ordering, authentication, concurrency, protocols, or recovery. If the impact is unclear, establish the boundary first. The main may take this responsibility when current evidence confirms it is Astra Max and that allocation applies.
- Use reviewer to check existing requirements, contracts, and defects; use architect for choosing or changing the contract itself or resolving a serious design conflict. Scope review to concrete impact and check original requirements, actual code, callers, persistence, and recovery. Agreement among models is not evidence.
- Have the research owner fill evidence gaps. When sufficient evidence still leaves a specific hard question, use deep_reasoner. Provide the question, constraints, observations, rejected alternatives, and remaining contradiction. Do not repeat research just to reach a failure count.
- Follow each role's TOML for deliverables and handoff conditions. All roles report actual work, evidence, verification coverage, and unfinished work, separating facts, inferences, and unknowns. GUI success does not validate every internal code contract.

## Role invocation and context

- Use `.codex/config.toml` and `.codex/agents/*.toml`. The general subagent default is Sol 6.1 Max. Follow the model, effort, and instructions in the role TOML, and do not silently replace unsupported values.
- Use a named role when the tool exposes it and its actual definition matches the file. If unavailable or stale, read the TOML and pass supported explicit model/effort arguments and role instructions. A role name alone is not proof of loading.
- By default, new subagents do not inherit the parent conversation. Explicitly use `fork_turns="none"` or `fork_context=false`, according to the actual tool. Supply sufficient goals, necessary context, decisions, constraints, evidence locations, owned files, acceptance criteria, delegation permission, and slot budget.
- When full history is needed or requested by the user, explicitly use `fork_turns="all"` or `fork_context=true` and state why. Supply selected original evidence instead only when an actual conflict with model/effort selection is established. Do not invent unsupported arguments or TOML keys.
- Send existing agents only necessary updates. The none/false choice controls parent-history inheritance for a new agent; it does not erase an existing agent's history.

## Delegation and concurrent work

- The main assigns the appropriate role directly. A Sol/Astra owner may delegate one level to luna_researcher or luna_worker only within child slots already allocated by the main. Direct main-to-Luna assignments are also allowed.
- Read-only owners may invoke only luna_researcher. Give luna_worker only an approved contract, owned files, and acceptance criteria. Invoke Luna through a matching named role or explicit `gpt-6-luna` / `max` plus role instructions.
- Terra is a leaf assigned directly by the main. Sol/Astra owners request GUI work through the main, which defines Terra's scope and execution order. Terra and Luna do not create agents or delegate further.
- Return consequential new decisions, expanded scope, or wholesale rework to the main with evidence, options, and questions, while continuing independently authorized work. Owners verify core evidence, changes, and validation in child results. Do not use an authoring agent as the independent reviewer of that change.
- At most 5 subagent threads may remain open, excluding the main. This includes descendants, waiting parents, and completed but unclosed threads. Reserve child-delegation and independent-review slots; start with only one Astra agent. If no slot is available, handle the task directly or coordinate rather than repeatedly retrying creation.
- Avoid concurrent changes to the same files, GUI, device, or shared execution resource. Reuse existing owners, preserve other workers' changes, and close threads after collecting results when no longer needed.
- Read-only scope, file ownership, and delegation depth are behavior rules, not independent permission isolation or cost caps. Follow the parent's permission and approval policy.
- Distinguish syntax checks, actual configuration loading, role exposure, and model execution. Confirm application during the first genuinely needed delegation; do not create model calls solely for validation. Do not assume editing a role file changes an existing subagent's model.

## Project-specific additions

- Find preservation requirements, shared resources, execution procedures, and core contracts in the project's documentation and include the relevant parts in task assignments.
- The main assigns ownership and execution order for shared apps, browsers, devices, build outputs, and runtime state. Unassigned owners must not change or test them concurrently.
- This template defines allocation policy and roles. Confirm actual tools, models, permissions, and project configuration loading in the session, and use the tool schema actually exposed for invocation arguments.
