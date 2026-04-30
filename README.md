# jetpack-skill

Plugin project for Jetpack and modern Android development skills.

## Project Structure

```text
jetpack-skill/
  .codex-plugin/
    plugin.json
  .agents/
    plugins/
      marketplace.json
  .claude-plugin/
    plugin.json
    marketplace.json
  assets/
  plugins/
    compose-best-practices/
      SKILL.md
      references/
```

## Development

Add skills under `plugins/`. Each skill should live in its own directory and include a `SKILL.md` file. Long examples, specs, or topic-specific rules can be split into a local `references/` directory and linked from `SKILL.md`.

Before publishing, review the plugin manifests:

- `.codex-plugin/plugin.json`
- `.claude-plugin/plugin.json`

If this repository is used as a local marketplace entry, keep the root plugin registered in both marketplace files:

- `.agents/plugins/marketplace.json`
- `.claude-plugin/marketplace.json`

Then add real assets for any plugin icon and logo used by Codex.

## License

MIT
