# Xcelerator Test Automation Framework — Deep Technical Documentation

> A BDD-driven, multi-module, multi-environment test automation framework for Siemens
> Xcelerator (FDS – Foundational Digital Services: **CCS** & **CDS** pillars). It supports
> **UI** automation (Selenium + Playwright), **API** automation (REST Assured), encrypted
> tenant data, cross-region execution, and Xray / Grafana reporting.

---

## 1. High-Level Overview

| Aspect | Detail |
|--------|--------|
| **Language** | Java (17+) |
| **Build Tool** | Gradle (multi-project build) |
| **BDD Engine** | Cucumber 7.19.0 (Gherkin `.feature` files) |
| **Test Runner** | JUnit 4.13.2 (`@RunWith(Cucumber.class)`) |
| **UI Automation** | Selenium 4.25.0 + Microsoft Playwright 1.58.0 |
| **API Automation** | REST Assured 5.5.0 |
| **Logging** | Log4j 2.24.0 (`@Log4j2` Lombok) |
| **Reporting** | Cucumber Reporting 5.8.2 (Masterthought) + Xray (Jira) + custom overview JSON |
| **Boilerplate** | Lombok 1.18.34 |
| **Design Pattern** | Page Object Model (POM), Singleton config, Data-driven, BDD |
| **Group / Version** | `com.siemens.xf` / `1.0-SNAPSHOT` |
| **Root Package (core engine)** | `com.siemens.cas` (in `utaf`) |

The framework is split into **two Gradle projects**:

1. **`:utaf`** — *Unified Test Automation Framework* — the **reusable core engine**
   (base pages, driver/browser management, config engine, hooks, reporting, Xray,
   API utilities). Base package: `com.siemens.cas`. Published/consumed as a library.
2. **Root project (`XceleratorTestAutomation`)** — the **product test project**
   consuming `:utaf`, holding feature files, page objects, step definitions, runners,
   and product-specific configuration.

```
settings.gradle:
  rootProject.name = 'XceleratorTestAutomation'
  include ':utaf'
  project(':utaf').projectDir = new File(settingsDir, 'utaf')
```

---

## 2. Architecture Diagram (Logical Layers)

