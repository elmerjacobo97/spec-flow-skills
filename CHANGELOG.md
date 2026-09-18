# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.0](https://github.com/elmerjacobo97/spec-flow-skills/compare/v0.2.0...v0.3.0) (2026-09-18)


### Features

* extract spec close flow into /spec-close skill ([e058998](https://github.com/elmerjacobo97/spec-flow-skills/commit/e058998d7a29f025b64541eece75ab9171e6b74f))
* **spec-close:** auto-include the spec's own files in the close commit ([006b639](https://github.com/elmerjacobo97/spec-flow-skills/commit/006b6393833acd2471a4abd307cc13c1ca0d24fb))
* **spec-close:** sync project memory on close ([fe7cb87](https://github.com/elmerjacobo97/spec-flow-skills/commit/fe7cb874e1e312af6f194c7dff831d1b52f7634e))
* **spec-edit:** add in-place spec revision skill ([3dfb035](https://github.com/elmerjacobo97/spec-flow-skills/commit/3dfb0355c60d020901e8d5085edf6dea466facbb))
* **spec-impl:** accept the close-spec request in any language ([2dfaed8](https://github.com/elmerjacobo97/spec-flow-skills/commit/2dfaed867a7b25310e1b3692a2e0b2ff5b95473f))
* **spec-impl:** add CloseMode (local vs MR flow) and post-merge cleanup ([72dc6ba](https://github.com/elmerjacobo97/spec-flow-skills/commit/72dc6ba56b39559933c15d742e3a282f540b700b))
* **spec-impl:** close the spec on request (state, commit, merge, delete branch) ([f28f623](https://github.com/elmerjacobo97/spec-flow-skills/commit/f28f623e0abe4688cd0559c7f0efa650ed0e5284))
* **spec-impl:** implement group by group with in-spec checkboxes and --one-shot ([c54cbbc](https://github.com/elmerjacobo97/spec-flow-skills/commit/c54cbbc23681e7b414a1364c3db19682e11421a1))
* **spec-impl:** keep existing work branch when running in ticket flow ([49a9b7b](https://github.com/elmerjacobo97/spec-flow-skills/commit/49a9b7b8b32386fa5165b1981a40d989f8bb8467))
* **spec-verify,spec-status,spec-explore:** add auxiliary audit, board, and explore skills ([c41a9d0](https://github.com/elmerjacobo97/spec-flow-skills/commit/c41a9d04c1c3cf94068fc30621e5b3aad36f790e))
* **spec:** use the selection tool only for decision questions ([e87be30](https://github.com/elmerjacobo97/spec-flow-skills/commit/e87be3095a1cf4e1e73c04cd97c308d3353f8f2c))


### Bug Fixes

* **spec-verify:** make frontmatter valid YAML so skills.sh can index it ([1c82050](https://github.com/elmerjacobo97/spec-flow-skills/commit/1c8205004ff64f1e2fb50b7043a1fbec426c7e80))

## [0.2.0](https://github.com/elmerjacobo97/spec-flow-skills/compare/v0.1.2...v0.2.0) (2026-06-29)


### Features

* **spec:** introduce AutoCreateBranch configuration for branch management ([1273a5c](https://github.com/elmerjacobo97/spec-flow-skills/commit/1273a5c1a152711f024c0afc336c2cb2a3b019cd))

## [0.1.2](https://github.com/elmerjacobo97/spec-flow-skills/compare/v0.1.1...v0.1.2) (2026-06-26)


### Bug Fixes

* **spec:** improve metadata formatting in spec template ([96e4c64](https://github.com/elmerjacobo97/spec-flow-skills/commit/96e4c64f04c8992260fe1e24c96fc05a9981d47e))

## [0.1.1](https://github.com/elmerjacobo97/spec-flow-skills/compare/v0.1.0...v0.1.1) (2026-05-23)


### Bug Fixes

* remove shell command directives to pass Gen Agent Trust Hub audit ([b5d4402](https://github.com/elmerjacobo97/spec-flow-skills/commit/b5d4402e3ffe6305971104882487192d0e603a53))
* remove shell injection vector in spec-impl Phase 2 ([b946f46](https://github.com/elmerjacobo97/spec-flow-skills/commit/b946f46f752e71a74c85080a85ce20c478d8df99))
* **spec:** stop after saving spec and point to /spec-impl ([1cbde98](https://github.com/elmerjacobo97/spec-flow-skills/commit/1cbde987627171305411d2be20ff0caad2739c20))
* use single quotes for argument-hint in spec SKILL.md ([7f6252a](https://github.com/elmerjacobo97/spec-flow-skills/commit/7f6252a5f16a05cc4b4469366697aaf892809f23))

## [Unreleased]

## [0.1.0] - 2026-05-07

### Added

- `/spec` skill — guides Claude through a four-phase spec authoring workflow (context → clarification → section-by-section drafting → save). Writes specs to `specs/NN-slug.md` with status `Draft`.
- `/spec-impl` skill — validates spec status is `Approved`, creates a dedicated git branch (`spec-NN-slug`), and implements the spec step by step with diff pauses between each step.
- `scripts/link-skills.sh` — symlinks all skills into `~/.claude/skills/` for local development.
- `scripts/install-to-agent.sh` — translates skills for Cursor, Codex, and Antigravity agents.
- `scripts/list-skills.sh` — lists all available skills.

[Unreleased]: https://github.com/elmerjacobo97/spec-flow-skills/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/elmerjacobo97/spec-flow-skills/releases/tag/v0.1.0
