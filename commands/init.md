---
description: Initialize the WordPress coding standards profile for this project.
---

# WP Coding Standards — Init Command

You are running `speckit.wp-coding-standards.init`.

This command configures the WordPress coding standards profile for the current project and writes `.specify/memory/wp-standards-constitution.md`.

---

## Purpose

Help the developer intentionally configure:

- which WordPress coding standards profile to enforce
- the project-specific global prefix
- minimum WordPress and PHP version targets
- project type and distribution channel
- which rule categories to prioritize

This configuration becomes the authoritative source for all subsequent `review`, `violation-detection`, and `apply` commands.

---

## Step 0 — Auto-Detect from AGENTS.md

Before starting the interview, check for an `AGENTS.md` file in the project root.

If found, extract the following fields and use them as pre-filled defaults:

| AGENTS.md Field | Maps To | Notes |
|---|---|---|
| `naming_prefix` | `prefix` | Use as-is |
| `wordpress_min_version` | `min_wp_version` | Use as-is |
| `php_min_version` | `min_php_version` | Use as-is |
| `coding_standard` | `profile` | See mapping below |
| `enforce_nonces` + `enforce_capabilities` + `sanitize_input` + `escape_output` + `sql_prepared_statements` | Security category enforcement | All `true` → enforce Security + DB categories |

### coding_standard → profile mapping

| AGENTS.md value | Profile |
|---|---|
| `wpcs-strict` | `wordpress` |
| `wpcs-core` | `wordpress-core` |
| `wpcs-docs` | `wordpress-docs` |
| `wpcs-vip` or `vip` | `wordpress-vip` |
| `wordpress` | `wordpress` |
| *(any unrecognised value)* | `wordpress` (default) |

If `AGENTS.md` is found and values are extracted, show a summary and ask:

```text
I found AGENTS.md with the following settings:

  Profile:     wordpress  (from coding_standard: wpcs-strict)
  Prefix:      acrossai_  (from naming_prefix)
  Min WP:      6.9        (from wordpress_min_version)
  Min PHP:     7.4        (from php_min_version)

Would you like to:
  1. Use these settings as-is
  2. Review and adjust individual settings
  3. Ignore AGENTS.md and start fresh
```

If the user selects **1**, skip all interview phases and write the constitution immediately.
If the user selects **2**, pre-fill all interview questions with the extracted values so they only need to confirm or change specific settings.
If the user selects **3**, proceed with the full interview from scratch.

If `AGENTS.md` is not found, proceed with the full interview below.

---

## Core Principles

### 1. Sequential Interviewing

DO NOT ask all questions at once.

Interview the user in logical phases. Only move to the next phase when the current phase is sufficiently understood.

### 2. Suggestions, Not Forced Opinions

Suggest sensible defaults based on project type. NEVER force a profile or prefix. A setting only becomes active when the user explicitly accepts it or confirms the default.

### 3. Detect Existing Configuration

Before starting the interview, check for:

- `.specify/memory/wp-standards-constitution.md`
- `.specify/config/wp-coding-standards.yml`

If either exists, summarize the current configuration and ask:

```text
Would you like to:
- Keep the current profile and update specific settings
- Switch to a different profile
- Regenerate the configuration from scratch
```

NEVER overwrite existing configuration automatically.

### 4. Graceful Pause and Resume

If the user needs to pause mid-interview:

1. Save current answers to `.specify/memory/wp-standards-draft.md` with a timestamp
2. Ask: "Do you want to resume these answers later?"
3. On next `init` run, detect the draft and offer to resume

---

## Interview Flow

---

### Phase 1 — Project Identity

**Question: Project Type**

```text
What type of WordPress project is this?

- Plugin
- Theme
- Block Theme
- Must-Use Plugin (mu-plugin)
```

---

**Question: Distribution Channel**

```text
Where will this project be distributed?

- WordPress.org Plugin/Theme Directory
- WordPress VIP Go (managed hosting)
- Private / Internal use
- WooCommerce.com Marketplace
```

