# Deep Technical Questions & Answers – XceleratorTest Automation Framework

---

## Q1: Explain the complete execution flow when a CI pipeline triggers a test run with `./gradlew ciTest -Dtags=XT-178,XT-48 -DparallelThreadCount=5`.

**Answer:**

1. **Gradle Task Resolution:** Gradle resolves the `ciTest` task (type: `Test`) defined in `build.gradle`. It sets `maxHeapSize=2048m`, passes all system properties (`-D` flags), and includes only `**/CIRunner.class`.

2. **BaseRunner @BeforeClass:** JUnit triggers `BaseRunner.setUpBeforeClass()`:
   - Sets Log4j2 `ThreadContext` for routing logs to `ExecutionDetails.log`.
   - Calls `setFDSPillar()` — reads the `tags` system property, detects prefix (`XT-` → CCS, `CQV-` → CDS), and sets `FDS_PILLAR` system property used later by `AppConfig` to load pillar-specific config files.
   - Calls `AppConfig.getInstance().loadConfiguration()` — singleton loads environment-specific properties (platform, browser, environment, Xray settings, etc.).
   - Loads locale-specific `.properties` file if `locale` config is set (i18n support).
   - Initializes `Xray` singleton if `exportToXray` flag is true.

3. **CIRunner.test() Method:**
   - Converts comma-separated tags to Cucumber tag expression: `( @XT-178 or @XT-48 ) and not ( @local or @slow or @noparallel )`.
   - Constructs a `String[] params` array with feature paths, glue code packages, plugins (JSON/XML/rerun reporters + `CustomCucumberEventListener`), tags, and thread count.
   - Invokes `io.cucumber.core.cli.Main.run(params, ...)` — this is **programmatic Cucumber execution** (not annotation-based), enabling dynamic tag/thread injection from CI variables.

4. **Cucumber Lifecycle:**
   - Scans glue packages for step definitions and hooks.
   - `UtafHooks` is instantiated via Cucumber's DI (PicoContainer/Guice) with `ScenarioContext` injected.
   - `@Before("@UI or @UI_API")` or `@Before("@API")` fires per scenario — sets up logging context, Xray tracking, and optional test data loading from external JSON.

5. **Driver/Browser Initialization (UI tests):**
   - Step definitions call `DriverFactory.INSTANCE.createDriver(key)` or `BrowserManager.getInstance()` (Playwright).
   - `DriverFactory` is a thread-safe **enum singleton** using `ThreadLocal<WebDriver>` + `ConcurrentHashMap` for parallel execution.
   - Supports local Chrome/Firefox/Edge and remote SauceLabs/SauceVisual via `RemoteWebDriver`.

6. **After Hooks (order matters):**
   - `order=1`: Soft assertion check — fails scenario if any soft asserts accumulated.
   - `order=0`: Screenshot capture (Selenium `TakesScreenshot` or Playwright `page.screenshot`), log attachment, network log writing, driver/browser cleanup, `ScenarioContext.reset()`.

7. **BaseRunner @AfterClass:**
   - Generates Masterthought HTML report from `cucumber.json`.
   - Exports results to Xray/Jira if enabled.
   - Generates `OverviewReport.json` for performance tracking.
   - Throws `Exception("Build failure...")` if `rerun.txt` is non-empty → Gradle marks build as FAILED.

---

## Q2: How does the framework achieve thread safety during parallel Cucumber execution, and what are the potential race conditions?

**Answer:**

**Thread Safety Mechanisms:**
- `DriverFactory` uses `ThreadLocal<WebDriver>` so each parallel thread gets its own WebDriver instance.
- `ConcurrentHashMap<String, WebDriver>` stores drivers by key for multi-window scenarios.
- `ScenarioContext` is injected per scenario via Cucumber's DI container (new instance per scenario in PicoContainer).
- `BrowserManager.getInstance()` likely uses `ThreadLocal` for Playwright Page instances.
- Log4j2 `ThreadContext` (MDC) is thread-local for per-scenario log file routing.