```
┌─────────────────────────────────────────────────────────────────────┐
│                     TEST / SPECIFICATION LAYER                        │
│   Gherkin .feature files  (src/test/java/**/features/*.feature)       │
│   Tags: @CQV-xxxx (CDS), @XT-xxxx (CCS), @QAGP-xxxxxx (test plan)     │
└───────────────────────────────┬─────────────────────────────────────┘
                                 │  glued by
┌───────────────────────────────▼─────────────────────────────────────┐
│                       STEP DEFINITION LAYER                           │
│   com.siemens.xf.ui.stepdefinitions / api.stepdefinitions            │
│   com.ccs.ui.stepdefinitions / com.cds.*.stepdefinitions             │
│   com.siemens.cas.* (base step defs, hooks) [utaf]                    │
└───────────────────────────────┬─────────────────────────────────────┘
                                 │  drives
┌───────────────────────────────▼─────────────────────────────────────┐
│                      PAGE OBJECT / SERVICE LAYER                      │
│   com.siemens.xf.ui.pages.*  (LoginPage, AdminConsolePage, ...)       │
│   com.ccs.ui.pages.*  (CorsAppPage, SdkLoginPage, DesktopAppPage)     │
│   com.cds.pages.*  (cleanup helpers)                                  │
│   extends com.siemens.cas.ui.pages.BasePage  [utaf]                   │
└───────────────────────────────┬─────────────────────────────────────┘
                                 │  uses
┌───────────────────────────────▼─────────────────────────────────────┐
│                     CORE ENGINE (utaf / com.siemens.cas)              │
│  Config │ Driver/Browser Factory │ Hooks │ Reporting │ Xray │ Locale  │
│  ScenarioContext │ REST utilities │ Secret decryption │ AWS S3 utils  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Repository / Folder Structure

```
XceleratorTestAutomation/
├── build.gradle                      # Root build: tasks test/ciTest/multiEnvTest
├── settings.gradle                   # Declares :utaf submodule
├── gradlew / gradlew.bat             # Gradle wrapper
├── run_ci.py                         # CI entrypoint -> gradlew ciTest
├── browserstack-push.py             # BrowserStack result push
├── requirements.txt / pyproject.toml # Python tooling deps
│
├── utaf/                             # === CORE ENGINE (reusable library) ===
│   ├── build.gradle                  # All framework dependencies (Selenium/Playwright/…)
│   └── src/main/java/com/siemens/cas/
│       ├── config/                   # Configuration, ConfigReader, LocaleMessage
│       ├── ui/pages/                 # BasePage (base POM)
│       ├── ui/components/            # IXTableComponent, reusable widgets
│       ├── ui/constants/             # KeyboardKeys
│       ├── api/stepdefinitions/      # BaseStepDefinitions
│       ├── hooks/                    # CustomCucumberEventListener, before/after
│       ├── report/                   # PerformanceReportGenerator
│       ├── xray/                     # Xray, XRayUtils
│       ├── utils/                    # ScenarioContext, SecretKeyDecryption, etc.
│       └── constants/                # UTAFConstant
│
├── src/                              # === PRODUCT TEST PROJECT ===
│   ├── main/java/
│   │   ├── com/siemens/xf/
│   │   │   ├── config/AppConfig.java         # Env/tenant/URL resolution facade
│   │   │   ├── constants/Environments.java   # PRODUS/PRODEUROP/PRODAP/PRODCN enum
│   │   │   ├── ui/pages/                      # Page objects (LoginPage, AdminConsolePage…)
│   │   │   ├── ui/stepdefinitions/            # UI step definitions
│   │   │   ├── api/stepdefinitions/           # API step definitions
│   │   │   └── utilis/                        # DemoUtils (PKCE), Signature, Terminal
│   │   ├── com/ccs/                           # CCS pillar (IAM/onboarding/SDK) pages+steps
│   │   └── com/cds/                           # CDS pillar (agents/workflow) pages+cleanup
│   └── test/
│       ├── java/
│       │   ├── runners/                       # BaseRunner, LocalRunner, CIRunner, MultiEnvRunner
│       │   ├── com/siemens/xf/ui/features/    # UTM, SAM1, iam, gateway, devconsole
│       │   ├── com/siemens/xf/api/features/
│       │   ├── com/ccs/ui/features/           # iam, onboardingconsole, UTS, UTS metrics
│       │   ├── com/cds/                        # CDS features
│       │   └── com/adoption/                   # Adoption features
│       └── resources/
│           ├── config.properties              # Default runtime config
│           └── locators/aws/                   # UI element locators per page (properties)
│
├── XceleratorTestAutomationConfig/   # External env/tenant config (per pillar & platform)
│   ├── ccs/  config.properties, aws/{common,PROD*}-config.properties, *.enc
│   └── cds/  config.properties, aws/…
│
├── payloads/                         # JSON request bodies for API tests
├── reports/                          # cucumber.json/.xml, html, trends, xray-response
├── scripts/                          # push2_datadog.py, test_insights.py
└── docs/                             # Existing markdown knowledge base
```

---

## 4. Build System (Gradle)

### 4.1 Root `build.gradle`

```groovy
plugins { id 'java' }
group 'com.siemens.xf'
version '1.0-SNAPSHOT'
apply plugin: 'java-library'

