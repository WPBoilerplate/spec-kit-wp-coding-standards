---
description: WordPress VIP Go profile — full WordPress standards plus VIP Go platform-specific rules for managed enterprise hosting.
---

# WP Coding Standards — WordPress VIP Profile

This profile enforces all rules from the `wordpress` profile plus additional rules required for [WordPress VIP Go](https://docs.wpvip.com/) managed enterprise hosting.

Use this profile when your project is deployed on or targeting the WordPress VIP platform.

---

## Includes

All rules from the `wordpress` profile, plus the VIP-specific categories below.

---

## VIP Go Platform Rules

---

### No Direct Database Queries

Direct `$wpdb` queries are heavily restricted on VIP Go.

**Prohibited patterns:**
```php
// Violation — direct query without caching or VIP approval
$wpdb->query( ... );
$wpdb->get_results( ... );
```

**Required pattern:**
- Use WordPress Options API, Transients API, or Post Meta API where possible
- When direct queries are unavoidable, wrap with `wp_cache_get()` / `wp_cache_set()`
- Add a comment: `// phpcs:ignore WordPress.DB.DirectDatabaseQuery — [reason]`

---

### No `$_SESSION`

PHP sessions are not supported on VIP Go (stateless multi-server architecture).

```php
// Violation
$_SESSION['user_data'] = $data;

// Alternatives
// - User meta: update_user_meta( $user_id, 'my_plugin_data', $data )
// - Transients: set_transient( 'my_plugin_' . $user_id, $data, HOUR_IN_SECONDS )
// - Cookies: setcookie() with appropriate scope
```

---

### No Remote `file_get_contents()`

`file_get_contents()` on HTTP/HTTPS URLs is blocked by VIP's PHP configuration.

```php
// Violation
$data = file_get_contents( 'https://api.example.com/data' );

// Required
$response = vip_safe_wp_remote_get( 'https://api.example.com/data' );
$data = wp_remote_retrieve_body( $response );
```

`vip_safe_wp_remote_get()` includes automatic timeouts, error handling, and rate limiting.

---

### No `sleep()` or `usleep()` in Request Paths

Blocking sleep calls in synchronous web requests prevent server capacity from handling other requests.

```php
// Violation
sleep( 2 );
usleep( 500000 );

// Alternatives
// - Use WP-Cron or Action Scheduler for deferred processing
// - Use exponential backoff in retry logic without blocking sleep
```

---

### `switch_to_blog()` Must Be Paired

Every `switch_to_blog()` call must have a corresponding `restore_current_blog()` call in all code paths.

```php
// Violation — no restore
switch_to_blog( $blog_id );
do_something();

// Correct
switch_to_blog( $blog_id );
do_something();
restore_current_blog();

// Correct — with exception handling
switch_to_blog( $blog_id );
try {
    do_something();
} finally {
    restore_current_blog();
}
```

---

### Batched Write Operations

Bulk inserts, updates, or deletes must be batched. Maximum 500 rows per batch.

```php
// Violation — unbounded bulk insert
foreach ( $large_array as $item ) {
    $wpdb->insert( $wpdb->posts, $item );
}

// Required — batched
$chunks = array_chunk( $large_array, 500 );
foreach ( $chunks as $chunk ) {
    foreach ( $chunk as $item ) {
        $wpdb->insert( $wpdb->posts, $item );
    }
    // Optional: sleep(0) to yield execution between batches
}
```

---

### Object Cache Required for Repeated Queries

Any query or expensive operation that executes more than once per request must use the WordPress Object Cache.

```php
// Violation — same query on every page load without caching
function my_plugin_get_stats() {
    return $wpdb->get_var( $wpdb->prepare( "SELECT COUNT(*) FROM ..." ) );
}

// Required
function my_plugin_get_stats() {
    $cache_key = 'my_plugin_stats';
    $count = wp_cache_get( $cache_key );
    if ( false === $count ) {
        $count = $wpdb->get_var( $wpdb->prepare( "SELECT COUNT(*) FROM ..." ) );
        wp_cache_set( $cache_key, $count, '', 5 * MINUTE_IN_SECONDS );
    }
    return $count;
}
```

---

### No `WP_Query` Full-Text Search

VIP Go uses Elasticsearch for search. Direct `WP_Query` with the `s` (search) parameter is prohibited.

```php
// Violation
$query = new WP_Query( [ 's' => $search_term ] );

// Required — use VIP Search / Elasticsearch integration
// See https://docs.wpvip.com/how-tos/vip-search/
```

---

### No Unbounded `wp_remote_get()` Without Timeout

All outbound HTTP requests must specify a timeout.

```php
// Violation — no timeout
wp_remote_get( 'https://api.example.com/data' );

// Required
wp_remote_get(
    'https://api.example.com/data',
    [ 'timeout' => 3 ]
);
```

VIP recommends a maximum timeout of 3 seconds for synchronous requests. Use Action Scheduler for longer-running HTTP operations.

---

### No `wp_redirect()` Without `exit`

All redirect calls must be followed immediately by `exit` or `wp_die()`.

```php
// Violation
wp_safe_redirect( admin_url() );

// Correct
wp_safe_redirect( admin_url() );
exit;
```

---

## Severity Reference

| VIP Rule | Severity |
|---|---|
| Direct `$wpdb` query without caching | CRITICAL |
| `$_SESSION` usage | CRITICAL |
| Remote `file_get_contents()` | CRITICAL |
| `sleep()` / `usleep()` in request path | CRITICAL |
| `switch_to_blog()` without `restore_current_blog()` | CRITICAL |
| Unbounded bulk write (> 500 rows) | HIGH |
| Missing object cache on repeated query | HIGH |
| `WP_Query` full-text search | HIGH |
| `wp_remote_get()` without timeout | HIGH |
| `wp_redirect()` without `exit` | HIGH |
