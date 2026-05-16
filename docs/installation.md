# Installation

## Requirements

- Spec Kit `>=0.1.0`
- A WordPress plugin, theme, mu-plugin, or block theme project

---

## From Registry

```text
specify extension add wp-coding-standards
```

---

## From GitHub

```text
specify extension add wp-coding-standards --from \
  https://github.com/WPBoilerplate/spec-kit-wp-coding-standards/archive/refs/tags/v1.0.0.zip
```

---

## Local Development

```text
specify extension add --dev /path/to/spec-kit-wp-coding-standards
```

---

## After Installation

Run the initializer to configure your project:

```text
/speckit.wp-coding-standards.init
```

This creates two files:
- `.specify/memory/wp-standards-constitution.md` — active profile and project settings
- `.specify/config/wp-coding-standards.yml` — machine-readable configuration

Both files should be committed to version control.

---

## Companion Extensions (Optional)

WP Coding Standards works standalone but integrates with:

- **Architecture Guard** — adds structural boundary enforcement on top of standards compliance
- **Security Review** — adds deep security threat modelling on top of surface-level WPCS security sniffs

Install companions:

```text
specify extension add architecture-guard
specify extension add security-review
```
