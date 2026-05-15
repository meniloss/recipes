# SHbyJM — Symfony Flex Recipes

Private Symfony Flex recipe repository for SHbyJM packages. Automates configuration when installing `shbyjm/*` packages via Composer.

## Available recipes

| Package | Version | Description |
|---------|---------|-------------|
| `shbyjm/admin-shell` | 1.1, 1.1.1, 1.1.2, 1.1.3, 2.0 | Admin shell — auth, users, dashboard, branding uploads |
| `shbyjm/lead-forwarding` | 1.0, 2.0 | Lead forwarding via outbox pattern toward the Hub |
| `shbyjm/lead-management` | 1.0 | CRM léger — leads, activités, rappels, session d'appel, settings |

## Usage on consumer sites

Add this endpoint to the `composer.json` of each consumer site:

```json
{
    "extra": {
        "symfony": {
            "endpoint": [
                "https://api.github.com/repos/meniloss/recipes/contents/index.json",
                "flex://defaults"
            ]
        }
    }
}
```

Then install the package normally:

```bash
composer require shbyjm/admin-shell
composer require shbyjm/lead-forwarding
```

Flex will automatically:
- Register the bundle in `config/bundles.php`
- Copy the default configuration YAML
- Add environment variable placeholders to `.env`
- Add relevant `.gitignore` entries
- Display post-install instructions

## Re-applying a recipe on an existing site

If a site already has the package installed and you want to apply (or re-apply) the recipe:

```bash
composer recipes:install shbyjm/admin-shell --force
composer recipes:install shbyjm/lead-forwarding --force
```

## Adding or updating a recipe

### File format

Each recipe is a single JSON file at the repository root, named:

```
{vendor}.{package}.{version}.json
```

For example: `shbyjm.admin-shell.1.1.json`

The JSON structure:

```json
{
    "manifests": {
        "vendor/package": {
            "manifest": {
                "bundles": { ... },
                "env": { ... },
                "copy-from-recipe": { ... },
                "gitignore": [ ... ],
                "post-install-output": [ ... ]
            },
            "files": {
                "path/to/file.yaml": {
                    "contents": ["line1", "line2"],
                    "executable": false
                }
            },
            "ref": "40-char-hex-string"
        }
    }
}
```

### Workflow: new version of an existing recipe

When the expected configuration of a package changes (new env vars, new config keys):

1. Create a new recipe file with the bumped version: `shbyjm.admin-shell.1.2.json`
2. Update `index.json` to add the new version to the package's version array
3. Commit and push to `main`

### Workflow: new package

1. Create the recipe file: `shbyjm.new-package.1.0.json`
2. Add the package entry in `index.json` under `"recipes"`
3. Commit and push to `main`

## Conventions

- This repo only hosts recipes for `shbyjm/*` packages — no third-party recipes
- Never put real secrets (API keys, tokens) in recipes — use placeholders only
- Paths assume PlanetHoster N0C hosting structure: `symfony/` + `public_html/` at the same level
- The `ref` field is a random 40-character hex string — regenerate it when updating a recipe (`php -r "echo bin2hex(random_bytes(20));"`)

## Release checklist for SHbyJM packages

Before tagging a new major or minor version of a `shbyjm/*` package:

1. The package code is tested and committed
2. The tag is created on the package repo (e.g. `v2.0.0`)
3. The corresponding recipe is created or updated in `meniloss/recipes`
4. `index.json` is updated with the new version
5. This README table is updated
6. Push `meniloss/recipes` to `main`
7. The release is only announced after steps 1–6 are complete

If the new version changes the installation (new env vars, new config keys, new routes, removed dependencies), a **new recipe version** is required. Patch versions that don't change the installation (bug fixes, template changes) do **not** need a new recipe — they are covered by the existing recipe for their minor version (e.g. recipe `2.0` covers `2.0.0`, `2.0.1`, `2.0.2`, etc.).