**Potential Race Conditions:**
1. **`PayloadLoader.getInstance()`** — if this is a true singleton (not ThreadLocal), concurrent `loadJsonFile()` and `clearData()` calls from parallel scenarios will corrupt shared state.
2. **`SuiteContext.getInstance().setScenarioLogFileName()`** — if SuiteContext is a global singleton, setting log file name is not thread-safe.
3. **Static `parallelThreadCount`** in BaseRunner — read/write from multiple threads without synchronization (though in practice only written once).
4. **File I/O** — `rerun.txt` is appended by Cucumber's rerun plugin from multiple threads; file writes may interleave.
5. **`Xray.getInstance().updateScenarioStartDetails()`** — if Xray maintains a shared collection, concurrent modifications need synchronization.

---

## Q3: Why does CIRunner use `io.cucumber.core.cli.Main.run()` instead of `@RunWith(Cucumber.class)` like LocalRunner? What are the trade-offs?

**Answer:**

| Aspect | `Main.run()` (CIRunner) | `@RunWith(Cucumber.class)` (LocalRunner) |
|--------|--------------------------|------------------------------------------|
| **Dynamic Configuration** | Tags, threads, features are constructed at runtime from system properties | Hardcoded in `@CucumberOptions` annotation (compile-time) |
| **Parallel Execution** | Supports `--threads N` for native Cucumber parallel | No built-in parallel (needs `cucumber-junit-platform-engine` or Surefire forks) |
| **Tag Filtering** | Programmatic tag expression with exclusion logic | Static tag expression |
| **IDE Support** | Cannot right-click run easily | Full IDE integration with green play button |
| **Report Plugins** | Specified in params array | Specified in annotation |

**Trade-off:** CIRunner sacrifices IDE convenience for CI flexibility. The `Main.run()` approach also bypasses JUnit's lifecycle partially — `@Test` method wraps entire Cucumber execution as a single JUnit test, meaning JUnit sees 1 test regardless of how many scenarios run. This affects JUnit-level reporting but Cucumber's own plugins handle per-scenario reporting.

---

## Q4: Explain the dual-browser-engine architecture (Selenium + Playwright) and the design implications.

**Answer:**

The framework supports **both Selenium WebDriver and Microsoft Playwright** simultaneously:

- **Selenium:** `DriverFactory` (enum singleton with ThreadLocal) manages `WebDriver` instances. Used for legacy tests, SauceLabs integration, and scenarios requiring network log capture via `LogType.PERFORMANCE`.
- **Playwright:** `BrowserManager` manages Playwright `Page` instances. Used for modern tests requiring better auto-wait, network interception, or faster execution.

**Design Implications:**
1. **Hook Complexity:** `afterUIScenario()` has two separate try-catch blocks — one for Selenium screenshot capture, one for Playwright. Both must be null-checked independently.
2. **Dual Cleanup:** `DriverFactory.INSTANCE.quitDrivers()` AND `BrowserManager.getInstance().closeBrowsers()` are always called, even if only one engine was used.
3. **No Unified Interface:** There's no abstraction layer (e.g., `BrowserEngine` interface) unifying both — step definitions must know which engine they're using.
4. **Resource Leaks:** If a Playwright test accidentally triggers Selenium DriverFactory initialization (or vice versa), resources may not be properly cleaned.

---

## Q5: What are the critical architectural improvements this framework needs?

**Answer:**

### 1. **Replace Singleton Anti-patterns with Dependency Injection**
- `PayloadLoader.getInstance()`, `SuiteContext.getInstance()`, `Xray.getInstance()` are global singletons causing thread-safety issues.
- **Fix:** Use Cucumber-Picocontainer or Spring DI with scenario-scoped beans.

### 2. **Migrate from JUnit 4 to JUnit 5 (Jupiter) + cucumber-junit-platform-engine**
- Current: JUnit 4 `@RunWith`, `@BeforeClass/@AfterClass`, `@Rule`.
- **Fix:** Use `@Suite` with `cucumber-junit-platform-engine` for native parallel execution, better lifecycle hooks, and `@Tag` filtering.

### 3. **Unify Browser Abstraction**
```java
public interface BrowserEngine {
    void navigate(String url);
    byte[] screenshot();
    void close();
}
// SeleniumEngine implements BrowserEngine
// PlaywrightEngine implements BrowserEngine
```
This eliminates dual try-catch blocks in hooks and enables engine switching via config.

