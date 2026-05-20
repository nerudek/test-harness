---
name: test-harness
description: Automated test suite runner with coverage, mutation testing, and multi-model validation — catch regressions before they ship
version: 1.0.0
author: nerudek
compatible-with: hermes-agent, claude-code, openclaw
---

# Test Harness — Automated Quality Gate

## Problem

You push a "small fix" and break three other features. Manual testing takes 30 minutes and you still miss edge cases. AI-generated code has no tests and you have no way to validate it across multiple models. Regressions slip through because there is no automated safety net.

## Solution

A comprehensive test harness that runs automatically on every commit: static analysis, unit tests, integration tests, mutation testing, and multi-model validation for AI-generated outputs. Fails fast on any regression.

## Quick Start

```bash
# Install
npm install --save-dev vitest @testing-library/react happy-dom

# Add to package.json
"scripts": {
  "test": "vitest run",
  "test:watch": "vitest",
  "test:coverage": "vitest run --coverage",
  "test:mutation": "stryker run"
}
```

## Test Types

### 1. Static Analysis (fails instantly, zero setup)

```bash
npm run lint          # ESLint
npx tsc --noEmit      # TypeScript type check
npx prettier --check . # Format check
```

### 2. Unit Tests (Vitest)

```javascript
// sum.test.ts
import { describe, it, expect } from 'vitest';
import { sum } from './sum';

describe('sum', () => {
  it('adds positive numbers', () => { expect(sum(2, 3)).toBe(5); });
  it('handles zero', () => { expect(sum(0, 5)).toBe(5); });
  it('handles negatives', () => { expect(sum(-2, -3)).toBe(-5); });
  it('throws on non-number', () => { expect(() => sum('a', 3)).toThrow(); });
});
```

### 3. Integration Tests (Playwright)

```javascript
// e2e.spec.ts
import { test, expect } from '@playwright/test';
test('homepage loads and navigates', async ({ page }) => {
  await page.goto('http://localhost:5173');
  await expect(page.locator('h1')).toHaveText('Banner Editor');
  await page.click('button:has-text("Wczytaj")');
});
```

### 4. Mutation Testing (Stryker)

```bash
npx stryker run
# Reports: "3 mutants survived out of 45" → your tests miss edge cases
```

### 5. AI Output Validation

```javascript
// validate-ai-output.test.ts
const MODELS = ['deepseek-v4-pro', 'qwen-27b', 'qwen-35b'];

describe.each(MODELS)('Model: %s', (model) => {
  it('returns valid JSON', async () => {
    const out = await queryModel(model, 'Return {"key":"value"} as JSON');
    expect(() => JSON.parse(out)).not.toThrow();
  });
  
  it('responds in correct language', async () => {
    const out = await queryModel(model, 'Odpowiedz po polsku: co to jest DOM?');
    expect(out.toLowerCase()).toMatch(/html|dokument|przeglądark/i);
  });

  it('completes within timeout', async () => {
    await expect(queryModel(model, 'Count from 1 to 10'))
      .resolves.toBeDefined();
  }, 30000);
});
```

## GitHub Actions Integration

```yaml
name: Quality Gate
on: [push, pull_request]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run lint
      - run: npx tsc --noEmit
      - run: npm test -- --coverage
      - name: Check coverage threshold
        run: npx istanbul check-coverage --lines 80 --functions 80 --branches 70
      - run: npx playwright test
```

## Coverage Thresholds

```json
// vitest.config.ts
{
  coverage: {
    thresholds: {
      lines: 80,
      functions: 80,
      branches: 70,
      statements: 80
    }
  }
}
```

## Common Pitfalls

1. Tests that pass in isolation but fail in CI — use `--runInBand` for sequential execution
2. Flaky E2E tests due to timing — use `waitForSelector` instead of `sleep`
3. Mutation testing on untested code — run coverage first, target only covered lines
4. Model validation timeout — set realistic timeouts (30-60s for LLM calls)
5. Coverage ignores generated files — exclude `*.gen.ts`, `dist/`, `node_modules/`

## FAQ

**Q: How many tests are enough?**
Coverage is a proxy, not a goal. Focus on critical paths: auth, payments, data mutations. 80% line coverage is a good start.

**Q: What is mutation testing and why do I need it?**
It changes small parts of your code (e.g., `>` to `<`) and checks if tests catch it. If a mutant survives, your tests have gaps.

**Q: Can I test AI agent behavior?**
Yes — define contracts: expected response shape, required fields, language, sentiment. Validate with regex, JSON schema, or assertions.

**Q: How do I test CLI tools?**
Use `execa` or `child_process.execSync`. Capture stdout/stderr and assert on exit code and output content.

**Q: E2E tests are too slow — alternatives?**
Run critical path E2E only. Use component tests (Cypress CT, Playwright CT) for UI. Mock API calls with MSW.

**Q: How to run tests in parallel?**
Vitest runs in parallel by default. For E2E, Playwright can shard across workers: `npx playwright test --shard=1/3`.

**Q: What about snapshot tests?**
Use sparingly. Snapshots break on any markup change, creating noise. Good for serialized data outputs, bad for React components (use visual regression instead).

**Q: How do I test WebSocket/realtime connections?**
Use `ws` library in Node, or Playwright's `page.waitForWebSocket()`. Mock the server side for unit tests.

**Q: Can I test across multiple Node versions?**
Yes — use matrix strategy in CI: `node-version: [18, 20, 22]`.

**Q: How to handle flaky tests?**
Retry: `vitest --retry=3`. Investigate root cause (timing, state leak). Never just increase retry count — flaky tests erode trust.

**Q: What about testing with real databases?**
Use testcontainers: `pg-test` or `testcontainers-node` to spin up real PostgreSQL in Docker for integration tests.

**Q: How to test error handling paths?**
Force errors: mock API to return 500, pass invalid data, trigger timeout. Test both happy path AND failure modes.

**Q: Can I skip tests temporarily?**
Use `it.skip()` or `describe.skip()` with a comment explaining why. Remove skips before merging — dead skipped tests are technical debt.

**Q: What about property-based testing?**
Use `fast-check` library. Instead of hand-written inputs, it generates random values and verifies invariants. Catches edge cases you never thought of.

**Q: How to visualize test results?**
Vitest UI (`vitest --ui`), Playwright HTML report, Coverage HTML report. All auto-generated in CI artifacts.

---

If this saved you time: [PayPal.me/nerudek](https://www.paypal.me/nerudek)
GitHub: [github.com/nerudek](https://github.com/nerudek)
