# Playwright API Reference

## Overview

Playwright is a powerful browser automation library that enables reliable end-to-end testing and automation across Chromium, Firefox, and WebKit browsers. This API reference provides comprehensive documentation for Playwright's core functionality.

## Table of Contents

1. [Browser Automation](#browser-automation)
   - [Browsers](#browsers)
   - [Browser Contexts](#browser-contexts)
   - [Pages](#pages)
   - [Frames](#frames)
2. [Selectors and Actions](#selectors-and-actions)
   - [Locators](#locators)
   - [Element Handling](#element-handling)
   - [Input Actions](#input-actions)
3. [Test Runner](#test-runner)
   - [Test Fixtures](#test-fixtures)
   - [Assertions](#assertions)
   - [Configuration](#configuration)
4. [Component Testing](#component-testing)
   - [Framework-specific Mounting](#framework-specific-mounting)
   - [Component Interactions](#component-interactions)
5. [Utilities](#utilities)
   - [Network Mocking](#network-mocking)
   - [API Testing](#api-testing)
   - [File System Access](#file-system-access)
6. [Debugging and Tracing](#debugging-and-tracing)
   - [Trace Viewer](#trace-viewer)
   - [Screenshots and Videos](#screenshots-and-videos)

---

## Browser Automation

### Browsers

The entry point to Playwright automation is through browser objects, which represent browser instances.

#### `playwright.chromium`

```typescript
import { chromium } from 'playwright';

// Launch a browser
const browser = await chromium.launch({
  headless: false, // Set to true for headless mode
  slowMo: 100, // Slow down operations by 100ms
});

// Close browser when done
await browser.close();
```

#### `playwright.firefox`

```typescript
import { firefox } from 'playwright';

const browser = await firefox.launch();
await browser.close();
```

#### `playwright.webkit`

```typescript
import { webkit } from 'playwright';

const browser = await webkit.launch();
await browser.close();
```

#### Browser Launch Options

| Option | Type | Description |
|--------|------|-------------|
| `headless` | boolean | Whether to run browser in headless mode. Defaults to `true`. |
| `slowMo` | number | Slows down Playwright operations by the specified amount of milliseconds. |
| `executablePath` | string | Path to a browser executable to use instead of the bundled one. |
| `args` | string[] | Additional arguments to pass to the browser instance. |
| `ignoreDefaultArgs` | boolean or string[] | Whether to ignore default arguments or only specific ones. |
| `proxy` | Object | Network proxy settings. |
| `downloadsPath` | string | Path to save downloaded files. |
| `chromiumSandbox` | boolean | Enable Chromium sandboxing. Defaults to `true`. |

### Browser Contexts

Browser contexts provide isolated browser sessions within a browser instance.

```typescript
// Create a browser context
const context = await browser.newContext({
  viewport: { width: 1280, height: 720 },
  userAgent: 'My Custom User Agent',
  locale: 'en-US',
  timezoneId: 'America/Los_Angeles',
  permissions: ['geolocation'],
});

// Create multiple isolated browser contexts
const context1 = await browser.newContext();
const context2 = await browser.newContext();

// Close context when done
await context.close();
```

#### Context Options

| Option | Type | Description |
|--------|------|-------------|
| `viewport` | Object | Sets a viewport size. |
| `userAgent` | string | Specific user agent to use. |
| `bypassCSP` | boolean | Bypasses Content-Security-Policy. |
| `geolocation` | Object | Sets geolocation. |
| `locale` | string | Specify locale. |
| `permissions` | string[] | Specify permissions to grant. |
| `extraHTTPHeaders` | Object | Additional HTTP headers. |
| `offline` | boolean | Emulates network being offline. |
| `httpCredentials` | Object | Provides credentials for HTTP authentication. |
| `deviceScaleFactor` | number | Specify device scale factor. |
| `isMobile` | boolean | Whether the meta viewport tag is taken into account. |
| `hasTouch` | boolean | Specifies if viewport supports touch events. |
| `colorScheme` | string | Emulates 'light', 'dark' or 'no-preference' color scheme. |
| `reducedMotion` | string | Emulates 'reduce', 'no-preference' reduced motion preference. |
| `forcedColors` | string | Emulates 'active', 'none' forced colors. |

### Pages

Pages represent individual tabs or windows within a browser context.

```typescript
// Create a new page in a context
const page = await context.newPage();

// Navigate to a URL
await page.goto('https://example.com', {
  waitUntil: 'networkidle',
  timeout: 30000
});

// Get page title
const title = await page.title();

// Close page when done
await page.close();
```

#### Page Navigation

```typescript
// Navigate to a URL
await page.goto('https://example.com');

// Wait for navigation after clicking
await Promise.all([
  page.waitForNavigation(), // Wait for navigation to complete
  page.click('a.navigation-link') // Click that triggers navigation
]);

// Go back in history
await page.goBack();

// Go forward in history
await page.goForward();

// Reload the page
await page.reload();
```

### Frames

Frames represent frame elements within a page.

```typescript
// Get a frame by name
const frame = page.frame('frame-name');

// Get a frame using a selector
const frameElement = await page.locator('.frame-class').elementHandle();
const frame = await frameElement.contentFrame();

// Interact with elements inside a frame
await frame.fill('input#username', 'JohnDoe');
```

## Selectors and Actions

### Locators

Locators are the recommended way to find and interact with elements.

```typescript
// Create a locator
const button = page.locator('button.submit');

// Combine locators for more specific selection
const submitButton = page.locator('form').locator('button[type="submit"]');

// Use various selector engines
const element = page.locator('text=Click me'); // Text selector
const cssElement = page.locator('div.class'); // CSS selector
const xpathElement = page.locator('//button[@type="submit"]'); // XPath selector
const testIdElement = page.locator('data-testid=submit-button'); // TestID selector
```

#### Locator Methods

| Method | Description |
|--------|-------------|
| `locator.click()` | Click the element. |
| `locator.fill(value)` | Fill an input field with text. |
| `locator.check()` | Check a checkbox or radio button. |
| `locator.uncheck()` | Uncheck a checkbox. |
| `locator.selectOption(values)` | Select options in a `<select>` element. |
| `locator.hover()` | Hover over the element. |
| `locator.press(key)` | Press a key on the element. |
| `locator.count()` | Get the number of matching elements. |
| `locator.isVisible()` | Check if element is visible. |
| `locator.isEnabled()` | Check if element is enabled. |
| `locator.isChecked()` | Check if checkbox is checked. |
| `locator.textContent()` | Get text content of the element. |
| `locator.getAttribute(name)` | Get attribute value. |
| `locator.screenshot()` | Take a screenshot of the element. |

### Element Handling

```typescript
// Wait for element to be visible
await page.locator('.loading').waitFor({ state: 'visible' });

// Wait for element to be hidden
await page.locator('.loading').waitFor({ state: 'hidden' });

// Get element properties
const text = await page.locator('h1').textContent();
const value = await page.locator('input').inputValue();
const isChecked = await page.locator('input[type="checkbox"]').isChecked();
```

### Input Actions

```typescript
// Type into an input field
await page.locator('input#username').fill('JohnDoe');

// Press keys
await page.locator('input#username').press('Tab');

// Type with delay between keystrokes
await page.locator('textarea').type('Slow typing', { delay: 100 });

// Check/uncheck checkboxes
await page.locator('input[type="checkbox"]').check();
await page.locator('input[type="checkbox"]').uncheck();

// Select options from dropdown
await page.locator('select#country').selectOption('US');
await page.locator('select#colors').selectOption(['red', 'blue']); // Multiple options

// Upload files
await page.locator('input[type="file"]').setInputFiles('path/to/file.jpg');
```

## Test Runner

Playwright Test is a test runner specifically designed for end-to-end testing with Playwright.

### Basic Test Structure

```typescript
import { test, expect } from '@playwright/test';

test('basic test', async ({ page }) => {
  await page.goto('https://example.com');
  await expect(page).toHaveTitle(/Example/);
});
```

### Test Fixtures

Playwright Test uses fixtures to provide test functions with the required resources.

```typescript
import { test, expect } from '@playwright/test';

// Using built-in fixtures
test('using page fixture', async ({ page, context, browser }) => {
  // page, context, and browser are built-in fixtures
  await page.goto('https://example.com');
});

// Define custom fixtures
const test = base.extend({
  loggedInPage: async ({ page }, use) => {
    await page.goto('https://example.com/login');
    await page.fill('input#username', 'user');
    await page.fill('input#password', 'password');
    await page.click('button[type="submit"]');
    await page.waitForURL('https://example.com/dashboard');
    
    // Provide the fixture value to the test
    await use(page);
  },
});

// Use custom fixture
test('test with logged in user', async ({ loggedInPage }) => {
  await expect(loggedInPage.locator('.user-name')).toHaveText('user');
});
```

#### Built-in Fixtures

| Fixture | Description |
|---------|-------------|
| `page` | Isolated page for this test. |
| `context` | Isolated context for this test. |
| `browser` | Shared browser instance. |
| `browserName` | Name of the browser being used. |
| `headless` | Whether the browser is running in headless mode. |
| `isMobile` | Whether the browser is running with mobile emulation. |

### Assertions

Playwright Test includes built-in assertions powered by Expect.

```typescript
import { test, expect } from '@playwright/test';

test('assertions example', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Element assertions
  await expect(page.locator('h1')).toBeVisible();
  await expect(page.locator('button')).toBeEnabled();
  await expect(page.locator('.error')).toBeHidden();
  await expect(page.locator('input[type="checkbox"]')).toBeChecked();
  
  // Text assertions
  await expect(page.locator('.message')).toHaveText('Success');
  await expect(page.locator('.items')).toContainText('Item 1');
  
  // Value assertions
  await expect(page.locator('input')).toHaveValue('example');
  await expect(page.locator('select')).toHaveValue('option1');
  
  // Attribute assertions
  await expect(page.locator('a')).toHaveAttribute('href', 'https://example.com');
  await expect(page.locator('img')).toHaveAttribute('alt');
  
  // Page assertions
  await expect(page).toHaveTitle('Example Domain');
  await expect(page).toHaveURL('https://example.com/');
  
  // Screenshot assertions
  await expect(page).toHaveScreenshot('homepage.png');
  await expect(page.locator('.logo')).toHaveScreenshot('logo.png');
});
```

### Configuration

Playwright Test can be configured using the `playwright.config.ts` file.

```typescript
import { PlaywrightTestConfig } from '@playwright/test';

const config: PlaywrightTestConfig = {
  // Basic options
  testDir: './tests',
  timeout: 30000,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  
  // Reporter configuration
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results.json' }]
  ],
  
  // Browser configuration
  use: {
    headless: true,
    viewport: { width: 1280, height: 720 },
    ignoreHTTPSErrors: true,
    video: 'on-first-retry',
    screenshot: 'only-on-failure',
    trace: 'on',
  },
  
  // Configure projects for different browsers
  projects: [
    {
      name: 'chromium',
      use: { browserName: 'chromium' },
    },
    {
      name: 'firefox',
      use: { browserName: 'firefox' },
    },
    {
      name: 'webkit',
      use: { browserName: 'webkit' },
    },
    {
      name: 'Mobile Chrome',
      use: {
        browserName: 'chromium',
        ...devices['Pixel 5'],
      },
    },
    {
      name: 'Mobile Safari',
      use: {
        browserName: 'webkit',
        ...devices['iPhone 12'],
      },
    },
  ],
};

export default config;
```

## Component Testing

Playwright supports component testing for React, Vue, and Svelte components.

### Framework-specific Mounting

#### React Components

```typescript
// Using @playwright/experimental-ct-react
import { test, expect } from '@playwright/experimental-ct-react';
import Button from '../components/Button';

test('Button component test', async ({ mount }) => {
  // Mount the component
  const component = await mount(<Button label="Click me" onClick={() => {}} />);
  
  // Interact with the component
  await component.click();
  
  // Make assertions
  await expect(component).toContainText('Click me');
});
```

#### Vue Components

```typescript
// Using @playwright/experimental-ct-vue
import { test, expect } from '@playwright/experimental-ct-vue';
import Button from '../components/Button.vue';

test('Vue button test', async ({ mount }) => {
  const component = await mount(Button, {
    props: {
      label: 'Click me',
    },
  });
  
  await component.click();
  await expect(component).toContainText('Click me');
});
```

#### Svelte Components

```typescript
// Using @playwright/experimental-ct-svelte
import { test, expect } from '@playwright/experimental-ct-svelte';
import Button from '../components/Button.svelte';

test('Svelte button test', async ({ mount }) => {
  const component = await mount(Button, {
    props: {
      label: 'Click me',
    },
  });
  
  await component.click();
  await expect(component).toContainText('Click me');
});
```

### Component Interactions

```typescript
test('component interactions', async ({ mount }) => {
  const component = await mount(FormComponent);
  
  // Fill form fields
  await component.locator('input[name="username