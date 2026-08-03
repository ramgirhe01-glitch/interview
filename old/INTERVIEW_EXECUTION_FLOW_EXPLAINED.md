# Complete Test Execution Flow – Interview Explanation Guide

> Use this to explain **"How does your API/UI test execute end-to-end?"** in an interview.

---

## 🔥 One-Liner Answer (Start with this)

> "Our framework is a **Gradle + Cucumber BDD + Java** based hybrid automation framework that uses **RestAssured for API testing** and **Selenium/Playwright for UI testing**, with parallel execution, Xray/Jira integration, SauceLabs cloud support, and Masterthought HTML reporting."

---

## 📦 Architecture Overview (Draw this on whiteboard)

```
┌─────────────────────────────────────────────────────────────────────┐
│                        GRADLE BUILD SYSTEM                          │
│   build.gradle → ciTest task / test task                            │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     RUNNER LAYER (JUnit 4)                           │
│   BaseRunner (@BeforeClass → Config + Xray + Locale setup)          │
│   CIRunner  (Main.run() → dynamic tags/threads from CI)             │
│   LocalRunner (@CucumberOptions → static tags for local dev)        │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     CUCUMBER ENGINE                                  │
│   Feature Files (.feature) → Step Definitions → Hooks               │
│   Parallel threads controlled by --threads N                        │
└──────────┬───────────────────────────────────┬──────────────────────┘
           │                                   │
           ▼                                   ▼
┌─────────────────────────┐     ┌──────────────────────────────────┐
│      API LAYER          │     │          UI LAYER                 │
│  RestAssured RestClient │     │  Selenium DriverFactory (enum)    │
│  RequestBuilder         │     │  Playwright BrowserManager        │
│  AccessTokenHandler     │     │  Page Object Model (UtafPage)     │
│  PayloadLoader (JSON)   │     │  SauceLabs / Local / Headless     │
└─────────────────────────┘     └──────────────────────────────────┘
           │                                   │
           └───────────────┬───────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    SHARED UTILITIES                                  │
│  ScenarioContext (DI per scenario) │ AppConfig (singleton)          │
│  SoftAssert │ JsonUtil │ FileUtil │ Xray Integration                │
└──────────────────────────────┬──────────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    REPORTING & TEARDOWN                              │
│  Masterthought HTML │ cucumber.json │ Xray Export │ Screenshots     │
│  rerun.txt (failed scenarios) │ Performance Overview JSON           │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🟢 API TEST EXECUTION FLOW (Step-by-Step)

### Example Command:
```bash
./gradlew ciTest -Dtags=XT-178 -DparallelThreadCount=3
```

### Flow:

```
1. GRADLE
   └─→ Picks ciTest task → includes CIRunner.class

2. JUNIT @BeforeClass (BaseRunner.setUpBeforeClass)
   ├─→ Sets Log4j2 ThreadContext → logs go to "ExecutionDetails.log"
   ├─→ setFDSPillar() → reads tag prefix (XT- = CCS pillar, CQV- = CDS pillar)
   ├─→ AppConfig.loadConfiguration() → loads environment config (URLs, credentials, browser)
   ├─→ Loads locale properties (i18n) if configured
   └─→ Initializes Xray singleton if exportToXray=true

3. CIRunner.test() METHOD
   ├─→ Reads -Dtags=XT-178 → converts to: "( @XT-178 ) and not ( @local or @slow or @noparallel )"
   ├─→ Builds cucumber params: features paths, glue packages, plugins, --threads 3
   └─→ Calls io.cucumber.core.cli.Main.run(params) → Cucumber engine starts

4. CUCUMBER PICKS UP FEATURE FILE
   Feature: Verify user API
     @API @XT-178
     Scenario: Create user via API
       Given I load request from file "createUser.json" at path "request"
       When I call API using request details
       Then The response status should be 200
       And I verify response json path "name" equals "John"

5. HOOKS FIRE (@Before("@API"))
   ├─→ UtafHooks.beforeScenario(scenario)
   ├─→ Sets ThreadContext log file = test case ID (per-scenario log)
   ├─→ Updates Xray with scenario start time
   └─→ ScenarioContext is injected via Cucumber DI (PicoContainer)

