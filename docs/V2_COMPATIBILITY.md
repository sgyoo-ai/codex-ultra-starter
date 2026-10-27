[English](V2_COMPATIBILITY.md) | [한국어](../kr/docs/V2_COMPATIBILITY.md)

# v1/v2 Tool Calls and Context Passing

Roles, models, reasoning efforts, task scopes, and delegation policies are shared by v1 and v2. Match the arguments to those provided by the actual calling tool.

The template does not force or switch versions. Do not invent unsupported version-switching, context-control, or delegation-depth settings.

## Per-agent reasoning effort and Ultra conversion

- The main agent and each subagent select their own model and reasoning effort. Convert an agent's selection to that model's `multi_agent_reasoning_effort` only when the **final selected effort for that agent**, after accounting for role settings, call arguments, defaults, and other applicable rules, is `Ultra`. A parent's Ultra selection does not determine a child's effort.
- If the model metadata has no valid override, choose a supported `Max`; otherwise choose the last supported non-Ultra value, then `Medium`. Explicit `high`, `xhigh`, and `max` selections remain unchanged by this Ultra conversion. [Normalization implementation](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/protocol/src/openai_models/reasoning_effort.rs#L8-L38).
- The same normalization is used to configure `reasoning.effort` for ordinary reasoning requests and in the subagent creation path. Even with the same conversion rules, the result can differ when agents select different models or efforts. [Request configuration code](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/core/src/client.rs#L957-L976), [subagent creation code](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/core/src/agent/control/spawn.rs#L958-L968).
- In the checked `0.164.0-alpha.11.1` tag and installed build's default catalog, the value for both `gpt-6-astra` and `gpt-6.1-sol` is `xhigh`. [Astra](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/models-manager/models.json#L22), [Sol](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/models-manager/models.json#L195).
- This template explicitly sets `high`, `xhigh`, or `max` for every subagent role. Even when the main agent is Ultra, the researcher uses `high`, implementer and Terra use `max`, and architect uses `xhigh`. `[agents].default_subagent_reasoning_effort = "max"` is the default for subagents; it does not set the main agent's effort. For other configurations without explicit settings, check the engine's defaults and inheritance rules separately.
- Verify the effective value against the model metadata and engine version used by the session. This template does not override the main model or mode; it sets per-role subagent settings. Checking the default catalog is distinct from measuring actual requests or internal server behavior in the current session.

## Context selection

| Intent | v2 tool: `fork_turns` | v1 tool: `fork_context` |
|---|---|---|
| Pass only the information the new subagent needs | `fork_turns="none"` | `fork_context=false` |
| Pass the full parent conversation history to the new subagent | `fork_turns="all"` | `fork_context=true` |

- Use these arguments and values only if the actual schema supports them. Selective context is the default.
- Provide enough information directly: goal, required context, confirmed decisions, fixed constraints, evidence locations, owned files, completion criteria, whether further delegation is allowed, and the slot budget.
- If the full history is necessary or the user requests it, explicitly pass it and briefly record why.
- Do not send both arguments or pass an argument from one interface to the other tool. If neither is available, check for another documented way to pass context; report the limitation if no method exists.
- Apply a conflict between full-history inheritance and model or effort selection only if it has been confirmed in that actual tool. If they conflict, pass the necessary source text selectively and do not silently change the selected model.
- `none` and `false` select whether a newly created subagent inherits the parent's history. Sending a follow-up to an existing subagent preserves that subagent's history.

## Role and model selection

1. If a named-role argument is available and its exposed definition matches the current file, use that role.
2. If no named role is available, or its definition is outdated, read the relevant TOML and pass the role instructions with supported explicit model and effort arguments.
3. If a required argument is unsupported, report the limitation. Mentioning a model name in the prompt does not change which model actually runs.
4. Distinguish the model and effort specified in the role file from the actual selection. In a path that applies the role file, its model and effort take precedence; do not claim it ran with a different effort without evidence.
5. Do not automatically substitute another value when the account or client does not support the requested model or effort.

## What to check in a new project

- Check the target project's existing settings and trust state, as well as model availability for the account.
- Record syntax checks, actual settings loading, current tool definitions, and real task results separately. Do not infer creation-tool arguments from a general thread API or from a protocol artifact's versioned directory name.
- Check that the role and call arguments are applied as intended during the first necessary real delegation. Do not create meaningless test agents or model requests.
- Do not restrict Terra or other roles to one specific calling-tool version. Keep each role's shared instructions and adapt only the tool call itself.

## Official references

- [Custom agents and role-file precedence](https://learn.chatgpt.com/docs/agent-configuration/subagents#custom-agents)
- [Project settings loading and trust](https://learn.chatgpt.com/docs/config-file/config-basic)
- [Supported models](https://learn.chatgpt.com/docs/models)
