# Baseline Test: php-native-audit (without skill)

## What the agent got right

- Scans everything silently first before presenting (good)
- Comprehensive list of what to check in each area
- Professional, direct tone — no hedging or alarm language
- Correctly identified CDN dependencies as critical for desktop apps
- Would not apply changes without asking

## What the agent got wrong — the skill must correct these

1. **All-at-once report, not one-by-one** — Agent presented findings as a structured report (Critical/Suboptimal/Advisory tiers). Spec requires presenting one finding at a time, waiting for user decision before moving to next.

2. **Batch approval** — Agent would ask "want me to fix all, some, or none?" instead of asking per-finding. Spec requires individual consent per change.

3. **No pre-checks** — Agent didn't verify this is actually a Laravel + NativePHP project before auditing. Spec requires checking for artisan and nativephp/electron in composer.json.

4. **Mentioned correctly-configured areas** — Agent wants to list "Confirmed Correct" items to "build trust." Spec says skip areas already optimally configured — don't waste the user's time.

5. **Summary at top** — Agent put summary first as "executive overview." Spec has summary at the end after the walkthrough.

6. **Severity tiers not needed** — Agent categorized by Critical/Suboptimal/Advisory. The one-by-one format makes this unnecessary; each finding speaks for itself.

## Key insight

The agents want to produce a REPORT. The skill must enforce a CONVERSATION — one finding at a time, individual decisions, skip what's fine. The content knowledge is already there; the interaction model is what needs enforcing.