6. STEP DEFINITIONS EXECUTE
   ├─→ RequestStepDefinitions extends BaseStepDefinitions
   │     constructor receives ScenarioContext (DI injection)
   │
   ├─→ "I load request from file" step:
   │     PayloadLoader.loadJsonFile("createUser.json")
   │     Reads JSON → stores request details (URL, headers, body, method)
   │
   ├─→ "I call API using request details" step:
   │     BaseStepDefinitions.callApiUsingFile()
   │     → UtafStepUtil.callServiceUsingRequestDetails()
   │       → RequestBuilder builds RestAssured RequestSpecBuilder
   │         (sets baseURI, headers, auth token, body)
   │       → AccessTokenHandler.getToken() → fetches OAuth token if needed
   │       → RestClient.post(requestBuilder)
   │         → RestAssured.given(spec).filter(logFilter).post()
   │         → CustomLogFilter captures request/response for logging
   │       → Response stored in ScenarioContext.set("response", response)
   │
   ├─→ "response status should be 200" step:
   │     ResponseStepDefinitions.verifiedStatusCodeAs(200)
   │     → Gets response from ScenarioContext
   │     → assertEquals(200, response.getStatusCode())
   │
   └─→ "verify response json path" step:
         → JsonPath extracts value from response body
         → assertEquals("John", actualValue)

7. HOOKS FIRE (@After("@API", order=0))
   ├─→ PayloadLoader.clearData()
   ├─→ Attaches log file to cucumber report
   ├─→ Prints: "2026-04-30T10:15:30 - XT-178:PASSED"
   └─→ scenarioContext.reset() → clears all stored data

8. @AfterClass (BaseRunner.teardownAfterClass)
   ├─→ Generates Masterthought HTML report from cucumber.json
   ├─→ Exports results to Xray/Jira (if enabled)
   ├─→ Generates OverviewReport.json (performance stats)
   └─→ Checks rerun.txt → if non-empty, throws Exception → BUILD FAILED
```

---

## 🔵 UI TEST EXECUTION FLOW (Step-by-Step)

### Flow:

```
1-3. SAME AS API (Gradle → BaseRunner → CIRunner → Cucumber starts)

4. CUCUMBER PICKS UP FEATURE FILE
   Feature: Login page validation
     @UI @XT-39109
     Scenario: Verify login functionality:[TD001,loginData.json]
       Given I open browser and navigate to application
       When I enter username "admin@test.com"
       And I click login button
       Then I should see dashboard page

5. HOOKS FIRE (@Before("@UI or @UI_API"))
   ├─→ UtafHooks.beforeUIScenario(scenario)
   ├─→ logScenarioStartDetails() → sets ThreadContext, ScenarioContext
   ├─→ populateTestData(scenarioName):
   │     Parses "[TD001,loginData.json]" from scenario name
   │     PayloadLoader.loadJsonFile("loginData.json")
   │     Extracts JSONObject for key "TD001"
   │     scenarioContext.setTestData(jsonObject)
   └─→ Xray.updateScenarioStartDetails()

6. STEP DEFINITIONS EXECUTE
   ├─→ UiStepDefinitions / StafUIStepDefinitions (extends BaseStepDefinitions)
   │     constructor receives ScenarioContext
   │     creates UtafPage (Page Object)
   │
   ├─→ "I open browser" step:
   │     DriverFactory.INSTANCE.createDriver("default")
   │     → Reads Configuration: executionEnvironment (local/saucelabs)
   │     → IF LOCAL:
   │         ChromeOptions → set headless/download path/window size
   │         new ChromeDriver(options) → WebDriver created
   │     → IF SAUCELABS:
   │         MutableCapabilities → platform, browser, version
   │         new RemoteWebDriver(sauceURL, capabilities)
   │     → driver stored in ThreadLocal<WebDriver> (thread-safe)
   │     → driver.get(applicationURL)
   │
   │   OR (Playwright path):
   │     BrowserManager.getInstance().createBrowser()
   │     → Playwright.create() [ThreadLocal per thread]
   │     → browser.newContext() → context.newPage()
   │     → page stored in ThreadLocal<Page>
   │     → page.navigate(url)
   │
   ├─→ "I enter username" step:
   │     Page Object locates element (CSS/XPath)
   │     driver.findElement(By.id("email")).sendKeys("admin@test.com")
   │     OR page.locator("#email").fill("admin@test.com")
   │
   ├─→ "I click login button" step:
   │     element.click() OR page.locator("button").click()
   │
   └─→ "I should see dashboard" step:
         Wait for element visible
         assertTrue(dashboardElement.isDisplayed())

