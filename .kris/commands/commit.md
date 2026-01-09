# Intelligent Batch Commit

Analyze changes and create organized, semantic commits with automatic grouping, staging, and versioning.

## Default Behavior (Fully Automatic)

1. **Analyze all unstaged and staged changes** in the repository
2. **Group related changes** by their logical category/topic (e.g., feature, bugfix, refactor, docs, style, config, etc.)
3. **For EACH group, create a SEPARATE commit**:
   - **Stage only the files** belonging to the current group
   - **Determine version increment** for this specific commit (vW.X.Y.Z format):
     - Parse the previous commit message to get current version
     - W = Major version (breaking changes, architectural changes, major features)
     - X = Minor version (new features, significant enhancements, API changes)
     - Y = Patch version (bugfixes, small improvements, minor refactors)
     - Z = Iteration (formatting, documentation, config tweaks, minor style changes)
     - Auto-increment appropriate level based on this commit's scope
     - Auto-reset lower levels when higher ones increment (e.g., v0.0.13.0 → v0.0.14.0 resets Z to 0)
   - **Detect GitHub issue references** in:
     - Branch names (e.g., `feature/123-new-feature`)
     - File contents or comments containing `#123`, `issue #123`, etc.
     - Include appropriate closure keywords if applicable: "Resolves #123", "Fixes #123", "Closes #123", "Completes #123"
   - **Create commit message** in this format:
     ```
     <category>: <concise title describing the change>
     
     <GitHub issue reference if applicable>
     vW.X.Y.Z
     <additional context if needed>
     ```
   - **Execute the commit** with `git commit -m "title" -m "description"`
4. **Move to next group** and repeat step 3 until all groups are committed
5. **Result**: Multiple sequential commits, each focused on one logical category

## CRITICAL: One Commit Per Category Group

- DO NOT stage and commit all files at once
- Each category group gets its own separate commit
- Version numbers increment sequentially across commits
- Example sequence:
  ```
  Commit 1: config: Enforce 2-space indentation (v0.0.13.0)
  Commit 2: style: Reformat all frontend files (v0.0.13.1)
  Commit 3: refactor: Enhance batch operations service (v0.0.13.2)
  ```

## Available Flags for Manual Control

Use these flags to enable manual prompts at specific steps:

- `--manual-categories` or `-c`: Prompt for approval/modification of detected categories before grouping
- `--manual-files` or `-f`: Prompt for approval/modification of which files are included in each group
- `--manual-version` or `-v`: Prompt for manual version number input instead of auto-increment
- `--manual-issues` or `-i`: Prompt for manual GitHub issue reference input
- `--manual-proceed` or `-p`: Prompt for approval before proceeding to each next commit (commit-by-commit review)
- `--manual-all` or `-a`: Enable all manual prompts (full interactive mode)

## Examples

**Fully automatic:**
```
/commit
```

**Review categories and files before committing:**
```
/commit --manual-categories --manual-files
```

**Review everything step-by-step:**
```
/commit --manual-all
```

**Only control version numbers manually:**
```
/commit -v
```

## Commit Message Format Details

**First `-m` flag (title):**
- Format: `<category>: <concise description>`
- Category examples: `feat`, `fix`, `refactor`, `docs`, `perf`, `test`, `chore`, `style`
- Keep under 72 characters
- Use imperative mood ("add" not "added")

**Second `-m` flag (description):**
- Line 1: GitHub issue reference (if applicable)
- Line 2: Version number (vW.X.Y.Z)
- Line 3+: Additional context if needed (optional)

## Notes

- **Each category gets its own commit** - never combine multiple categories into one commit
- Commits are created sequentially, one after another
- Each commit should be self-contained and logically coherent
- Version increments with each commit based on the scope of that specific change
- Prioritize clarity - the title should make the change immediately understandable
- If no previous version found, start at v1.0.0.0
- For formatting/style changes (like reformatting files), increment Z (iteration)
- For config changes (like .editorconfig, .prettierrc), increment Z (iteration)
- For refactors or enhancements without breaking changes, increment Y (patch)
- For new features or API additions, increment X (minor)
- For breaking changes or major architectural shifts, increment W (major)
- If unsure about category boundaries, prefer smaller, more focused commits over large ones