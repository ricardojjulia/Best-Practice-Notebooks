# Contributing

Thanks for helping improve **Best-Practice-Notebooks**. This repository is a curated, content-only collection of Dynatrace best-practice notebook series. Please keep changes narrow, factual, and synchronized across the documentation layers that help readers and importers use the repository correctly.

## What to Contribute

Appropriate contributions include:

- Fixing repository-level documentation, navigation, and metadata
- Correcting obvious factual or formatting issues in series-level documentation
- Improving cross-links between topic series and the `-START-HERE-` playbook
- Proposing new notebook series or missing Dynatrace use cases

## Contribution Guardrails

- Preserve existing source attribution and the unofficial-support disclaimer.
- Do not invent licensing terms or imply official Dynatrace support.
- Avoid broad reformatting or drive-by edits across generated exports.
- Do not commit secrets, credentials, customer data, or private information.
- Follow the repository's [minimal check-in policy](CHECK_IN_POLICY.md).

## Updating Existing Series

If you modify a topic series, keep the repository structure consistent:

1. Update the series `README.md` when notebook scope, ordering, or prerequisites change.
2. Keep the notebook formats aligned: `markdown/`, `notebooks/` or `NOTEBOOKS/`, and `pdfs/` or `PDFs/`.
3. Preserve notebook-level `Created` / `Last Updated` metadata where present.
4. Preserve source-attribution sections and links inside the notebooks.
5. If routing changes, update the series `AGENTS.md`.

## Updating Cross-Series Navigation

If you add a new series or materially change repository scope:

1. Update the root [`README.md`](README.md).
2. Update the [`-START-HERE-` playbook](-START-HERE-/README.md) and any affected index files in that directory.
3. Recheck any stated series counts or inventory summaries for accuracy.

## Verification Expectations

This repository has no build system or automated test suite. Before opening a PR, manually verify that:

- links you changed still resolve
- counts or inventory statements match the repository
- filenames and directory casing (`notebooks/` vs `NOTEBOOKS/`, `pdfs/` vs `PDFs/`) are accurate
- any import guidance still matches the files present in the repository

## Support and Ownership

This repository is not officially supported by Dynatrace. Contributions are reviewed on a best-effort basis by the repository owner or maintainers. When in doubt, prefer small, well-explained pull requests over sweeping content changes.
