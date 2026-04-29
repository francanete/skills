# Reference file template

Reference files hold domain-specific knowledge that Claude loads only when the SKILL.md body links to them. Keep them terse — no 101 explanations of concepts Claude already knows.

```markdown
# <Domain Name>

<If file >100 lines, add an explicit Contents list here so partial reads still see structure.>

## <Subdomain or rule group>

- <Rule 1 — short, imperative>
- <Rule 2>
- <Rule 3 — explain *why* if non-obvious>

## <Subdomain 2>

- ...
```

## Rules for reference files

- **No frontmatter.** Only `SKILL.md` has frontmatter.
- **One topic per file.** If you're tempted to add a second `## Domain B` section, it's probably a separate file.
- **One level deep from `SKILL.md`.** Reference files should not link to other reference files — Claude does partial reads on nested links.
- **Keep terse.** Bullet rules beat prose paragraphs.
- **Use forward slashes** in any paths, even on Windows.
