# Agent Runbook — Automated Testing (Playwright)

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to configure and run automated End-to-End (E2E) testing using Playwright.

---

## 🎯 What You Get
After completing this setup, you gain the ability to write and execute robust browser-based tests.
- **Automated Execution**: Run tests headlessly in CI or during development.
- **Agent Integration**: Pairs cleanly with Chrome DevTools for deeper inspection.
- **Cross-Browser**: Tests can run against Chromium, Firefox, and WebKit.

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. When tasked with setting up or running tests, execute the phases below in sequence.

---

## 🔍 Phase 1: Environment Discovery
Before applying configurations, run diagnostic checks to identify existing configurations.

1. **Verify if Node.js is installed**:
   - Command: `node -v`
   - Expect: `v18.x.x` or higher.
2. **Verify Playwright presence**:
   - Command: `npx playwright --version`
   - Expect: Playwright CLI version output. If not found, it will be installed in Phase 3.

---

## ⚙️ Phase 2: Configuration Injection

Inject a standardized `playwright.config.ts` if one does not exist at the project root.

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
});
```

---

## 🚀 Phase 3: Background Service Automation

1. **Install Playwright and Browsers**:
   - Command: `npm init playwright@latest -y` (if starting fresh) or `npx playwright install --with-deps` (if adding to an existing project).
2. **Create Dummy Test**:
   - Command: `mkdir -p tests && echo "import { test, expect } from '@playwright/test'; test('basic test', async ({ page }) => { await page.goto('https://playwright.dev/'); await expect(page).toHaveTitle(/Playwright/); });" > tests/example.spec.ts`

---

## 🩺 Phase 4: Health Check & Verification

1. **Run the Test Suite**:
   - Command: `npx playwright test`
   - Expect: A summary showing 1 passed test.
2. **Verify HTML Report Generation**:
   - Command: `ls playwright-report/index.html`
   - Expect: The file exists.

---

## 🛠️ Phase 5: Automated Troubleshooting

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **Missing System Dependencies (Linux)** | Playwright browsers require OS-level shared libraries (e.g., `libgbm1`). | 1. Run `npx playwright install-deps`. (Requires sudo permissions; prompt user if needed). |
| **Test Timeout** | Network issues or the page load took too long. | 1. Increase timeout in `playwright.config.ts`. <br>2. Re-run tests with `DEBUG=pw:api npx playwright test` for verbose logging. |

---

## 🚫 Out of Scope
- Visual regression testing setups (requires explicit baseline management).
- Mobile device emulation configurations (handled per-project).
