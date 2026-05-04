# Contributing

Thanks for your interest in improving `chestertons-fence`.

## Quick ways to help

1. **Open an issue** with a real fence you wish this skill had caught — the more specific the better. We use these to grow the example list in the SKILL.md.
2. **Submit a catalog** for a specific stack/framework (e.g., `examples/django.md`, `examples/rails.md`, `examples/terraform.md`). Catalogs are the highest-leverage contribution because they make the skill actually-useful in a real codebase, not just a generic one.
3. **Improve the workflow** — if you've hit a case the 6-step workflow misses, open a PR with a tighter version.

## PR guidelines

- Keep changes focused. One PR = one improvement.
- Update `CHANGELOG.md` under an `[Unreleased]` heading.
- For SKILL.md edits, preserve the YAML frontmatter shape; the `description` is what Claude uses to decide when to trigger.
- Be opinionated in writing. The skill's value comes from being prescriptive, not flexible.

## Local testing

After cloning, install your local copy in Claude Code:

```sh
/plugin marketplace add /absolute/path/to/your/clone
/plugin install chestertons-fence@<marketplace-name>
```

Then trigger with phrases like *"clean this up"*, *"refactor this"*, *"this looks redundant"* in any project, and verify the 6-step workflow runs.

## License

By contributing, you agree your contributions are licensed under MIT (see `LICENSE`).
