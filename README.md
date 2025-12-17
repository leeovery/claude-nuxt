<h1 align="center">Claude Nuxt</h1>

<p align="center">
  <strong>Nuxt.js Development Skills & Commands for Claude Code</strong>
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

Nuxt.js development patterns and practices for Claude Code.

**This is a work in progress.** Skills and commands are actively being developed.

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

*Coming soon.*

## Commands

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

Please open an issue first to discuss significant changes.

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
