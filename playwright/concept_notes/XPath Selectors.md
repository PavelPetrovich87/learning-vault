---
status: archived
tags: [playwright, testing, e2e, selectors, xpath]
source:
  - https://playwright.dev/docs/other-locators#xpath-locator
  - https://playwright.dev/docs/locators
created: 2026-07-01
updated: 2026-10-21
difficulty: beginner
---

# 🧠 Topic: XPath Selectors

> [!abstract] Executive Summary (ELI5)
> XPath selectors are a way to find elements by describing a path through the DOM tree, like "the button inside the second form that has the text 'Save'." They can walk up, down, and sideways through the tree, which makes them more powerful than CSS selectors for complex queries.

---

## 🎯 The "Why"

- **Problem Solved:** Some targets are hard to express with CSS. You may need to select by text content, traverse up to a parent, or find an element based on a sibling. XPath handles all of these.
- **Alternatives:** [[CSS Selectors]], Playwright user-facing locators (`getByRole`, `getByText`, `getByLabel`, `getByTestId`), and `locator.filter`.
- **Trade-offs:** XPath is expressive and bidirectional, but selectors are often harder to read and more brittle. They can also be slower than CSS selectors because the browser evaluates them differently.

## 💡 Mental Model & Analogy

- **Analogy:** XPath is like giving someone directions to a house by describing the route: "Go to the main street, turn at the second traffic light, then it's the third house on the left." CSS is more like pointing and saying "the red house." XPath gives you more detail, but the directions get confusing fast.
- **Visual/Diagram:**

```mermaid
graph TD
    A[//button] --> B[all button elements]
    C[//form//button] --> D[buttons inside any form]
    E[//button[text='Save']] --> F[specific button by text]
    G[//button[@data-testid='save']] --> H[specific button by attribute]
```

## 🛠️ Core Principles & Code Snippets

### Basic Syntax in Playwright

Use the `xpath=` prefix. Playwright defaults to `css=`, so the prefix is required for XPath.

```typescript
import { test, expect } from '@playwright/test';

test('basic XPath selectors', async ({ page }) => {
  await page.goto('/dashboard');

  // Select all buttons
  await page.locator('xpath=//button').click();

  // Select by attribute
  await page.locator('xpath=//button[@data-testid="save"]').click();

  // Select by text content
  await page.locator('xpath=//button[text()="Save"]').click();

  // Select by partial text
  await page.locator('xpath=//button[contains(text(), "Save")]').click();
});
```

### Traversing the DOM Tree

```typescript
// Parent of an element
await page.locator('xpath=//input[@id="email"]/parent::div').click();

// Ancestor of an element
await page.locator('xpath=//input[@id="email"]/ancestor::form').click();

// Sibling
await page.locator('xpath=//label[@for="email"]/following-sibling::input').fill('alice@example.com');

// Descendant
await page.locator('xpath=//form//button').click();
```

### Positional Predicates

```typescript
// First match
await page.locator('xpath=(//button)[1]').click();

// Last match
await page.locator('xpath=(//button)[last()]').click();

// Nth match
await page.locator('xpath=(//button)[3]').click();

// Inside a specific parent by index
await page.locator('xpath=//table[1]//tr[2]//td[3]').click();
```

### Combining Conditions

```typescript
// Element that has both attributes
await page.locator('xpath=//button[@type="submit" and @disabled]').toBeVisible();

// Element that contains a class among others
await page.locator('xpath=//button[contains(@class, "primary")]').click();

// Element with visible text inside
await page.locator('xpath=//a[contains(., "Dashboard")]').click();
```

> [!tip] Implementation Note
> Wrap positional queries in parentheses, such as `(//button)[2]`, because `//button[2]` means "the second button child of each parent" rather than "the second button in the whole document."

## ⚠️ Pitfalls & Anti-patterns

- **Overly complex XPath expressions:** Long paths like `//div[3]/div[1]/table/tbody/tr[5]/td[2]/button` break when the layout changes. Prefer test IDs or role-based locators.
- **Forgetting the `xpath=` prefix:** Without it, Playwright treats the string as a CSS selector and the query fails.
- **Using XPath for simple queries that CSS handles better:** If you only need a class or attribute, CSS is faster and more readable.
- **Absolute paths starting with `/html/body/...`:** These are extremely brittle. Always use `//` (search anywhere) or a stable anchor.
- **Text matching with whitespace:** `text()` comparisons can fail when the DOM contains extra spaces or line breaks. Use `contains(normalize-space(), 'text')` or prefer Playwright's `getByText`.
- **Case sensitivity:** XPath text matching is case-sensitive by default. Use `translate()` if you need case-insensitive matching.

## 🔗 Deep Connections

- **Parent Topic:** [[Playwright-Test-Object-and-Fixtures]]
- **Related Concepts:** [[CSS Selectors]], Playwright locators (`getByRole`, `getByText`, `getByTestId`)
- **Prerequisites:** Basic CSS, DOM tree structure, XPath syntax, Playwright locators.

## ❓ Active Recall Questions

> [!faq]- When should you choose XPath over CSS selectors?
> When you need to select by text content, traverse up to ancestors or siblings, or combine complex conditions that CSS cannot express.

> [!faq]- Why is `//button[2]` different from `(//button)[2]`?
> `//button[2]` selects the second button child of each parent, while `(//button)[2]` selects the second button in the entire document.

> [!faq]- What prefix must you use so Playwright treats a selector string as XPath?
> `xpath=` — without it, Playwright assumes the selector is a CSS selector.

> [!faq]- What is the main readability risk with XPath selectors?
> They quickly become long, nested paths that are hard to parse and maintain when the DOM changes.

---

## 📝 Related Notes

- [[CSS Selectors]]
- [[Playwright-Test-Object-and-Fixtures]]
