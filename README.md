# PHP Native Skills

Agent skills for setting up and auditing Laravel + NativePHP Desktop v2 applications. These skills guide AI coding agents through desktop-specific optimizations that Laravel's web-first defaults don't account for.

## Why These Skills Exist

Laravel and PHP are configured for shared web servers by default. When you build a desktop app with NativePHP, those defaults become a problem:

- **PHP limits are too restrictive.** `memory_limit=128M` and `max_execution_time=30` make sense on shared hosting — not on a machine with 16GB RAM running a single-user app.
- **CSRF protection is pointless.** There's no cross-site anything in a desktop app. The middleware just adds friction.
- **SQLite is undertuned.** Conservative defaults protect against multi-user concurrency you'll never have. WAL mode, relaxed synchronous, and bigger caches make a real difference.
- **External dependencies break offline.** Google Fonts from a CDN? Works in dev, fails silently when the user has no internet. Desktop apps must bundle everything.
- **Startup feels slow.** Without a loading page, the Electron window sits blank while PHP boots. Users think the app is broken.
- **Missing extensions crash the app.** A PHP extension that's available in development but not bundled in the NativePHP build means a broken app with no fix for the end user.

These skills encode all of this knowledge into a guided, conversational workflow so you don't have to remember it yourself.

## Installation

```bash
npx skills add <owner>/php-native-skills
```

This installs both skills into your project. They're available as slash commands in any AI coding agent that supports the [Agent Skills](https://agentskills.io) standard (Claude Code, Cursor, GitHub Copilot, VS Code, and others).

## Skills

### `/php-native-setup`

**Use when:** Starting a new Laravel + NativePHP desktop app or adding NativePHP to an existing Laravel project.

The setup skill detects your project's current state and adapts:

| State | What happens |
|---|---|
| Empty directory | Tells you to run `laravel new .` first (respects Laravel's interactive installer) |
| Laravel, no NativePHP | Asks Electron or Tauri, installs your choice, then optimizes |
| Laravel + NativePHP | Skips installation, goes straight to optimization |
| NativePHP Mobile | Stops — these skills are for Desktop v2 only |

Then it walks through all 9 optimization areas one by one, explaining each change and waiting for your approval before applying anything.

### `/php-native-audit`

**Use when:** You have an existing Laravel + NativePHP project and want to check if it's optimized for desktop use.

The audit skill works differently from setup:

1. **Scans silently** — checks all 9 areas without printing anything
2. **Filters** — skips areas already optimally configured
3. **Presents findings one at a time** — professional tone, clear rationale, your decision on each
4. **Summarizes** — what was changed, what was already fine, what you skipped

It's designed to be re-run periodically. After adding new dependencies, pulling in a package that uses a CDN, or updating Laravel — run the audit again.

## The 9 Optimization Areas

Both skills cover the same areas with consistent naming and logic:

### a) PHP Configuration

Checks `memory_limit` and `max_execution_time`. Doesn't blindly set values — asks about your app's workload first. A simple CRUD app might be fine with defaults; an app that processes images or imports large datasets needs more headroom. Changes are applied via `ini_set()` in a service provider so they work in both development and production builds.

### b) Middleware Cleanup

Desktop apps don't need web-server middleware. The skills check for and offer to remove:

| Middleware | Why it's unnecessary |
|---|---|
| `VerifyCsrfToken` | No cross-site context in a desktop app |
| `PreventRequestsDuringMaintenance` | Maintenance mode is meaningless locally |
| `TrustProxies` | No reverse proxy between user and app |

Handles both Laravel 10-11 (`app/Http/Kernel.php`) and Laravel 12+ (`bootstrap/app.php`) middleware registration by checking the `laravel/framework` version in `composer.json`.

### c) SQLite Tuning

NativePHP apps use SQLite — the right choice for single-user desktop. But SQLite's defaults are conservative for multi-user scenarios you'll never have. The skills propose these pragmas:

| Pragma | Value | Why |
|---|---|---|
| `journal_mode` | `wal` | Concurrent reads during writes |
| `synchronous` | `normal` | Safe for single-user, faster writes |
| `cache_size` | `-20000` | 20MB in-memory page cache |
| `busy_timeout` | `5000` | Graceful wait instead of lock errors |
| `mmap_size` | `134217728` | 128MB memory-mapped I/O |
| `temp_store` | `memory` | RAM for temporary storage |

On Laravel 12+, these go in the native `pragmas` array in `config/database.php`. On Laravel 10-11, they're set as individual keys in the SQLite connection config.

### d) Service Drivers

Laravel defaults to Redis, Pusher, and cloud mail services that won't exist on a user's desktop. The skills check and propose desktop-appropriate alternatives:

| Service | Default (web) | Desktop alternative |
|---|---|---|
| Queue | `redis` | `sync` or `database` |
| Broadcasting | `pusher` | `log` or `null` |
| Mail | Cloud provider | `log` or local `smtp` |

### e) Loading Page

NativePHP has no built-in desktop splash screen. Without a loading page, the Electron/Tauri window sits blank while PHP boots — users think the app is broken.

The skills create a dedicated `/loading` route with `Window::open()->route('loading')` in `NativeAppServiceProvider`. The view is a lightweight Blade template with inline CSS only (no Vite, no external assets) so it renders near-instantly. JavaScript redirects to the main app route when ready.

You choose what to display: app name, logo, spinner, custom content.

### f) CDN Asset Bundling

Desktop apps must work offline. Any CDN reference is a liability. The skills scan:

- `resources/` — Blade templates and CSS files for `<link>`, `<script>`, and `@import` tags pointing to external URLs
- `tailwind.config.js` — font-family references that assume internet access (fonts load in dev via the browser, fail silently offline)

For each found reference, the skills present it and offer to download and bundle locally. After bundling fonts, they verify that `@font-face` declarations point to the local files.

### g) PHP Extensions

A missing PHP extension in the packaged app means a broken app with no fix for the end user. The skills:

1. Scan `composer.json` for `ext-*` requirements
2. Scan PHP source files for extension-specific function calls
3. Cross-check against the `php_extensions` list in `config/nativephp.php`
4. Flag any required extension that isn't configured for bundling

### h) Build Optimization

These optimizations belong in the **build/packaging pipeline**, not in development:

- **OPcache preloading** — a preload script for Laravel core classes
- **Composer autoloader** — `composer dump-autoload --classmap-authoritative`
- **Laravel caches** — `config:cache`, `route:cache`, `view:cache`

The skills help wire these into your build script. They explicitly do NOT run them during development setup (cached config hides changes and causes confusion during dev).

### i) Laravel Octane

Octane keeps the Laravel application in memory between requests, eliminating bootstrap overhead on every request. The skills check if it's installed and suggest adding it if not — with an explanation of the tradeoffs.

## Supported Platforms

| Platform | Supported |
|---|---|
| NativePHP Desktop v2 (Electron) | Yes |
| NativePHP Desktop v2 (Tauri) | Yes |
| NativePHP Mobile v3 | No (different architecture, different optimization needs) |

## Requirements

- Laravel 10, 11, or 12+
- NativePHP Desktop v2
- An AI coding agent that supports [Agent Skills](https://agentskills.io) (Claude Code, Cursor, GitHub Copilot, VS Code, etc.)

## How It Works

These skills are markdown files that AI coding agents load and follow. They don't contain executable code — they contain structured instructions that guide the agent through a decision-making process with you in control.

The agent reads the skill, runs the detection logic, checks your project's current state, and walks you through each optimization area. You decide what to change. Nothing is applied without your explicit consent.

## Design Principles

- **You are in control.** Every change is presented and approved before applying.
- **Context-dependent.** PHP config values aren't blindly set — they depend on what your app actually does.
- **Version-aware.** Middleware location and SQLite pragma syntax adapt to your Laravel version.
- **Offline-first.** CDN bundling ensures the desktop app works without internet.
- **Dev vs production.** Build optimizations go in the build pipeline, not in your development workflow.
- **Professional tone.** Clear explanations, not chatty or alarmist. Like a consultant's recommendation.

## Contributing

Found a missing optimization area? Open an issue or PR. The skills are just markdown files in `skills/` — easy to read, review, and extend.

## License

MIT