dependencies {
    implementation project(':utaf')                       // core engine
    implementation fileTree(dir: 'lib', include: ['*.jar']) // xf_iam_client, lombok jar
    annotationProcessor 'org.projectlombok:lombok:1.18.34'
    implementation 'org.bouncycastle:bcprov-jdk18on:1.78.1' // crypto (PKCE/JWT/enc)
}
```

**Gradle tasks (test entry points):**

| Task | Includes | Purpose |
|------|----------|---------|
| `test` | `**/LocalRunner.class` | Local developer run (default) |
| `ciTest` | `**/CIRunner.class` | CI/CD pipeline run (tags + parallelism via system props) |
| `multiEnvTest` | `**/MultiEnvRunner.class` | Run one test across many environments in one execution |
| `downloadDependencies` | — | Copies runtime classpath jars into `lib/` |

All test tasks set `maxHeapSize = "2048m"`, propagate `System.getProperties()`,
and `ignoreFailures = false`.

Example multi-env invocation (from build.gradle comment):
```bash
./gradlew multiEnvTest -Dtags="@CQV-3283" -Denvironments="produs,prodeurop,prodap"
```

### 4.2 `utaf/build.gradle` — Core dependency catalogue

| Category | Library : Version |
|----------|-------------------|
| BDD | `io.cucumber:cucumber-java` / `-junit` / `-picocontainer` : 7.19.0 |
| Unit runner | `junit:junit` : 4.13.2 |
| UI | `org.seleniumhq.selenium:selenium-java` : 4.25.0, `selenium-remote-driver` (netty http/haproxy excluded) |
| UI (modern) | `com.microsoft.playwright:playwright` : 1.58.0 |
| API | `io.rest-assured:rest-assured` : 5.5.0 |
| JSON | `jackson-*` 2.18.0-rc1, `json-path`/`json-path-assert` 2.9.0, `json-simple` 1.1.1, `gson` 2.11.0, `jsonassert` 2.0-rc1 |
| YAML | `org.yaml:snakeyaml` 2.3, `jackson-dataformat-yaml` |
| Reporting | `net.masterthought:cucumber-reporting` 5.8.2 (jackson excluded) |
| Logging | `org.apache.logging.log4j:log4j-core` 2.24.0 |
| Waits | `org.awaitility:awaitility` 4.2.2 |
| AWS | `com.amazonaws:aws-java-sdk-s3` 1.12.+ |
| Mail | `jakarta.mail:jakarta.mail-api` 2.1.3, `commons-email` 1.6.0 |
| JWT / Auth | `com.auth0:java-jwt` 4.4.0, `com.nimbusds:nimbus-jose-jwt` 9.41.1 |
| CSV | `com.opencsv:opencsv` 5.9 |
| HTML parse | `org.jsoup:jsoup` 1.18.1 |
| Zip | `net.lingala.zip4j:zip4j` 2.11.5 |
| Commons | `commons-io` 2.17.0, `commons-text` 1.12.0, `guava` 33.3.1-jre |
| Boilerplate | `org.projectlombok:lombok` 1.18.34 |

> Dependencies are declared with the `api` configuration so they transitively flow to
> the consuming root project. Jackson `databind/core/jsr310` are excluded from several
> libraries to force a single, pinned Jackson version and avoid conflicts.

The `utaf` jar is a **fat/uber jar** (`jar { from configurations.runtimeClasspath… }`)
with `Main-Class: com.siemens.automation`.

---

## 5. Test Runners (`src/test/java/runners`)

### 5.1 `BaseRunner`
The lifecycle backbone. Responsibilities:

- **`@BeforeClass setUpBeforeClass()`**
  - Sets Log4j `ThreadContext` log file name (`ExecutionDetails`).
  - **`setFDSPillar()`** — reads Cucumber `tags` (system property or annotation) and
    sets `FDS_PILLAR`: tags containing `XT-` ⇒ **CCS**, `CQV-` ⇒ **CDS**.
  - **`loadConfigData()`** — `AppConfig.getInstance().loadConfiguration()`.
  - Loads locale data (`localedata/<locale>.properties`) via `LocaleMessage`.
  - Initializes Xray if `exportToXray` is enabled.
- **`@AfterClass teardownAfterClass()`**
  - Builds the Masterthought HTML report from `reports/cucumber.json` with
    classifications (Platform, environment, thread count, execution environment, OS/browser).
  - Sets trends file `reports/demo-trends.json`.
  - Exports report to Jira/Xray (`XRayUtils.exportReportToJIRA`) when enabled.
  - Generates overview JSON (`PerformanceReportGenerator.generateOverviewJsonReport()`).
  - **Fails the build** by throwing if `reports/rerun.txt` is non-empty (i.e. any scenario failed).
- `parallelThreadCount` field defaults to `"1"`.

### 5.2 `LocalRunner` (default local execution)
```java
@RunWith(Cucumber.class)
@CucumberOptions(
  features = { "src/test/java/com/siemens/xf/api/features",
               "src/test/java/com/siemens/xf/ui/features",
               "src/test/java/com/cds",
               "src/test/java/com/adoption",
               "src/test/java/com/ccs/ui/features" },
  glue = { "com/siemens/cas/api/stepdefinitions", "com/siemens/cas/ui/stepdefinitions",
           "com/siemens/xf/api/stepdefinitions", "com/siemens/xf/ui/stepdefinitions",
           "com/cds/api/stepdefinitions", "com/cds/ui/stepdefinitions",
           "com/siemens/cas/hooks", "com/adoption/api/stepdefinitions",
           "com/ccs/ui/pages", "com/ccs/ui/stepdefinitions" },
  plugin = { "pretty",
             "html:reports/cucumber-default-report/cucumber.html",
             "json:reports/cucumber.json",
             "junit:reports/cucumber.xml",
             "rerun:reports/rerun.txt",
             "com.siemens.cas.hooks.CustomCucumberEventListener" },
  monochrome = true, dryRun = false,
  tags = "(@CQV-6050)",                       // <-- edit locally to select scenarios
  snippets = CucumberOptions.SnippetType.CAMELCASE)
