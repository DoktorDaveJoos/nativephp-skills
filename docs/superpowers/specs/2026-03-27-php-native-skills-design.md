# PHP Native Skills Design

Two skills for setting up and auditing Laravel + NativePHP desktop applications.

## Context

PHP Native (NativePHP) builds desktop apps on top of Laravel using Electron. Several default PHP and Laravel settings are designed for shared web servers and don't make sense for a single-user desktop app running on local hardware. These skills guide developers through optimizing their setup.

---

## Skill 1: `/php-native-setup`

### Purpose

Bootstrap a Laravel + NativePHP desktop app with optimized settings, walking the user through each decision conversationally.

### Entry Point Detection

1. **Empty directory** — Tell user: "Run `laravel new .` first to set up Laravel with the interactive installer, then run this skill again." Stop.
2. **Laravel installed, no NativePHP** — Install NativePHP, then run the full optimization walkthrough.
3. **Laravel + NativePHP already installed** — Skip installation, go straight to optimizations.

Detection logic:
- Check for `artisan` file and `composer.json` with `laravel/framework` dependency
- Check for `nativephp/electron` in `composer.json`

### Installation Phase (if NativePHP not yet installed)

- `composer require nativephp/electron`
- `php artisan native:install`
- Verify installation succeeded

### Optimization Walkthrough

Each area is presented one-by-one. The skill explains what it wants to change and why, then the user decides. All changes are applied only with user consent.

#### a) PHP Configuration

- Inspect machine specs (RAM, CPU cores) to determine appropriate values
- Settings to optimize:
  - `memory_limit` — scale to available RAM (e.g. 512M-2G instead of 128M)
  - `max_execution_time` — increase significantly (e.g. 300s+), no web timeout concerns
  - `post_max_size` / `upload_max_filesize` — increase for local file handling
  - Other settings as appropriate for desktop context
- Show the user: current value, proposed value, reasoning

#### b) XSRF Token Validation

- Remove or disable `VerifyCsrfToken` middleware
- Rationale: Desktop app has no cross-site request context. XSRF protection adds friction with no security benefit here.

#### c) SQLite Database Tuning

- Configure for single-user desktop performance:
  - Enable WAL (Write-Ahead Logging) journal mode
  - Increase cache size
  - Set `synchronous=NORMAL` (safe for single-user, faster than FULL)
  - Set appropriate `busy_timeout`
- Rationale: No concurrent database users, no need for conservative multi-user defaults.

#### d) Startup Performance

- Run Laravel optimization commands:
  - `php artisan config:cache`
  - `php artisan route:cache`
  - `php artisan view:cache`
- Check for Laravel Octane and other performance boosters
- Generate a static loading page for Electron:
  - Ask the user what they want displayed (app name, logo, spinner, custom content)
  - Create a simple static HTML file that Electron shows immediately on launch
  - Wire it into the NativePHP/Electron configuration
- Rationale: PHP/Laravel boot takes seconds. Showing the Electron window instantly with a loading page eliminates the "is it running?" confusion.

#### e) CDN Asset Bundling

- Scan all Blade templates, CSS files, and JS files for external CDN references:
  - Google Fonts and other web fonts
  - CSS frameworks (Bootstrap, Tailwind CDN, etc.)
  - JavaScript libraries loaded from CDN
  - Icon libraries (Font Awesome, Material Icons, etc.)
- For each found CDN dependency:
  - Offer to download and bundle it locally
  - Update references to point to local files
- Rationale: Desktop app may run without internet. CDN dependencies will silently fail offline.

#### f) PHP Extensions

- Scan `composer.json` and all PHP source files for required extensions
- Cross-check against NativePHP's bundled extension configuration
- Also verify a known list of commonly needed extensions:
  - `sqlite3`, `pdo_sqlite`
  - `mbstring`
  - `openssl`
  - `fileinfo`
  - `json`
  - `tokenizer`
  - `xml`
  - `curl`
  - `dom`
  - `zip`
- Flag any extension that is required but not bundled
- Rationale: Missing extensions at runtime in a packaged desktop app means a broken app with no easy fix for the end user.

### Summary

After the walkthrough, display:
- What was configured
- What was skipped (user chose not to apply)
- Any warnings or recommendations for later

---

## Skill 2: `/php-native-audit`

### Purpose

Inspect an existing Laravel + NativePHP project against desktop-optimized settings. Report findings professionally and walk through each one, letting the user decide what to change.

### Pre-checks

- Verify `artisan` and `composer.json` exist (Laravel project)
- Verify `nativephp/electron` is in `composer.json`
- If either is missing, stop with a clear message

### Audit Process

1. **Silent scan** — Check all 6 optimization areas without making changes. Build internal report.

2. **Filter** — Only surface areas that need attention. Skip anything already optimally configured.

3. **Present findings one-by-one** — For each finding:
   - Current state (what's configured now)
   - Recommended state (what it should be and why)
   - Ask user: apply this change?

### Areas Scanned

Same 6 areas as the setup skill:

| Area | What it checks |
|------|---------------|
| PHP Config | Current values vs machine-appropriate desktop defaults |
| XSRF Token | Whether `VerifyCsrfToken` middleware is still active |
| SQLite Tuning | WAL mode, cache size, synchronous setting, busy_timeout |
| Startup Performance | Config/route/view caches, static loading page presence |
| CDN Dependencies | External CDN references in templates, CSS, JS |
| PHP Extensions | Required extensions vs bundled extensions |

### Tone

Professional and curated. Each finding reads like a consultant's recommendation:
- "Currently `memory_limit` is set to 128M. On this machine with 16GB RAM, a desktop app can safely use 1G. Apply this change?"
- Not: "Hey! You should totally bump that memory limit up!"

### Summary

After walking through all findings:
- List changes applied
- List items already configured correctly
- List items the user chose to skip

---

## Skill Structure

Both skills will be written as standalone skill `.md` files suitable for installation into any project's `.claude/skills/` directory or equivalent.

```
php-native-skills/
  skills/
    php-native-setup.md
    php-native-audit.md
```

## Key Design Principles

- **User is in control** — Every change is presented and approved before applying
- **Laravel-first** — These skills are specifically for Laravel + NativePHP/Electron
- **Machine-aware** — PHP config values are calculated based on actual hardware, not hardcoded
- **Idempotent detection** — Both skills detect current state before acting
- **Offline-ready** — CDN bundling ensures the desktop app works without internet
- **Professional tone** — Clear explanations, not chatty or casual