7. HOOKS FIRE (@After order=1 then order=0)
   
   order=1: softAssertHook
   ├─→ If scenarioContext.isSoftAssertFailed() → Assert.fail()
   
   order=0: afterUIScenario
   ├─→ IF FAILED + SELENIUM:
   │     DriverFactory.updateJobStatus(true) → marks SauceLabs as failed
   │     TakesScreenshot → captures PNG → attaches to Cucumber report
   │     Saves screenshot to reports/screenshots/XT-39109.PNG
   │
   ├─→ IF FAILED + PLAYWRIGHT:
   │     page.screenshot(fullPage=true) → attaches to report
   │     Logs current URL for debugging
   │
   ├─→ Attaches execution log file to report
   ├─→ Writes network logs (if enabled) from Chrome DevTools Protocol
   ├─→ DriverFactory.INSTANCE.quitDrivers() → closes all browser windows
   ├─→ BrowserManager.getInstance().closeBrowsers() → closes Playwright
   ├─→ PayloadLoader.clearData()
   └─→ scenarioContext.reset() + null

8. @AfterClass → SAME AS API (reports + Xray + build status)
```

---

## 🧩 Key Classes and Their Roles (Quick Reference)

| Class | Role |
|-------|------|
| `BaseRunner` | JUnit lifecycle: config loading, report generation, Xray export |
| `CIRunner` | Dynamic Cucumber execution via `Main.run()` for CI/CD |
| `LocalRunner` | Static `@CucumberOptions` for IDE-based local runs |
| `UtafHooks` | Cucumber `@Before/@After` hooks: setup, screenshot, cleanup |
| `ScenarioContext` | Per-scenario data store (DI injected), holds response, testData, variables |
| `BaseStepDefinitions` | Parent class for all steps, holds ScenarioContext + UtafStepUtil |
| `RequestStepDefinitions` | API request steps: load JSON, set headers, call API |
| `ResponseStepDefinitions` | API response steps: verify status, extract JSON path values |
| `UiStepDefinitions` | UI steps: browser actions, window management |
| `RestClient` | Wraps RestAssured: GET/POST/PUT/DELETE/PATCH with custom logging |
| `RequestBuilder` | Builds RestAssured `RequestSpecBuilder` from JSON config |
| `AccessTokenHandler` | OAuth2 token generation and caching |
| `DriverFactory` | Enum singleton, ThreadLocal WebDriver management, SauceLabs support |
| `BrowserManager` | Playwright browser/page management with ThreadLocal |
| `AppConfig` | Loads environment-specific configuration (URLs, credentials, platform) |
| `Configuration` | UTAF-level config: browser type, execution env, timeouts |
| `PayloadLoader` | Loads JSON test data files, provides JSONObject by ID |
| `UtafStepUtil` | Utility: variable replacement, API call orchestration |
| `SoftAssert` | Accumulates assertions without failing immediately |
| `Xray` | Tracks scenario results, exports to Jira/Xray |

---

## 🔄 Data Flow Diagram

```
┌──────────────┐    ┌──────────────────┐    ┌─────────────────┐
│ Feature File │───▶│ Step Definition  │───▶│ ScenarioContext │
│ (Gherkin)    │    │ (Glue Code)      │    │ (Data Store)    │
└──────────────┘    └────────┬─────────┘    └────────┬────────┘
                             │                       │
              ┌──────────────┼───────────────────────┘
              │              │
              ▼              ▼
