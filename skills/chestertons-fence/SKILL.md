---
name: chestertons-fence
description: |
  Use this skill BEFORE deleting, simplifying, or refactoring any code that you don't fully understand the history of. Stack-agnostic — applies to frontend, backend, infrastructure, scripts, schemas, migrations, and config. Trigger whenever you see "redundant-looking" lines (matchers, fallbacks, magic numbers, defensive `if`/`try`/`rescue` branches, module-scope singletons, browser storage flags, short-comment one-liners, polyfills, rate-limit gates, retry loops, idempotency keys, feature-flag fallthroughs). Especially trigger in middleware, auth/refresh flows, fetch/HTTP layers, error handlers, sentry/logging, database migrations, queue/job/worker retry logic, IaC defaults, CI workflows, and any file whose existence reason is non-obvious. The cleaner code "looks", the more suspicious it is. Always run this skill before applying any refactor — even one the user explicitly asked for. The goal is to prevent removing a fence (Chesterton's Fence) without understanding why it was put there.
allowed-tools: Read, Grep, Bash
---

# Chesterton's Fence — Ask "why is this here?" before touching it

> "Don't remove a fence until you know why it was put up." — G.K. Chesterton, *The Thing* (1929)

Code has intent. Removing it without understanding the intent is how AI-assisted refactors create production incidents that pass type-check, pass tests, and fail only in dev / app / one locale / under race conditions.

This skill is a **forcing function**: it makes Claude stop and investigate the history of any code it's about to change.

---

## Stack-agnostic by design

This skill applies to **any layer** — frontend, backend, infrastructure, scripts, configs, IaC, CI workflows, schemas, migrations. **Anywhere code has history, fences exist.** The examples below pull from multiple stacks; pick the ones that fit your repo and ignore the rest.

## When to trigger (be aggressive)

Trigger this skill if **any** of the following is true:

- About to **delete** or **simplify** a line, branch, fallback, set/cache, matcher, magic number, retry count, timeout, or module-scope variable
- About to refactor a `try/catch` (or `rescue`/`except`), `?? fallback`, `if (env === ...)`, `if (typeof window !== "undefined")`, idempotency check, or any environment-dependent guard
- About to drop a column, change a default, reorder a migration, or "tidy" a schema
- About to remove a feature flag, a deprecated route, or a legacy adapter
- The target has **no comment** or only a 1-line comment
- The target lives in fence-dense areas: `middleware.*`, `auth*`, `fetch*`/`http*`, `*Error*`, `polyfill*`, `sentry*`, `*Bridge*`, `*RateLimit*`, `migrations/*`, `*Worker*`/`*Job*`/`*Queue*`, `Dockerfile*`, CI workflows, terraform/k8s manifests
- A magic number or string is involved (`180`, `"INVALID_TOKEN"`, `(ko|en|ja)`, retry counts, backoff windows, etc.)
- The user said "clean this up" / "make it more readable" / "remove the redundant part"

When in doubt, trigger. False positives cost 30 seconds; false negatives cost a 3am page.

---

## The 6-step workflow (mandatory)

### 1. STOP

Do not edit yet. Re-read the file and the lines around it.

### 2. Identify the fence — one sentence

State the change in one sentence:

> "I am about to change `<file>:<line>` from `<X>` to `<Y>` because `<reason>`."

If you can't fill this in cleanly, you don't yet understand the change.

### 3. Investigate — 4 sources, in order

Stop at any step where you find a clear "why".

**(a) Adjacent comments** — read 5 lines above and below, and any comment in the same function.

**(b) `git blame`** — what commit last touched this line? What was the message?

```bash
git blame -L <line>,<line> <file>
git log <commit_sha> -1 --format='%B'
```

**(c) `git log -L`** — how has this line evolved over time?

```bash
git log -L <line>,<line>:<file> --oneline --no-patch
```

**(d) Related code / tests** — grep for the same variable, magic number, or string elsewhere.

```bash
rg "<the_token>" --type=ts --type=tsx
```

### 4. Hypothesize — what breaks if it's gone?

State a falsifiable hypothesis:

> "If this line is removed, in `<environment>` (dev / prod / app / locale X / under race), `<concrete failure>` will occur."

If you can't form a hypothesis, the fence is **still standing for a reason you haven't found yet**. Go back to step 3.

### 5. Decide — three options

