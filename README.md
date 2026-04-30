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

## Project Structure

```text
Jetpack_skills/
├── .agents/
│   └── plugins/
│       └── marketplace.json              # Codex marketplace config
├── .claude-plugin/
│   ├── marketplace.json                  # Claude plugin marketplace config
│   └── plugin.json                       # Claude plugin metadata
├── .codex-plugin/
│   └── plugin.json                       # Codex plugin metadata
├── assets/                               # Plugin icons and visual assets
├── plugins/
│   └── compose-best-practices/
│       ├── SKILL.md                      # Skill entry point
│       └── references/                   # Detailed topic references
│           ├── component-api-preview-accessibility.md
│           ├── performance-and-lazy-layout.md
│           ├── side-effects-and-lifecycle.md
│           └── state-and-architecture.md
├── LICENSE
├── PRIVACY.md
└── README.md
```

## Development

Add new skills under `plugins/`. Each skill should live in its own directory and include a `SKILL.md` file.

Recommended layout:

```text
plugins/
└── new-skill-name/
    ├── SKILL.md
    └── references/
        └── topic-reference.md
```

Keep `SKILL.md` concise. Put long examples, detailed rules, specs, and topic-specific guidance into a local `references/` directory, then link those references from `SKILL.md`.

Before publishing a change, review:

- `.codex-plugin/plugin.json`
- `.claude-plugin/plugin.json`
- `.agents/plugins/marketplace.json`
- `.claude-plugin/marketplace.json`

## Roadmap

This repository is intended to evolve into a broader Jetpack skill set. Planned areas include:

- Jetpack Compose performance and stability
- Compose navigation and screen architecture
- Lifecycle-aware state management
- Compose UI testing and semantics
- Material Design component implementation
- Paging, Room, DataStore, WorkManager, and other Jetpack library practices

## License

MIT
