# Playwright Configuration — Complete Reference

Everything that goes in `playwright.config.ts`, explained option by option. Covers timeouts, delays, retries, screenshots, video, trace, projects, and every other setting you're likely to touch.

---

## Table of Contents

1. [The Full Config File (annotated)](#1-the-full-config-file-annotated)
2. [Top-Level Options](#2-top-level-options)
3. [Timeouts & Delays](#3-timeouts--delays)
4. [Retries](#4-retries)
5. [Screenshots](#5-screenshots)
6. [Video](#6-video)
7. [Trace](#7-trace)
8. [The `use` Block (every option)](#8-the-use-block-every-option)
9. [Projects](#9-projects)
10. [Reporters](#10-reporters)
11. [Web Server](#11-web-server)
12. [Expect (Assertion) Config](#12-expect-assertion-config)
13. [Global Setup & Teardown](#13-global-setup--teardown)
14. [Overriding Config Per-Test](#14-overriding-config-per-test)
15. [Environment-Based Config](#15-environment-based-config)
16. [Quick Reference Table](#16-quick-reference-table)

---

## 1. The Full Config File (annotated)

This is a complete, production-ready config with **every common option** and a comment on each line. The sections below explain each group in detail.

```ts
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  // ───────── WHERE TESTS LIVE ─────────
  testDir: './tests',                  // folder Playwright scans for tests
  testMatch: '**/*.spec.ts',           // file pattern that counts as a test
  testIgnore: '**/helpers/**',         // files to skip

  // ───────── PARALLELISM ─────────
  fullyParallel: true,                 // run every test in parallel, even within a file
  workers: process.env.CI ? 4 : undefined, // number of parallel worker processes
                                            // undefined = Playwright picks (≈ half CPU cores)

  // ───────── SAFETY / CI ─────────
  forbidOnly: !!process.env.CI,        // fail the build if a stray test.only() is committed

  // ───────── RETRIES ─────────
  retries: process.env.CI ? 2 : 0,     // retry failed tests up to N times

  // ───────── TIMEOUTS ─────────
  timeout: 30_000,                     // max time per TEST (ms)
  globalTimeout: 60 * 60_000,          // max time for the WHOLE run (ms)

  // ───────── ASSERTION CONFIG ─────────
  expect: {
    timeout: 5_000,                    // max time for each expect() to retry
  },

  // ───────── REPORTERS ─────────
  reporter: [
    ['html', { open: 'never' }],       // rich HTML report
    ['list'],                          // live terminal output
    ['junit', { outputFile: 'results.xml' }], // for CI dashboards
  ],

  // ───────── SHARED DEFAULTS FOR EVERY TEST ─────────
  use: {
    baseURL: 'http://localhost:3000',  // lets you write page.goto('/login')

    // Timeouts that live inside `use`
    actionTimeout: 10_000,             // max time for a single action (click, fill, ...)
    navigationTimeout: 30_000,         // max time for page navigation

    // Debug artifacts
    screenshot: 'only-on-failure',     // 'on' | 'off' | 'only-on-failure'
    video: 'retain-on-failure',        // 'on' | 'off' | 'retain-on-failure' | 'on-first-retry'
    trace: 'on-first-retry',           // 'on' | 'off' | 'retain-on-failure' | 'on-first-retry'

    // Browser behaviour
    headless: true,
    viewport: { width: 1280, height: 720 },
    ignoreHTTPSErrors: true,
    locale: 'en-US',
    timezoneId: 'Asia/Kolkata',
    colorScheme: 'light',              // 'light' | 'dark' | 'no-preference'

    // Slow everything down (debugging only)
    launchOptions: {
      slowMo: 0,                       // ms of delay added before each action
    },
  },

  // ───────── BROWSERS / DEVICES ─────────
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox',  use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit',   use: { ...devices['Desktop Safari'] } },
    { name: 'mobile',   use: { ...devices['iPhone 13'] } },
  ],

  // ───────── DEV SERVER ─────────
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,                  // max time to wait for the server to boot
  },

  // ───────── GLOBAL SETUP / TEARDOWN ─────────
  globalSetup: './global-setup.ts',
  globalTeardown: './global-teardown.ts',

  // ───────── OUTPUT ─────────
  outputDir: './test-results',         // where screenshots/videos/traces are written
});
```

---

## 2. Top-Level Options

These sit at the root of `defineConfig({ ... })`.

| Option | Type | What it does |
|---|---|---|
| `testDir` | string | Folder Playwright scans for test files. |
| `testMatch` | string / RegExp / array | Pattern for files that are treated as tests. |
| `testIgnore` | string / RegExp / array | Files to exclude. |
| `fullyParallel` | boolean | If true, tests **within a file** also run in parallel (not just across files). |
| `workers` | number / string | How many parallel worker processes. `'50%'` = half the CPU cores. |
| `forbidOnly` | boolean | Fails the run if any `test.only()` is left in the code — set true in CI. |
| `outputDir` | string | Where artifacts (screenshots, videos, traces) are saved. |
| `globalTimeout` | number (ms) | Hard cap on the entire test run. |
| `maxFailures` | number | Stop the run after N failures (fail-fast). |
| `quiet` | boolean | Suppress stdout/stderr from tests. |

```ts
export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  workers: '50%',          // use half of available CPU cores
  forbidOnly: !!process.env.CI,
  maxFailures: process.env.CI ? 10 : undefined,  // bail out early in CI
  outputDir: './test-results',
});
```

---

## 3. Timeouts & Delays

This is the part people get confused about — Playwright has **several different timeouts**, each controlling a different thing.

### The timeout hierarchy

```
globalTimeout      → the whole test run (all tests combined)
  └── timeout      → one single test
        ├── actionTimeout       → one action (click, fill, check...)
        ├── navigationTimeout   → one navigation (goto, reload...)
        └── expect.timeout      → one assertion's auto-retry window
```

### Test timeout (`timeout`)

Maximum time a **single test** is allowed to run. Default is 30 seconds.

```ts
export default defineConfig({
  timeout: 30_000,   // 30 seconds per test
});
```

### Global timeout (`globalTimeout`)

Hard cap for the **entire run**. Useful in CI so a hung suite doesn't run forever. Off by default.

```ts
export default defineConfig({
  globalTimeout: 60 * 60_000,   // 1 hour for the whole suite
});
```

### Action timeout (`actionTimeout`)

Max time for **one action** like `click()`, `fill()`, `check()`. Lives inside `use`. Default is 0 (no limit — falls back to the test timeout).

```ts
use: {
  actionTimeout: 10_000,   // each click/fill must complete within 10s
}
```

### Navigation timeout (`navigationTimeout`)

Max time for **navigations** like `goto()`, `reload()`, `goBack()`. Lives inside `use`.

```ts
use: {
  navigationTimeout: 30_000,   // each page navigation must finish within 30s
}
```

### Expect timeout (`expect.timeout`)

How long a web-first assertion (`await expect(locator).toBeVisible()`) keeps **auto-retrying** before failing. Default is 5 seconds.

```ts
expect: {
  timeout: 5_000,
}
```

### Delay between actions (`slowMo`)

`slowMo` adds an artificial delay (in ms) **before every action**. This is a **debugging aid only** — it lets you watch what's happening. Never use it in CI.

```ts
use: {
  launchOptions: {
    slowMo: 500,   // pause 500ms before each action so you can see it
  },
}
```

### Per-test / per-action timeout overrides

You can override any timeout inside a test instead of globally:

```ts
// Override the whole test's timeout
test('slow flow', async ({ page }) => {
  test.setTimeout(60_000);   // this test gets 60s
  // ...
});

// Triple the timeout for a known-slow test
test('heavy report', async ({ page }) => {
  test.slow();   // 3x the configured timeout
});

// Override timeout on a single action
await page.getByRole('button').click({ timeout: 15_000 });

// Override timeout on a single assertion
await expect(page.getByText('Done')).toBeVisible({ timeout: 20_000 });

// Disable timeout entirely for one test (use with care)
test('manual debugging', async ({ page }) => {
  test.setTimeout(0);
});
```

---

## 4. Retries

A **retry** re-runs a failed test from scratch. This smooths over flaky tests without hiding them — a test that fails then passes is reported as **"flaky"**, not "passed."

```ts
export default defineConfig({
  retries: process.env.CI ? 2 : 0,   // 2 retries in CI, none locally
});
```

### Why `0` locally and `2` in CI?

- **Locally** you want failures to surface immediately so you can fix them.
- **In CI** transient issues (network blips, slow runners) shouldn't block a merge, but you still want the report to flag flakiness.

### Detecting flaky tests

After a run with retries enabled, the HTML report has a **"Flaky"** category for tests that needed a retry to pass. Treat these as bugs to fix, not as green.

### Per-test retry override

```ts
test.describe('flaky external integration', () => {
  test.describe.configure({ retries: 3 });   // these tests retry 3x

  test('payment webhook', async ({ page }) => { /* ... */ });
});
```

### Pairing retries with trace

The smartest combo — capture a trace **only when a test is retried**, so you get full debug info on flakes without bloating every run:

```ts
use: {
  trace: 'on-first-retry',
}
```

---

## 5. Screenshots

Screenshots can be captured automatically on failure, always, or never.

```ts
use: {
  screenshot: 'only-on-failure',   // 'on' | 'off' | 'only-on-failure'
}
```

| Value | Behaviour |
|---|---|
| `'off'` | Never capture (default). |
| `'on'` | Capture after every test. |
| `'only-on-failure'` | Capture only when a test fails — the usual choice. |

### Full-page and options form

You can pass an object for finer control:

```ts
use: {
  screenshot: {
    mode: 'only-on-failure',
    fullPage: true,              // capture the entire scrollable page
    omitBackground: false,
  },
}
```

### Manual screenshots inside a test

Config-level screenshots are automatic on failure; you can also take them yourself:

```ts
// Whole page
await page.screenshot({ path: 'home.png', fullPage: true });

// A single element
await page.locator('#chart').screenshot({ path: 'chart.png' });

// Hide dynamic regions (timestamps, ads) before capturing
await page.screenshot({
  path: 'home.png',
  mask: [page.locator('.timestamp')],
});
```

---

## 6. Video

Records a video of the browser for each test.

```ts
use: {
  video: 'retain-on-failure',   // 'on' | 'off' | 'retain-on-failure' | 'on-first-retry'
}
```

| Value | Behaviour |
|---|---|
| `'off'` | Never record (default). |
| `'on'` | Record every test (large artifacts). |
| `'retain-on-failure'` | Record everything but **keep only failures' videos** — the usual choice. |
| `'on-first-retry'` | Record only when a test is retried. |

### Options form (set video size)

```ts
use: {
  video: {
    mode: 'retain-on-failure',
    size: { width: 1280, height: 720 },
  },
}
```

Videos are saved to `outputDir` and linked in the HTML report.

---

## 7. Trace

A **trace** is the most powerful debug artifact: it records a full timeline you can replay in the Trace Viewer — DOM snapshots before/after each action, network calls, console logs, and screenshots.

```ts
use: {
  trace: 'on-first-retry',   // 'on' | 'off' | 'retain-on-failure' | 'on-first-retry'
}
```

| Value | Behaviour |
|---|---|
| `'off'` | Never trace (default). |
| `'on'` | Trace every test (heavy — avoid in CI). |
| `'retain-on-failure'` | Trace everything, keep only failures. |
| `'on-first-retry'` | Trace only on the first retry — **best balance** of cost vs insight. |

### Options form

```ts
use: {
  trace: {
    mode: 'on-first-retry',
    screenshots: true,
    snapshots: true,
    sources: true,        // include test source code in the trace
  },
}
```

### Viewing a trace

```bash
npx playwright show-trace trace.zip
```

Or just open the HTML report after a failed run and click the trace icon on the failed test.

---

## 8. The `use` Block (every option)

The `use` block sets **defaults for every test**. Any of these can be overridden per-project or per-test.

```ts
use: {
  // ── Base URL ──
  baseURL: 'https://example.com',     // page.goto('/login') resolves against this

  // ── Timeouts ──
  actionTimeout: 10_000,
  navigationTimeout: 30_000,

  // ── Artifacts ──
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
  trace: 'on-first-retry',

  // ── Browser launch ──
  headless: true,
  launchOptions: {
    slowMo: 0,                        // delay before each action (debug)
    args: ['--start-maximized'],      // raw browser flags
  },

  // ── Viewport & device ──
  viewport: { width: 1280, height: 720 },
  deviceScaleFactor: 1,
  isMobile: false,
  hasTouch: false,

  // ── Locale / region ──
  locale: 'en-IN',
  timezoneId: 'Asia/Kolkata',
  geolocation: { latitude: 17.385, longitude: 78.486 },  // Hyderabad
  permissions: ['geolocation', 'notifications'],

  // ── Appearance ──
  colorScheme: 'dark',                // 'light' | 'dark' | 'no-preference'

  // ── Network ──
  ignoreHTTPSErrors: true,
  offline: false,
  extraHTTPHeaders: {
    'x-test-run': 'true',
  },
  httpCredentials: {                  // HTTP basic auth
    username: 'user',
    password: 'pass',
  },
  proxy: { server: 'http://myproxy:3128' },

  // ── Auth state ──
  storageState: 'playwright/.auth/user.json',   // pre-logged-in session

  // ── Misc ──
  bypassCSP: false,                   // bypass Content-Security-Policy
  javaScriptEnabled: true,
  acceptDownloads: true,
  userAgent: 'my-custom-agent/1.0',
  testIdAttribute: 'data-testid',     // change which attribute getByTestId uses
}
```

### Notable options explained

- **`baseURL`** — relative paths in `page.goto()` resolve against this. Essential for switching between local/staging/prod.
- **`storageState`** — points to a saved auth session so tests start logged in. (See the auth setup pattern.)
- **`testIdAttribute`** — if your app uses `data-test` instead of `data-testid`, set it here so `getByTestId()` works.
- **`extraHTTPHeaders`** — inject headers on every request, e.g. a test bypass token.
- **`httpCredentials`** — fills HTTP Basic Auth dialogs automatically.

---

## 9. Projects

A **project** is a named configuration. The same tests run once per project. Use them for multiple browsers, viewports, or auth roles. Each project can override the `use` block.

### Multiple browsers

```ts
projects: [
  { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  { name: 'firefox',  use: { ...devices['Desktop Firefox'] } },
  { name: 'webkit',   use: { ...devices['Desktop Safari'] } },
  { name: 'Mobile Chrome', use: { ...devices['Pixel 5'] } },
  { name: 'Mobile Safari', use: { ...devices['iPhone 13'] } },
],
```

Run one project:

```bash
npx playwright test --project=chromium
```

### Auth-setup dependency (run login once, reuse everywhere)

```ts
projects: [
  // 1. A setup project that logs in and saves storage state
  { name: 'setup', testMatch: /.*\.setup\.ts/ },

  // 2. Real test projects that depend on setup and load the saved state
  {
    name: 'chromium',
    use: {
      ...devices['Desktop Chrome'],
      storageState: 'playwright/.auth/user.json',
    },
    dependencies: ['setup'],   // setup runs first
  },
],
```

### Different settings per project

```ts
projects: [
  {
    name: 'desktop',
    use: { viewport: { width: 1920, height: 1080 } },
  },
  {
    name: 'mobile',
    use: { ...devices['iPhone 13'], video: 'on' },  // override video just for mobile
  },
  {
    name: 'visual',
    testMatch: /.*\.visual\.spec\.ts/,              // only visual tests
    use: { ...devices['Desktop Chrome'] },
  },
],
```

---

## 10. Reporters

Reporters control how results are displayed. You can run several at once.

```ts
reporter: [
  ['list'],                                  // live terminal output (default)
  ['html', { open: 'never' }],               // rich self-contained HTML report
  ['json', { outputFile: 'results.json' }],  // machine-readable
  ['junit', { outputFile: 'results.xml' }],  // CI dashboards (Jenkins, GitLab)
  ['github'],                                // GitHub Actions inline annotations
],
```

| Reporter | Use |
|---|---|
| `list` | Detailed per-test lines in the terminal. |
| `line` | Compact single updating line. |
| `dot` | Minimal — one char per test. |
| `html` | Full interactive report with traces/screenshots. |
| `json` | For custom processing. |
| `junit` | XML for CI test dashboards. |
| `github` | Annotations on GitHub PRs. |

```bash
# Open the HTML report after a run
npx playwright show-report
```

---

## 11. Web Server

Playwright can **start your app automatically** before the tests and shut it down after. No more "start the dev server first."

```ts
webServer: {
  command: 'npm run dev',              // command to launch your app
  url: 'http://localhost:3000',        // Playwright waits until this responds
  reuseExistingServer: !process.env.CI, // locally reuse a running server; in CI always start fresh
  timeout: 120_000,                    // max time to wait for the server to boot
  stdout: 'pipe',
  stderr: 'pipe',
},
```

### Multiple servers (e.g. frontend + backend)

```ts
webServer: [
  {
    command: 'npm run start:api',
    url: 'http://localhost:4000/health',
    reuseExistingServer: !process.env.CI,
  },
  {
    command: 'npm run start:web',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
],
```

---

## 12. Expect (Assertion) Config

Configure how web-first assertions behave globally.

```ts
expect: {
  timeout: 5_000,            // how long each expect() auto-retries

  toHaveScreenshot: {        // visual comparison thresholds
    maxDiffPixels: 100,
    maxDiffPixelRatio: 0.01,
    threshold: 0.2,          // per-pixel color sensitivity (0–1)
    animations: 'disabled',  // freeze animations before snapshot
  },

  toMatchSnapshot: {
    maxDiffPixelRatio: 0.01,
  },
},
```

- **`timeout`** — the retry window for assertions like `toBeVisible()`. Separate from action/test timeouts.
- **`toHaveScreenshot`** — global tolerances for visual regression so you don't repeat them in every test.

---

## 13. Global Setup & Teardown

Run code **once** before all tests start and once after they finish — for seeding a database, starting a service, or cleaning up.

```ts
export default defineConfig({
  globalSetup: './global-setup.ts',
  globalTeardown: './global-teardown.ts',
});
```

```ts
// global-setup.ts
import { FullConfig } from '@playwright/test';

async function globalSetup(config: FullConfig) {
  // e.g. seed the database, create test users, fetch an auth token
  console.log('Running once before all tests');
}

export default globalSetup;
```

```ts
// global-teardown.ts
async function globalTeardown() {
  console.log('Running once after all tests');
  // e.g. wipe the database, delete test users
}

export default globalTeardown;
```

> **Note:** for auth, the **project dependency** pattern (section 9) is usually preferred over `globalSetup`, because it integrates with storage state and the HTML report.

---

## 14. Overriding Config Per-Test

Most config can be overridden at the file or describe-block level with `test.use()`.

```ts
import { test, expect } from '@playwright/test';

// Override for an entire file
test.use({
  viewport: { width: 375, height: 667 },   // mobile viewport
  locale: 'fr-FR',
  colorScheme: 'dark',
});

test('renders in French dark mode', async ({ page }) => {
  await page.goto('/');
  // ...
});
```

```ts
// Override for just one describe block
test.describe('mobile checkout', () => {
  test.use({ ...devices['iPhone 13'] });

  test('shows mobile cart', async ({ page }) => { /* ... */ });
});
```

```ts
// Per-test timeout / retries / mode
test.describe('slow suite', () => {
  test.describe.configure({ timeout: 60_000, retries: 1, mode: 'serial' });
  // ...
});
```

---

## 15. Environment-Based Config

A common real-world pattern: drive the config off environment variables so the same suite runs against local, staging, and prod.

```ts
import { defineConfig, devices } from '@playwright/test';

const ENV = process.env.TEST_ENV || 'local';

const baseURLs: Record<string, string> = {
  local:   'http://localhost:3000',
  staging: 'https://staging.example.com',
  prod:    'https://example.com',
};

export default defineConfig({
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 4 : undefined,
  timeout: 30_000,

  use: {
    baseURL: baseURLs[ENV],
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    headless: !!process.env.CI,        // headed locally, headless in CI
  },

  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
});
```

Run it:

```bash
TEST_ENV=staging npx playwright test
CI=true npx playwright test
```

---

## 16. Quick Reference Table

| What you want | Option | Where | Typical value |
|---|---|---|---|
| Time limit per test | `timeout` | top-level | `30_000` |
| Time limit for whole run | `globalTimeout` | top-level | `3600_000` |
| Time limit per click/fill | `actionTimeout` | `use` | `10_000` |
| Time limit per navigation | `navigationTimeout` | `use` | `30_000` |
| Time limit per assertion | `expect.timeout` | `expect` | `5_000` |
| Delay before each action (debug) | `slowMo` | `use.launchOptions` | `0` |
| Retry failed tests | `retries` | top-level | `2` in CI |
| Screenshot on failure | `screenshot` | `use` | `'only-on-failure'` |
| Video on failure | `video` | `use` | `'retain-on-failure'` |
| Trace on retry | `trace` | `use` | `'on-first-retry'` |
| Base URL for `goto('/x')` | `baseURL` | `use` | env-driven |
| Parallel within files | `fullyParallel` | top-level | `true` |
| Worker count | `workers` | top-level | `4` / `'50%'` |
| Block stray `test.only` | `forbidOnly` | top-level | `!!CI` |
| Pre-logged-in session | `storageState` | `use` | `'.auth/user.json'` |
| Auto-start dev server | `webServer` | top-level | object |
| Run on multiple browsers | `projects` | top-level | array |
| Where artifacts are saved | `outputDir` | top-level | `'./test-results'` |
| Stop after N failures | `maxFailures` | top-level | `10` in CI |

---

**Tip:** start from the annotated config in section 1, delete what you don't need, and drive the volatile values (base URL, headless, retries) off environment variables as shown in section 15.
