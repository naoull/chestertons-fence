# chestertons-fence

> A Claude Code plugin that stops AI from removing the fence before it knows why the fence is there.

A stack-agnostic skill that forces context-gathering before any deletion or refactor. Based on G.K. Chesterton's 1929 parable.
**License: MIT** · **Type: Claude Code plugin (skill)**

---

## Install

```sh
/plugin marketplace add naoull/chestertons-fence
/plugin install chestertons-fence@naoull/chestertons-fence
```

That's it. The skill auto-triggers when you ask Claude to *clean up*, *refactor*, *simplify*, or *remove* code — especially in fence-dense areas (middleware, auth, fetch wrappers, error handlers, migrations, IaC).

---

## What it does

Based on Chesterton's parable: *"Don't remove a fence until you know why it was put up."*

When you ask Claude to refactor or simplify code, this plugin makes it **stop and investigate the history first** — adjacent comments, `git blame`, `git log -L`, related code — so it doesn't quietly remove a load-bearing line whose intent isn't obvious.

It runs a 6-step workflow:

```
STOP → Identify → Investigate (blame, log, related) → Hypothesize → Decide → Document
```

It is **stack-agnostic** — works on frontend, backend, infrastructure, scripts, schemas, migrations, CI workflows, and IaC. Anywhere code has history, fences exist.

---

## What you'll see in action

```
You:    "Clean up this matcher — v2 looks redundant alongside v1."

Claude: This looks like a fence. Let me investigate before touching it.

        → git blame middleware.ts:78
          last touched 6 months ago — "fix: prevent v2 fall-through trap"
        → Adjacent comment found:
          "v2 누락 시 dev 환경에서 /v2/* fetch 가 미들웨어에 잡혀..."
        → Hypothesis: if v2 is removed, dev fetches to /v2/* will redirect
          to /[locale]/* and return 200 + HTML, breaking JSON.parse silently.

        Decision: KEEP — fence is load-bearing.
        I recommend strengthening the comment instead. Proceed?
```

That single 30-second pause prevents a class of incidents that pass type-check, pass tests, and only fail in one environment.

---

## Why it exists

The risk of AI-assisted refactors isn't *"AI writes buggy code"* — modern AI usually writes clean code that compiles and passes tests. The risk is more subtle:

**AI doesn't see *why* the prior code was shaped the way it was.** It knows boilerplate and anti-patterns, but it's indifferent to which environment-specific trap a given line is preventing. So the cleaned-up code fails only in places nobody re-tests — a specific dev config, a specific locale, a narrow race window.

This plugin is the smallest possible forcing function to fix that: a skill that puts a stop sign in front of every refactor, with a checklist Claude must walk through before touching the code.

---

## Recommended next step — build your repo's catalog

The plugin ships with a generic SKILL.md. The **highest-leverage** thing you can do after install is to populate a `CATALOG.md` next to it with the actual fences in *your* codebase. See the SKILL.md *"Build your repo's fence catalog"* section for the format.

Examples of what tends to live in real catalogs (any stack):

- Route matchers / URL patterns with "future" prefixes to prevent dev fall-through
- Version fallbacks during a phased API rollout (v2 → v1 caches, contract negotiation)
- Module-scope singletons that prevent concurrent token refresh, connection storms, or duplicate jobs
- Magic numbers that are race-condition leeway windows or retry/backoff thresholds
- 301 redirects that preserve query params for external (push / ad / deep-link / SEO) compatibility
- Browser storage flags that survive Strict Mode / Fast Refresh / hot reload
- Migration ordering, default values, or `NOT NULL` deferrals tied to rollout sequencing
- Feature-flag fallthrough branches that still cover legacy clients
- IaC defaults (timeouts, replicas) that exist because of a past outage

Once Claude can see the catalog, it stops at the right places with the right context — in your codebase, not a generic one.

---

## Contributing

Issues, PRs, and catalog templates for specific stacks/frameworks are very welcome. If you ship a fence catalog you're proud of, open a PR — they'll live under `examples/`.

## Author

[kibeom na](https://github.com/naoull)

## License

MIT — see [LICENSE](./LICENSE).