### 4. **Externalize Tag-to-Pillar Mapping**
- Current: Hardcoded `if (tags.contains("XT-"))` logic in `setFDSPillar()`.
- **Fix:** Config-driven mapping: `pillar.mapping=XT:CCS,CQV:CDS` in properties file.

### 5. **Implement Retry Mechanism Properly**
- Current: `rerun.txt` is generated but there's no automatic retry execution.
- **Fix:** Add a `rerunTest` Gradle task that reads `rerun.txt` and re-executes failed scenarios before final report generation.

### 6. **Replace `getBuildNumber()` Stub**
- Currently hardcoded to return `1`. Should read from CI environment variable (`CI_PIPELINE_IID`, `BUILD_NUMBER`).

### 7. **Add Circuit Breaker for Xray Integration**
- If Jira/Xray is down, test execution should not be affected. Wrap Xray calls with timeout + fallback (log warning, skip export).

### 8. **Eliminate Mixed Package Conventions**
- Glue paths mix `classpath:com.siemens.xf.api` (dot notation) with `com/cds/api/stepdefinitions` (path notation). Standardize to one format.

### 9. **Add Health Check Before Execution**
- Validate environment URLs are reachable, credentials are valid, and required services are up before running tests. Fail fast with clear error messages.

### 10. **Implement Structured Logging with Correlation IDs**
- Current: `ThreadContext` only tracks file name. Add scenario ID, thread ID, and timestamp to MDC for distributed log tracing.

---

## Q6: How does the test data flow from external JSON files to step definitions in a UI scenario?

**Answer:**

**Flow:**
1. Scenario name follows convention: `Scenario Name:[testDataId,testDataFile]`
2. `@Before("@UI or @UI_API")` → `populateTestData(scenario.getName())`
3. Parses scenario name: splits on `":[" `→ extracts `testDataId` and `testDataFile`
4. `PayloadLoader.getInstance().loadJsonFile(testDataFile)` — reads JSON file into memory
5. `PayloadLoader.getInstance().getJSONObjectPayload(testDataFile, testDataId)` — extracts specific test data object by ID
6. `scenarioContext.setTestData(requestObj)` — stores in scenario-scoped context
7. Step definitions access via `scenarioContext.getTestData()` — retrieves `JSONObject` with all test parameters

**Problems with this approach:**
- Coupling scenario naming convention to data loading logic (fragile parsing)
- No validation if testDataFile/testDataId doesn't exist (likely NPE at runtime)
- `PayloadLoader` singleton is not thread-safe for parallel execution
- Better alternative: Use Cucumber's native `Scenario Outline` with `Examples` table, or `@DataFile` custom annotation

---

## Q7: What happens if a test fails midway through execution — trace the complete teardown path.

**Answer:**

1. **Exception in step definition** → Cucumber marks scenario as FAILED, skips remaining steps
2. **`@After(order=1)` — `softAssertHook`**: Checks `scenarioContext.isSoftAssertFailed()`. If true, calls `Assert.fail()` (but scenario is already failed, so this is additive)
3. **`@After(order=0)` — `afterUIScenario`**:
   - Prints timestamp + test case ID + FAILED status
   - **Selenium path**: Calls `updateJobStatus(true)` on SauceLabs, prints Sauce URL, captures screenshot as bytes, attaches to Cucumber report, saves PNG to `reports/screenshots/`
   - **Playwright path**: Logs current URL, takes full-page screenshot, attaches to report
   - Attaches execution log file to report if `attachLogToReport` is enabled
   - Writes network performance logs if `networkLogsEnabled`
   - **Cleanup**: `DriverFactory.INSTANCE.quitDrivers()`, `PayloadLoader.clearData()`, `BrowserManager.closeBrowsers()`, `scenarioContext.reset()`, nullifies scenarioContext
4. **Cucumber continues** to next scenario (parallel thread picks next from queue)
5. **After all scenarios**: `@AfterClass teardownAfterClass()` generates reports, checks `rerun.txt` (non-empty = failed scenarios exist), throws exception to fail Gradle build

**Risk:** If `quitDrivers()` itself throws, `BrowserManager.closeBrowsers()` won't execute (no independent try-catch for cleanup sequence). This can leak browser processes.

