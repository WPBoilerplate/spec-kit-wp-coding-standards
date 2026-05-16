---
description: WordPress-Core profile — formatting, PHP style, and whitespace rules only. No security or naming enforcement.
---

# WP Coding Standards — WordPress-Core Profile

This profile maps to the official `WordPress-Core` ruleset from [WordPress/WordPress-Coding-Standards](https://github.com/WordPress/WordPress-Coding-Standards).

It covers non-controversial, generally-agreed-upon formatting and PHP style rules. It does NOT include security, DB, naming, or WP API rules.

Use this profile only when you want formatting compliance without broader WordPress standards enforcement.

---

## Rule Categories Active in This Profile

---

### PHP Tags

- Files must use full `<?php` opening tags — short tags (`<?`, `<?=`) are not permitted
- Multi-line PHP snippets embedded in HTML must have `<?php` and `?>` on their own lines

---

### Indentation

- Indent with tabs, not spaces
- Tab width: 4 spaces equivalent
- Align multi-line statements consistently

---

### Braces and Control Structures

- Opening braces go on the same line as the control structure
- `else`, `elseif`, `catch` go on the same line as the closing brace
- Single-line `if` statements must still use braces
- Alternative syntax (`if:` / `endif;`) is permitted in template files

---

### Whitespace

- No trailing whitespace at end of lines
- Blank line at end of file
- Space before and after binary operators (`=`, `+`, `-`, `&&`, etc.)
- Space after commas in function calls and definitions
- Space after `!` in negation
- No space between function name and opening parenthesis in calls
- Space before opening parenthesis of control structures (`if (`, `foreach (`, `while (`)

---

### String Concatenation

- Spaces before and after the concatenation operator: `$a . $b`
- Break long concatenation across lines with the `.` at the beginning of the continuation line

---

### Arrays

- Short array syntax `[]` is preferred over `array()`
- Multi-line arrays: one item per line, trailing comma on last item
- Keys and values aligned when there are 3+ items

---

### Yoda Conditions

- Literals and constants must be on the left side of comparisons:
  - `if ( true === $flag )` not `if ( $flag === true )`
  - `if ( null === $value )` not `if ( $value === null )`

---

### Class and Function Declarations

- `function` keyword followed by a space before the name in anonymous functions
- Visibility (`public`, `protected`, `private`) required on all methods and properties
- `static` keyword comes after visibility: `public static function`
- Abstract and final declarations before visibility: `abstract public function`

---

### Namespace and Use Statements

- One blank line after namespace declaration
- `use` statements immediately after namespace, one per line

---

### Comments

- Single-line comments use `//`
- Multi-line comments use `/* */`
- PHPDoc blocks use `/** */`
- No inline `#` comments

---

## What This Profile Does NOT Check

This profile intentionally excludes:

- Security rules (EscapeOutput, NonceVerification, ValidatedSanitizedInput)
- Database rules (PreparedSQL, DirectDatabaseQuery)
- Naming conventions (PrefixAllGlobals, ValidFunctionName, ValidHookName)
- WordPress API rules (DeprecatedFunctions, Capabilities, I18n)
- PHP best-practice rules (YodaConditions is included; StrictInArray, DontExtract are not)

For full WordPress standards enforcement, use the `wordpress` profile.

---

## Severity Reference

All violations in this profile are **LOW** severity — they represent style inconsistencies with no functional or security impact.
