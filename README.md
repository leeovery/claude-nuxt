<h1 align="center">Claude Nuxt</h1>

<p align="center">
  <strong>Opinionated Nuxt 4 Skills & Commands for Claude Code</strong>
</p>

<p align="center">
  <a href="#installation">Installation</a> •
  <a href="#skills">Skills</a> •
  <a href="#commands">Commands</a> •
  <a href="#how-it-works">How It Works</a> •
  <a href="#contributing">Contributing</a>
</p>

---

## About

This is a curated collection of Nuxt 4 + Vue 3 development patterns refined across multiple production applications in the Fabric ecosystem.

**These are opinionated.** They represent a domain-driven, type-safe, composable-first architecture—featuring the feature module pattern (queries/mutations/actions), model hydration with casting, and a clear separation between data and presentation layers. They won't be for everyone, and that's okay.

**This is a work in progress.** As I use these skills in real projects, I'm continuously refining them to better represent how I actually work. Expect updates as patterns evolve and edge cases reveal themselves.

## Installation

```bash
composer require --dev leeovery/claude-nuxt
```

That's it. The [Claude Manager](https://github.com/leeovery/claude-manager) handles everything else automatically.

## How It Works

This package depends on [`leeovery/claude-manager`](https://github.com/leeovery/claude-manager), which:

1. **Symlinks skills** into your project's `.claude/skills/` directory
2. **Symlinks commands** into your project's `.claude/commands/` directory
3. **Manages your `.gitignore`** with a deterministic list of linked skills and commands
4. **Handles installation/removal** automatically via Composer hooks

You don't need to configure anything—just install and start coding.

## Skills

Each skill provides focused guidance on a specific aspect of Nuxt development.

### Foundation

| Skill | Description |
|-------|-------------|
| [**nuxt-architecture**](skills/nuxt-architecture/) | Project structure, philosophy, technology stack, and pattern selection |
| [**nuxt-layers**](skills/nuxt-layers/) | Working with base, nuxt-ui, and x-ui shared layers |

### Core Patterns

| Skill | Description |
|-------|-------------|
| [**nuxt-models**](skills/nuxt-models/) | Domain models with hydration, relations, casts, and value objects |
| [**nuxt-enums**](skills/nuxt-enums/) | Class-based enums with Castable interface and behavior methods |
| [**nuxt-repositories**](skills/nuxt-repositories/) | Data access layer with BaseRepository and model hydration |

### Feature Layer

| Skill | Description |
|-------|-------------|
| [**nuxt-features**](skills/nuxt-features/) | Feature module pattern—queries, mutations, and actions |

### UI Layer

| Skill | Description |
|-------|-------------|
| [**nuxt-components**](skills/nuxt-components/) | Component patterns with script setup order convention |
| [**nuxt-pages**](skills/nuxt-pages/) | File-based routing with list/detail page patterns |
| [**nuxt-composables**](skills/nuxt-composables/) | Core composables—useWait, useFlash, useReactiveFilters, and more |
| [**nuxt-forms**](skills/nuxt-forms/) | Form handling with XForm component and validation |
| [**nuxt-tables**](skills/nuxt-tables/) | Column builder pattern and XTable component |

### Infrastructure

| Skill | Description |
|-------|-------------|
| [**nuxt-config**](skills/nuxt-config/) | Configuration with nuxt.config.ts, app.config.ts, and environment variables |
| [**nuxt-auth**](skills/nuxt-auth/) | Laravel Sanctum authentication and permission-based authorization |
| [**nuxt-realtime**](skills/nuxt-realtime/) | Real-time features with Laravel Echo and WebSockets |
| [**nuxt-errors**](skills/nuxt-errors/) | Error handling with typed error classes and interceptors |

## Commands

Slash commands for common Nuxt development tasks.

*Coming soon.*

## Requirements

- PHP ^8.2
- [leeovery/claude-manager](https://github.com/leeovery/claude-manager) ^1.0 (installed automatically)

## Contributing

Contributions are welcome! Whether it's:

- **Bug fixes** in the documentation
- **Improvements** to existing patterns
- **Discussion** about approaches and trade-offs
- **New skills** for patterns not yet covered

Please open an issue first to discuss significant changes. These are opinionated patterns, so let's talk through the approach before diving into code.

## Related Packages

- [**claude-manager**](https://github.com/leeovery/claude-manager) — The plugin manager that powers skill installation
- [**claude-laravel**](https://github.com/leeovery/claude-laravel) — Laravel development skills for Claude Code
- [**claude-technical-workflows**](https://github.com/leeovery/claude-technical-workflows) — Technical workflow skills for Claude Code

## License

MIT License. See [LICENSE](LICENSE) for details.

---

<p align="center">
  <sub>Built with care by <a href="https://github.com/leeovery">Lee Overy</a></sub>
</p>