┌─────────────────┐  ┌─────────────────┐
│   API Path      │  │    UI Path      │
│                 │  │                 │
│ PayloadLoader   │  │ DriverFactory   │
│ → loads JSON    │  │ → creates driver│
│                 │  │                 │
│ RequestBuilder  │  │ Page Object     │
│ → builds spec  │  │ → locates elem  │
│                 │  │                 │
│ AccessToken     │  │ Actions         │
│ → OAuth token   │  │ → click/type    │
│                 │  │                 │
│ RestClient      │  │ Assertions      │
│ → executes API  │  │ → verify visible│
│                 │  │                 │
│ Response stored │  │ Screenshot on   │
│ in Context      │  │ failure         │
└─────────────────┘  └─────────────────┘
```

---

## 🎯 How to Explain in Interview (Script)

### "Tell me about your framework architecture"

> "We have a **multi-module Gradle project** with two modules:
> 1. **UTAF (Unified Test Automation Framework)** – the core library containing hooks, drivers, REST client, utilities, and configurations.
> 2. **XceleratorTestAutomation** – the test project containing feature files, step definitions specific to our application, and runners.
>
> We use **Cucumber BDD** for test orchestration with **Gherkin feature files**. Tests are tagged with Jira IDs like `@XT-178`. 
>
> For **API testing**, we use **RestAssured** wrapped in a custom `RestClient` class. Request details are stored in external JSON files, loaded by `PayloadLoader`, and built into requests via `RequestBuilder`.
>
> For **UI testing**, we support **both Selenium and Playwright**. `DriverFactory` is an enum singleton with `ThreadLocal` for thread-safe parallel execution. We also support **SauceLabs** for cross-browser cloud testing.
>
> **Parallel execution** is handled natively by Cucumber's `--threads` parameter in the CI runner. Each thread gets its own `ScenarioContext` (via PicoContainer DI), its own WebDriver (via ThreadLocal), and its own log file (via Log4j2 MDC).
>
> After execution, we generate **Masterthought HTML reports**, export results to **Xray/Jira** for traceability, and if any test fails, `rerun.txt` is populated and the Gradle build fails."

### "How do you handle test data?"

> "For **API tests**, request details (URL, headers, body, method) are stored in external JSON files. `PayloadLoader` loads them, and we can parameterize values using `${variable}` syntax that gets replaced at runtime from `ScenarioContext`.
>
> For **UI tests**, we embed test data IDs in the scenario name like `Scenario:[TD001,loginData.json]`. The `@Before` hook parses this, loads the JSON file, and stores the data in `ScenarioContext` so step definitions can access it."

### "How do you handle failures?"

> "On failure, Cucumber skips remaining steps. Our `@After` hooks capture screenshots (Selenium or Playwright), attach execution logs to the report, update SauceLabs job status, and write the failed scenario path to `rerun.txt`. The `@AfterClass` checks if `rerun.txt` is non-empty and throws an exception to fail the CI build. We also have **soft assertions** that accumulate failures without stopping the scenario, checked in a higher-order `@After` hook."

### "How is parallel execution thread-safe?"

> "Three mechanisms:
> 1. `ThreadLocal<WebDriver>` in DriverFactory – each thread has its own browser
> 2. `ScenarioContext` is DI-injected per scenario via PicoContainer – each scenario has isolated data
> 3. Log4j2 `ThreadContext` (MDC) – each thread writes to its own log file
>
> The `ConcurrentHashMap` in DriverFactory handles multi-window scenarios within a single thread."

---

## ⚠️ Common Interview Follow-up Questions

**Q: Why Cucumber and not TestNG/JUnit directly?**
> BDD approach allows non-technical stakeholders (POs, BAs) to read/write scenarios. Gherkin acts as living documentation. Step reuse reduces code duplication.

**Q: Why both Selenium and Playwright?**
> Playwright was adopted later for better stability, auto-wait, and faster execution. Selenium is retained for SauceLabs compatibility and legacy tests. We're gradually migrating.

**Q: How do you manage environments (dev/staging/prod)?**
> `AppConfig` loads environment-specific properties based on the `FDS_PILLAR` system property and environment name. Config files like `config_ccs.properties`, `config_cds.properties` contain URLs, credentials per environment.

**Q: How do you integrate with CI/CD?**
> GitLab CI calls `./gradlew ciTest -Dtags=... -DparallelThreadCount=5`. Reports are published as artifacts. Xray integration updates Jira test execution status automatically. `run_ci.py` orchestrates the pipeline.

**Q: What reporting do you use?**
> - **Masterthought Cucumber Reports** (HTML with charts, trends)
> - **cucumber.json** (machine-readable)
> - **Xray export** (Jira traceability)
> - **OverviewReport.json** (performance metrics)
> - **Per-scenario log files** + screenshots on failure

