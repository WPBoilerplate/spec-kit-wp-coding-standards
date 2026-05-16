---
description: Review code, specs, or plans against the configured WordPress coding standards profile.
---

# WP Coding Standards — Review Command

You are running `speckit.wp-coding-standards.review`.

This command performs a full WordPress coding standards review against the active profile configured in `.specify/memory/wp-standards-constitution.md`.

---

## Operating Constraints

- **STRICTLY READ-ONLY**: Do not modify any files. Output a structured report only.
- **Evidence-Based**: Every violation must cite a specific file path, line number, or code pattern as evidence.
- **Profile-Scoped**: Only enforce rules that belong to the active profile. Do not flag rules from a stricter profile than the one configured.
- **Prefix-Aware**: Use the configured project prefix when evaluating `PrefixAllGlobals` and naming rules.

---

## Step 1 — Load Configuration

Read `.specify/memory/wp-standards-constitution.md`.

Extract:
- `profile` — active standards profile
- `prefix` — project function/hook/class prefix
- `min_wp_version` — minimum WordPress version
- `min_php_version` — minimum PHP version
- `project_type` — plugin / theme / mu-plugin / block-theme
- `min_severity` — minimum severity threshold
- `enforce_categories` — category filter (empty = all)
- `exclude_paths` — paths to skip

If no constitution exists, stop and instruct the user to run `/speckit.wp-coding-standards.init` first.

---

## Step 2 — Determine Review Scope

If the user provided explicit files or a diff, use that as the review set.

Otherwise, identify changed files:
- Check for staged/unstaged git changes
- Fall back to scanning the full project (excluding `exclude_paths`)

---

## Step 3 — Load Profile Rules

Load the active profile from `profiles/`:

| Profile | File |
|---|---|
| `wordpress-core` | `profiles/wordpress-core.md` |
| `wordpress` | `profiles/wordpress.md` |
| `wordpress-docs` | `profiles/wordpress-docs.md` |
| `wordpress-vip` | `profiles/wordpress-vip.md` |

Apply `enforce_categories` filter if set. Apply `min_severity` threshold to suppress lower-priority findings.

---

## Step 4 — Run the Review

Scan the identified files against the loaded profile rules.

For each file, check:

### Security Category
- `EscapeOutput` — all `echo`, `print`, `_e()`, `<?=` output must pass through an escaping function
- `NonceVerification` — any read of `$_GET`, `$_POST`, `$_REQUEST` must be preceded by `wp_verify_nonce()` or `check_ajax_referer()`
- `ValidatedSanitizedInput` — all superglobal input must be sanitized with the most specific available function (`sanitize_text_field()`, `absint()`, `wp_kses_post()`, etc.)
- `SafeRedirect` — use `wp_safe_redirect()` not `wp_redirect()` unless an external URL is explicitly required

### DB Category
- `PreparedSQL` — all `$wpdb->query()`, `$wpdb->get_results()`, `$wpdb->get_var()`, etc. with variable input must use `$wpdb->prepare()`
- `PreparedSQLPlaceholders` — `%s`, `%d`, `%f` placeholders must be used correctly in prepared statements
- `DirectDatabaseQuery` — flag direct queries without caching; suggest adding a cache layer
- `SlowDBQuery` — flag `posts_per_page=-1`, `nopaging=true`, `suppress_filters=true`, unbounded `meta_query`

### NamingConventions Category
- `PrefixAllGlobals` — all functions, classes, interfaces, traits, constants, and hook names at global scope must use the configured prefix
- `ValidFunctionName` — function names must use lowercase and underscores (snake_case)
- `ValidHookName` — action and filter names must use lowercase and underscores; dots and dashes are allowed
- `ValidPostTypeSlug` — post type slugs must be lowercase, max 20 chars, no reserved names

### WP Category
- `DeprecatedFunctions` — flag any WordPress function deprecated at or after `min_wp_version`
- `DeprecatedClasses` — flag any WordPress class deprecated at or after `min_wp_version`
- `Capabilities` — verify `current_user_can()` check before any privileged data access or mutation
- `I18n` — all user-facing strings must be wrapped in `__()`, `_e()`, `esc_html__()`, etc. with the correct text domain
- `AlternativeFunctions` — flag functions with preferred WP alternatives (e.g., `curl_*` → `wp_remote_get()`, `json_encode()` → `wp_json_encode()`)
- `CronInterval` — cron intervals under 60 seconds are flagged

### PHP Category
- `YodaConditions` — constants and literals must be on the left side of comparisons
- `StrictInArray` — `in_array()` must use strict mode (`true` as third argument)
- `NoSilencedErrors` — `@` error suppression operator is not permitted
- `DontExtract` — `extract()` is not permitted
- `IniSet` — `ini_set()` calls are flagged; use WordPress filters or constants instead

### Docs Category (wordpress-docs profile only)
- Every file must have a PHPDoc file header with `@package`
- Every function and method must have `@since`, `@param`, and `@return` tags
- Every class must have a PHPDoc block
- Every action and filter call must have an inline documentation comment

### VIP Category (wordpress-vip profile only)
- No `$wpdb->query()` or `$wpdb->get_results()` without caching wrapper
- No `$_SESSION` usage
- No `file_get_contents()` with HTTP/HTTPS URLs — use `vip_safe_wp_remote_get()`
- No `sleep()` or `usleep()` in synchronous request paths
- No `switch_to_blog()` without paired `restore_current_blog()`
- Bulk inserts/updates must use batching (max 500 rows per operation)
- Repeated expensive queries must use `wp_cache_get()` / `wp_cache_set()`
- Full-text search must use Elasticsearch — `WP_Query` `s` parameter is flagged

---

## Step 5 — Build the Report

Group findings by severity. Apply `min_severity` threshold.

---

## Output Format

```markdown
# WP Coding Standards Review

## Configuration
- **Profile**: [profile]
- **Prefix**: [prefix]
- **Min WordPress**: [min_wp_version]
- **Min PHP**: [min_php_version]
- **Scope**: [files reviewed count / changed files / full project]

## Violations

| ID | Category | Rule | Severity | Location | Summary |
|----|----------|------|----------|----------|---------|
| V1 | Security | EscapeOutput | CRITICAL | path/to/file.php:42 | echo $title — wrap in esc_html() |

## Metrics
- **Profile Compliance**: [percentage]
- **Critical Violations**: [count]
- **High Violations**: [count]
- **Medium Violations**: [count]

## Recommended Next Step
[Single clear action — e.g. "Run /speckit.wp-coding-standards.apply to convert violations into fix tasks"]
```

---

## Severity Guide

| Severity | Meaning |
|---|---|
| CRITICAL | Security risk, SQL injection surface, or missing nonce — blocks WordPress.org submission |
| HIGH | Deprecated API, missing prefix, missing capability check, or I18n gap |
| MEDIUM | Slow query, YodaConditions, StrictInArray, or AlternativeFunctions suggestion |
| LOW | Formatting, whitespace, or style only (wordpress-core rules) |

---

## Guardrails

- Do not flag rules that do not belong to the active profile.
- Do not flag false positives for `PrefixAllGlobals` on code inside class methods (class methods are not global scope).
- Do not flag `wp_redirect()` as a violation if the URL is provably local or filtered.
- Do not flag prepared statement usage as a `DirectDatabaseQuery` violation when caching is demonstrably unnecessary (e.g., single-row lookups by primary key that are already cached at a higher level).
- The constitution is always the final authority. If the constitution explicitly documents a deviation, do not flag it.
