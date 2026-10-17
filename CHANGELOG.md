# Changelog

Notable changes to this workspace. Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added

- `animejs` skill: Anime.js 4.5.0 reference for writing, debugging, reviewing, and migrating animations.
- `animejs-expert` agent: Anime.js V4 specialist that preloads the `animejs` skill.

### Changed

- Renamed the `angular-expert-skill` skill to `angular-expert` and `clearly-and-concisely` to `writing-clearly-and-concisely`. Copies installed under the old names do not update: copy the renamed folders again and remove the old ones.
- Aligned all skills with the skill authoring best practices: third-person descriptions with explicit triggers, tables of contents on reference files over 100 lines, direct links from each SKILL.md to its reference files, and no time-sensitive wording.

### Removed

- Both copies of `angular-knowledge-index.md`: they duplicated the table in the `angular-expert` SKILL.md.

