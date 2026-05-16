---
description: Detect WordPress coding standards violations in plans, tasks, or implementation. Lightweight scan for use in lifecycle hooks.
---

# WP Coding Standards — Violation Detection Command

You are running `speckit.wp-coding-standards.violation-detection`.

This command performs a **lightweight, focused scan** for WordPress coding standards violations. It is designed for use in plan and task lifecycle hooks where a full review would be excessive.

---

## Operating Constraints

- **STRICTLY READ-ONLY**: Do not modify any files.
- **Fast Path**: Prioritize CRITICAL and HIGH severity findings only. Skip LOW and MEDIUM unless `--all` flag is passed.
- **Scoped**: When invoked from a hook, scan only the artifact passed in context (plan, tasks, or diff). Do not scan the full project unless explicitly requested.

---

## Step 1 — Load Configuration

Read `.specify/memory/wp-standards-constitution.md` for the active profile and prefix.

If no constitution exists, output a single warning and stop:

```text
WP Coding Standards not initialized. Run /speckit.wp-coding-standards.init to configure.
```

---

## Step 2 — Determine Scan Target

| Invocation Context | Scan Target |
|---|---|
| `after_plan` hook | `specs/<feature>/plan.md` |
| `after_tasks` hook | `specs/<feature>/tasks.md` |
| `after_implement` hook | staged/changed PHP files |
| Manual invocation | files provided by user, or changed files |

When scanning plan/tasks documents, look for:
- References to raw `$_GET`/`$_POST` without mention of nonce verification
- References to `$wpdb->query()` or direct SQL without mention of `$wpdb->prepare()`
- References to `echo` or direct output without mention of escaping functions
- References to new functions or hooks without the project prefix
- References to deprecated WordPress functions

---

## Step 3 — Run Fast Scan

Apply only CRITICAL and HIGH rules from the active profile:

**Always checked (all profiles):**
- `EscapeOutput` — unescaped output
- `NonceVerification` — superglobal access without nonce
- `ValidatedSanitizedInput` — unsanitized input
- `PreparedSQL` — unparameterized queries
- `PrefixAllGlobals` — missing project prefix on new globals
- `DeprecatedFunctions` — use of deprecated WP functions
- `Capabilities` — missing capability checks

**VIP profile additionally checks:**
- `$_SESSION` usage
- `file_get_contents()` on remote URLs
- `sleep()` / `usleep()` calls
- Unbatched bulk write operations

---

## Output Format

If violations found:

```markdown
## WP Standards: Violations Detected

| ID | Rule | Severity | Location | Issue |
|----|------|----------|----------|-------|
| V1 | EscapeOutput | CRITICAL | admin/page.php:55 | echo $user_input — needs esc_html() |
| V2 | NonceVerification | CRITICAL | includes/handler.php:20 | $_POST access without nonce check |

Run /speckit.wp-coding-standards.apply to generate fix tasks.
```

If no violations found:

```markdown
## WP Standards: No Critical Violations Detected

Active profile: [profile] | Prefix: [prefix]
```

---

## Guardrails

- Do not produce a full review report — this command outputs a compact findings table only.
- Do not block implementation. Violations are advisory unless the constitution marks a rule as P0.
- Do not scan `vendor/`, `node_modules/`, or `tests/` paths.
