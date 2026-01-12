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

## Versions

| Version | Package Manager | Status     | Branch                                                  |
|---------|-----------------|------------|---------------------------------------------------------|
| 2.x     | npm             | **Active** | `main`                                                  |
| 1.x     | Composer        | Deprecated | [`v1`](https://github.com/leeovery/claude-nuxt/tree/v1) |

## About

This is a curated collection of Nuxt 4 + Vue 3 development patterns refined across multiple production applications in the Fabric ecosystem.

**These are opinionated.** They represent a domain-driven, type-safe, composable-first architecture—featuring the feature module pattern (queries/mutations/actions), model hydration with casting, and a clear separation between data and presentation layers. They won't be for everyone, and that's okay.

**This is a work in progress.** As I use these skills in real projects, I'm continuously refining them to better represent how I actually work. Expect updates as patterns evolve and edge cases reveal themselves.

**Model compatibility:** These skills have been developed and refined for Claude Code running on **Opus 4.5**. Different models may exhibit different edge cases, and future model releases may require adjustments to the prompts and workflows.

## Installation

Two installation methods are available:

| Method | Best for | Trade-off |
|--------|----------|-----------|
| **Marketplace** | Local Claude Code | Simple install, skills cached globally |
| **npm** | Claude Code for Web | Skills copied to repo, requires npm |

### Option 1: Claude Marketplace

```
/plugin marketplace add leeovery/claude-plugins-marketplace
/plugin install claude-nuxt@claude-plugins-marketplace
```

> **Note:** Marketplace plugins are cached globally (`~/.claude/plugins/`) and won't be available in Claude Code for Web since files aren't in your repository.

### Option 2: npm (Recommended for Web)

```bash
npm install -D @leeovery/claude-nuxt
```

Skills are copied to `.claude/` in your project and can be committed to your repository—making them available in Claude Code for Web.

<details>
<summary>pnpm users</summary>

pnpm doesn't expose binaries from transitive dependencies, so install the manager directly:

```bash
pnpm add -D @leeovery/claude-manager @leeovery/claude-nuxt
pnpm approve-builds  # approve when prompted
pnpm install         # triggers postinstall
```
</details>

<details>
<summary>Removal (npm/pnpm)</summary>

Due to bugs in npm 7+ ([issue #3042](https://github.com/npm/cli/issues/3042)) and pnpm ([issue #3276](https://github.com/pnpm/pnpm/issues/3276)), preuninstall hooks don't run reliably. Remove files manually first:

```bash
npx claude-manager remove @leeovery/claude-nuxt && npm rm @leeovery/claude-nuxt
```
</details>

## How It Works

This package depends on [`@leeovery/claude-manager`](https://github.com/leeovery/claude-manager), which:

1. **Copies skills** into your project's `.claude/skills/` directory
2. **Copies commands** into your project's `.claude/commands/` directory
3. **Copies agents** into your project's `.claude/agents/` directory
4. **Tracks installed plugins** via a manifest file

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

- Node.js 18+
- [@leeovery/claude-manager](https://github.com/leeovery/claude-manager) ^2.0.0 (installed automatically)

## Contributing

Contributions are welcome! Whether it's:

- **Bug fixes** in the documentation
- **Improvements** to existing patterns
- **Discussion** about approaches and trade-offs
- **New skills** for patterns not yet covered

Please open an issue first to discuss significant changes. These are opinionated patterns, so let's talk through the approach before diving into code.

## Related Packages

- [**@leeovery/claude-manager**](https://github.com/leeovery/claude-manager) — The plugin manager that powers skill installation
- [**@leeovery/claude-laravel**](https://github.com/leeovery/claude-laravel) — Laravel development skills for Claude Code
- [**@leeovery/claude-technical-workflows**](https://github.com/leeovery/claude-technical-workflows) — Technical workflow skills for Claude Code

## License

MIT License. See [LICENSE](LICENSE) for details.

---

<p align="center">
  <sub>Built with care by <a href="https://github.com/leeovery">Lee Overy</a></sub>
</p>
