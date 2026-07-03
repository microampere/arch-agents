# arch-agents Plugin

## Version bumps

The plugin version string is duplicated in **three** files. Whenever the version changes, update all three in the same commit:

1. `.claude-plugin/plugin.json` — `"version"` field (source of truth for the Claude Code plugin system).
2. `README.md` — `## Version` section near the bottom of the file.
3. `showcase.html` — appears twice: the hero `eyebrow` div (`v2.0.1 · Claude Code Plugin`) and the footer `<p>` (`arch-agents · v2.0.1 · Claude Code Plugin`).

Before bumping, grep the repo for the current version string to catch any new location that isn't listed above:

```
grep -rn "2\.0\.1" .
```

Follow semver: MINOR (2.x.0) for new/changed behavior in the skills (new heuristics, new rules, new domains), PATCH (2.0.x) for pure fixes/typos/refactors with no behavior change, MAJOR for breaking changes to skill contracts or file formats.