public class LocalRunner extends BaseRunner {}
```

### 5.3 `CIRunner`
Uses the Cucumber CLI for **parallel execution** and reads **tags** & thread count
from system properties, so the pipeline controls scope. Included by the `ciTest` task.

### 5.4 `MultiEnvRunner`
Runs a **single tagged scenario across multiple environments** in one JVM run
(e.g. `produs,prodeurop,prodap`) generating separate reports under `reports/multi-env/`.
Included by the `multiEnvTest` task.

---

## 6. Configuration Engine

### 6.1 Layers of configuration (precedence: system property → external file → default)

1. **`src/test/resources/config.properties`** — default runtime knobs.
2. **External config project `XceleratorTestAutomationConfig/`** — pillar + platform + env
   specific files, resolved through template `XceleratorTestAutomationConfig/{pillar}/{platform}/`.
3. **Encrypted tenant data `*.enc`** — decrypted at runtime via `SecretKeyDecryption`.
4. **Locators** — `locators/aws/*.properties` loaded as key/value UI locators.

### 6.2 `config.properties` keys

| Key | Values / Meaning |
|-----|------------------|
| `platform` | `AWS` / `AZURE` / `ALI` / `OPENSHIFTCLOUD` / `OPENSHIFTPHYSICAL` / `RANCHERLOCAL` |
| `environment` | `PRODUS` / `PRODEUROP` / `PRODAP` / `PRODCN` / `INTEG` / `PREPRODEUROP_WITHECA` … |
| `executionEnvironment` | `local` / `saucelabs` / `saucevisual` / `grid` / `remote` |
| `browser` | `Chrome` / `Firefox` / `MicrosoftEdge` / `Safari` / `Opera` (local only) |
| `remoteCapabilities` | JSON caps for saucelabs/grid |
| `element.load.timeout.seconds` | Global element wait (default `60`) |
| `locale` | `en` / `fr` (loads `localedata/<locale>.properties`) |
| `scenarioTagKey` | e.g. `@XT` — tag prefix used for Xray mapping |
| `exportToXray` / `exportToGrafana` | Push results to Jira Xray / Grafana |
| `uploadExecutionLogFileToXray` | Attach exec log to Xray execution |
| `testPlanID` / `testExecutionID` | Jira Xray identifiers |
| `remoteDebuggingAddress` | Attach to a running Chrome (`--remote-debugging-port=9222`) |

### 6.3 `AppConfig` (Singleton facade — `com.siemens.xf.config.AppConfig`)

A thread-safe **double-checked Singleton** wrapping `com.siemens.cas.config.Configuration`.
Key responsibilities and methods:

- **`loadConfiguration()`** — branches on FDS pillar: `loadCCSConfiguration()` or `loadCDSConfiguration()`.
- **CCS loading** merges: `ccs/config.properties` → platform `common-config.properties`
  → env var `CCS_CONFIG_DATA` → `ExternalConfigFiles` list (`.properties` and encrypted `.enc`)
  → cross-region file (US↔EUROP, AP↔CN) → env-specific file → decrypted tenant `.enc`
  → `locators/aws/`.
- **CDS loading** merges: `cds/config.properties` → platform common → `CDS_CONFIG_DATA`
  → env-specific → decrypted tenant `.enc` → locators.
- **Pillar detection** — `getFDSPillar()` reads `FDS_PILLAR` (system prop/env); throws if unset.
  Helpers `isFDSCCS()` / `isFDSCDS()`.
- **Environment resolution** — `getPlatform()`, `getEnvironment()`, `getRegion()`, `getEnv()`
  (all overridable via `-Dplatform` / `-Denvironment`).
- **Property access** — `getPropertyValue`, `getEnvPropertyValue` (prefixes key with env),
  `getDecryptedPropertyValue`, `getDecryptedEnvPropertyValue`.
- **URL builders (with `StringSubstitutor` token replacement `${…}`)**:
  `getWebUrl`, `getBaseUrl`, `getAuthBaseUri`, `getRegionalBaseUrl`, `getCrossRegionalBaseUrl`,
  `getSamTwoAppAccessUrl`, `getSaasProdAppAccessUrl`, `getSaasAppAccessUrl`, `getNxxAppAccessUrl`,
  `getGlobalGateWayAppAccessUrl`, `getAppAccessUrl`, `getQueryAuthUrl` (OAuth authorize URL).
- **Tenant** — `getTenantName(key)` resolves from system property or env-prefixed config key.

### 6.4 `Environments` enum (`com.siemens.xf.constants`)
Represents `PRODUS`, `PRODEUROP`, `PRODAP`, `PRODCN` and drives **cross-region pairing**
logic in `AppConfig` (US↔EUROP, AP↔CN) for cross-region token/data tests.

---

## 7. Page Object Model (POM)

All page objects extend **`com.siemens.cas.ui.pages.BasePage`** (core engine) and use
**Playwright `Locator`s** and/or Selenium. Representative page objects:

| Package | Page objects |
|---------|--------------|
| `com.siemens.xf.ui.pages` | `LoginPage`, `AdminConsolePage`, `AdminConsoleProductsPage`, `AdminConsoleCreditsPage`, `SAMConsolePage`, `SamAuthConsolePage`, `DevConsolePage`, `UTSPage`, `IXDemoPage`, `DempPage`/`DemoPage` |
| `com.ccs.ui.pages` | `CorsAppPage`, `SdkLoginPage`, `DesktopAppPage`, `ForceAuthLoginPage` |
| `com.cds.pages` | `CDSCleanup`, `CustomWorkFlowCleanup`, `AgentRegistryCleanup`, `VectorServiceKbCleanup` (teardown/cleanup helpers) |

Example (`CorsAppPage`): encapsulates login flows for UI & API apps, handles tenant-type
branching, uses Playwright `Locator` objects for elements, and exposes intent-level methods
(login, submit credentials) consumed by step definitions.

**BasePage (core)** typically provides: element waits (backed by `awaitility`/explicit waits),
click/type/getText helpers, screenshot capture, frame/window handling, and locator lookup
from the loaded `locators/aws/*.properties`.

---

## 8. Step Definitions & Hooks

- **Glue packages** (see `LocalRunner`): product step defs live under
  `com.siemens.xf.ui/api.stepdefinitions`, `com.ccs.ui.stepdefinitions`,
  `com.cds.*.stepdefinitions`, `com.adoption.api.stepdefinitions`; base/shared step defs
  live in the core `com.siemens.cas.*.stepdefinitions` and `com.siemens.cas.hooks`.
- **`com.siemens.cas.hooks.CustomCucumberEventListener`** — Cucumber plugin registered in
  runner `plugin` list; handles cross-cutting concerns (logging per scenario, screenshots
  on failure, timing, Xray status mapping).
- **`ScenarioContext`** (core `com.siemens.cas.utils`) — shared state store passed between
  steps within a scenario (via PicoContainer DI — `cucumber-picocontainer`).

---

## 9. API Testing

- Built on **REST Assured 5.5.0**; base step definitions in `com.siemens.cas.api.stepdefinitions.BaseStepDefinitions`.
- **Request payloads** stored as JSON under `payloads/`
  (e.g. `agent_record_a2ui.json`, `create_agent_multistep_a2ui.json`).
- JSON assertions via `json-path`, `json-path-assert`, `jsonassert`.
- **Auth**: OAuth2 with PKCE — `com.siemens.xf.utilis.DemoUtils` generates PKCE code
  challenge; JWT via `java-jwt` / `nimbus-jose-jwt`; crypto via BouncyCastle.
- Feature files under `src/test/java/com/siemens/xf/api/features`.

---

## 10. Reporting & Observability

| Output | Location | Produced by |
|--------|----------|-------------|
| Cucumber JSON | `reports/cucumber.json` | Cucumber `json:` plugin |
| Cucumber JUnit XML | `reports/cucumber.xml` | Cucumber `junit:` plugin |
| Default HTML | `reports/cucumber-default-report/cucumber.html` | Cucumber `html:` plugin |
| Rich HTML report | `reports/cucumber-html-reports/` | Masterthought `ReportBuilder` (BaseRunner `@AfterClass`) |
| Trends | `reports/demo-trends.json` | Masterthought trends |
| Rerun list | `reports/rerun.txt` | Cucumber `rerun:` plugin (drives build failure) |
| Overview JSON | `reports/OverviewReport.json` | `PerformanceReportGenerator` |
| Xray response | `reports/xray-response.json` | `XRayUtils.exportReportToJIRA` |
| Screenshots | `reports/screenshots/` | Hooks on failure |
| Logs | `reports/logs/` | Log4j2 |

**Xray/Jira integration** (`com.siemens.cas.xray.Xray`, `XRayUtils`): when `exportToXray=true`,
creates an Xray JSON payload and pushes to Jira, associating results with `testPlanID` /
`testExecutionID`. Optional Grafana (`exportToGrafana`) and Datadog (`scripts/push2_datadog.py`).

**BrowserStack** results can be pushed via `browserstack-push.py`.

---

## 11. Execution Environments

Controlled by `executionEnvironment` in `config.properties`:

- **`local`** — launches a local browser (`browser=Chrome/Firefox/Edge/Safari/Opera`).
- **`saucelabs` / `saucevisual`** — Sauce Labs cloud grid using `remoteCapabilities`.
- **`grid`** — Selenium Grid.
- **`remote`** — attach to an already-running browser (`remoteDebuggingAddress`).

---

## 12. How to Run

### 12.1 Local (developer)
1. Set desired scope in `LocalRunner`’s `tags` (e.g. `@CQV-6050`) or pass `-Dtags`.
2. Configure `src/test/resources/config.properties` (platform, environment, browser).
3. Set the pillar (or rely on tag inference): `-DFDS_PILLAR=CCS` or `CDS`.
4. Run:
   ```bash
   ./gradlew test -Dtags="@CQV-6050" -Denvironment=PRODUS -Dplatform=AWS
   ```

### 12.2 CI/CD
```bash
python run_ci.py            # loads env vars, invokes: ./gradlew ciTest -Dtags=... -DthreadCount=...
# or directly:
./gradlew ciTest -Dtags="@XT-405" -Denvironment=prodeurop -Dplatform=aws
```
`run_ci.py` loads environment variables (tags, environment, platform, config data paths,
credentials) and shells out to the Gradle `ciTest` task.

### 12.3 Multi-environment
```bash
./gradlew multiEnvTest -Dtags="@CQV-3283" -Denvironments="produs,prodeurop,prodap"
```

---

## 13. Tagging Convention

| Tag prefix | Meaning |
|------------|---------|
| `@CQV-####` | CDS pillar scenario / Jira issue key (sets `FDS_PILLAR=CDS`) |
| `@XT-####` | CCS pillar scenario / execution key (sets `FDS_PILLAR=CCS`) |
| `@QAGP-######` | Xray Test Plan ID |
| `scenarioTagKey` (`@XT`) | Prefix used to map scenarios to Xray tests |

---

## 14. Security & Secrets

- **Encrypted config** (`*.enc`) decrypted via `com.siemens.cas.utils.SecretKeyDecryption`
  using a decrypt key from `Configuration.getDecryptKey()`.
- **BouncyCastle** (`bcprov-jdk18on`) for cryptography (PKCE, JWT signing, decryption).
- **`lib/xf_iam_client-1.2.1.jar`** + `desktop/XFApp/xf_iam_client.dll` — Siemens IAM client
  used for identity/token flows (desktop app tests in `desktop/`).
- Credentials/tenant data are **never** stored in the main repo — they live in the external
  `XceleratorTestAutomationConfig/` project and encrypted `.enc` files.

---

## 15. Product Modules Covered (FDS)

- **CCS (Common Core Services / IAM)** — `com.ccs.*`: Onboarding Console, IAM (Admin/Tech
  User client credentials, secret management, well-known endpoints), UTS (Usage/Token/Metrics),
  SDK/CORS/Desktop app credential journeys.
- **CDS (Common Data Services / Agentic AI)** — `com.cds.*`: Agents, custom & dynamic
  workflows, agent registry, vector service knowledge banks (+ cleanup helpers).
- **XF core (`com.siemens.xf`)** — Admin Console (products, credits), SAM/SAM Auth Console,
  Dev Console, UTS, IX Demo, gateway.
- **Adoption** — `com.adoption.*` adoption-flow features.

---

## 16. Known TODO / Refactoring (from README)

- Rename `utaf` package structure to `com.siemens.cas` (in progress) and update runners.
- Support loading config files from the main project / runner class.
- Create **project-level** `BaseStepDefinitions` and `BasePage`.
- Remove unused steps/imports; move Mindsphere-specific steps to its own project.

---

## 17. Quick Reference — Key Files

| File | Role |
|------|------|
| `build.gradle` | Root build, test tasks |
| `settings.gradle` | Declares `:utaf` module |
| `utaf/build.gradle` | Core dependency catalogue + fat jar |
| `src/test/java/runners/BaseRunner.java` | Lifecycle, config load, reporting, Xray, build-fail |
| `src/test/java/runners/LocalRunner.java` | Local Cucumber options (features/glue/plugins/tags) |
| `src/test/java/runners/CIRunner.java` | Parallel CI execution |
| `src/test/java/runners/MultiEnvRunner.java` | Cross-environment execution |
| `src/main/java/com/siemens/xf/config/AppConfig.java` | Config/env/tenant/URL facade (Singleton) |
| `src/main/java/com/siemens/xf/constants/Environments.java` | Environment enum + cross-region pairing |
| `src/test/resources/config.properties` | Default runtime configuration |
| `src/test/resources/locators/aws/` | UI locators (properties) |
| `run_ci.py` | CI entrypoint |
| `reports/` | All generated reports & logs |

---

*Document generated as framework architecture reference. Update alongside code changes.*

