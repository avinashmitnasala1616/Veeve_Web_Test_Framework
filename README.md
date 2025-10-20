# Veeva Web Test Automation Framework

A modular **Selenium + Java** automation framework using **Cucumber/TestNG** with a shared core and three runnable test modules.

> **Modules**
>
> - **automation-framework** – reusable engine (DriverFactory, WaitUtils, ConfigManager, ExtentManager, CSV/DOM/Date utilities, Scroll/Popup helpers).
> - **core-product-tests** – Cucumber UI tests for *Core Product* (CP) flows (pages: Warriors*, steps, runners, feature files).
> - **derived-product1-tests** – Cucumber UI tests for *Derived Product 1* (DP1) with CSV-driven expectations.
> - **derived-product2-tests** – Cucumber UI tests for *Derived Product 2* (DP2).
>
> Parent folder: `veeva-web-taf` (aggregator).

---

## Repository Layout (abridged)

```
veeva-web-taf/
├─ automation-framework/
│  ├─ src/main/java/com/avinash/veeva/framework/
│  │  ├─ ConfigManager.java    # env, baseUrl, browser, grid
│  │  ├─ DriverFactory.java    # local + grid, ThreadLocal<WebDriver>
│  │  ├─ WaitUtils.java        # explicit waits (By, WebElement, anyVisible)
│  │  ├─ ExtentManager.java    # reporting hooks + screenshots
│  │  ├─ CSVUtils.java, DateUtils.java, DomUtils.java, ScrollUtils.java, PopUpUtils.java
│  │  └─ ... (helpers)
│  ├─ src/main/resources/log4j2.xml
│  ├─ src/test/java/com/avinash/veeva/framework/Hooks.java
│  └─ src/test/resources/config.properties
│
├─ core-product-tests/
│  ├─ src/main/java/com/avinash/veeva/cp/pages/Warriors*Page.java
│  ├─ src/test/java/com/avinash/veeva/cp/runner/CpTestRunner.java
│  ├─ src/test/java/com/avinash/veeva/cp/steps/Cp*Steps.java
│  ├─ src/test/resources/features/cp_*.feature
│  ├─ logs/test.log
│  └─ reports/cucumber-cp.html
│
├─ derived-product1-tests/
│  ├─ src/test/java/com/avinash/veeva/dp1/runner/Dp1TestRunner.java
│  ├─ src/test/java/com/avinash/veeva/dp1/steps/Dp1Steps.java
│  ├─ src/test/resources/features/dp1_tickets.feature
│  └─ src/test/resources/testdata/expected_*.csv
│
├─ derived-product2-tests/
│  ├─ src/main/java/com/avinash/veeva/dp2/pages/BullsHomePage.java
│  ├─ src/test/java/com/avinash/veeva/dp2/runner/Dp2TestRunner.java
│  ├─ src/test/java/com/avinash/veeva/dp2/steps/Dp2Steps.java
│  └─ src/test/resources/features/dp2_footer_links.feature
└─ pom.xml (aggregator/parent)
```

---

## Architecture

See **[ARCHITECTURE.md](ARCHITECTURE.md)** for Mermaid + PlantUML diagrams.

High-level flow:

- Each test module (CP/DP1/DP2) depends on **automation-framework**.
- `DriverFactory` builds drivers (local or Grid) → `WaitUtils`/`DomUtils` stabilize interactions.
- `Hooks` wires **ExtentManager** to capture **screenshots** and **logs** on failure.
- Feature files drive **Cucumber** steps → Pages encapsulate locators and actions (POM).

---

## Prerequisites

- Java **15+** (your workspace shows JavaSE-15; Java 17+ also works)
- Maven **3.9+**
- Chrome/Edge browser installed (or run against Selenium Grid)
- (Optional) Docker for local Grid

---

## Configuration

`automation-framework/src/test/resources/config.properties` (extend as needed):

```properties
env=dev
baseUrl=https://example-app.dev
browser=chrome
headless=false
grid.enabled=false
grid.url=http://localhost:4444/wd/hub
implicitWaitSec=0
explicitWaitSec=15
screenshot.on.fail=true
reporter=extent
```

You can override any key at runtime with `-D`:

```bash
mvn test -DbaseUrl=https://app.qa.example -Dbrowser=edge -Dheadless=true
```

---

## Build & Run

From repo root (`veeva-web-taf`):

### Run **all** modules
```bash
mvn -T 1C -U clean verify
```

### Run a **single** module
```bash
# Core Product
mvn -pl core-product-tests -am clean test

# Derived Product 1
mvn -pl derived-product1-tests -am clean test

# Derived Product 2
mvn -pl derived-product2-tests -am clean test
```

### Run with **tags** (Cucumber)
```bash
# Example: run CP smoke or regression
mvn -pl core-product-tests -am test -Dcucumber.filter.tags="@smoke or @regression"
```

### Parallel execution (example)
```bash
mvn -pl core-product-tests test -Ddataproviderthreadcount=4 -DforkCount=2C
```

---

## Reports & Logs

- **Extent / HTML reports**: module-specific under `target/` or `reports/` (e.g., `core-product-tests/reports/cucumber-cp.html`).
- **Screenshots on failure**: saved under `target/screenshots/` (module-scoped).
- **Logs**: `logs/test.log` (Log4j2).

If using Allure (optional):
```bash
mvn allure:report
# open target/site/allure-maven-plugin/index.html
```

---

## Key Framework Classes (quick notes)

- **DriverFactory** – Thread-safe driver creation (local/Grid), life-cycle, window/timeouts.
- **WaitUtils**
  - `visible(By)` / `visibleAll(By)`
  - `visible(WebElement)` – wait until existing element is visible
  - `anyVisible(List<WebElement>)` – return first displayed element or timeout
- **ConfigManager** – typed access to `config.properties` with `System.getProperty` overrides.
- **ExtentManager** – singleton for reporters, step and scenario hooks, screenshot attachments.
- **DomUtils/ScrollUtils/PopUpUtils** – small helpers to stabilize flaky UI flows.
- **CSVUtils** – read expectation files for DP1/DP2 validations.

---

## Example: Add a New Test

1. Create a new **feature** in the right module under `src/test/resources/features`.
2. Add **step definitions** under `src/test/java/.../steps`.
3. Reuse **Page Objects** under `src/main/java/.../pages` (or add new ones).
4. Ensure hooks and `DriverFactory` are available via the shared **automation-framework**.
5. Run the module with Maven (see *Build & Run*).

---

## CI/CD (Jenkins/GitHub Actions)

Typical stages:

1. **Build & Lint** (checkstyle/spotless, PMD/SpotBugs)
2. **Unit/Component tests** for `automation-framework`
3. **CP tests** (tagged suites)
4. **DP1 tests**
5. **DP2 tests**
6. **Publish reports** + **Artifacts**
7. (Optional) Slack/Teams notifications

---

## Contribution & Branching

- `main` – stable
- `feature/*` – new feature branches
- Conventional commits (e.g., `feat: add dp1 expected durations csv validator`)

---

## .gitignore (starter)

```
/target/
/reports/
/logs/
/allure-results/
/allure-report/
/idea/
/.settings/
/.project
/.classpath
.DS_Store
*.iml
*.log
*.tmp
*.local
```

---

## License

MIT (or update as required).
