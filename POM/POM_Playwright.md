# Enterprise-Grade Playwright TypeScript Framework
## Web UI + Mobile App (Appium) + API Automation

For an **enterprise-grade Playwright TypeScript framework** supporting **Web UI + Mobile App (Appium) + API Automation**, use a **Layered Page Object Model (POM)** architecture with reusable components, fixtures, utilities, test data, and reporting.

---

# Recommended Project Structure

```text
playwright-framework/
│
├── package.json
├── playwright.config.ts
├── tsconfig.json
│
├── src
│   │
│   ├── pages                      # Web POM
│   │   ├── BasePage.ts
│   │   ├── LoginPage.ts
│   │   ├── HomePage.ts
│   │   └── ProfilePage.ts
│   │
│   ├── mobile                     # Mobile Appium POM
│   │   ├── BaseMobilePage.ts
│   │   ├── MobileLoginPage.ts
│   │   ├── MobileHomePage.ts
│   │   └── MobileProfilePage.ts
│   │
│   ├── api                        # API Object Model (AOM)
│   │   ├── BaseApi.ts
│   │   ├── UserApi.ts
│   │   ├── PolicyApi.ts
│   │   └── ClaimsApi.ts
│   │
│   ├── components                 # Common UI Components
│   │   ├── Header.ts
│   │   ├── Menu.ts
│   │   └── Footer.ts
│   │
│   ├── fixtures
│   │   ├── webFixture.ts
│   │   ├── mobileFixture.ts
│   │   └── apiFixture.ts
│   │
│   ├── data
│   │   ├── testData.json
│   │   └── users.json
│   │
│   ├── models
│   │   ├── User.ts
│   │   ├── Policy.ts
│   │   └── Claim.ts
│   │
│   ├── utils
│   │   ├── ConfigReader.ts
│   │   ├── Logger.ts
│   │   ├── FakerUtil.ts
│   │   ├── WaitUtil.ts
│   │   └── JsonUtil.ts
│   │
│   └── constants
│       ├── URLs.ts
│       └── Endpoints.ts
│
├── tests
│   ├── web
│   │   ├── login.spec.ts
│   │   └── profile.spec.ts
│   │
│   ├── mobile
│   │   ├── mobileLogin.spec.ts
│   │   └── mobileProfile.spec.ts
│   │
│   └── api
│       ├── userApi.spec.ts
│       └── claimsApi.spec.ts
│
├── reports
├── screenshots
└── logs
```

---

# 1. Base Page (Web)

```typescript
import { Page } from '@playwright/test';

export class BasePage {
  protected page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  async open(url: string) {
    await this.page.goto(url);
  }

  async getTitle() {
    return await this.page.title();
  }

  async waitForPageLoad() {
    await this.page.waitForLoadState('networkidle');
  }

  async click(locator: string) {
    await this.page.locator(locator).click();
  }

  async type(locator: string, value: string) {
    await this.page.locator(locator).fill(value);
  }
}
```

---

# 2. Login Page

```typescript
import { Page } from '@playwright/test';
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  txtUser = '#username';
  txtPassword = '#password';
  btnLogin = '#loginBtn';

  constructor(page: Page) {
    super(page);
  }

  async login(user: string, password: string) {
    await this.page.locator(this.txtUser).fill(user);
    await this.page.locator(this.txtPassword).fill(password);
    await this.page.locator(this.btnLogin).click();
  }
}
```

---

# 3. Home Page

```typescript
import { Page } from '@playwright/test';
import { BasePage } from './BasePage';

export class HomePage extends BasePage {
  lblWelcome = '.welcome';

  constructor(page: Page) {
    super(page);
  }

  async getWelcomeMessage() {
    return await this.page.locator(this.lblWelcome).textContent();
  }
}
```

---

# Web Test

```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from '../../src/pages/LoginPage';
import { HomePage } from '../../src/pages/HomePage';

test('Login Test', async ({ page }) => {
  const loginPage = new LoginPage(page);
  const homePage = new HomePage(page);

  await loginPage.open('https://demoapp.com');

  await loginPage.login(
    'admin',
    'password'
  );

  expect(
    await homePage.getWelcomeMessage()
  ).toContain('Welcome');
});
```

---

