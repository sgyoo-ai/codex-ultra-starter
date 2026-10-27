# Codex Ultra Starter

English | [한국어](kr/README.md)

Settings for preserving as much Ultra-mode performance as possible while reducing usage through task-appropriate models and roles.

## Intended environment

- Codex with **GPT-6.1 Sol Ultra** or **GPT-6 Astra Ultra** as the main agent.
- Select the main model and Ultra mode in Codex. Subagent models and reasoning efforts follow the role definitions below.

## Main and subagent reasoning effort

**The main agent and each subagent select their model and reasoning effort separately.** `multi_agent_reasoning_effort` is model metadata used to resolve an agent's final selected `Ultra` effort into an inference-request effort. Main agents and subagents use the same resolution rule, each with its own selected model's metadata. [Resolution code](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/protocol/src/openai_models/reasoning_effort.rs#L8-L38), [ordinary request path](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/core/src/client.rs#L957-L976), [subagent startup path](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/core/src/agent/control/spawn.rs#L958-L968).

- **Main agent selected as Ultra:** In the checked Codex CLI `0.164.0-alpha.11.1` bundled catalog, both Sol 6.1 and Astra resolve to **`xhigh`**. The actual value can vary with the engine and model metadata in use. [Astra catalog](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/models-manager/models.json#L22), [Sol catalog](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/models-manager/models.json#L195).
- **This template's subagent roles:** Each uses its explicitly assigned `high`, `xhigh`, or `max`. For example, with an Ultra main agent, researcher still uses `high`, while implementer and Terra use `max`.
- **A subagent itself selected as Ultra:** Its own model's `multi_agent_reasoning_effort` is used. See the [compatibility guide](docs/V2_COMPATIBILITY.md) for fallback behavior when that metadata has no valid value.

**For the same model, a subagent explicitly set to `max` uses a higher reasoning-effort setting than this setup's Ultra main agent (`xhigh`).** Examples are a Sol 6.1 main agent versus implementer or reviewer, and an Astra main agent versus deep_reasoner. Some roles use other effort levels; comparisons of actual performance across different models require separate evidence.

`[agents].default_subagent_reasoning_effort = "max"` in `.codex/config.toml` sets the subagent default only. Select the main agent's effort in the app; subagents using a role follow that role TOML's explicit settings.

## What the main agent does

The main agent retains the model and mode selected in the app and focuses on **coordinating with the user, assigning work to subagents, evaluating results, and integrating them**. It clarifies goals and constraints, selects roles, provides scope, acceptance criteria and necessary context, and coordinates dependencies and execution order for shared resources.

Research, design, implementation, and GUI work go to the appropriate subagent when independent delegation is worthwhile. The main checks evidence, validation coverage, and unresolved issues before deciding what to accept. It handles short, clear tasks and tightly coupled sequential work directly.

## Included roles

Roles are listed from common, broadly applicable work to more specialized work. Select them by the task and the judgment required.

| Role | Model / effort | When to use |
|---|---|---|
| implementer | `gpt-6.1-sol` / `max` | Ordinary detailed design, implementation-contract drafts, code analysis, changes, and validation |
| gui_operator | `gpt-6.1-terra` / `max` | Screen, browser, and app interaction; visual interpretation; GUI reproduction and behavior checks |
| researcher | `gpt-6.1-sol` / `high` | Read-only research requiring search-scope decisions, code-path tracing, or synthesis of multiple sources |
| reviewer | `gpt-6.1-sol` / `max` | Independent checks of existing requirements, contracts, and defects |
| luna_worker | `gpt-6-luna` / `max` | Small implementation or repetitive changes with a fixed contract and owned files |
| luna_researcher | `gpt-6-luna` / `max` | Bounded extraction, comparison, organization, or summaries from specified sources |
| architect | `gpt-6-astra` / `xhigh` | Choosing or changing core architecture and consequential contracts; resolving design conflicts |
| deep_reasoner | `gpt-6-astra` / `max` | Focused examination of a specific hard question remaining after evidence has been organized |

- At most 5 open subagent threads, excluding the main. Descendants, waiting parents, and completed but unclosed threads all count.
- Start with one Astra agent and reserve slots for independent review and child tasks.
- Delegation is limited to main → Sol/Astra owner → Luna. Read-only owners may delegate only to luna_researcher.
- Assign Terra directly from the main. Terra and Luna are leaves; the main schedules access to shared GUIs and devices.
- Keep each role's explicit model and effort; do not silently substitute unsupported values.
- The main handles short, clear work directly. The role list is not a mandatory execution sequence.

## Files and language variants

```text
README.md                       # English overview
AGENTS.md                       # English project work and delegation rules
.codex/config.toml              # Subagent defaults and open-thread limit
.codex/agents/*.toml            # Eight role definitions with English descriptions/instructions
docs/APPLY.md                   # Applying, updating, and restoring the template
docs/V2_COMPATIBILITY.md        # Tool arguments, actual loading, and verification stages
kr/README.md                    # Korean overview
kr/AGENTS.md                    # Korean project work and delegation rules
kr/.codex/                     # Korean variant of the same role settings
kr/docs/                       # Korean application and compatibility guides
.gitignore                     # Excludes runtime output, backups, and secrets
```

Choose the repository root for English or `kr/` for Korean. Each is an installable variant: copy **only the selected variant's `AGENTS.md` and `.codex`** into a project. The English variant follows the user's preferred language and defaults to English when none is established; the Korean variant retains informal Korean communication. Both use the same models, effort levels, role assignments, and delegation limits.

README files and Git settings are repository documentation and do not need to replace a target project's files. `.codex/config.toml` and `.codex/agents/*.toml` are intentionally versioned.

## Apply to a project

1. Review the desired template revision and choose English (repository root) or Korean (`kr/`). Use a verified tag or commit when taking the template from Git.
2. Back up the target project's AGENTS.md, project settings, colliding role definitions, and the fact that any new file did not previously exist.
3. For an empty project, copy the selected variant's AGENTS.md and `.codex`. For an existing project, merge the required parts while preserving its rules and settings. Use one language variant at a time.
4. Add the project's own data, service, permission, shared-resource, execution, and core-contract constraints.
5. Open the target directory as the project root and check actual configuration loading. Project configuration loads for trusted projects.
6. Use role, model, effort, and context arguments supported by the session's actual spawning tool. Check behavior during the first genuinely needed delegation.

See the [application and rollback guide](docs/APPLY.md) and [compatibility guide](docs/V2_COMPATIBILITY.md).

## Passing context

| Policy | v2 tool's `fork_turns` | v1 tool's `fork_context` |
|---|---|---|
| Default: task instructions and necessary context only | `"none"` | `false` |
| When needed: the full parent conversation history | `"all"` | `true` |

Use these arguments and values only when the actual tool exposes them. Select arguments from the real schema; do not infer them from a version name or send both forms together. Context selection is a tool-call policy. Do not invent TOML options for it. Briefly state why full history is needed.

## Verification and Git boundaries

Checks cover TOML syntax, role order, models, efforts, limits, removal of project-specific material, and documentation links. Both language variants retain the same operational settings.

In each target project, check configuration conflicts, actual project configuration loading, and available tools and models. Verify the result during the first genuinely needed task.

Keep only this shared template and its documentation in this repository. Keep actual project code, personal absolute paths, device addresses, service configuration, credentials, screenshots, conversations, logs, runtime state, and backups outside it. Do not turn the original working project into the template repository.

Record changes and evidence when role policies or tool interfaces change, and update the template revision. No global installation hook or resident service is included.

## Official references

- [Custom roles and subagent defaults](https://learn.chatgpt.com/docs/agent-configuration/subagents)
- [Project configuration scope and precedence](https://learn.chatgpt.com/docs/config-file/config-basic)
- [AGENTS.md loading rules](https://learn.chatgpt.com/docs/agent-configuration/agents-md)
