# DNOO GIS Enhancement 2026

## Scope
This repository is only for GIS-related enhancement work under DNOO Enhancement 2026.

## Repository Structure
- `.github/` contains Copilot instructions, prompts, and Speckit agent definitions.
- `.specify/` contains SpecKit workflow state, templates, extensions, scripts, and the project constitution.
- `docs/` contains supporting repository documentation:
	- `docs/decisions/` for decision records
	- `docs/discovery/` for discovery inputs and reference material
	- `docs/discovery/design-docs/` for design-document extracts grouped by enhancement item
	- `docs/discovery/emails/`, `docs/discovery/notes/`, and `docs/discovery/screenshots/` for supporting discovery evidence
	- `docs/templates/` for reusable document templates
- `specs/` contains the generated project and feature specification artifacts:
	- `specs/000-project-spec/` for the repository-level project specification
	- `specs/item-01-electric-line-splitting-tool/`
	- `specs/item-02-user-length-alignment-tool/`
	- `specs/item-03-update-substation-task/`
	- `specs/item-04-session-monitoring/`
	- `specs/item-09-11kv-cable-overview-report/`
	- `specs/item-12-meter-spsid-population/`
	- `specs/item-13-tee-off-cable-lifecycle/`
	- `specs/item-15-load-taking-mobile-app/`
- `.vscode/` contains workspace settings for this repository.

## Out of scope
- ADMS-only enhancement items
- Non-GIS workstreams
- General DNOO topics not required for GIS enhancement delivery

## Objectives
- Improve operational efficiency
- Strengthen data integrity
- Reduce manual workaround

## Governance
Work in this repository is governed by the project constitution in
.specify/memory/constitution.md. Requirements, plans, and tasks must follow the GIS
scope, ADMS boundary, data integrity, and non-regression rules defined there.
