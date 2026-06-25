---
name: add-datatest-id
description: Add stable, semantic, and unique test selectors (`data-testid`, `data-test`, `data-cy`) to frontend components without changing application behavior. Detect framework and existing convention before making changes. Use when user asks to add test IDs, data-testid attributes, test selectors, or testing hooks to UI elements.
---

# Add data-testid

## Rules

* Detect the frontend framework before making changes.
* Detect the existing testing convention (`data-testid`, `data-test`, `data-cy`, etc.) and continue using it unless instructed otherwise.
* Never rename, replace, or remove existing test IDs.
* Only add missing test IDs.
* Every interactive element should have a test ID.
* Important containers (modals, forms, tables, sections, cards, drawers, dialogs, tabs, navigation) should also have test IDs when useful for testing.
* Prefix every test ID with the component or page filename (without the extension) where the element is defined.
* Preserve formatting, styling, and application behavior.
* When an element contains visible text content (≤40 chars), derive the `description` from that text — kebab-case it. When the text is longer than 40 chars, empty, or purely decorative (icon, spacer), write a manual description instead.

## Naming Convention

```
<filename>:<scope>-<description>
```

* **filename** = component/page filename without extension
* **description** = kebab-case of the element's visible text content when ≤40 chars; otherwise a manual kebab-case description
* **scope** = btn, input, modal, table, row, card, section, heading, link, icon, etc.
Examples:

```
LoginPage:btn-sign-in
LoginPage:input-email
UserTable:table-users
UserTable:row-user-${user.id}
CaseDiary:btn-save
CaseDiary:modal-create-entry
OfficerForm:input-first-name
```

When the element's visible text is ≤40 chars, derive `description` from it:

```text
<button>Sign In</button>                    → LoginPage:btn-sign-in
<h2>User Settings</h2>                      → SettingsPage:heading-user-settings
<a>View Order #{order.id}</a>               → OrderPage:link-view-order-{order.id}
<th>First Name</th>                         → UserTable:heading-first-name
```

When text is missing, decorative, or >40 chars, write a manual description:

```text
<button aria-label="Close"><X/></button>    → Header:btn-close
<Icon name="search"/>                       → Header:icon-search
<p>This is a very long description text that exceeds forty characters easily</p> → InfoSection:text-long-description
```

If the project uses lowercase filenames, preserve that style:

```text
login-page:btn-sign-in
user-table:row-user-${user.id}
```

Use the actual filename exactly as it appears in the project (excluding the file extension).

## Dynamic Elements

Collections must include a unique identifier:

```text
UserTable:row-user-${user.id}
OrderTable:row-order-${order.id}
InvoiceCard:card-invoice-${invoice.id}
```

## Validation

Before finishing:

* every interactive element has a test ID
* no duplicate test IDs exist
* existing test IDs remain unchanged
* IDs follow the filename prefix convention
* lint/build/tests pass when available

## Validation Script

Create a script that checks whether all required elements have test IDs. This script is generated once and runs automatically.

### Script approach

Detect the framework and pick the right approach:

**React / JSX projects (recommended):** Generate a custom ESLint plugin rule (e.g. `eslint-plugin-test-selectors`) that flags JSX elements (`button`, `input`, `a`, `select`, `textarea`, `form`, `dialog`, `table`, `nav`, `section`, `article`, `main`, `header`, `footer`, `aside`, `li` in navigation/loops, `img` without an `alt`) missing a `data-testid` attribute. Add it to the project's ESLint config. Since it's a real ESLint rule, editors show it inline and `eslint --fix` can auto-insert placeholders.

**Non-JSX or no ESLint:** Create a standalone script (Node.js, Python, or shell) that scans source files with a regex for common interactive HTML/component patterns and reports elements without test IDs. Keep it in the project root as `scripts/check-test-ids.js` (or equivalent).

### Integrate into the pipeline

Wire the check into the project's existing automation so it runs on every build/run/test:

* **`package.json`** — add to the `lint` script, or create a `lint:test-ids` script and include it in `lint`:
  ```json
  "scripts": {
    "lint": "eslint . && npm run lint:test-ids",
    "lint:test-ids": "node scripts/check-test-ids.js",
    "build": "npm run lint && vite build",
    "dev": "npm run lint && vite"
  }
  ```
* **Pre-commit hooks** — add to the existing lint-staged config.
* **CI** — the `lint` step already catches it since `lint:test-ids` is part of `lint`.
* **Framework scripts** — if the project uses `next lint`, `vue-cli-service lint`, or `ng lint`, include the check as a separate lint script or custom rule.

### What the script checks

* Every interactive element (`button`, `input`, `select`, `textarea`, `a` without external href, `form`) has a test ID.
* Every important container (modal, dialog, table, nav, section, drawer, card in a collection) has a test ID.
* No duplicate test IDs across the project.
* IDs follow the `<filename>:<scope>-<description>` convention.
* Dynamic collections include a unique identifier (`${user.id}`, `item.id`, etc.).
* Description derives from element's visible text when ≤40 chars, manual otherwise.

## Workflow

1. Detect framework.
2. Detect existing selector convention.
3. Determine the current component/page filename.
4. Preserve existing test IDs.
5. Add missing test IDs using `<filename>:<scope>-<description>`.
6. Ensure uniqueness.
7. Update E2E selectors if necessary.
8. Validate changes.
9. Report modified files.
