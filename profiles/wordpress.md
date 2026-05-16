---
description: WordPress full profile — complete standard including security, DB, naming, WP API, and PHP best-practice rules. Recommended for all plugins and themes.
---

# WP Coding Standards — WordPress Profile

This profile maps to the official `WordPress` ruleset from [WordPress/WordPress-Coding-Standards](https://github.com/WordPress/WordPress-Coding-Standards), which is the superset of `WordPress-Core` and `WordPress-Extra`.

This is the **recommended profile** for all plugins and themes, including those targeting the WordPress.org Plugin/Theme Directory.

---

## Includes

All rules from the `wordpress-core` profile, plus the categories below.

---

## Security Category

### EscapeOutput

All data output to HTML, attributes, URLs, or JavaScript must be escaped.

| Context | Required Function |
|---|---|
| HTML content | `esc_html()` / `esc_html__()` / `esc_html_e()` |
| HTML attributes | `esc_attr()` / `esc_attr__()` / `esc_attr_e()` |
| URLs in HTML | `esc_url()` |
| URLs in database/redirects | `esc_url_raw()` |
| JavaScript strings | `esc_js()` |
| Translated output | `esc_html__()`, `esc_attr__()` |
| Rich HTML (trusted) | `wp_kses()` / `wp_kses_post()` |

Violations: `echo $var`, `echo get_option(...)`, `<a href="<?php echo $url ?>">`

### NonceVerification

Any read of `$_GET`, `$_POST`, `$_REQUEST`, or `$_COOKIE` for a state-changing operation must be preceded by nonce verification.

```php
// AJAX handlers
check_ajax_referer( 'my_action', 'nonce' );

// Form submissions
if ( ! wp_verify_nonce( $_POST['_wpnonce'], 'my_action' ) ) {
    wp_die( 'Security check failed.' );
}
```

Reading superglobals for display-only operations (e.g., current page, search term) requires sanitization but not necessarily a nonce.

### ValidatedSanitizedInput

All superglobal input must be sanitized immediately upon receipt.

| Data Type | Required Function |
|---|---|
| Plain text | `sanitize_text_field()` |
| Integer | `absint()` / `intval()` |
| Email | `sanitize_email()` |
| URL | `esc_url_raw()` |
| Filename | `sanitize_file_name()` |
| HTML content | `wp_kses_post()` |
| Multi-line text | `sanitize_textarea_field()` |
| Key/slug | `sanitize_key()` |

### SafeRedirect

Use `wp_safe_redirect()` instead of `wp_redirect()` unless the redirect target is provably external and intentional.

---

## DB Category

### PreparedSQL

All database queries that include variable input must use `$wpdb->prepare()`.

```php
// Violation
$wpdb->get_results( "SELECT * FROM {$wpdb->posts} WHERE ID = $id" );

// Correct
$wpdb->get_results( $wpdb->prepare( "SELECT * FROM {$wpdb->posts} WHERE ID = %d", $id ) );
```

Applies to: `$wpdb->query()`, `$wpdb->get_results()`, `$wpdb->get_var()`, `$wpdb->get_row()`, `$wpdb->insert()`, `$wpdb->update()`, `$wpdb->delete()`.

### PreparedSQLPlaceholders

Use typed placeholders in prepared statements:
- `%d` for integers
- `%s` for strings
- `%f` for floats

Never use `%1$s` style positional placeholders with sprintf-style formatting.

### DirectDatabaseQuery

Direct `$wpdb` queries (any query not going through a WordPress API) must include a comment explaining why a native WordPress function could not be used, and should include a caching layer.

```php
// Flagged — no caching, no comment
$results = $wpdb->get_results( $wpdb->prepare( "SELECT ..." ) );

// Better
$cache_key = 'my_plugin_results_' . md5( $query );
$results = wp_cache_get( $cache_key );
if ( false === $results ) {
    $results = $wpdb->get_results( $wpdb->prepare( "SELECT ..." ) );
    wp_cache_set( $cache_key, $results, '', HOUR_IN_SECONDS );
}
```

### SlowDBQuery

Flag queries known to cause full-table scans or excessive load:
- `posts_per_page => -1` or `nopaging => true` without a hard upper bound
- `suppress_filters => true` (bypasses caching plugins)
- `meta_query` on non-indexed columns
- `tax_query` with `include_children => true` on large taxonomies

---

## NamingConventions Category

### PrefixAllGlobals

All globally-scoped symbols must use the project prefix from the constitution.

Applies to:
- Functions: `{prefix}my_function()`
- Classes: `{Prefix}_Class_Name` (PascalCase with prefix)
- Interfaces/Traits: same as classes
- Constants: `{PREFIX}_CONSTANT_NAME` (uppercase with prefix)
- Hook names: `{prefix}hook_name`
- Global variables: `${prefix}var_name`

Does NOT apply to:
- Methods inside classes
- Local variables inside functions
- Constants defined inside classes

### ValidFunctionName

Functions must use lowercase letters and underscores only (snake_case). No camelCase.

```php
// Violation
function myPluginDoSomething() {}

// Correct
function my_plugin_do_something() {}
```

### ValidHookName

Action and filter hook names must use lowercase letters, underscores, forward slashes, and hyphens only. No camelCase or PascalCase.

```php
// Violation
add_action( 'myPlugin_afterSave', ... );

// Correct
add_action( 'my_plugin/after_save', ... );
add_action( 'my_plugin_after_save', ... );
```

### ValidPostTypeSlug

Post type slugs must:
- Be lowercase
- Use underscores or hyphens only
- Be 20 characters or fewer
- Not use any WordPress reserved post type names (`post`, `page`, `attachment`, `revision`, `nav_menu_item`, etc.)

---

## WP Category

### DeprecatedFunctions

Flag any WordPress function that is deprecated at or after the configured `min_wp_version`. The review engine should suggest the current replacement.

Common deprecated functions: `get_currentuserinfo()`, `wp_get_http()`, `like_escape()`, `the_attachment_link()`.

### DeprecatedClasses

Flag WordPress classes deprecated at or after `min_wp_version`.

### Capabilities

Every privileged operation must be gated by `current_user_can()`.

```php
// Violation — no capability check
function my_plugin_delete_item( $id ) {
    $wpdb->delete( ... );
}

// Correct
function my_plugin_delete_item( $id ) {
    if ( ! current_user_can( 'manage_options' ) ) {
        wp_die( esc_html__( 'Permission denied.', 'my-plugin' ) );
    }
    $wpdb->delete( ... );
}
```

Use the most granular capability appropriate to the operation. Avoid `manage_options` for operations that don't require full admin access.

### I18n

All user-facing strings must use WordPress translation functions with the correct text domain.

| Function | Use When |
|---|---|
| `__( 'string', 'textdomain' )` | Return translated string |
| `_e( 'string', 'textdomain' )` | Echo translated string |
| `_n( 'single', 'plural', $n, 'textdomain' )` | Pluralization |
| `_x( 'string', 'context', 'textdomain' )` | String with disambiguation context |
| `esc_html__( 'string', 'textdomain' )` | Translated + escaped for HTML |
| `esc_attr__( 'string', 'textdomain' )` | Translated + escaped for attributes |

Violations: hardcoded English strings in output, wrong text domain, variable passed as text domain.

### AlternativeFunctions

Flag PHP functions that have WordPress alternatives:

| PHP Function | WordPress Alternative |
|---|---|
| `curl_*` functions | `wp_remote_get()` / `wp_remote_post()` |
| `json_encode()` | `wp_json_encode()` |
| `json_decode()` | OK — no WP alternative |
| `strip_tags()` | `wp_strip_all_tags()` |
| `parse_url()` | OK — no WP alternative |
| `file_get_contents( 'http...' )` | `wp_remote_get()` |
| `header()` | `wp_redirect()` / `wp_safe_redirect()` |
| `setcookie()` | WordPress cookie functions |

### CronInterval

Flag cron schedule intervals under 60 seconds. WordPress cron is not designed for sub-minute intervals.

---

## PHP Category

### YodaConditions

Constants and literals must be on the left side of comparisons.

```php
// Violation
if ( $status === 'publish' ) {}

// Correct
if ( 'publish' === $status ) {}
```

### StrictInArray

`in_array()` must always use strict mode.

```php
// Violation
in_array( $value, $allowed )

// Correct
in_array( $value, $allowed, true )
```

### NoSilencedErrors

The `@` error suppression operator is not permitted. Handle errors explicitly.

### DontExtract

`extract()` creates unpredictable variable scope and is not permitted. Use explicit variable assignment.

### IniSet

`ini_set()` is not permitted. Use WordPress constants or filters to configure PHP behavior.

---

## Severity Reference

| Category | Default Severity |
|---|---|
| Security (EscapeOutput, NonceVerification, ValidatedSanitizedInput, SafeRedirect) | CRITICAL |
| DB (PreparedSQL, DirectDatabaseQuery) | CRITICAL |
| DB (SlowDBQuery) | HIGH |
| NamingConventions (PrefixAllGlobals) | HIGH |
| NamingConventions (ValidFunctionName, ValidHookName) | MEDIUM |
| WP (DeprecatedFunctions, DeprecatedClasses) | HIGH |
| WP (Capabilities) | CRITICAL |
| WP (I18n) | HIGH |
| WP (AlternativeFunctions, CronInterval) | MEDIUM |
| PHP (YodaConditions, StrictInArray, NoSilencedErrors, DontExtract, IniSet) | MEDIUM |
| Core formatting rules | LOW |
