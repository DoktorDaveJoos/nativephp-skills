# PHP Native Skills

Agent skills for setting up and auditing Laravel + NativePHP Desktop v2 applications.

## Installation

```bash
npx skills add <owner>/php-native-skills
```

## Skills

### `/php-native-setup`

Guides you through setting up a Laravel + NativePHP desktop app from scratch. Detects your project state, installs NativePHP if needed, and walks through 9 optimization areas with your consent on each change.

### `/php-native-audit`

Audits an existing Laravel + NativePHP project for desktop optimization issues. Scans silently, then presents only the findings that need attention — one at a time, professionally, with your decision on each.

## Optimization Areas

Both skills cover these 9 areas:

| Area | What it covers |
|---|---|
| PHP Configuration | `memory_limit`, `max_execution_time` via `ini_set()` |
| Middleware Cleanup | CSRF, maintenance mode, proxy trust |
| SQLite Tuning | WAL, synchronous, cache, busy timeout, mmap |
| Service Drivers | Queue, broadcasting, mail — desktop-appropriate drivers |
| Loading Page | Lightweight `/loading` route for instant window display |
| CDN Asset Bundling | Offline-proof fonts, CSS, JS — including Tailwind |
| PHP Extensions | Required vs bundled extension verification |
| Build Optimization | OPcache, Composer classmap, Laravel caches |
| Laravel Octane | Application kept in memory between requests |

## Requirements

- Laravel 10+
- NativePHP Desktop v2 (Electron or Tauri)
