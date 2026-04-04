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

Codex hooks are experimental. This scaffold uses only the documented `SessionStart`, `UserPromptSubmit`, and `Stop` events.

Local validation on April 3, 2026 enabled `codex_hooks`, linked this plugin into `/Users/bfirestone/plugins/worktrunk`, and ran `codex exec` in this repository. No Worktrunk marker transition was observed during or after the run, so plugin-bundled hook loading is still unconfirmed in this install flow.

The bundled `hooks.json` is therefore a candidate/default configuration, not a guarantee that automatic marker tracking works in every Codex environment. Use manual marker commands if you need reliable branch state today.

## Attribution

Worktrunk is authored by Max Rozen / worktrunk.dev. This Codex packaging follows the upstream Worktrunk repository while adapting the plugin surface for this marketplace.
