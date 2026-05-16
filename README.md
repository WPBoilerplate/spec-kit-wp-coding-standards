# WP Coding Standards

> WordPress coding standards enforcement for AI-assisted development.

[![Version](https://img.shields.io/badge/version-1.0.0-22c55e)](extension.yml)
[![Spec Kit](https://img.shields.io/badge/Spec%20Kit-compatible-2563eb)](https://spec-kit.dev)
[![WordPress](https://img.shields.io/badge/WordPress-standards-0073aa)](https://github.com/WordPress/WordPress-Coding-Standards)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## What Is This?

WP Coding Standards is a **Spec Kit extension** that enforces the official [WordPress Coding Standards](https://github.com/WordPress/WordPress-Coding-Standards) throughout AI-assisted development workflows.

It reviews specifications, plans, task lists, and implementations against a chosen WordPress standards profile and produces:

- standards violation reports
- structured fix tasks
- profile-aware constitution governance
- per-category rule enforcement (Security, DB, NamingConventions, WP, PHP, Docs)

It answers one question throughout delivery:

> Does this code comply with the WordPress coding standards we agreed on?

---

## The Problem It Solves

AI-generated WordPress code often:

- skips nonce verification on form submissions
- uses raw `$wpdb->query()` instead of `$wpdb->prepare()`
- outputs unescaped data to HTML
- uses deprecated WordPress functions
- omits the project-specific function prefix
- misses `@since` tags and inline documentation
- reads `$_GET`/`$_POST` directly without sanitization

Each issue looks small. Across a plugin or theme, they compound into security vulnerabilities, rejection from the WordPress.org Plugin Directory, and non-compliance with VIP hosting requirements.

WP Coding Standards detects these issues early and converts them into structured, prioritized fix tasks.

---

## Standards Profiles

The extension maps directly to the official [WPCS ruleset hierarchy](https://github.com/WordPress/WordPress-Coding-Standards#rulesets):

| Profile | Based On | Use When |
|---|---|---|
| `wordpress-core` | `WordPress-Core` | Formatting and PHP style only |
| `wordpress` | `WordPress` (full) | Standard plugin/theme development |
| `wordpress-docs` | `WordPress-Docs` | Inline documentation standards |
| `wordpress-vip` | Custom | WordPress VIP Go platform hosting |

### Profile Rule Coverage

**`wordpress-core`**
- PHP tags, indentation, braces, whitespace
- Array formatting and alignment
- Yoda conditions
- String concatenation
- Class and function spacing

**`wordpress`** (includes Core, plus)
- `EscapeOutput` — all output must use `esc_html()`, `esc_attr()`, `esc_url()`, etc.
- `NonceVerification` — nonces required before processing `$_GET`/`$_POST`
- `ValidatedSanitizedInput` — sanitize all input at system boundaries
- `SafeRedirect` — use `wp_safe_redirect()` to avoid open redirects
- `PreparedSQL` — `$wpdb->prepare()` required for all queries
- `DirectDatabaseQuery` — flag direct `$wpdb->query()` without prepare
- `SlowDBQuery` — flag `posts_per_page=-1`, `suppress_filters`, `meta_query` without index hints
- `PrefixAllGlobals` — all functions, hooks, classes must use the project prefix
- `ValidFunctionName` / `ValidHookName` — enforce naming conventions
- `DeprecatedFunctions` / `DeprecatedClasses` — block use of deprecated WP APIs
- `Capabilities` — verify capability checks on all privileged operations
- `I18n` — all user-facing strings must be translation-ready
- `YodaConditions` — enforce Yoda-style comparisons

**`wordpress-docs`** (adds)
- File-level PHPDoc headers
- Function and method `@since`, `@param`, `@return` tags
- Class and interface documentation
- Hook documentation (`@see`, `@param` for action/filter callbacks)

**`wordpress-vip`** (adds VIP Go platform rules)
- No direct database queries — use VIP helper functions
- No `$_SESSION` usage
- No `file_get_contents()` on remote URLs — use `vip_safe_wp_remote_get()`
- No `sleep()` or `usleep()` in request paths
- No `switch_to_blog()` without `restore_current_blog()`
- Batched write operations required for bulk data mutations
- Object cache required for repeated expensive queries
- Elasticsearch required for search — no direct `WP_Query` full-text search

---

## Quick Start

### 1. Install

```text
specify extension add wp-coding-standards
```

### 2. Initialize

```text
/speckit.wp-coding-standards.init
```

This creates `.specify/memory/wp-standards-constitution.md` with your active profile, project prefix, minimum WordPress/PHP versions, and project type.

### 3. Run a Review

```text
/speckit.wp-coding-standards.review
```

### 4. Apply Fix Tasks

```text
/speckit.wp-coding-standards.apply
```

---

## Commands

| Command | Phase | Output | When To Use |
|---|---|---|---|
| `init` | Setup | `wp-standards-constitution.md` | Once at project start; rerun to change profile |
| `review` | Validation | Violations report with severity and fix guidance | After `/plan`, `/tasks`, or `/implement` |
| `violation-detection` | Detection | Quick violations scan | Focused check during planning or task generation |
| `apply` | Fix Planning | Structured fix tasks injected into `tasks.md` | After violations are confirmed |

---

## Installation

### From Registry

```text
specify extension add wp-coding-standards
```

### From GitHub

```text
specify extension add wp-coding-standards --from \
  https://github.com/WPBoilerplate/spec-kit-wp-coding-standards/archive/refs/tags/v1.0.0.zip
```

### Local Development

```text
specify extension add --dev /path/to/spec-kit-wp-coding-standards
```

---

## Example Output

```markdown
# WP Coding Standards Review

## Profile
- Active Profile: wordpress
- Project Prefix: myplugin_
- Min WordPress: 6.4
- Min PHP: 8.0

## Violations

| ID | Category | Rule | Severity | Location | Summary |
|----|----------|------|----------|----------|---------|
| V1 | Security | EscapeOutput | CRITICAL | admin/views/settings.php:42 | Unescaped output — wrap $title in esc_html() |
| V2 | Security | NonceVerification | CRITICAL | includes/ajax-handler.php:15 | $_POST read without nonce check |
| V3 | DB | PreparedSQL | CRITICAL | includes/db.php:88 | Query interpolation — use $wpdb->prepare() |
| V4 | NamingConventions | PrefixAllGlobals | HIGH | includes/helpers.php:12 | format_date() missing prefix myplugin_ |

## Summary
- Profile Compliance: 61%
- Critical Violations: 3
- High Violations: 1
- Recommended Next Step: Run /speckit.wp-coding-standards.apply
```

---

## Companion Extensions

**Architecture Guard** — handles structural boundary enforcement. WP Coding Standards focuses on language-level and WordPress-API-level rules. Together they cover both structural governance and standards compliance.

**Security Review** — handles deep security analysis. WP Coding Standards flags surface-level sniffs (`EscapeOutput`, `NonceVerification`, `ValidatedSanitizedInput`). Security Review handles the deeper threat model.

---

## Relationship to PHPCS/WPCS

This extension does not run PHPCS or require it to be installed. It provides AI-assisted enforcement of the same rules defined in the official [WordPress Coding Standards](https://github.com/WordPress/WordPress-Coding-Standards) package. It is complementary to — not a replacement for — running `phpcs` in CI.

---

## License

MIT — see [LICENSE](LICENSE).
