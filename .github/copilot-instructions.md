# Copilot instructions for Best-Practice-Notebooks

## Big picture
- This repo is a content library of Dynatrace best‑practice notebook series.
- Each topic is a folder (e.g., AUTOM, IAMADM, K8S) with the same layout: `NOTEBOOKS/` (Dynatrace notebook JSON), `markdown/` (Markdown exports), `PDFs/` (printable exports), plus a topic `README.md`.
- Root index and navigation live in [README.md](README.md).

## Content structure & naming
- Notebook filenames use a strict prefix pattern:
  - Markdown: `markdown/-TOPIC-##_slug.md` (example: [AUTOM - Dynatrace automation/markdown/-AUTOM-01-automation-landscape.md](AUTOM%20-%20Dynatrace%20automation/markdown/-AUTOM-01-automation-landscape.md))
  - Notebook JSON: `NOTEBOOKS/--TOPIC-##_slug.json` (example: [AUTOM - Dynatrace automation/NOTEBOOKS/README.md](AUTOM%20-%20Dynatrace%20automation/NOTEBOOKS/README.md) describes the folder)
- Topic `README.md` files list the notebook sequence and should stay aligned with the Markdown exports (example: [AUTOM - Dynatrace automation/README.md](AUTOM%20-%20Dynatrace%20automation/README.md)).

## Authoring workflow (repo‑specific)
- The recommended consumption path is importing JSON from `NOTEBOOKS/` into Dynatrace Notebooks for interactive DQL queries; Markdown/PDF are exports of the same content (see [README.md](README.md)).
- When adding or renaming a notebook, update **all three** representations (`NOTEBOOKS/`, `markdown/`, `PDFs/`) and the topic `README.md` lineup to keep them consistent.
- Keep the numbered order and prefixes consistent across the series (e.g., `AUTOM-01`, `AUTOM-02`, …) so the root table of contents stays accurate.
## Git workflow
- For all git operations (branching, committing, pushing, etc.), prefer using GitKraken MCP tools over terminal commands.
- Available GitKraken MCP tools include: `mcp_gitkraken_git_branch`, `mcp_gitkraken_git_checkout`, `mcp_gitkraken_git_add_or_commit`, `mcp_gitkraken_git_push`, `mcp_gitkraken_git_log_or_diff`, and others.
## What to avoid
- Don’t add build/test steps—this is a documentation‑only repo with static exports.
- Don’t introduce new folder conventions; follow the existing topic layout.
