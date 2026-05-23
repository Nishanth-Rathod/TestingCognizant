# Playwright — Complete Notes (Basic to Advanced)

Study notes in proper learning order. Each section explains the **concept** first, then shows the **code**. Network is at the end as requested.

---

## Table of Contents

1. [Introduction & Setup](#1-introduction--setup)
2. [Project Structure & First Test](#2-project-structure--first-test)
3. [Browser → Context → Page](#3-browser--context--page)
4. [Page Navigation](#4-page-navigation)
5. [Locators](#5-locators)
6. [Actions & Interactions](#6-actions--interactions)
7. [Assertions](#7-assertions)
8. [Waits & Auto-Waiting](#8-waits--auto-waiting)
9. [Frames (iframes)](#9-frames-iframes)
10. [New Tabs, Windows, Popups](#10-new-tabs-windows-popups)
11. [Dialogs (Alerts, Confirms, Prompts)](#11-dialogs-alerts-confirms-prompts)
12. [File Uploads & Downloads](#12-file-uploads--downloads)
13. [Keyboard & Mouse](#13-keyboard--mouse)
14. [Scrolling](#14-scrolling)
15. [Cookies, Storage & Authentication](#15-cookies-storage--authentication)
16. [Hooks & Test Lifecycle](#16-hooks--test-lifecycle)
17. [Fixtures](#17-fixtures)
18. [Page Object Model (POM)](#18-page-object-model-pom)
19. [Configuration & Projects](#19-configuration--projects)
20. [Parallel Execution & Sharding](#20-parallel-execution--sharding)
21. [Debugging Tools](#21-debugging-tools)
22. [Reporters](#22-reporters)
23. [Visual Testing](#23-visual-testing)
24. [Network — Interception, Mocking, API Testing](#24-network--interception-mocking-api-testing)
25. [Advanced Tips & Best Practices](#25-advanced-tips--best-practices)

---

## 1. Introduction & Setup

### Concept

**Playwright** is an end-to-end testing framework from Microsoft. It controls real browsers (Chromium, Firefox, WebKit) using a single API.

**Why it stands out:**
- **Auto-waiting** — every action waits for the element to be ready (no `sleep`).
- **Cross-browser** — same test code runs on Chrome, Edge, Firefox, and Safari (WebKit).
- **Out-of-process** — Playwright drives the browser from outside, so it can handle multiple tabs, multiple origins, and frames natively (Cypress can't).
- **Parallel by default** — every test gets its own isolated browser context.
- **Built-in tracing, video, screenshots, codegen, network interception.**

### Installation

```bash
# Create new Playwright project (recommended)
npm init playwright@latest

# Or add to existing project
npm install -D @playwright/test
npx playwright install         # downloads browser binaries
npx playwright install chromium # only one browser
```

The init command creates:
```
playwright.config.ts        # central config
tests/                      # your tests live here
tests-examples/             # sample tests
```

---

## 2. Project Structure & First Test

### Concept

A test file uses `test` and `expect` exports. Each `test(...)` block is one independent test. Tests in the same file run sequentially by default; files run in parallel.

### First Test

```ts
// tests/example.spec.ts
import { test, expect } from '@playwright/test';

test('homepage has Playwright in title', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  await expect(page).toHaveTitle(/Playwright/);
});
```

### Running Tests

```bash
npx playwright test                  # all tests, headless
npx playwright test --headed         # show the browser
npx playwright test --ui             # watch mode (best for development)
npx playwright test login.spec.ts    # specific file
npx playwright test -g "smoke"       # filter by test name
npx playwright test --project=chromium
npx playwright test --debug          # step debugger
npx playwright show-report           # open last HTML report
```

---

## 3. Browser → Context → Page

### Concept

Three levels of hierarchy — understand this and a lot of Playwright clicks into place:

```
Browser  ─ the actual Chromium/Firefox/WebKit process (heavy)
  │
  └── BrowserContext  ─ an isolated incognito-like profile (cheap, fast)
        │              cookies, storage, cache are isolated per context
        │
        └── Page  ─ a single tab inside a context
```

**Each test gets its own fresh context.** That's why tests don't pollute each other.

You can have multiple contexts (= multiple "users") in one test, and multiple pages (= multiple tabs) in one context.

### Code

```ts
test('three-level hierarchy', async ({ browser }) => {
  const context = await browser.newContext();   // like opening incognito
  const page1 = await context.newPage();
  const page2 = await context.newPage();

  await page1.goto('https://example.com');
  await page2.goto('https://playwright.dev');

  await context.close();   // closes both pages
});

test('two users at once', async ({ browser }) => {
  const userA = await browser.newContext();
  const userB = await browser.newContext();
  const pageA = await userA.newPage();
  const pageB = await userB.newPage();
  // Separate cookies, sessions, localStorage
});
```

When you write `async ({ page })`, Playwright automatically creates a fresh context + page for that test.

---

## 4. Page Navigation

### Concept

Navigation methods control the URL of the current page. Most also wait for the page to reach a load state by default.

### Code

```ts
await page.goto('https://example.com');             // navigate
await page.goto('/dashboard');                       // uses baseURL from config
await page.goBack();                                 // browser back
await page.goForward();                              // browser forward
await page.reload();                                 // refresh

// Wait until specific load state
await page.goto('/heavy', { waitUntil: 'domcontentloaded' });
// Options: 'load' (default), 'domcontentloaded', 'networkidle', 'commit'

// Get current URL / title
const url = page.url();
const title = await page.title();

// Set viewport size
await page.setViewportSize({ width: 1280, height: 720 });
```

> **Note:** `'networkidle'` waits for no network activity for 500ms — convenient but **flaky** on apps with constant polling/analytics. Prefer `domcontentloaded` + a specific assertion.

---

## 5. Locators

### Concept

A **Locator** represents an element (or a set of elements) on the page. Locators are **lazy** — they don't query the DOM until you act on them. They re-find the element every time, so they never go stale.

**Old way (deprecated):** `page.$('selector')` — returns a snapshot that can break.
**New way:** `page.locator(...)` and the user-facing helpers below.

### Recommended (User-Facing) Locators

These match how a real user perceives the page:

```ts
page.getByRole('button', { name: 'Sign in' });   // ⭐ best — accessibility tree
page.getByLabel('Email');                         // form fields by label
page.getByPlaceholder('you@example.com');
page.getByText('Welcome back');
page.getByAltText('Profile picture');
page.getByTitle('Close');
page.getByTestId('submit-btn');                   // last-resort, but stable
```

### CSS / XPath (fallback)

```ts
page.locator('button.submit');             // CSS
page.locator('//button[@type="submit"]');  // XPath — avoid if possible
page.locator('text=Sign in');              // text engine
```

### Filtering & Chaining

```ts
// Filter by text content
page.getByRole('listitem').filter({ hasText: 'Pending' });

// Filter by child element
page.getByRole('listitem').filter({
  has: page.getByRole('button', { name: 'Delete' }),
});

// Filter "not"
page.getByRole('listitem').filter({ hasNotText: 'Done' });

// Scope inside another locator
const row = page.getByRole('row', { name: 'Alice' });
await row.getByRole('button', { name: 'Edit' }).click();
```

### Working with Multiple Matches

```ts
const items = page.getByRole('listitem');

await items.first().click();
await items.last().hover();
await items.nth(2).click();

const count = await items.count();
const all = await items.all();
for (const el of all) {
  console.log(await el.textContent());
}
```

### Locator Best Practices

1. **Prefer role-based locators.** They survive UI redesigns.
2. **Avoid XPath** unless absolutely necessary.
3. **Use `data-testid`** when the UI doesn't have meaningful labels.
4. **Don't chain CSS deeply** — `div > div > .btn` breaks easily.
5. **Use `getByRole` + `name`** to disambiguate same-role elements.

---

## 6. Actions & Interactions

### Concept

Actions are the verbs you call on locators: click, type, check, hover. **Every action auto-waits** for the element to be:
- **Attached** (in the DOM)
- **Visible**
- **Stable** (not animating)
- **Enabled**
- **Receiving events** (not covered by another element)

This is why you almost never need manual waits before clicking.

### Common Actions

```ts
// Click
await page.getByRole('button', { name: 'Save' }).click();
await locator.click({ button: 'right' });           // right-click
await locator.dblclick();                            // double-click
await locator.click({ position: { x: 10, y: 10 } }); // at coordinates
await locator.click({ modifiers: ['Shift'] });       // Shift+click
await locator.click({ force: true });                // skip checks (sparingly)

// Typing
await page.getByLabel('Email').fill('test@a.com');                       // clears + types
await page.getByLabel('Email').pressSequentially('hello', { delay: 50 }); // key by key
await page.getByLabel('Email').clear();

// Checkboxes / radios
await page.getByRole('checkbox', { name: 'Agree' }).check();
await page.getByRole('checkbox', { name: 'Agree' }).uncheck();
await page.getByRole('radio', { name: 'Yes' }).check();

// Dropdowns (<select>)
await page.getByLabel('Country').selectOption('US');
await page.getByLabel('Country').selectOption({ label: 'India' });
await page.getByLabel('Country').selectOption(['US', 'IN']);   // multi-select

// Hover / focus / blur
await page.getByText('Tooltip target').hover();
await page.getByLabel('Email').focus();
await page.getByLabel('Email').blur();

// Drag and drop
await page.locator('#source').dragTo(page.locator('#target'));
```

### Reading Element State

```ts
const text  = await locator.textContent();
const inner = await locator.innerText();
const html  = await locator.innerHTML();
const value = await locator.inputValue();
const attr  = await locator.getAttribute('href');

const visible = await locator.isVisible();
const enabled = await locator.isEnabled();
const checked = await locator.isChecked();
```

> Reading state is a snapshot (no auto-retry). For checks, prefer `await expect(locator).toX()`.

---

## 7. Assertions

### Concept

Two kinds of assertions:

1. **Web-first assertions** (`await expect(locator).toX()`) — auto-retry until true or timeout. **Always prefer for UI checks.**
2. **Generic assertions** (`expect(value).toBe(...)`) — Jest-style, no retry. For non-DOM values.

### Web-First Assertions

```ts
// Visibility & state
await expect(locator).toBeVisible();
await expect(locator).toBeHidden();
await expect(locator).toBeEnabled();
await expect(locator).toBeDisabled();
await expect(locator).toBeChecked();
await expect(locator).toBeFocused();
await expect(locator).toBeEmpty();
await expect(locator).toBeEditable();

// Text & values
await expect(locator).toHaveText('Hello');
await expect(locator).toHaveText(/welcome/i);
await expect(locator).toContainText('elcom');
await expect(locator).toHaveValue('admin');
await expect(locator).toHaveAttribute('href', '/home');
await expect(locator).toHaveClass(/active/);
await expect(locator).toHaveCount(5);
await expect(locator).toHaveCSS('color', 'rgb(255, 0, 0)');

// Page-level
await expect(page).toHaveURL(/dashboard/);
await expect(page).toHaveTitle('Home');
```

### Custom Timeout & Negation

```ts
await expect(locator).toBeVisible({ timeout: 10_000 });
await expect(locator).not.toBeVisible();
```

### Soft Assertions

A failing soft assertion doesn't stop the test — it logs and continues.

```ts
await expect.soft(page.getByTestId('total')).toHaveText('$120');
await expect.soft(page.getByTestId('tax')).toHaveText('$10');
await expect.soft(page.getByTestId('shipping')).toHaveText('$5');
```

### Polling Assertions (`expect.poll`)

For non-locator values that change over time:

```ts
await expect.poll(async () => {
  const res = await fetch('/api/jobs/123');
  return (await res.json()).status;
}, { timeout: 30_000 }).toBe('completed');
```

---

## 8. Waits & Auto-Waiting

### Concept

Playwright auto-waits for actions and assertions, so manual waits are rare. Use them only for:
- URL changes
- Network responses
- Specific load states
- Arbitrary conditions

### Avoid `waitForTimeout` in Real Tests

```ts
await page.waitForTimeout(2000);  // ❌ flaky and slow — only for debugging
```

### Useful Waits

```ts
// URL & navigation
await page.waitForURL('**/dashboard');
await page.waitForURL(/\/orders\/\d+/);

// Load states
await page.waitForLoadState('domcontentloaded');
await page.waitForLoadState('load');
await page.waitForLoadState('networkidle');

// Custom condition (runs in browser)
await page.waitForFunction(() => document.fonts.ready);
await page.waitForFunction(() => (window as any).appReady === true);

// Element-level
await locator.waitFor({ state: 'visible' });
await locator.waitFor({ state: 'hidden' });
await locator.waitFor({ state: 'attached' });
await locator.waitFor({ state: 'detached' });

// Wait for specific event
const popup = await page.waitForEvent('popup');
const download = await page.waitForEvent('download');
```

### The "Listen-Before-Act" Pattern

When an action triggers an async event (popup, download, network call), set up the wait **before** the click:

```ts
// ❌ Wrong order — race condition
await page.getByText('Open').click();
const popup = await page.waitForEvent('popup');

// ✅ Right order
const popupPromise = page.waitForEvent('popup');
await page.getByText('Open').click();
const popup = await popupPromise;
```

### Element Appears & Disappears

For loading spinners, toasts, etc., let assertions handle it:

```ts
await expect(page.getByText('Saving...')).toBeVisible();
await expect(page.getByText('Saving...')).toBeHidden();
```

---

## 9. Frames (iframes)

### Concept

An `<iframe>` is a separate document inside the page. Normal page locators can't see inside — you need a **FrameLocator**.

### Code

```ts
// Get a handle to an iframe
const frame = page.frameLocator('#payment-iframe');

// Now use it like a page
await frame.getByLabel('Card number').fill('4242 4242 4242 4242');
await frame.getByLabel('Expiry').fill('12/30');
await frame.getByRole('button', { name: 'Pay' }).click();

// Nested iframes
const inner = page.frameLocator('#outer').frameLocator('#inner');
await inner.getByText('Confirm').click();

// By name attribute
const named = page.frameLocator('iframe[name="checkout"]');

// Programmatic access
const f  = page.frame({ name: 'checkout' });
const f2 = page.frame({ url: /stripe\.com/ });
```

> A FrameLocator is lazy too — re-resolves on each call. Safe across re-renders.

---

## 10. New Tabs, Windows, Popups

### Concept

When a link/button opens a new tab, Playwright fires a `page` event on the **context** (the new tab lives in the same context). Wait for that event and grab the new page.

### Code

```ts
test('opens detail in new tab', async ({ context, page }) => {
  await page.goto('/products');

  const newTabPromise = context.waitForEvent('page');
  await page.getByText('Open in new tab').click();
  const newTab = await newTabPromise;

  await newTab.waitForLoadState();
  await expect(newTab).toHaveURL(/\/products\/\d+/);
});
```

### Popups (`window.open`)

```ts
const popupPromise = page.waitForEvent('popup');
await page.getByText('Open popup').click();
const popup = await popupPromise;
await popup.waitForLoadState();
```

### Switching Between Tabs

```ts
const allPages = context.pages();   // every open tab
await allPages[0].bringToFront();   // focus first tab
```

---

## 11. Dialogs (Alerts, Confirms, Prompts)

### Concept

JavaScript dialogs (`alert`, `confirm`, `prompt`) **block the page**. Playwright auto-dismisses them unless you register a handler before they trigger.

### Code

```ts
// Register handler BEFORE the action that triggers the dialog
page.on('dialog', async dialog => {
  console.log('type:', dialog.type());        // 'alert' | 'confirm' | 'prompt' | 'beforeunload'
  console.log('message:', dialog.message());
  console.log('default:', dialog.defaultValue());

  await dialog.accept();              // OK
  // await dialog.accept('answer');   // for prompt()
  // await dialog.dismiss();          // Cancel
});

await page.getByRole('button', { name: 'Delete' }).click();
```

### One-Time Handler

```ts
page.once('dialog', d => d.accept());
```

---

## 12. File Uploads & Downloads

### Concept

Uploads use a method that bypasses the OS file picker. Downloads use a `download` event you wait for.

### Uploads

```ts
// Single file
await page.getByLabel('Upload').setInputFiles('path/to/photo.png');

// Multiple files
await page.getByLabel('Upload').setInputFiles([
  'path/to/a.pdf',
  'path/to/b.pdf',
]);

// Clear selected files
await page.getByLabel('Upload').setInputFiles([]);

// In-memory file (no real file on disk)
await page.getByLabel('Upload').setInputFiles({
  name: 'note.txt',
  mimeType: 'text/plain',
  buffer: Buffer.from('hello world'),
});
```

### Hidden File Inputs

If the upload UI hides the actual `<input type="file">`, intercept the file chooser event:

```ts
const fileChooserPromise = page.waitForEvent('filechooser');
await page.getByText('Upload').click();
const fileChooser = await fileChooserPromise;
await fileChooser.setFiles('path/to/file.pdf');
```

### Downloads

```ts
const downloadPromise = page.waitForEvent('download');
await page.getByRole('link', { name: 'Download report' }).click();
const download = await downloadPromise;

console.log(await download.suggestedFilename());
await download.saveAs(`./downloads/${await download.suggestedFilename()}`);
```

---

## 13. Keyboard & Mouse

### Concept

Most interactions go through `locator.click()` / `locator.fill()`. For low-level control (drawing on canvas, complex shortcuts), use `page.keyboard` and `page.mouse`.

### Keyboard

```ts
// Press a single key on a focused locator
await locator.press('Enter');
await locator.press('Control+A');
await locator.press('Shift+ArrowRight');

// Type with delay (key-by-key)
await locator.pressSequentially('hello', { delay: 50 });

// Low-level keyboard
await page.keyboard.down('Shift');
await page.keyboard.press('ArrowDown');
await page.keyboard.press('ArrowDown');
await page.keyboard.up('Shift');

await page.keyboard.type('Slow typing', { delay: 100 });
```

### Mouse

```ts
await page.mouse.move(100, 200);
await page.mouse.down();
await page.mouse.move(300, 400, { steps: 10 });   // smooth move
await page.mouse.up();

await page.mouse.click(100, 200);
await page.mouse.dblclick(100, 200);
```

### Manual Drag-and-Drop (for finicky UIs)

```ts
const src = page.locator('#item');
const dst = page.locator('#dropzone');

const sb = await src.boundingBox();
const db = await dst.boundingBox();

await page.mouse.move(sb!.x + sb!.width / 2, sb!.y + sb!.height / 2);
await page.mouse.down();
await page.mouse.move(db!.x + 10, db!.y + 10, { steps: 10 });
await page.mouse.move(db!.x + db!.width / 2, db!.y + db!.height / 2, { steps: 10 });
await page.mouse.up();
```

---

## 14. Scrolling

### Concept

Scrolling is a frequent need: bringing an element into view, triggering lazy-loaded content, testing infinite scroll, or checking scroll-based animations. Playwright doesn't have one single `scroll()` method — instead, there are **several techniques**, and you pick the right one for the job.

**Important:** for most actions you don't need to scroll manually — `click()`, `hover()`, `fill()`, etc. **automatically scroll the element into view** before acting. You only need explicit scrolling when:
- You're triggering scroll-based behavior (lazy loading, sticky headers, animations).
- You want to verify scroll position.
- You need to scroll a specific scrollable container (not the page).

### 1. Scroll an Element Into View

The simplest method — works on any locator:

```ts
await page.locator('#footer').scrollIntoViewIfNeeded();
await page.getByText('Section 5').scrollIntoViewIfNeeded();
```

This scrolls the **nearest scrollable ancestor** until the element is visible. Works for both page-level and inner-container scrolling.

### 2. Scroll Using the Mouse Wheel

Useful for scroll events that depend on real wheel input (parallax, sticky behavior, infinite scroll triggers):

```ts
// Scroll down 500 pixels
await page.mouse.wheel(0, 500);

// Scroll up 300 pixels
await page.mouse.wheel(0, -300);

// Scroll horizontally
await page.mouse.wheel(500, 0);
```

> **Note:** `mouse.wheel` scrolls wherever the mouse currently is. To scroll inside a specific element, move the mouse over it first:
>
> ```ts
> await page.locator('#chat-box').hover();
> await page.mouse.wheel(0, 1000);
> ```

### 3. Scroll Using Keyboard

```ts
// Page must be focused first (click somewhere or focus body)
await page.locator('body').focus();

await page.keyboard.press('PageDown');
await page.keyboard.press('PageUp');
await page.keyboard.press('End');         // jump to bottom of page
await page.keyboard.press('Home');        // jump to top
await page.keyboard.press('ArrowDown');
await page.keyboard.press('Space');       // scrolls down by one viewport
```

### 4. Scroll Using JavaScript (most flexible)

When you need precise control, run JS in the page context with `page.evaluate`:

```ts
// Scroll to absolute position
await page.evaluate(() => window.scrollTo(0, 0));                            // top
await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));   // bottom

// Scroll by relative amount
await page.evaluate(() => window.scrollBy(0, 500));

// Smooth scroll
await page.evaluate(() => window.scrollTo({ top: 1000, behavior: 'smooth' }));
```

### 5. Scroll a Specific Element (not the page)

For scrolling inside a `<div>` with `overflow: scroll` (chat panes, modals, custom dropdowns):

```ts
// Scroll a specific container to the bottom
await page.locator('#chat-list').evaluate(el => {
  el.scrollTop = el.scrollHeight;
});

// Scroll to top
await page.locator('#chat-list').evaluate(el => { el.scrollTop = 0; });

// Scroll by amount
await page.locator('#chat-list').evaluate(el => { el.scrollTop += 200; });

// Horizontal scroll
await page.locator('#carousel').evaluate(el => { el.scrollLeft += 300; });
```

### 6. Read Current Scroll Position

```ts
const y = await page.evaluate(() => window.scrollY);
const x = await page.evaluate(() => window.scrollX);

// For a specific element
const top = await page.locator('#list').evaluate(el => el.scrollTop);
const max = await page.locator('#list').evaluate(el => el.scrollHeight);
```

### 7. Pattern: Infinite Scroll / Lazy Loading

Keep scrolling until the desired number of items is loaded:

```ts
test('loads 50 items via infinite scroll', async ({ page }) => {
  await page.goto('/feed');
  const items = page.getByRole('listitem');

  while ((await items.count()) < 50) {
    await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
    await page.waitForTimeout(500);   // wait for next batch to render
  }

  await expect(items).toHaveCount(50);
});
```

A cleaner version — wait for the next API response instead of a timeout:

```ts
const nextBatch = page.waitForResponse('**/api/feed?page=*');
await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
await nextBatch;
```

### 8. Pattern: Scroll to Element That Loads Lazily

```ts
const target = page.getByText('Item 100');

// Keep scrolling until the element shows up
while (!(await target.isVisible())) {
  await page.mouse.wheel(0, 1000);
  await page.waitForTimeout(200);
}
await target.click();
```

### 9. Pattern: Verify Scroll Position After Action

```ts
await page.getByRole('link', { name: 'Back to top' }).click();
await expect.poll(() => page.evaluate(() => window.scrollY)).toBe(0);
```

### Quick Cheat Sheet

| Goal | Best Method |
|---|---|
| Bring an element into view | `locator.scrollIntoViewIfNeeded()` |
| Scroll page by pixels | `page.mouse.wheel(0, 500)` |
| Jump to top / bottom | `window.scrollTo` via `page.evaluate` |
| Scroll a specific container | `locator.evaluate(el => el.scrollTop = ...)` |
| Trigger keyboard scroll | `page.keyboard.press('PageDown' / 'End')` |
| Read current scroll position | `page.evaluate(() => window.scrollY)` |
| Infinite scroll loop | `scrollTo(bottom)` + `waitForResponse` |

---

## 15. Cookies, Storage & Authentication

### Concept

Each context has its own cookies, localStorage, sessionStorage. You can save the entire authenticated state to a JSON file once, then load it for every test — avoiding repeated logins.

### Cookies

```ts
await context.addCookies([{
  name: 'session',
  value: 'abc123',
  domain: 'example.com',
  path: '/',
}]);

const cookies = await context.cookies();
const apiCookies = await context.cookies('https://api.example.com');
await context.clearCookies();
```

### LocalStorage / SessionStorage

```ts
await page.evaluate(() => localStorage.setItem('token', 'abc'));
const token = await page.evaluate(() => localStorage.getItem('token'));
await page.evaluate(() => localStorage.clear());
```

### Storage State (the auth pattern)

```ts
// auth.setup.ts — log in once and save state
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('user@test.com');
  await page.getByLabel('Password').fill('pw');
  await page.getByRole('button', { name: 'Sign in' }).click();
  await page.waitForURL('/dashboard');

  await page.context().storageState({ path: 'playwright/.auth/user.json' });
});
```

```ts
// playwright.config.ts wires it up
projects: [
  { name: 'setup', testMatch: /.*\.setup\.ts/ },
  {
    name: 'chromium',
    use: {
      ...devices['Desktop Chrome'],
      storageState: 'playwright/.auth/user.json',
    },
    dependencies: ['setup'],
  },
],
```

Now every test in the `chromium` project starts already logged in.

---

## 16. Hooks & Test Lifecycle

### Concept

Hooks run setup/teardown code at specific points:

```
beforeAll  →  beforeEach  →  test  →  afterEach  →  afterAll
```

`beforeAll` runs once per **worker**. `beforeEach` runs before every test.

### Code

```ts
import { test, expect } from '@playwright/test';

test.beforeAll(async () => {
  console.log('once before all tests in this file');
});

test.beforeEach(async ({ page }) => {
  await page.goto('/');
});

test.afterEach(async ({ page }, testInfo) => {
  if (testInfo.status !== 'passed') {
    await page.screenshot({ path: `failed-${testInfo.title}.png` });
  }
});

test.afterAll(async () => {
  console.log('once after all tests');
});

test.describe('Cart', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/cart');
  });

  test('adds item', async ({ page }) => { /* ... */ });
  test('removes item', async ({ page }) => { /* ... */ });
});
```

### Test Annotations

```ts
test.skip('not ready', async ({ page }) => { });
test.fixme('broken bug', async ({ page }) => { });
test.only('focus me', async ({ page }) => { });
test.slow();   // 3x timeout

test('only on mobile', async ({ isMobile }) => {
  test.skip(!isMobile, 'mobile only test');
});
```

---

## 17. Fixtures

### Concept

A **fixture** is a reusable piece of setup/teardown injected into tests. The built-in `page`, `context`, `browser`, and `request` are all fixtures. You can create your own.

Fixtures are **lazy** (only created when a test asks for them) and **isolated** (fresh per test by default).

### Custom Fixture Example

```ts
// fixtures.ts
import { test as base, Page } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

type MyFixtures = {
  loginPage: LoginPage;
  authedPage: Page;
};

export const test = base.extend<MyFixtures>({
  // Simple fixture
  loginPage: async ({ page }, use) => {
    const lp = new LoginPage(page);
    await use(lp);
  },

  // With setup + teardown
  authedPage: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: 'playwright/.auth/user.json',
    });
    const page = await context.newPage();
    await use(page);              // test runs here
    await context.close();        // teardown
  },
});

export { expect } from '@playwright/test';
```

```ts
// usage in tests
import { test, expect } from './fixtures';

test('uses login fixture', async ({ loginPage }) => {
  await loginPage.goto();
  await loginPage.login('a@b.com', 'pw');
});
```

### Worker-Scoped Fixtures

Run **once per worker** instead of per test — useful for expensive setup:

```ts
export const test = base.extend<{}, { dbAccount: Account }>({
  dbAccount: [async ({}, use) => {
    const account = await createAccount();   // expensive
    await use(account);
    await deleteAccount(account.id);
  }, { scope: 'worker' }],
});
```

---

## 18. Page Object Model (POM)

### Concept

POM wraps each page or component in a class that exposes locators and actions. Tests then read like a story.

**Goals:**
- Selectors live in one place — easier to maintain.
- Reusable actions (`login`, `addToCart`).
- Clear test intent.

### Example

```ts
// pages/LoginPage.ts
import { Page, Locator, expect } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly email: Locator;
  readonly password: Locator;
  readonly submit: Locator;
  readonly error: Locator;

  constructor(page: Page) {
    this.page = page;
    this.email    = page.getByLabel('Email');
    this.password = page.getByLabel('Password');
    this.submit   = page.getByRole('button', { name: 'Sign in' });
    this.error    = page.getByRole('alert');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.email.fill(email);
    await this.password.fill(password);
    await this.submit.click();
  }

  async expectError(text: string | RegExp) {
    await expect(this.error).toContainText(text);
  }
}
```

```ts
// tests/login.spec.ts
import { test } from '../fixtures';

test('invalid credentials show error', async ({ loginPage }) => {
  await loginPage.goto();
  await loginPage.login('a@b.com', 'wrong');
  await loginPage.expectError(/invalid credentials/i);
});
```

> Keep assertions in the test file unless integral to the action (like `expectError`).

---

## 19. Configuration & Projects

### Concept

`playwright.config.ts` sets defaults for every test: timeouts, base URL, browsers, retries, reporters. **Projects** let you run the same tests under different configs (different browsers, viewports, auth states).

### Sample Config

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 4 : undefined,
  reporter: [['html'], ['list']],

  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    actionTimeout: 10_000,
    navigationTimeout: 30_000,
  },

  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox',  use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit',   use: { ...devices['Desktop Safari'] } },
    { name: 'mobile',   use: { ...devices['iPhone 13'] } },
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Common `use` Options

```ts
use: {
  baseURL: 'https://example.com',
  viewport: { width: 1280, height: 720 },
  locale: 'en-US',
  timezoneId: 'Asia/Kolkata',
  geolocation: { latitude: 17.385, longitude: 78.486 },
  permissions: ['geolocation'],
  ignoreHTTPSErrors: true,
  headless: false,
  storageState: 'auth.json',
  extraHTTPHeaders: { 'x-test': '1' },
  colorScheme: 'dark',
}
```

---

## 20. Parallel Execution & Sharding

### Concept

By default each test file runs in its own **worker process**. Tests within a file run sequentially unless you opt in to parallel mode.

- `fullyParallel: true` — every test runs in parallel, even within a file.
- `workers: 4` — limit parallelism.
- **Sharding** splits tests across machines in CI.

### Code

```ts
test.describe.configure({ mode: 'parallel' });   // tests in this describe run in parallel
test.describe.configure({ mode: 'serial' });     // run in order, stop on first failure
```

### Sharding (split across CI machines)

```bash
npx playwright test --shard=1/4
npx playwright test --shard=2/4
```

### GitHub Actions Example

```yaml
name: Playwright Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test --shard=${{ matrix.shard }}/4
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: report-${{ matrix.shard }}
          path: playwright-report
```

---

## 21. Debugging Tools

### Concept

Playwright has the best debugging tooling of any e2e framework. Four tools to know:

1. **UI Mode** — watch mode with timeline, locator picker, time travel.
2. **Inspector** — step-by-step debugger.
3. **Trace Viewer** — replay a finished test as a timeline.
4. **Codegen** — record actions and generate test code.

### Commands

```bash
npx playwright test --ui                     # ⭐ best for development
npx playwright test --debug                  # step debugger (Inspector)
npx playwright codegen example.com           # record-and-replay code generator
npx playwright show-trace trace.zip          # open a trace file
npx playwright show-report                   # open last HTML report
```

### In-Code Debugging

```ts
await page.pause();   // opens Inspector at this point
```

### Capturing Trace, Screenshot, Video

```ts
use: {
  trace: 'on-first-retry',         // 'on' | 'off' | 'retain-on-failure'
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
}
```

---

## 22. Reporters

### Concept

Reporters format test results. You can use multiple at once — a human-readable one in the terminal and a machine-readable one for CI.

### Built-in Reporters

```ts
reporter: [
  ['list'],                                              // default terminal output
  ['html', { open: 'never' }],                           // self-contained HTML report
  ['json', { outputFile: 'results.json' }],              // for custom processing
  ['junit', { outputFile: 'results.xml' }],              // for Jenkins, GitLab, etc.
  ['github'],                                            // GitHub Actions annotations
  ['line'],                                              // single-line progress
  ['dot'],                                               // minimal
]
```

### Custom Reporter

```ts
// my-reporter.ts
import { Reporter, TestCase, TestResult } from '@playwright/test/reporter';

export default class MyReporter implements Reporter {
  onTestEnd(test: TestCase, result: TestResult) {
    console.log(`${test.title}: ${result.status}`);
  }
}
```

```ts
// playwright.config.ts
reporter: [['./my-reporter.ts']],
```

---

## 23. Visual Testing

### Concept

Visual testing captures a screenshot and compares it pixel-by-pixel against a stored "baseline." It catches CSS regressions a functional test can't.

### Code

```ts
test('home page matches snapshot', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('home.png', {
    maxDiffPixelRatio: 0.01,
    mask: [page.locator('.timestamp')],   // ignore dynamic regions
  });
});

// Element-level snapshot
await expect(page.locator('#chart')).toHaveScreenshot('chart.png');
```

### First Run vs Update

The first run **creates** the baseline. To update after a UI change:

```bash
npx playwright test --update-snapshots
```

### Tips

- Visual tests are **OS- and font-sensitive**. Run them in a consistent environment (usually CI Linux).
- Mask volatile regions (timestamps, ads, animations).
- Keep them in their own project so flaky pixel diffs don't block functional runs.

---

## 24. Network — Interception, Mocking, API Testing

### Concept

Playwright can **observe**, **modify**, **mock**, and **block** any network request the page makes. It can also make **direct API calls** without a browser. This makes tests fast, deterministic, and capable of testing edge cases that are hard to reproduce.

### 1. Listening to Requests & Responses

```ts
page.on('request',  req => console.log('>>', req.method(), req.url()));
page.on('response', res => console.log('<<', res.status(), res.url()));

// Filter to specific calls
page.on('response', async res => {
  if (res.url().includes('/api/users')) {
    console.log(await res.json());
  }
});
```

### 2. Wait for a Specific Request/Response

```ts
const responsePromise = page.waitForResponse('**/api/orders');
await page.getByRole('button', { name: 'Place order' }).click();
const response = await responsePromise;

expect(response.status()).toBe(201);
const body = await response.json();
expect(body.orderId).toBeTruthy();

// More complex matching
const r = await page.waitForResponse(
  res => res.url().includes('/api/users') && res.status() === 200
);
```

### 3. Mock an API Response (`page.route`)

The most useful network feature. Intercept a request and return a fake response:

```ts
await page.route('**/api/users', async route => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify([{ id: 1, name: 'Mock Alice' }]),
  });
});

await page.goto('/users');
await expect(page.getByText('Mock Alice')).toBeVisible();
```

### 4. Modify a Real Response

Let the request go to the real server, then tweak what comes back:

```ts
await page.route('**/api/products', async route => {
  const response = await route.fetch();
  const json = await response.json();
  json.push({ id: 999, name: 'Injected Product' });
  await route.fulfill({ response, json });
});
```

### 5. Block Resources (speed up tests, kill ads/analytics)

```ts
// Block all images
await page.route('**/*.{png,jpg,jpeg,svg,webp}', route => route.abort());

// Block analytics
await page.route('**/google-analytics.com/**', route => route.abort());
await page.route('**/segment.io/**', route => route.abort());
```

### 6. Modify a Request Before it Goes Out

```ts
await page.route('**/api/**', async route => {
  const headers = { ...route.request().headers(), 'x-test': 'true' };
  await route.continue({ headers });
});
```

### 7. Pure API Testing (no browser)

Use the `request` fixture to hit endpoints directly. Great for backend-only tests, or for setting up state before a UI test.

```ts
import { test, expect } from '@playwright/test';

test('POST /api/login returns token', async ({ request }) => {
  const res = await request.post('/api/login', {
    data: { email: 'a@b.com', password: 'pw' },
  });
  expect(res.ok()).toBeTruthy();
  const body = await res.json();
  expect(body.token).toBeTruthy();
});
```

### 8. The "API Setup, UI Assert" Pattern (powerful)

Use API to prepare state quickly, then validate via UI. Much faster than driving setup through the browser.

```ts
test('shows todo created via API', async ({ page, request }) => {
  // Setup via API — instant
  await request.post('/api/todos', { data: { text: 'buy milk' } });

  // Assert via UI
  await page.goto('/');
  await expect(page.getByText('buy milk')).toBeVisible();
});
```

### 9. Asserting on Request Payloads

```ts
const requests: any[] = [];
page.on('request', req => {
  if (req.url().includes('/api/track')) {
    requests.push(req.postDataJSON());
  }
});

await page.getByRole('button', { name: 'Subscribe' }).click();
await expect.poll(() => requests.length).toBeGreaterThan(0);
expect(requests[0]).toMatchObject({ event: 'subscribe' });
```

### 10. Offline / Network Throttling

```ts
// Offline
await context.setOffline(true);
await page.reload();
await expect(page.getByText('You are offline')).toBeVisible();
await context.setOffline(false);

// Throttle via CDP (Chromium only)
const client = await context.newCDPSession(page);
await client.send('Network.emulateNetworkConditions', {
  offline: false,
  latency: 400,
  downloadThroughput: (500 * 1024) / 8,   // 500 kbps
  uploadThroughput:   (500 * 1024) / 8,
});
```

### 11. HAR Replay (record once, replay forever)

Record real network traffic and replay it later without hitting the server:

```ts
// Record
await context.routeFromHAR('network.har', { update: true });

// Replay (after first run)
await context.routeFromHAR('network.har');
```

---

## 25. Advanced Tips & Best Practices

### Selector Strategy
- **Role-first**, then `data-testid`, then text, then CSS. Avoid XPath.
- Add stable `data-testid` to components that lack good labels.
- Don't reuse the same locator string in multiple files — wrap in POM.

### Speed
- Use **storage state** for auth instead of logging in each test.
- Use **API calls** for setup and teardown.
- **Block unnecessary resources** (images, analytics) when not testing them.
- Run **fully parallel** with multiple workers in CI.
- **Shard** across CI machines.

### Stability
- Always use **web-first assertions** (`await expect(locator).toX()`).
- Never use `page.waitForTimeout` in real tests.
- Avoid `'networkidle'` if your app polls.
- Use `trace: 'on-first-retry'` — small artifact, full debug info on flakes.
- For flaky tests, set `retries: 2` so flaky-but-passing tests aren't blockers, but the report still flags them.

### Test Architecture

```
project/
├── tests/                  # spec files
│   ├── login.spec.ts
│   └── checkout.spec.ts
├── pages/                  # Page Object Model
│   ├── LoginPage.ts
│   └── CheckoutPage.ts
├── fixtures/               # custom fixtures
│   └── index.ts
├── utils/                  # helpers
├── data/                   # test data
├── playwright/.auth/       # storage states (gitignored)
└── playwright.config.ts
```

### Tagging Tests for Selective Runs

```ts
test('checkout flow @smoke @critical', async ({ page }) => { /* ... */ });
```

```bash
npx playwright test -g "@smoke"
npx playwright test --grep-invert "@slow"
```

### Mocking the Clock

```ts
await page.clock.install({ time: new Date('2025-01-01T10:00:00Z') });
await page.goto('/scheduler');
await page.clock.fastForward('02:00');     // jump 2 hours
await page.clock.runFor(60_000);            // tick 60 seconds
```

### Geolocation, Timezone, Locale

```ts
test.use({
  geolocation: { latitude: 17.385, longitude: 78.486 },  // Hyderabad
  permissions: ['geolocation'],
  locale: 'en-IN',
  timezoneId: 'Asia/Kolkata',
});
```

### Final Checklist for Production Suites

- ✅ TypeScript with strict mode
- ✅ POM for every meaningful page/component
- ✅ Custom fixtures for repeatable setup
- ✅ Storage state for auth
- ✅ Parallel + sharded in CI
- ✅ HTML + JUnit reporters in CI
- ✅ Trace on first retry
- ✅ Stable role-based selectors
- ✅ No `waitForTimeout` in code
- ✅ API + UI hybrid tests where it makes sense
- ✅ Visual tests in their own project, run on Linux only

---

**End of notes.** Walk through each section in order, build a small project as you go, and the API will stick fast.
