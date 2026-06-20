# Commit message style

> Gitmoji prefix + [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/). No Claude co-author trailer.

**When** — every commit in this repo. Likely the same convention applies in this user's other repos unless `git log` shows otherwise.

**Format**

```
<gitmoji> <type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

**Steps**

1. Pick a gitmoji that fits the change. Common picks:
   - 🌱 — scaffolding, seeds, initial setup
   - ✨ — new feature
   - 🐛 — bug fix
   - 📝 — documentation
   - ♻️ — refactor
   - 🔧 — config / tooling
   - 🙈 — `.gitignore` changes
   - 🎨 — formatting / style
   - 🗑️ — removal / deletion
2. Append a Conventional Commits type. Required types: `feat` (new feature, SemVer MINOR) and `fix` (bug fix, SemVer PATCH). Other allowed: `docs`, `chore`, `refactor`, `style`, `test`, `build`, `ci`, `perf`.
3. Optionally add a scope in **parentheses**: `feat(parser): ...`. Scope is a noun describing the section of the codebase. Use parentheses — not square brackets.
4. Add `!` before the colon for breaking changes (`feat!:` or `feat(api)!:`), or include a `BREAKING CHANGE: <detail>` footer.
5. Write a concise description after `:`, lowercase, no trailing period.
6. Add a body explaining **why** the change matters, not just what it does. Blank line between subject and body. Keep body lines under ~80 characters.
7. Footers use `Token: value` form (`Refs: #123`, `BREAKING CHANGE: ...`). One footer per line.
8. **Do not** add `Co-Authored-By: Claude` (or any other Claude trailer). The user does not want Claude listed as an author.

**Examples**

```
✨ feat: add eslint flat config seed
🐛 fix(prettier): respect lf line endings on windows checkouts
📝 docs: document commit style in wiki
♻️ refactor(seeds)!: collapse §A and §B into a single bootstrap section
🙈 chore: add .gitignore
```

**Gotchas**

- Use the actual emoji glyph, not the `:shortcode:` form — `git log --oneline` renders the glyph but not the shortcode.
- Parentheses for scope, not brackets. `feat(api):` not `[feat]:` and not `feat[api]:`.
- This is a switch from the earlier `[type]:` bracketed style used in commits `eeda7b8`, `8993fd5`, and `ace01ca`. Going forward, follow Conventional Commits. Don't rewrite history — those commits stay as-is.
- If a sibling repo's `git log` shows a different style, follow that repo's local style — this is the user's default, not a universal rule.

**References**

- [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)
- [Gitmoji](https://gitmoji.dev/)
