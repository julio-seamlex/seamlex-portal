# Seamlex Portal — Lifecycle state

> Local to this workspace, written by `/hi-seamlex` and `/seamlex-setup`. It records which step of the
> lifecycle this company is on, nothing more. Settings live in the plugin's fixed config
> (`config/seamlex.config.md`) and are never copied here.
>
> Safe to commit — it holds no secrets. Delete it and `/hi-seamlex` will infer the step again from what it
> can see in the workspace and on the board.

| Key | Placeholder | Value |
|---|---|---|
| Current step | `{{STAGE}}` | <setup \| discovery \| requirement \| status> |
| Changed on | `{{STAGE_UPDATED}}` | <YYYY-MM-DD> |
| Why | `{{STAGE_NOTE}}` | <one line: what moved it here> |

## History

Newest first. One line per change, so the path through the engagement stays visible.

| Date | Step | Note |
|---|---|---|