# Mobile Page Object Model (Appium + Playwright)

Although Playwright cannot directly automate native mobile apps, it can work alongside **Appium**.

## Base Mobile Page

```typescript
import { Browser } from 'webdriverio';

export class BaseMobilePage {
  protected driver: Browser;

  constructor(driver: Browser) {
    this.driver = driver;
  }

  async click(locator: string) {
    await this.driver.$(locator).click();
  }

  async type(locator: string, value: string) {
    await this.driver.$(locator).setValue(value);
  }
}
```

---

## Mobile Login Page

```typescript
export class MobileLoginPage extends BaseMobilePage {

  txtUsername = '~username';
  txtPassword = '~password';
  btnLogin = '~login';

  async login(user: string, password: string) {
    await this.type(this.txtUsername, user);
    await this.type(this.txtPassword, password);
    await this.click(this.btnLogin);
  }
}
```

---

# API Object Model (AOM)

Similar to Page Object Model (POM), create an API abstraction layer.

## Base API Class

```typescript
import { APIRequestContext } from '@playwright/test';

export class BaseApi {

  protected request: APIRequestContext;

  constructor(request: APIRequestContext) {
    this.request = request;
  }
}
```

---

## User API

```typescript
import { BaseApi } from './BaseApi';

export class UserApi extends BaseApi {

  async getUser(userId: number) {
    return await this.request.get(
      `/users/${userId}`
    );
  }

  async createUser(body: any) {
    return await this.request.post(
      '/users',
      {
        data: body
      }
    );
  }
}
```

---

# API Test

```typescript
import { test, expect } from '@playwright/test';
import { UserApi } from '../../src/api/UserApi';

test('Get User', async ({ request }) => {

  const userApi = new UserApi(request);

  const response = await userApi.getUser(1);

  expect(response.status()).toBe(200);

  const body = await response.json();

  expect(body.id).toBe(1);
});
```

---

# Custom Fixtures

```typescript
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

type Fixtures = {
  loginPage: LoginPage;
};

export const test = base.extend<Fixtures>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  }
});

export { expect } from '@playwright/test';
```

## Usage

```typescript
import { test, expect } from '../../src/fixtures/webFixture';

test('sample', async ({ loginPage }) => {
  await loginPage.open('https://demo.com');
});
```

---

# Enterprise Enhancements

A **Senior Test Architect-level Framework** should additionally include the following capabilities.

## Web

1. Page Object Model (POM)
2. Component Object Model
3. Custom Fixtures
4. Parallel Execution
5. Retry Mechanism
6. Screenshot Capture
7. Video Recording
8. Trace Viewer
9. Accessibility Testing
10. Visual Regression Testing

---

## API

1. API Object Model (AOM)
2. Request/Response Logging
3. Schema Validation
4. Contract Testing
5. OAuth/JWT Support
6. Data-Driven Testing

---

## Mobile

1. Appium Driver Factory
2. Android Support
3. iOS Support
4. Deep Link Testing
5. Gesture Library
6. Device Farm Integration
7. Mobile-Specific Reporting

---

## Common Framework Capabilities

1. Environment Management
2. Faker-Based Test Data Generation
3. CI/CD Integration
4. Allure Reporting
5. Azure DevOps Integration
6. GitHub Actions Integration
7. Docker Support
8. SonarQube Quality Gates
9. Logging Framework
10. Test Data Management
11. Secrets Management

---

# Design Pattern Recommendation

For a large-scale enterprise project:

```text
Tests
  │
  ▼
Fixtures
  │
  ▼
Pages / APIs / Mobile Pages
  │
  ▼
Components
  │
  ▼
Base Classes
  │
  ▼
Utilities
  │
  ▼
Playwright / Appium Libraries
```

---

# Architecture Benefits

This architecture provides:

- Single repository for Web, API, Android, and iOS automation
- High reusability through layered design
- Better maintainability through separation of concerns
- Scalability for enterprise applications
- Consistent reporting and logging
- Easy CI/CD integration
- Cross-team collaboration and standardization
- Support for modern DevOps and Quality Engineering practices

The framework cleanly supports **Web UI + API + Android + iOS automation** in a single Playwright TypeScript repository while maintaining **scalability, reusability, maintainability, and enterprise readiness**.
