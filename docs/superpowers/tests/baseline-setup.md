# Baseline Test: php-native-setup (without skill)

## What the agent got right

- All 6 optimization areas were covered with good specifics
- Machine-aware PHP config (inspects RAM/cores, not hardcoded)
- XSRF removal with correct rationale
- SQLite WAL, synchronous, cache_size, busy_timeout
- Config/route/view caching for startup
- CDN scanning and bundling approach
- Extension cross-checking
- One-at-a-time presentation with user consent
- Agent 1 even suggested additional valid optimizations (OPcache preloading, maintenance mode middleware, composer autoload optimization)

## What the agent missed — the skill must teach these

1. **Entry point detection flow** — Neither agent checked if the directory was empty, had Laravel, or had NativePHP. They jumped straight into recommendations without detecting project state.

2. **`laravel new` vs `composer create-project`** — Agent 1 used `composer create-project laravel/laravel .` instead of telling the user to run the interactive `laravel new .` installer. The skill must enforce: detect empty dir → tell user to run `laravel new .` → pick up after.

3. **Structured walkthrough discipline** — While both agents covered the areas, they did it as a knowledge dump, not as a structured conversational walkthrough. The skill must enforce: present one area → wait for decision → next area.

4. **Loading page as first-class step** — Agent 2 covered it well, but Agent 1 buried it in startup performance. The skill must make the loading page a distinct, actionable step that asks the user what to display.

5. **Consistent detection commands** — The skill must provide exact commands for each check (e.g., `grep "nativephp/electron" composer.json`) rather than leaving detection to agent judgment.

## Key insight

The agents already know the CONTENT well. The skill's value is in:
- Structuring the PROCESS (detect → install → walkthrough)
- Enforcing entry point detection logic
- Ensuring the `laravel new` flow (not composer create-project)
- Making the walkthrough conversational, not a knowledge dump
- Ensuring ALL 6 areas are always covered (no cherry-picking)
