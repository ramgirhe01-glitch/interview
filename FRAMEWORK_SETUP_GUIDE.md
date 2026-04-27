# Complete Guide: Cucumber + RestAssured + Playwright + Selenium Framework

## Table of Contents
1. [Project Structure](#1-latest-project-structure)
2. [Gradle Setup](#2-gradle-build-setup)
3. [Cucumber Setup](#3-cucumber-setup)
4. [Selenium Setup](#4-selenium-setup)
5. [Playwright Setup](#5-playwright-setup)
6. [RestAssured Setup](#6-restassured-setup)
7. [Framework Utilities](#7-framework-utilities)
8. [Running Tests](#8-running-tests)

---

## 1. Latest Project Structure

```
project-root/
├── build.gradle
├── settings.gradle
├── gradle.properties
├── gradlew / gradlew.bat
├── src/
│   ├── main/java/com/framework/
│   │   ├── config/
│   │   │   ├── ConfigReader.java          # Read properties files
│   │   │   ├── DriverFactory.java         # Selenium WebDriver factory
│   │   │   └── PlaywrightFactory.java     # Playwright browser factory
│   │   ├── constants/
│   │   │   └── Constants.java             # Global constants
│   │   ├── enums/
│   │   │   ├── BrowserType.java
│   │   │   └── EnvironmentType.java
│   │   ├── exceptions/
│   │   │   └── FrameworkException.java
│   │   ├── models/                        # POJOs for API
│   │   │   ├── request/
│   │   │   │   └── UserRequest.java
│   │   │   └── response/
│   │   │       └── UserResponse.java
│   │   └── utils/
│   │       ├── ApiHelper.java             # RestAssured wrapper
│   │       ├── DateUtil.java
│   │       ├── ExcelReader.java
│   │       ├── JsonUtil.java
│   │       ├── LoggerUtil.java
│   │       ├── ReportHelper.java
│   │       ├── RetryAnalyzer.java
│   │       ├── ScreenshotUtil.java
│   │       └── WaitHelper.java
│   ├── main/resources/
│   │   ├── config.properties
│   │   ├── log4j2.xml
│   │   └── environments/
│   │       ├── dev.properties
│   │       ├── staging.properties
│   │       └── prod.properties
│   └── test/
│       ├── java/com/framework/
│       │   ├── hooks/
│       │   │   └── Hooks.java             # Cucumber Before/After hooks
│       │   ├── runners/
│       │   │   ├── TestRunner.java         # Main Cucumber runner
│       │   │   ├── APITestRunner.java      # API-only runner
│       │   │   └── UITestRunner.java       # UI-only runner
│       │   ├── stepdefinitions/
│       │   │   ├── ui/
│       │   │   │   ├── LoginSteps.java
│       │   │   │   └── DashboardSteps.java
│       │   │   ├── api/
│       │   │   │   ├── UserApiSteps.java
│       │   │   │   └── AuthApiSteps.java
│       │   │   └── common/
│       │   │       └── CommonSteps.java
│       │   └── pages/                      # Page Object Model
│       │       ├── BasePage.java
│       │       ├── selenium/
│       │       │   ├── LoginPage.java
│       │       │   └── DashboardPage.java
│       │       └── playwright/
│       │           ├── PWLoginPage.java
│       │           └── PWDashboardPage.java
│       └── resources/
│           ├── features/
│           │   ├── ui/
│           │   │   ├── login.feature
│           │   │   └── dashboard.feature
│           │   └── api/
│           │       ├── user_api.feature
│           │       └── auth_api.feature
│           ├── testdata/
│           │   ├── users.json
│           │   └── testdata.xlsx
│           └── schemas/                    # JSON schemas for API validation
│               └── user_schema.json
├── reports/
├── logs/
└── download/
```

---

## 2. Gradle Build Setup

### `build.gradle`

```groovy
plugins {
    id 'java'
}

group = 'com.framework'
version = '1.0-SNAPSHOT'
sourceCompatibility = '17'
targetCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    // === Cucumber ===
    implementation 'io.cucumber:cucumber-java:7.18.1'
    implementation 'io.cucumber:cucumber-spring:7.18.1'  // for DI
    testImplementation 'io.cucumber:cucumber-junit-platform-engine:7.18.1'
    testImplementation 'org.junit.platform:junit-platform-suite:1.10.3'
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.3'

    // === Selenium ===
    implementation 'org.seleniumhq.selenium:selenium-java:4.25.0'
    implementation 'io.github.bonigarcia:webdrivermanager:5.9.2'

    // === Playwright ===
    implementation 'com.microsoft.playwright:playwright:1.47.0'

    // === RestAssured ===
    implementation 'io.rest-assured:rest-assured:5.5.0'
    implementation 'io.rest-assured:json-schema-validator:5.5.0'
    implementation 'io.rest-assured:json-path:5.5.0'

    // === Reporting ===
    implementation 'tech.grasshopper:extentreports-cucumber7-adapter:1.14.0'
    implementation 'net.masterthought:cucumber-reporting:5.8.2'

    // === Utilities ===
    implementation 'com.google.code.gson:gson:2.11.0'
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.17.2'
    implementation 'org.apache.poi:poi-ooxml:5.3.0'
    implementation 'org.apache.logging.log4j:log4j-core:2.23.1'
    implementation 'org.projectlombok:lombok:1.18.34'
    annotationProcessor 'org.projectlombok:lombok:1.18.34'

    // === Assertions ===
    testImplementation 'org.assertj:assertj-core:3.26.3'
    testImplementation 'org.hamcrest:hamcrest:2.2'
}

// Cucumber test task
tasks.named('test') {
    useJUnitPlatform()
    systemProperty 'cucumber.filter.tags', System.getProperty('cucumber.filter.tags', '')
    systemProperty 'browser', System.getProperty('browser', 'chrome')
    systemProperty 'env', System.getProperty('env', 'staging')
}
```

### `settings.gradle`

```groovy
rootProject.name = 'test-automation-framework'
```

---

## 3. Cucumber Setup

### Step 1: Create Feature File

**`src/test/resources/features/ui/login.feature`**

```gherkin
@ui @login
Feature: Login Functionality

  Background:
    Given the user is on the login page

  @smoke
  Scenario: Successful login with valid credentials
    When the user enters username "admin" and password "admin123"
    And clicks the login button
    Then the user should be redirected to the dashboard

  @regression
  Scenario Outline: Login with invalid credentials
    When the user enters username "<username>" and password "<password>"
    And clicks the login button
    Then an error message "<message>" should be displayed

    Examples:
      | username | password  | message              |
      | invalid  | admin123  | Invalid username     |
      | admin    | wrong     | Invalid password     |
      |          |           | Fields are required  |
```

**`src/test/resources/features/api/user_api.feature`**

```gherkin
@api @users
Feature: User API

  @smoke
  Scenario: Get all users
    Given the API base URL is configured
    When I send a GET request to "/api/users"
    Then the response status code should be 200
    And the response should contain a list of users

  @regression
  Scenario: Create a new user
    Given the API base URL is configured
    When I send a POST request to "/api/users" with body:
      """
      {
        "name": "John Doe",
        "email": "john@example.com",
        "job": "QA Engineer"
      }
      """
    Then the response status code should be 201
    And the response should contain "name" as "John Doe"
```

### Step 2: Create Test Runner

**`src/test/java/com/framework/runners/TestRunner.java`**

```java
package com.framework.runners;

import org.junit.platform.suite.api.ConfigurationParameter;
import org.junit.platform.suite.api.IncludeEngines;
import org.junit.platform.suite.api.SelectPackages;
import org.junit.platform.suite.api.Suite;

import static io.cucumber.junit.platform.engine.Constants.*;

@Suite
@IncludeEngines("cucumber")
@SelectPackages("com.framework")
@ConfigurationParameter(key = FEATURES_PROPERTY_NAME, value = "src/test/resources/features")
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "com.framework.stepdefinitions,com.framework.hooks")
@ConfigurationParameter(key = PLUGIN_PROPERTY_NAME, value = "pretty,html:reports/cucumber.html,json:reports/cucumber.json,rerun:reports/rerun.txt")
@ConfigurationParameter(key = SNIPPET_TYPE_PROPERTY_NAME, value = "camelcase")
public class TestRunner {
}
```

### Step 3: Create Hooks

**`src/test/java/com/framework/hooks/Hooks.java`**

```java
package com.framework.hooks;

import com.framework.config.DriverFactory;
import com.framework.config.PlaywrightFactory;
import io.cucumber.java.After;
import io.cucumber.java.Before;
import io.cucumber.java.Scenario;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;

public class Hooks {

    @Before("@ui and not @playwright")
    public void beforeUITest(Scenario scenario) {
        String browser = System.getProperty("browser", "chrome");
        DriverFactory.initDriver(browser);
        System.out.println("Starting Selenium UI test: " + scenario.getName());
    }

    @Before("@playwright")
    public void beforePlaywrightTest(Scenario scenario) {
        String browser = System.getProperty("browser", "chromium");
        PlaywrightFactory.initBrowser(browser);
        System.out.println("Starting Playwright UI test: " + scenario.getName());
    }

    @Before("@api")
    public void beforeAPITest(Scenario scenario) {
        System.out.println("Starting API test: " + scenario.getName());
    }

    @After("@ui and not @playwright")
    public void afterUITest(Scenario scenario) {
        if (scenario.isFailed()) {
            byte[] screenshot = ((TakesScreenshot) DriverFactory.getDriver())
                    .getScreenshotAs(OutputType.BYTES);
            scenario.attach(screenshot, "image/png", scenario.getName());
        }
        DriverFactory.quitDriver();
    }

    @After("@playwright")
    public void afterPlaywrightTest(Scenario scenario) {
        if (scenario.isFailed()) {
            byte[] screenshot = PlaywrightFactory.getPage().screenshot();
            scenario.attach(screenshot, "image/png", scenario.getName());
        }
        PlaywrightFactory.closeBrowser();
    }
}
```

---

## 4. Selenium Setup

### Step 1: DriverFactory (Thread-Safe)

**`src/main/java/com/framework/config/DriverFactory.java`**

```java
package com.framework.config;

import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.edge.EdgeDriver;
import org.openqa.selenium.firefox.FirefoxDriver;

import java.time.Duration;

public class DriverFactory {

    private static final ThreadLocal<WebDriver> driver = new ThreadLocal<>();

    public static void initDriver(String browser) {
        WebDriver webDriver;
        switch (browser.toLowerCase()) {
            case "firefox":
                WebDriverManager.firefoxdriver().setup();
                webDriver = new FirefoxDriver();
                break;
            case "edge":
                WebDriverManager.edgedriver().setup();
                webDriver = new EdgeDriver();
                break;
            case "chrome-headless":
                WebDriverManager.chromedriver().setup();
                ChromeOptions options = new ChromeOptions();
                options.addArguments("--headless=new", "--no-sandbox", "--disable-dev-shm-usage");
                webDriver = new ChromeDriver(options);
                break;
            default:
                WebDriverManager.chromedriver().setup();
                webDriver = new ChromeDriver();
        }
        webDriver.manage().window().maximize();
        webDriver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        webDriver.manage().timeouts().pageLoadTimeout(Duration.ofSeconds(30));
        driver.set(webDriver);
    }

    public static WebDriver getDriver() {
        return driver.get();
    }

    public static void quitDriver() {
        if (driver.get() != null) {
            driver.get().quit();
            driver.remove();
        }
    }
}
```

### Step 2: Base Page (Selenium)

**`src/test/java/com/framework/pages/BasePage.java`**

```java
package com.framework.pages;

import com.framework.config.DriverFactory;
import org.openqa.selenium.*;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.time.Duration;

public abstract class BasePage {

    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage() {
        this.driver = DriverFactory.getDriver();
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(15));
        PageFactory.initElements(driver, this);
    }

    protected void click(WebElement element) {
        wait.until(ExpectedConditions.elementToBeClickable(element)).click();
    }

    protected void type(WebElement element, String text) {
        wait.until(ExpectedConditions.visibilityOf(element));
        element.clear();
        element.sendKeys(text);
    }

    protected String getText(WebElement element) {
        return wait.until(ExpectedConditions.visibilityOf(element)).getText();
    }

    protected boolean isDisplayed(WebElement element) {
        try {
            return wait.until(ExpectedConditions.visibilityOf(element)).isDisplayed();
        } catch (TimeoutException e) {
            return false;
        }
    }

    protected void waitForUrl(String partialUrl) {
        wait.until(ExpectedConditions.urlContains(partialUrl));
    }
}
```

### Step 3: Page Object (Selenium)

**`src/test/java/com/framework/pages/selenium/LoginPage.java`**

```java
package com.framework.pages.selenium;

import com.framework.pages.BasePage;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class LoginPage extends BasePage {

    @FindBy(id = "username")
    private WebElement usernameInput;

    @FindBy(id = "password")
    private WebElement passwordInput;

    @FindBy(id = "loginBtn")
    private WebElement loginButton;

    @FindBy(css = ".error-message")
    private WebElement errorMessage;

    public void navigateTo(String url) {
        driver.get(url);
    }

    public void enterUsername(String username) {
        type(usernameInput, username);
    }

    public void enterPassword(String password) {
        type(passwordInput, password);
    }

    public void clickLogin() {
        click(loginButton);
    }

    public String getErrorMessage() {
        return getText(errorMessage);
    }

    public void login(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        clickLogin();
    }
}
```

### Step 4: Step Definitions (Selenium)

**`src/test/java/com/framework/stepdefinitions/ui/LoginSteps.java`**

```java
package com.framework.stepdefinitions.ui;

import com.framework.config.ConfigReader;
import com.framework.pages.selenium.LoginPage;
import io.cucumber.java.en.*;
import static org.assertj.core.api.Assertions.*;

public class LoginSteps {

    private LoginPage loginPage;

    @Given("the user is on the login page")
    public void theUserIsOnTheLoginPage() {
        loginPage = new LoginPage();
        loginPage.navigateTo(ConfigReader.get("base.url") + "/login");
    }

    @When("the user enters username {string} and password {string}")
    public void theUserEntersCredentials(String username, String password) {
        loginPage.enterUsername(username);
        loginPage.enterPassword(password);
    }

    @When("clicks the login button")
    public void clicksTheLoginButton() {
        loginPage.clickLogin();
    }

    @Then("the user should be redirected to the dashboard")
    public void theUserShouldBeRedirectedToDashboard() {
        assertThat(loginPage.driver.getCurrentUrl()).contains("/dashboard");
    }

    @Then("an error message {string} should be displayed")
    public void anErrorMessageShouldBeDisplayed(String expectedMessage) {
        assertThat(loginPage.getErrorMessage()).isEqualTo(expectedMessage);
    }
}
```

---

## 5. Playwright Setup

### Step 1: PlaywrightFactory (Thread-Safe)

**`src/main/java/com/framework/config/PlaywrightFactory.java`**

```java
package com.framework.config;

import com.microsoft.playwright.*;

public class PlaywrightFactory {

    private static final ThreadLocal<Playwright> playwright = new ThreadLocal<>();
    private static final ThreadLocal<Browser> browser = new ThreadLocal<>();
    private static final ThreadLocal<BrowserContext> context = new ThreadLocal<>();
    private static final ThreadLocal<Page> page = new ThreadLocal<>();

    public static Page initBrowser(String browserType) {
        Playwright pw = Playwright.create();
        playwright.set(pw);

        Browser br;
        BrowserType.LaunchOptions options = new BrowserType.LaunchOptions()
                .setHeadless(Boolean.parseBoolean(System.getProperty("headless", "false")));

        switch (browserType.toLowerCase()) {
            case "firefox":
                br = pw.firefox().launch(options);
                break;
            case "webkit":
                br = pw.webkit().launch(options);
                break;
            default:
                br = pw.chromium().launch(options);
        }
        browser.set(br);

        BrowserContext ctx = br.newContext(new Browser.NewContextOptions()
                .setViewportSize(1920, 1080)
                .setRecordVideoDir(java.nio.file.Paths.get("reports/videos/")));
        context.set(ctx);

        Page pg = ctx.newPage();
        page.set(pg);
        return pg;
    }

    public static Page getPage() {
        return page.get();
    }

    public static void closeBrowser() {
        if (page.get() != null) page.get().close();
        if (context.get() != null) context.get().close();
        if (browser.get() != null) browser.get().close();
        if (playwright.get() != null) playwright.get().close();
        page.remove(); context.remove(); browser.remove(); playwright.remove();
    }
}
```

### Step 2: Page Object (Playwright)

**`src/test/java/com/framework/pages/playwright/PWLoginPage.java`**

```java
package com.framework.pages.playwright;

import com.microsoft.playwright.Page;
import com.microsoft.playwright.Locator;
import com.framework.config.PlaywrightFactory;

public class PWLoginPage {

    private final Page page;

    // Locators
    private final String usernameInput = "#username";
    private final String passwordInput = "#password";
    private final String loginButton = "#loginBtn";
    private final String errorMessage = ".error-message";

    public PWLoginPage() {
        this.page = PlaywrightFactory.getPage();
    }

    public void navigateTo(String url) {
        page.navigate(url);
    }

    public void enterUsername(String username) {
        page.fill(usernameInput, username);
    }

    public void enterPassword(String password) {
        page.fill(passwordInput, password);
    }

    public void clickLogin() {
        page.click(loginButton);
    }

    public String getErrorMessage() {
        return page.textContent(errorMessage);
    }

    public void login(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        clickLogin();
    }

    public String getCurrentUrl() {
        return page.url();
    }
}
```

---

## 6. RestAssured Setup

### Step 1: API Helper (Reusable Wrapper)

**`src/main/java/com/framework/utils/ApiHelper.java`**

```java
package com.framework.utils;

import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import io.restassured.response.Response;
import io.restassured.specification.RequestSpecification;

import java.util.Map;

public class ApiHelper {

    private RequestSpecification request;

    public ApiHelper(String baseUrl) {
        RestAssured.baseURI = baseUrl;
        request = RestAssured.given()
                .contentType(ContentType.JSON)
                .accept(ContentType.JSON)
                .log().all();
    }

    public ApiHelper withAuth(String token) {
        request = request.header("Authorization", "Bearer " + token);
        return this;
    }

    public ApiHelper withHeaders(Map<String, String> headers) {
        request = request.headers(headers);
        return this;
    }

    public ApiHelper withQueryParams(Map<String, String> params) {
        request = request.queryParams(params);
        return this;
    }

    public Response get(String endpoint) {
        return request.get(endpoint).then().log().all().extract().response();
    }

    public Response post(String endpoint, Object body) {
        return request.body(body).post(endpoint).then().log().all().extract().response();
    }

    public Response put(String endpoint, Object body) {
        return request.body(body).put(endpoint).then().log().all().extract().response();
    }

    public Response patch(String endpoint, Object body) {
        return request.body(body).patch(endpoint).then().log().all().extract().response();
    }

    public Response delete(String endpoint) {
        return request.delete(endpoint).then().log().all().extract().response();
    }
}
```

### Step 2: API Step Definitions

**`src/test/java/com/framework/stepdefinitions/api/UserApiSteps.java`**

```java
package com.framework.stepdefinitions.api;

import com.framework.config.ConfigReader;
import com.framework.utils.ApiHelper;
import io.cucumber.java.en.*;
import io.restassured.response.Response;

import static org.assertj.core.api.Assertions.*;
import static io.restassured.module.jsv.JsonSchemaValidator.*;

public class UserApiSteps {

    private ApiHelper apiHelper;
    private Response response;

    @Given("the API base URL is configured")
    public void theAPIBaseURLIsConfigured() {
        apiHelper = new ApiHelper(ConfigReader.get("api.base.url"));
    }

    @When("I send a GET request to {string}")
    public void iSendAGETRequestTo(String endpoint) {
        response = apiHelper.get(endpoint);
    }

    @When("I send a POST request to {string} with body:")
    public void iSendAPOSTRequestToWithBody(String endpoint, String body) {
        response = apiHelper.post(endpoint, body);
    }

    @Then("the response status code should be {int}")
    public void theResponseStatusCodeShouldBe(int statusCode) {
        assertThat(response.getStatusCode()).isEqualTo(statusCode);
    }

    @Then("the response should contain a list of users")
    public void theResponseShouldContainAListOfUsers() {
        assertThat(response.jsonPath().getList("data")).isNotEmpty();
    }

    @Then("the response should contain {string} as {string}")
    public void theResponseShouldContainAs(String key, String value) {
        assertThat(response.jsonPath().getString(key)).isEqualTo(value);
    }

    @Then("the response should match the schema {string}")
    public void theResponseShouldMatchTheSchema(String schemaFile) {
        response.then().assertThat()
                .body(matchesJsonSchemaInClasspath("schemas/" + schemaFile));
    }
}
```

---

## 7. Framework Utilities

### ConfigReader

**`src/main/java/com/framework/config/ConfigReader.java`**

```java
package com.framework.config;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

public class ConfigReader {

    private static final Properties properties = new Properties();

    static {
        try {
            String env = System.getProperty("env", "staging");
            // Load default config
            properties.load(new FileInputStream("src/main/resources/config.properties"));
            // Load environment-specific config (overrides defaults)
            properties.load(new FileInputStream("src/main/resources/environments/" + env + ".properties"));
        } catch (IOException e) {
            throw new RuntimeException("Failed to load config: " + e.getMessage());
        }
    }

    public static String get(String key) {
        String sysVal = System.getProperty(key);
        return sysVal != null ? sysVal : properties.getProperty(key);
    }
}
```

### Config Properties

**`src/main/resources/config.properties`**

```properties
# Default Config
browser=chrome
headless=false
implicit.wait=10
explicit.wait=15
page.load.timeout=30

# URLs
base.url=https://app.example.com
api.base.url=https://api.example.com
```

**`src/main/resources/environments/staging.properties`**

```properties
base.url=https://staging.example.com
api.base.url=https://staging-api.example.com
```

---

## 8. Running Tests

### Command Line

```bash
# Run all tests
./gradlew test

# Run by tag
./gradlew test -Dcucumber.filter.tags="@smoke"
./gradlew test -Dcucumber.filter.tags="@api and @regression"
./gradlew test -Dcucumber.filter.tags="@ui and not @playwright"

# Run with specific browser
./gradlew test -Dbrowser=firefox
./gradlew test -Dbrowser=chrome-headless

# Run with specific environment
./gradlew test -Denv=prod

# Combined
./gradlew test -Dcucumber.filter.tags="@smoke" -Dbrowser=chrome-headless -Denv=staging

# Install Playwright browsers (first time only)
npx playwright install
# OR via Java
mvn exec:java -e -D exec.mainClass=com.microsoft.playwright.CLI -D exec.args="install"
```

### Parallel Execution

Add to `src/test/resources/junit-platform.properties`:

```properties
cucumber.execution.parallel.enabled=true
cucumber.execution.parallel.config.strategy=fixed
cucumber.execution.parallel.config.fixed.parallelism=4
cucumber.execution.parallel.config.fixed.max-pool-size=4
```

---

## 9. Quick Setup Checklist

| Step | Action |
|------|--------|
| 1 | Create project with `gradle init` |
| 2 | Add dependencies in `build.gradle` |
| 3 | Create folder structure as shown above |
| 4 | Create `ConfigReader.java` + properties files |
| 5 | Create `DriverFactory.java` (Selenium) |
| 6 | Create `PlaywrightFactory.java` (Playwright) |
| 7 | Create `ApiHelper.java` (RestAssured) |
| 8 | Create `BasePage.java` + Page Objects |
| 9 | Create `.feature` files |
| 10 | Create Step Definitions |
| 11 | Create `Hooks.java` (Before/After) |
| 12 | Create `TestRunner.java` |
| 13 | Create `junit-platform.properties` for parallel execution |
| 14 | Run: `./gradlew test -Dcucumber.filter.tags="@smoke"` |

---

## 10. Key Design Principles

1. **Thread Safety** — Use `ThreadLocal` for WebDriver/Playwright so tests run in parallel
2. **Page Object Model** — Separate page interactions from test logic
3. **Tag-Based Execution** — Use `@ui`, `@api`, `@smoke`, `@regression`, `@playwright` tags
4. **Environment Switching** — Properties files per environment, override via `-Denv=`
5. **Screenshot on Failure** — Automatic via Hooks
6. **API Schema Validation** — Use JSON Schema Validator with RestAssured
7. **Reusable API Helper** — Fluent builder pattern for API calls
8. **Cucumber Reports** — HTML, JSON, and Extent Reports integration

