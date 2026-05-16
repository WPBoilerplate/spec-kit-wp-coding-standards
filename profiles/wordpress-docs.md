---
description: WordPress-Docs profile — inline documentation standards on top of the full WordPress profile. Requires @since, @param, @return, and file headers.
---

# WP Coding Standards — WordPress-Docs Profile

This profile maps to the official `WordPress-Docs` ruleset from [WordPress/WordPress-Coding-Standards](https://github.com/WordPress/WordPress-Coding-Standards), layered on top of the full `WordPress` profile.

Use this profile when inline documentation coverage is required — for example, projects contributing to WordPress core, public APIs intended for developer use, or plugins with strict documentation requirements.

---

## Includes

All rules from the `wordpress` profile, plus the documentation categories below.

---

## File Headers

Every PHP file must begin with a PHPDoc file header block.

**Required tags:**
- `@package` — the project package name

**Recommended tags:**
- `@subpackage` — for subdivided packages
- `@since` — version the file was introduced

```php
<?php
/**
 * Admin settings page renderer.
 *
 * @package My_Plugin
 * @since 1.0.0
 */
```

Violations: missing file header, missing `@package`, file header placed after `<?php` on the same line.

---

## Function and Method Documentation

Every function and method must have a PHPDoc block immediately above it.

### Required Tags

**`@since`**

Every function must declare the version it was introduced.

```php
/**
 * Returns the active plugin settings.
 *
 * @since 1.0.0
 */
function my_plugin_get_settings() {}
```

The version must be a valid semantic version string or a placeholder like `{VERSION}` during development.

**`@param`**

Every parameter must have a `@param` tag.

```php
/**
 * @param string $key     Option key to retrieve.
 * @param mixed  $default Default value if option is not set.
 */
function my_plugin_get_option( $key, $default = null ) {}
```

Format: `@param {type} ${name} {description}.`
- Type must be a valid PHP type or `mixed`
- Description ends with a period
- Parameters aligned by longest type + name

**`@return`**

Every non-void function must have a `@return` tag.

```php
/**
 * @return array<string, mixed> Plugin settings array.
 */
function my_plugin_get_settings() {}
```

Void functions may omit `@return` or use `@return void`.

### Recommended Tags

**`@throws`** — document any exceptions the function may throw

**`@see`** — link to related functions, classes, or external documentation

**`@link`** — external URL reference

**`@deprecated`** — if the function is deprecated, include:

```php
/**
 * @deprecated 2.0.0 Use my_plugin_get_settings() instead.
 */
function my_plugin_old_settings() {}
```

---

## Class Documentation

Every class, interface, abstract class, and trait must have a PHPDoc block.

**Required:**
- Description (at least one sentence)
- `@since` tag

```php
/**
 * Handles plugin admin settings.
 *
 * @since 1.0.0
 */
class My_Plugin_Settings {}
```

Class properties should have inline `@var` type declarations:

```php
/**
 * @var string Plugin version.
 */
private string $version;
```

---

## Hook Documentation

Every `do_action()` and `apply_filters()` call must be preceded by a documentation block.

### Actions

```php
/**
 * Fires after plugin settings are saved.
 *
 * @since 1.0.0
 *
 * @param int    $post_id Post ID.
 * @param string $status  New status value.
 */
do_action( 'my_plugin_settings_saved', $post_id, $status );
```

### Filters

```php
/**
 * Filters the plugin settings array before it is returned.
 *
 * @since 1.0.0
 *
 * @param array<string, mixed> $settings Plugin settings.
 * @param int                  $user_id  Current user ID.
 */
$settings = apply_filters( 'my_plugin_settings', $settings, $user_id );
```

**Required for hook documentation:**
- `@since` — version when the hook was introduced
- `@param` — one tag per argument passed to the hook callback

---

## Inline Comments

Inline comments that describe non-obvious logic must end with a period.

```php
// Set a transient to cache the result for 1 hour.
set_transient( $cache_key, $result, HOUR_IN_SECONDS );
```

TODO comments must include the author and a ticket/issue reference:

```php
// TODO: Replace with wp_cache_get() after cache invalidation is implemented. See #42.
```

---

## Severity Reference

| Rule | Severity |
|---|---|
| Missing file header | HIGH |
| Missing @package | HIGH |
| Missing @since on function/method | HIGH |
| Missing @param | MEDIUM |
| Missing @return on non-void function | MEDIUM |
| Missing class documentation | HIGH |
| Missing hook documentation | MEDIUM |
| Malformed @param type or format | LOW |
