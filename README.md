
<h1 align="center">Murphiyii-skills</h1>

<p align="center">
  <img src="assets/logo/combo-03a-06a.svg" width="128" alt="Murphiyii-skills logo">
</p>

<p align="center">
  <a href="https://github.com/Murphiyii/murphiyii-skills/releases"><img src="https://img.shields.io/github/v/release/Murphiyii/murphiyii-skills" alt="Release"></a>
  <a href="https://github.com/Murphiyii/murphiyii-skills/stargazers"><img src="https://img.shields.io/github/stars/Murphiyii/murphiyii-skills?style=social" alt="Stars"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-Apache%202.0-blue.svg" alt="License: Apache 2.0"></a>
  <img src="https://img.shields.io/badge/built%20for-Claude%20Code-D97757" alt="Built for Claude Code">
  <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs welcome">
  <img src="https://img.shields.io/badge/skills-8-blue.svg" alt="Skills">
</p>

<p align="center">A workspace of Claude Code skills and agents.</p>

## Contents

<details>
<summary>Folder layout</summary>

```
murphiyii-skills/
├── CHANGELOG.md                     Notable changes
├── CLAUDE.md                        Working rules for Claude Code in this repo
├── LICENSE                          Apache 2.0 license
├── README.md                        This file
├── agents/
│   ├── animejs-expert.md            Writes, debugs, and migrates Anime.js V4 animations
│   └── skill-linter.md              Read-only subagent that validates skills
└── skills/
    ├── angular-expert/              Angular 20+ guidance
    ├── animejs/                     Anime.js 4.5.0 guidance
    ├── writing-clearly-and-concisely/ Strunk's writing rules
    ├── error-logging/               Logs problems to agent-errors.log
    ├── handoff/                     Handoff documents for the next agent
    ├── premortem/                   Failure analysis of a concrete plan
    ├── prompt-builder/              Rewrites prompts for a specific Claude model
    └── vanilla-web-architecture/    Architecture for no-framework web apps
```

</details>

## Install

Claude Code loads skills from `~/.claude/skills/` and agents from `~/.claude/agents/`. Copy the ones you want, for example:

```bash
cp -r skills/handoff ~/.claude/skills/
cp agents/skill-linter.md ~/.claude/agents/
```

Copies do not update when this repo changes, so copy again after edits.

## Skills

| Skill | What it does |
|---|---|
| [angular-expert](skills/angular-expert/SKILL.md) | Writes, reviews, and architects Angular 20+ components, services, routing, and forms |
| [animejs](skills/animejs/SKILL.md) | Writes, debugs, reviews, and migrates Anime.js 4.5.0 animations, including V3 to V4 migration |
| [writing-clearly-and-concisely](skills/writing-clearly-and-concisely/SKILL.md) | Applies Strunk's rules to any prose people will read |
| [error-logging](skills/error-logging/SKILL.md) | Appends problems to `agent-errors.log` so the next agent inherits the warning |
| [handoff](skills/handoff/SKILL.md) | Writes a `handoff-YYYY-MM-DD-HHmm.md` summary for another agent. Optional argument: the next session's focus |
| [premortem](skills/premortem/SKILL.md) | Assumes a concrete plan has failed and works backward to failure modes and mitigations. Not for idea-stage brainstorms |
| [prompt-builder](skills/prompt-builder/SKILL.md) | Rewrites a prompt for a specific Claude model |
| [vanilla-web-architecture](skills/vanilla-web-architecture/SKILL.md) | Architecture for plain HTML, CSS, and JavaScript apps with no framework or build step |


## Agents

**[animejs-expert](agents/animejs-expert.md)** writes, debugs, reviews, and migrates Anime.js V4 animations. Requires the `animejs` skill to be installed.

**[skill-linter](agents/skill-linter.md)** checks `SKILL.md` files for broken frontmatter, a `name` that does not match its folder, missing referenced files, and references to agents or skills that do not exist.


## Changelog

Notable changes are listed in [CHANGELOG.md](CHANGELOG.md).

## License

Released under the [Apache License 2.0](LICENSE).
