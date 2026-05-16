# Changelog

All notable changes to WP Coding Standards will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] — 2026-05-17

### Added

- Initial release
- Four standards profiles: `wordpress-core`, `wordpress`, `wordpress-docs`, `wordpress-vip`
- `init` command — phased interview to configure project profile and write `wp-standards-constitution.md`
- `review` command — full standards review against active profile with structured report
- `violation-detection` command — lightweight scan for use in plan/tasks hooks
- `apply` command — converts violations into prioritized fix tasks injected into `tasks.md`
- Profile-aware rule enforcement across 11 WPCS sniff categories:
  - Security: EscapeOutput, NonceVerification, ValidatedSanitizedInput, SafeRedirect
  - DB: PreparedSQL, PreparedSQLPlaceholders, DirectDatabaseQuery, SlowDBQuery
  - NamingConventions: PrefixAllGlobals, ValidFunctionName, ValidHookName, ValidPostTypeSlug
  - WP: DeprecatedFunctions, DeprecatedClasses, Capabilities, I18n, AlternativeFunctions, CronInterval
  - PHP: YodaConditions, StrictInArray, NoSilencedErrors, DontExtract, IniSet
  - Docs (wordpress-docs profile): file headers, @since, @param, @return, hook documentation
  - VIP (wordpress-vip profile): no direct DB, no $_SESSION, no remote file_get_contents, no sleep(), batched writes, object cache requirements
- Spec Kit lifecycle hooks: `after_plan`, `after_tasks`, `after_implement`
- `config-template.yml` for project-level configuration
- Constitution template at `templates/wp-standards-constitution.md`
