# Commit message style

> Gitmoji prefix + Angular-style `[type]: subject`. No Claude co-author trailer.

**When** — every commit in this repo. Likely the same convention applies in this user's other repos unless `git log` shows otherwise.

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
2. Append `[type]:` using a bracketed Angular type. Common: `feat`, `fix`, `docs`, `chore`, `refactor`, `style`, `test`, `build`, `ci`, `perf`.
3. Write a concise subject after `:`.
4. Add a body explaining **why** the change matters, not just what it does. Keep the body under ~80 characters per line.
5. **Do not** add `Co-Authored-By: Claude` (or any other Claude trailer) to the commit. The user does not want Claude listed as an author.

**Examples** (from this repo's `git log`):

```
🌱 [chore]: scaffold repo with seeds, code-quality configs, and docs
🙈 [chore]: add .gitignore
```

**Gotchas**

- Use the actual emoji glyph, not the `:shortcode:` form — `git log --oneline` renders the glyph but not the shortcode.
- Brackets, not parentheses. `[chore]:` not `chore(scope):`. The Angular *convention* is `type(scope):`; this project uses the bracketed variant.
- Convention was set on 2026-06-20 (after commit `eeda7b8`). If a sibling repo's `git log` shows a different style, follow that repo's local style — this is the user's default, not a universal rule.