| Decision | Meaning | Action |
|---|---|---|
| **Keep** | The fence is still load-bearing | Don't change it. **Reinforce the comment** if it's missing or thin. |
| **Move** | The intent is valid; the implementation can be clearer | Re-implement the same intent (test, type, named constant, explicit guard) |
| **Remove** | The risk that justified it is gone | Document **why it's now safe** in the commit body and PR description |

**Remove only after** confirming all three:

- [ ] The original risk no longer exists (BE deployed everywhere, library version bumped, deprecated API gone, etc.)
- [ ] You've checked the change in **all variant environments** (dev/staging/prod, web/app, every locale)
- [ ] You have a rollback plan

### 6. Document — leave a trail

- **Keep / Move**: add or strengthen the comment in this format:

  ```ts
  // Chesterton: <one-line reason this exists>
  // Removable when: <the condition that would let us delete it>
  ```

- **Remove**: in the commit body and PR description, write:

  > Removed because: `<concrete reason the original risk is gone>`. Verified in: `<environments>`. Rollback: `<plan>`.

---

## Build your repo's fence catalog

Generic rules only get you so far. The highest leverage is keeping a **catalog of the actual fences in your codebase** so this skill can short-circuit straight to "here be dragons" warnings.

Create a file at `.claude/skills/chestertons-fence/CATALOG.md` (or extend this SKILL.md) with entries shaped like:

````md
### `<file>:<line-or-symbol>` — <one-line label>

```<lang>
<the actual code or excerpt>
```

**Why it exists**: <one paragraph — what would happen if this were removed>

**Removable when**: <the future condition that would let you delete it, if any>

**Last verified**: <YYYY-MM-DD by whom>
````

Examples of what tends to live in real catalogs (stack-agnostic — pick what fits):

- Route matchers / URL patterns that include "future" prefixes to prevent dev fall-through
- Version fallbacks during a phased API rollout (v2 → v1 with session cache, contract negotiation, etc.)
- Module-scope singletons that prevent concurrent token refresh, connection storms, or duplicate jobs
- Magic numbers that are race-condition leeway windows, retry/backoff thresholds, or jitter ranges
- 301 redirects that preserve query params for external (push / ad / deep-link / SEO) compatibility
- Browser storage flags (`sessionStorage`/`localStorage`) that survive Strict Mode, Fast Refresh, or hot reload
- Empty `catch {}` / `rescue` / `except` blocks that intentionally swallow specific known errors
- Database migration ordering, default values, or `NOT NULL` deferrals tied to rollout sequencing
- Queue / cron retry counts, deadletter offsets, idempotency keys
- Feature-flag fallthrough branches that look dead but still cover legacy clients/users
- Hardcoded client IDs, region codes, or version pins that anchor backwards compatibility
- IaC defaults (timeouts, replicas, security groups) that exist to survive a specific outage replay

When you populate this catalog, **the skill becomes 10x more valuable**: Claude stops at the right places, with the right context, in *your* codebase — not a generic one.

---

## Anti-patterns (do not do these)

- "No comment, so it's redundant" — the absence of a comment is the highest-risk signal, not the lowest
- "Tests pass, so it's safe" — if the e2e/integration coverage is thin in this area, silent intent is more likely
- "TypeScript is green, so it's safe" — types do not capture runtime environments, race windows, or cross-platform branches
- "Works on my machine" — dev / staging / prod / web / app / locale variation is exactly where fences live
- "AI suggested a cleaner version, ship it" — cleanness is often the artifact of removing intent

---

## Pre-PR checklist

Before opening the PR, confirm:

- [ ] You ran step 1 (STOP) — no auto-edit
- [ ] You ran `git blame` on the changed lines
- [ ] You wrote down the failure hypothesis (step 4) somewhere — at minimum in the PR body
- [ ] You wrote a `// Chesterton: ...` comment OR a "Removed because" paragraph in the PR
- [ ] You considered every environment matrix (dev/prod, web/app, all locales, under race)

---

## Why this skill exists

The risk of AI-assisted refactors is rarely *"AI writes buggy code"* — modern AI usually writes clean code that compiles and passes tests. The risk is more subtle.

**AI doesn't see *why* the prior code was shaped the way it was.** It knows boilerplate and anti-patterns, but it's indifferent to which environment-specific trap a given line is preventing. So the cleaned-up code fails only in places nobody re-tests — a specific dev config, a specific locale, a narrow race window.

This skill is the smallest possible forcing function to fix that: a stop sign in front of every refactor, with a checklist Claude must walk through before touching the code.

---

## References

- G.K. Chesterton, *The Thing* (1929) — the original "Chesterton's Fence" passage

## License

MIT — see `LICENSE` in the plugin root.