If the user selects **WordPress VIP Go**, automatically set profile to `wordpress-vip` and skip the profile selection question.

If the user selects **WordPress.org**, note that the `wordpress` full profile is required and flag any `wordpress-core`-only selection as insufficient for directory submission.

---

### Phase 2 — Standards Profile

Skip this phase if VIP was selected in Phase 1.

```text
Which WordPress coding standards profile should be enforced?

- wordpress       — Full standard (Core + security, DB, naming, WP, PHP rules). Recommended for most projects.
- wordpress-core  — Formatting and PHP style only. No security or naming rules.
- wordpress-docs  — Adds inline documentation requirements (@since, @param, @return) on top of the full standard.
```

Recommend `wordpress` for plugins and themes targeting WordPress.org.
Recommend `wordpress-docs` for projects that require full PHPDoc coverage.

---

### Phase 3 — Project Prefix

```text
What is the function/hook/class prefix for this project?

This prefix must be applied to all globally-scoped functions, hooks, classes,
and constants to avoid conflicts with other plugins and themes.

Example:
- Plugin slug: my-awesome-plugin → prefix: my_awesome_plugin_
- Theme: my-theme → prefix: my_theme_
```

Validate that the prefix:
- ends with an underscore
- contains only lowercase letters, numbers, and underscores
- is at least 3 characters long (excluding the trailing underscore)

If the user provides a slug without underscores, suggest the converted form.

---

### Phase 4 — Version Targets

**Question: Minimum WordPress Version**

```text
What is the minimum WordPress version this project supports?

Current WordPress latest: 6.8
Recommended minimum: 6.4

(Used to flag deprecated functions and APIs introduced after your minimum version)
```

**Question: Minimum PHP Version**

```text
What is the minimum PHP version this project supports?

WordPress 6.4+ requires PHP 7.4 minimum.
Recommended: 8.0 or higher for new projects.
```

---

### Phase 5 — Enforcement Preferences

**Question: Severity Threshold**

```text
What is the minimum violation severity to report?

- all      — Report everything including formatting style
- high     — Report security, DB, naming, and WP API violations (recommended)
- critical — Report only blocking violations (security, DB prepare, nonces)
```

Recommend `high` for most projects. Recommend `critical` for brownfield codebases with many existing style issues.

**Question: Categories**

```text
Are there any rule categories you want to exclude from enforcement?

Categories:
- security   (EscapeOutput, NonceVerification, ValidatedSanitizedInput, SafeRedirect)
- db         (PreparedSQL, DirectDatabaseQuery, SlowDBQuery)
- naming     (PrefixAllGlobals, ValidFunctionName, ValidHookName)
- wp         (DeprecatedFunctions, Capabilities, I18n, AlternativeFunctions)
- php        (YodaConditions, StrictInArray, NoSilencedErrors, DontExtract)
- docs       (PHPDoc headers, @since, @param, @return — wordpress-docs profile only)

Leave empty to enforce all categories for the active profile.
```

NEVER suggest excluding `security` or `db` categories. If the user tries to exclude them, warn:

```text
Warning: Excluding security or DB categories removes enforcement of nonce verification,
output escaping, and SQL injection prevention. This is not recommended for any
project targeting WordPress.org or production use.
```

---

## Output

Generate or update `.specify/memory/wp-standards-constitution.md` using the constitution template at `templates/wp-standards-constitution.md`.

Populate the template with all answers collected during the interview.

Also write `.specify/config/wp-coding-standards.yml` if it does not already exist, using `config-template.yml` as the base.

---

## Completion Message

After writing the constitution, confirm:

```text
WP Coding Standards initialized.

Profile:     [profile]
Prefix:      [prefix]
Min WP:      [min_wp_version]
Min PHP:     [min_php_version]
Project:     [project_type]
Severity:    [min_severity]

Configuration saved to:
- .specify/memory/wp-standards-constitution.md
- .specify/config/wp-coding-standards.yml

Run /speckit.wp-coding-standards.review to check your current codebase.
```
