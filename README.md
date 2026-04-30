# Jetpack Skills

Jetpack Skills is a Codex plugin collection for Android modern development. It focuses on Jetpack, Jetpack Compose, MAD architecture, lifecycle-aware UI, state management, navigation, performance, testability, and code review practices.

This repository currently keeps both marketplace entry formats:

- Codex marketplace: `.agents/plugins/marketplace.json`
- Claude plugin marketplace: `.claude-plugin/marketplace.json`

The project will continue to grow as a curated set of Jetpack-related skills.

## Skill List

| Skill | Status | Summary | Main Use Cases |
| --- | --- | --- | --- |
| [compose-best-practices](./plugins/compose-best-practices/SKILL.md) | Available | Jetpack Compose best practices for MAD architecture, recomposition performance, state, side effects, Lazy layouts, component APIs, accessibility, and Preview coverage. | Compose code review, UI refactor, architecture governance |

## Quick Start

```bash
# Clone the skill repository
git clone git@github.com:huanglinqing123/Jetpack_skills.git ~/Jetpack_skills

# Add it as a local Codex plugin marketplace if your environment supports local marketplaces
# codex plugin marketplace add ~/Jetpack_skills

# For local plugin development, keep the repository path available to Codex
# and iterate on skills under ./plugins/<skill-name>/SKILL.md
```

## License

MIT
