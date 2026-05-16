# Usage Guide

## Initial Setup

Run once at project start:

```text
/speckit.wp-coding-standards.init
```

The command interviews you about your project and writes `.specify/memory/wp-standards-constitution.md`.

Rerun at any time to change profile or update settings.

---

## Running a Review

```text
/speckit.wp-coding-standards.review
```

Reviews the current changed files (or full project if no changes are staged) against the active profile.

**Options:**

```text
/speckit.wp-coding-standards.review --all
```
Reviews the full project, not just changed files.

```text
/speckit.wp-coding-standards.review path/to/file.php
```
Reviews a specific file.

---

## Generating Fix Tasks

After a review identifies violations:

```text
/speckit.wp-coding-standards.apply
```

Injects a `## WP Standards Fix Tasks` section into `specs/<feature>/tasks.md` with prioritized fix tasks (P0–P3).

---

## Lifecycle Hook Usage

The extension automatically offers to scan at key points in the Spec Kit workflow:

| Hook | Trigger | What It Does |
|---|---|---|
| `after_plan` | After `/plan` completes | Quick `violation-detection` scan of the plan document |
| `after_tasks` | After `/tasks` completes | Quick `violation-detection` scan of the task list |
| `after_implement` | After `/implement` completes | Full `review` of changed PHP files |

All hooks are optional — you can decline the prompt.

---

## Switching Profiles

Rerun init and select a different profile:

```text
/speckit.wp-coding-standards.init
```

Choose "Update specific settings" and select a new profile. The constitution is updated in place.

---

## Common Workflows

### New plugin targeting WordPress.org

```text
/speckit.wp-coding-standards.init
→ Plugin | WordPress.org | wordpress profile | prefix: myplugin_

/speckit.wp-coding-standards.review
→ Fix P0 violations

/speckit.wp-coding-standards.apply
→ Inject fix tasks into tasks.md
```

### Existing plugin — brownfield adoption

```text
/speckit.wp-coding-standards.init
→ Plugin | Internal | wordpress profile | min_severity: critical

/speckit.wp-coding-standards.review --all
→ Identify all CRITICAL violations first

/speckit.wp-coding-standards.apply
→ Generates P0/P1 fix tasks; tackle incrementally
```

### VIP Go project

```text
/speckit.wp-coding-standards.init
→ Plugin | WordPress VIP Go | (auto: wordpress-vip profile)

/speckit.wp-coding-standards.review
→ Reviews against full VIP rule set
```

---

## Documenting Intentional Deviations

If your project intentionally violates a rule (legacy code, documented exception), add it to the constitution:

```markdown
## Documented Deviations

- The `legacy/` directory is excluded from PrefixAllGlobals enforcement
  pending planned refactor. Tracked in GitHub issue #42.
- wp_redirect() without exit used in OAuth callback (oauth-handler.php:88).
  The exit is handled by the OAuth library's own redirect flow. See /docs/oauth.md.
```

Documented deviations are not flagged as violations.
