# PHP Native Skills

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![NativePHP Desktop v2](https://img.shields.io/badge/NativePHP-Desktop%20v2-orange.svg)](https://nativephp.com)
[![Laravel 10|11|12+](https://img.shields.io/badge/Laravel-10%20%7C%2011%20%7C%2012%2B-red.svg)](https://laravel.com)

Agent skills for setting up and auditing Laravel + NativePHP Desktop v2 applications. These skills guide AI coding agents through desktop-specific optimizations that Laravel's web-first defaults don't account for.

## Quick Start

```bash
npx skills@latest add DoktorDaveJoos/nativephp-skills
```

Then run `/nativephp-setup` to set up a new project or `/nativephp-audit` to check an existing one.

## Table of Contents

- [Why These Skills Exist](#why-these-skills-exist)
- [Why Not Just Ask the AI?](#why-not-just-ask-the-ai)
- [Skills](#skills)
  - [/nativephp-setup](#nativephp-setup)
  - [/nativephp-audit](#nativephp-audit)
- [The 8 Optimization Areas](#the-8-optimization-areas)
- [Supported Platforms](#supported-platforms)
- [Requirements](#requirements)
- [How It Works](#how-it-works)
- [Design Principles](#design-principles)
- [FAQ](#faq)
- [Contributing](#contributing)
- [License](#license)

## Why These Skills Exist

Laravel and PHP are configured for shared web servers by default. When you build a desktop app with NativePHP, those defaults become a problem:

- **PHP limits are too restrictive.** `memory_limit=128M` and `max_execution_time=30` make sense on shared hosting — not on a machine with 16GB RAM running a single-user app.
- **CSRF protection is pointless.** There's no cross-site anything in a desktop app. The middleware just adds friction.
- **SQLite is undertuned.** Conservative defaults protect against multi-user concurrency you'll never have. WAL mode, relaxed synchronous, and bigger caches make a real difference.
- **External dependencies break offline.** Google Fonts from a CDN? Works in dev, fails silently when the user has no internet. Desktop apps must bundle everything.
- **Startup feels slow.** NativePHP hides the window until content loads — no blank screen, but no visible window either. Without a loading page, the user sees nothing but a dock icon while PHP boots.
- **Missing extensions crash the app.** A PHP extension that's available in development but not bundled in the NativePHP build means a broken app with no fix for the end user.

These skills encode all of this knowledge into a guided, conversational workflow so you don't have to remember it yourself.

## Why Not Just Ask the AI?

AI coding agents already know most of the individual optimizations. What they lack is structure. We tested agents without these skills and found consistent gaps:

| Without the skill | With the skill |
|---|---|
| Jumps straight into recommendations | Detects project state first (empty dir, Laravel only, NativePHP installed, Mobile) |
| Uses `composer create-project` for new projects | Enforces `laravel new .` to respect Laravel's interactive installer |
| Dumps all recommendations as a knowledge dump | Presents one area at a time, waits for your decision |
| Offers "fix all" batch approval | Requires informed consent per change |
| Audit produces a tiered report (Critical/Suboptimal/Advisory) | Audit is a conversation: scan silently, present only what needs attention |
| Lists correctly-configured areas to "build trust" | Skips what's already fine — doesn't waste your time |
| Buries loading page in startup performance | Makes the loading page a distinct, actionable step |
| Leaves detection to agent judgment | Provides exact detection commands for consistent results |

The content knowledge is already there. The interaction model is what these skills enforce.

## Skills

### `/nativephp-setup`

**Use when:** Starting a new Laravel + NativePHP desktop app or adding NativePHP to an existing Laravel project.

The setup skill detects your project's current state and adapts:

| State | What happens |
|---|---|
| Empty directory | Tells you to run `laravel new .` first (respects Laravel's interactive installer) |
| Laravel, no NativePHP | Asks Electron or Tauri, installs your choice, then optimizes |
| Laravel + NativePHP | Skips installation, goes straight to optimization |
| NativePHP Mobile | Stops — these skills are for Desktop v2 only |

Then it walks through all 8 optimization areas one by one, explaining each change and waiting for your approval before applying anything.

**Example interaction:**

> `/nativephp-setup`
>
> Found a Laravel 12 project with `nativephp/electron` installed.
> Skipping installation — going straight to optimization.
>
> **a) PHP Configuration**
> Your current `memory_limit` is `128M`. What kind of workload does this app handle — simple CRUD, file processing, image manipulation?
>
> *It processes CSV imports up to 50MB.*
>
> For large file imports, `512M` is reasonable. I'll add `ini_set('memory_limit', '512M')` to `AppServiceProvider::boot()`. Apply this change?
>
> *Yes.*
>
> Done. Next up:
>
> **b) Middleware Cleanup**
> `VerifyCsrfToken` is active. Desktop apps have no cross-site context — this middleware adds friction without security benefit. Remove it?

### `/nativephp-audit`

**Use when:** You have an existing Laravel + NativePHP project and want to check if it's optimized for desktop use.

The audit skill works differently from setup:

1. **Scans silently** — checks all 8 areas without printing anything
2. **Filters** — skips areas already optimally configured
3. **Presents findings one at a time** — professional tone, clear rationale, your decision on each
4. **Summarizes** — what was changed, what was already fine, what you skipped

It's designed to be re-run periodically. After adding new dependencies, pulling in a package that uses a CDN, or updating Laravel — run the audit again.

**Example interaction:**

> `/nativephp-audit`
>
> Verified: Laravel 12 + NativePHP Desktop (Electron). Scanning...
>
> Found 2 areas needing attention. 6 areas are already optimally configured.
>
> **CDN Asset Bundling**
> Currently: Google Fonts loaded from `fonts.googleapis.com` in `resources/views/layouts/app.blade.php:12`
> Recommended: Download and bundle locally
> Rationale: Desktop apps must work offline. CDN fonts fail silently when the user has no internet connection.
> Apply this change?
>
> *Yes.*
>
> Downloaded Inter font family to `public/fonts/`. Updated `@font-face` declarations to reference local files.
>
> **Service Drivers**
> Currently: Queue driver set to `redis` in `config/queue.php`
> Recommended: Switch to `sync` or `database`
> Rationale: Redis won't be available on the end user's machine.
> Apply this change?
>
> *Switch to database.*
>
> Done. Here's your summary:
> - **Applied:** CDN Asset Bundling (Google Fonts bundled locally), Service Drivers (queue: `redis` → `database`)
> - **Already optimal:** PHP Configuration, Middleware, SQLite Tuning, Loading Page, PHP Extensions, Build Optimization
> - **Skipped:** none

## The 8 Optimization Areas

Both skills cover the same areas with consistent naming and logic:

| # | Area | What it optimizes |
|---|------|-------------------|
| a | [PHP Configuration](#a-php-configuration) | Memory and execution limits for desktop context |
| b | [Middleware Cleanup](#b-middleware-cleanup) | Removes web-only middleware (CSRF, maintenance, proxies) |
| c | [SQLite Tuning](#c-sqlite-tuning) | WAL mode, cache, sync settings for single-user |
| d | [Service Drivers](#d-service-drivers) | Replaces cloud services with local alternatives |
| e | [Loading Page](#e-loading-page) | Reduces perceived startup time with instant visual feedback |
| f | [CDN Asset Bundling](#f-cdn-asset-bundling) | Downloads external assets for offline use |
| g | [PHP Extensions](#g-php-extensions) | Ensures required extensions are bundled |
| h | [Build Optimization](#h-build-optimization) | OPcache, autoloader, Laravel caches for production |

#### a) PHP Configuration

Checks `memory_limit` and `max_execution_time`. Doesn't blindly set values — asks about your app's workload first. A simple CRUD app might be fine with defaults; an app that processes images or imports large datasets needs more headroom. Changes are applied via `ini_set()` in a service provider so they work in both development and production builds.

#### b) Middleware Cleanup

Desktop apps don't need web-server middleware. The skills check for and offer to remove:

| Middleware | Why it's unnecessary |
|---|---|
| `VerifyCsrfToken` | No cross-site context in a desktop app |
| `PreventRequestsDuringMaintenance` | Maintenance mode is meaningless locally |
| `TrustProxies` | No reverse proxy between user and app |

Handles both Laravel 10-11 (`app/Http/Kernel.php`) and Laravel 12+ (`bootstrap/app.php`) middleware registration by checking the `laravel/framework` version in `composer.json`.

#### c) SQLite Tuning

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

#### d) Service Drivers

Laravel defaults to Redis, Pusher, and cloud mail services that won't exist on a user's desktop. The skills check and propose desktop-appropriate alternatives:

| Service | Default (web) | Desktop alternative |
|---|---|---|
| Queue | `redis` | `sync` or `database` |
| Broadcasting | `pusher` | `log` or `null` |
| Mail | Cloud provider | `log` or local `smtp` |

#### e) Loading Page

NativePHP creates every `BrowserWindow` with `show: false` and waits for `did-finish-load` before showing it — so there's never a blank window. But there's also no window at all during startup (just a dock/taskbar icon), which can feel slow on heavier apps.

A lightweight `/loading` route makes the window appear faster: it loads instantly (inline HTML, no Vite, no external assets), giving the user immediate visual feedback while the main app bootstraps. The skills create a dedicated `/loading` route with `Window::open()->route('loading')` in `NativeAppServiceProvider`. JavaScript redirects to the main app route when ready.

You choose what to display: app name, logo, spinner, custom content.

#### f) CDN Asset Bundling

Desktop apps must work offline. Any CDN reference is a liability. The skills scan:

- `resources/` — Blade templates and CSS files for `<link>`, `<script>`, and `@import` tags pointing to external URLs
- `tailwind.config.js` — font-family references that assume internet access (fonts load in dev via the browser, fail silently offline)

For each found reference, the skills present it and offer to download and bundle locally. After bundling fonts, they verify that `@font-face` declarations point to the local files.

#### g) PHP Extensions

A missing PHP extension in the packaged app means a broken app with no fix for the end user. The skills:

1. Scan `composer.json` for `ext-*` requirements
2. Scan PHP source files for extension-specific function calls
3. Cross-check against the `php_extensions` list in `config/nativephp.php`
4. Flag any required extension that isn't configured for bundling

#### h) Build Optimization

These optimizations belong in the **build/packaging pipeline**, not in development:

- **OPcache preloading** — a preload script for Laravel core classes
- **Composer autoloader** — `composer dump-autoload --classmap-authoritative`
- **Laravel caches** — `config:cache`, `route:cache`, `view:cache`

The skills help wire these into your build script. They explicitly do NOT run them during development setup (cached config hides changes and causes confusion during dev).

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

These skills are markdown files that AI coding agents load and follow. They're built on the [Agent Skills](https://agentskills.io) standard — a way to package domain knowledge as structured instructions that compatible editors load into the AI's context when you invoke them via slash commands.

The skills don't contain executable code. The agent reads the skill, runs the detection logic, checks your project's current state, and walks you through each optimization area. You decide what to change. Nothing is applied without your explicit consent.

## Design Principles

- **You are in control.** Every change is presented and approved before applying.
- **Context-dependent.** PHP config values aren't blindly set — they depend on what your app actually does.
- **Version-aware.** Middleware location and SQLite pragma syntax adapt to your Laravel version.
- **Offline-first.** CDN bundling ensures the desktop app works without internet.
- **Dev vs production.** Build optimizations go in the build pipeline, not in your development workflow.
- **Professional tone.** Clear explanations, not chatty or alarmist. Like a consultant's recommendation.

## FAQ

**Can I run the audit on a project I already set up with the setup skill?**
Yes. The audit scans current state and skips anything already optimized. It's designed to be re-run after adding dependencies, pulling in new packages, or updating Laravel.

**What if I decline a recommendation?**
It's skipped and noted in the summary. No changes are made. You can re-run the skill later if you change your mind.

**Do these work with NativePHP Mobile v3?**
No. Mobile has a different architecture and different optimization needs. Both skills detect Mobile and stop with a clear message.

**What Laravel versions are supported?**
10, 11, and 12+. The skills detect your version and adapt — middleware location (`Kernel.php` vs `bootstrap/app.php`) and SQLite pragma syntax change between versions.

**Are any code changes made without my permission?**
No. Every change is presented with its rationale and requires your explicit consent before applying. You can approve, decline, or modify each recommendation.

**What if my project uses Tauri instead of Electron?**
Both are fully supported. The setup skill asks which backend you want if neither is installed. The audit skill detects whichever is present and proceeds.

## Contributing

Found a missing optimization area? Open an issue or PR.

The skills are markdown files in `nativephp-setup/` and `nativephp-audit/` — easy to read, review, and extend. Design rationale and decisions are documented in `docs/superpowers/specs/`. If you want to understand why the skills work the way they do, start there.

To validate changes, test both skills against a real Laravel + NativePHP project. The `docs/superpowers/tests/` directory contains baseline test results showing how agents behave without the skills — useful context for understanding what each instruction in the skills is correcting.

## License

MIT
