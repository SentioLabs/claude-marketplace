# Worktrunk for Codex

This directory is a Codex-only packaging of Worktrunk's git worktree guidance. It is intended to feel native in the Codex marketplace while staying honest about which Claude-era behaviors do and do not translate cleanly.

## What is included

- Codex marketplace metadata for the `worktrunk` plugin
- A Codex-facing hook contract using only documented Codex events
- A plugin-local entry point for the Worktrunk skill and reference docs that later tasks will fill in
- Repo-local release registration so the plugin can be versioned alongside the rest of this marketplace

## Install in Codex

1. Make sure this repo's `.agents/plugins/marketplace.json` is available to Codex.
2. Install or enable the `worktrunk` plugin from the Sentio Labs marketplace entry.
3. Refresh Codex if the plugin does not appear immediately after registration.

Use the Codex marketplace path for installation and configuration here. Do not follow Claude-specific install commands in this package.

## Hook status

Codex hooks are experimental. This scaffold uses only the documented `SessionStart`, `UserPromptSubmit`, and `Stop` events, and plugin-bundled hook loading still needs runtime validation in this environment.

If runtime validation shows hooks are not loaded by Codex here, the docs should treat the hook behavior as best-effort rather than guaranteed.

## Attribution

Worktrunk is authored by Max Rozen / worktrunk.dev. This Codex packaging follows the upstream Worktrunk repository while adapting the plugin surface for this marketplace.
