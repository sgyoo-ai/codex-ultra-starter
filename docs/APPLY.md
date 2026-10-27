[English](APPLY.md) | [한국어](../kr/docs/APPLY.md)

# Applying and Restoring the Template

This document describes how to apply the settings to a target project. Publishing or downloading the repository alone does not change another project's settings. Verify that the settings load and run in the target project itself.

## Before applying

1. Confirm the target project's root and the template tag or commit you will use.
2. Inspect the target's `AGENTS.md`, `.codex/config.toml`, and role files with matching names. Check only the relevant conflicts in user-wide settings, without displaying secrets.
3. Save the original bytes and hashes, along with the absence of any new files, in a local backup in the target project. Do not put the backup in Git.
4. Confirm which role format, models, and reasoning efforts the target app and session support. Do not change unrelated user-wide models, permissions, or services.

## A new, empty project

- Choose the repository root for English or `kr/` for Korean, then copy that selected directory's `AGENTS.md` and `.codex` into the target project. Never copy both language bundles.
- Add the target project's introduction, run and verification procedures, preservation requirements, shared resources, and core contracts to its own documentation and `AGENTS.md`.
- The template README explains how to use the settings; it does not need to serve as an introduction to the new project.

## A project with existing settings

- Preserve existing rules in `AGENTS.md` and merge the common delegation rules. Do not replace the entire original file.
- Review and merge only the four `[agents]` entries in `config.toml` for conflicts. Do not declare the same table twice or overwrite MCP, permission, or personal settings.
- Compare role files with the same names and merge them. Preserve role files with different names.
- If a same-named file still has an older definition, verify whether it is actually loaded. Do not blindly install duplicate user-wide or project settings.

## Verification sequence

1. Check TOML syntax, the names, models, and efforts of all eight roles, and the open-subagent limit of five.
2. Check the target root, project trust, settings precedence, and actual loading. Account or administrator policies may limit support.
3. Check the arguments of the actual creation tool as described in the [compatibility guide](V2_COMPATIBILITY.md).
4. Verify the role instructions, model, effort, context passed, and result during the first necessary real task. Do not create a separate model or subagent call just for verification.
5. Record syntax checks, settings loading, tool definitions, and actual execution as separate results in the project documentation.

## Updating

- Update from a verified tag or commit and compare it with the target project's additions.
- Review changes to models, efforts, permissions, and core contracts separately from ordinary wording updates.
- Back up existing files and record which new files were absent before each update. Do not assume Git alone can restore local uncommitted state.

## Restoring

- If nothing has changed since applying the template, restore the backed-up files and remove only the files newly created by this change.
- If `.codex` was absent before applying and no further settings have been added since, you may remove the project `.codex` created by this change. If settings already existed, restore files individually instead of removing the entire directory.
- If other changes have been made since then, compare the differences and preserve them while reverting only this change.
- The `.codex` directory in the user's home folder is not part of restoration. Restoring files does not remove conversation history, active subagent history, or model settings. Clean up any subagents as needed and verify again in a new project conversation.
