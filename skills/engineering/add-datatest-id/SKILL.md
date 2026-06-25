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

## Naming Convention

```
<filename>:<scope>-<description>
```

* **filename** = component/page filename without extension
* **scope** = btn, input, modal, table, row, card, section, etc.
* **description** = human-readable kebab-case description

Examples:

```text
LoginPage:btn-sign-in
LoginPage:input-email
UserTable:table-users
UserTable:row-user-${user.id}
CaseDiary:btn-save
CaseDiary:modal-create-entry
OfficerForm:input-first-name
Header:icon-search
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
