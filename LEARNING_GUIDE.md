# 📚 Complete Learning Guide: RestAssured, RestClient, Playwright & Selenium

agent id :- 9def4784-b7a3-4044-b80b-8cf5e8bce527 , 26c388b9-8b24-4410-b953-9afebdc95497

## 🎯 Learning Path Overview

This guide will teach you **4 key technologies** using real code from your XceleratorTestAutomation framework:

| Technology | Framework Location | Purpose |
|------------|-------------------|---------|
| **RestAssured** | `utaf/src/main/java/com/siemens/cas/api/core/` | HTTP Client Library |
| **RestClient** | `utaf/src/main/java/com/siemens/cas/api/core/RestClient.java` | Custom wrapper for RestAssured |
| **Playwright** | `utaf/src/main/java/com/siemens/cas/ui/core/BrowserManager.java` | Modern UI Automation |
| **Selenium** | `utaf/src/main/java/com/siemens/cas/ui/core/DriverFactory.java` | Traditional UI Automation |

---

# 📘 PART 1: RestAssured Deep Dive

## What is RestAssured?
RestAssured is a Java library for testing and validating REST APIs. It provides a **fluent API** (method chaining) to make HTTP requests easy to write and read.

## 1.1 Basic RestAssured Syntax

```java
// Standard RestAssured pattern
import io.restassured.RestAssured;
import io.restassured.response.Response;

// GET Request
Response response = RestAssured
    .given()                              // Start building request
        .baseUri("https://api.example.com")
        .header("Authorization", "Bearer token123")
        .queryParam("page", 1)
    .when()                               // Execute
        .get("/users")                    // HTTP method + endpoint
    .then()                               // Validate
        .statusCode(200)                  // Assert status
        .extract()
        .response();                      // Get response object

// POST Request with Body
Response response = RestAssured
    .given()
        .baseUri("https://api.example.com")
        .header("Content-Type", "application/json")
        .body("{\"name\": \"John\", \"email\": \"john@test.com\"}")
    .when()
        .post("/users")
    .then()
        .statusCode(201)
        .extract()
        .response();
```

## 1.2 How Your Framework Uses RestAssured

### File: `utaf/src/main/java/com/siemens/cas/api/core/RestClient.java`

```java
// Your framework wraps RestAssured in RestClient class
public class RestClient {
    
    // GET request using RequestSpecBuilder
    public Response get(RequestSpecBuilder requestBuilder) {
        logFilter = new CustomLogFilter();
        final Response response = RestAssured
            .given(requestBuilder.build())    // Build request from spec
            .filter(logFilter)                // Add logging filter
            .when()
            .get();                           // Execute GET
        logRequestResponseDetails(response.time());
        return response;
    }
    
    // POST request
    public Response post(RequestSpecBuilder requestBuilder) {
        logFilter = new CustomLogFilter();
        final Response response = RestAssured
            .given(requestBuilder.build())
            .when()
            .filter(logFilter)
            .post();                          // Execute POST
        logRequestResponseDetails(response.time());
        return response;
    }
    
    // PUT request
    public Response put(RequestSpecBuilder requestBuilder) {
        // Similar pattern with .put()
    }
    
    // DELETE request  
    public Response delete(RequestSpecBuilder requestBuilder) {
        // Similar pattern with .delete()
    }
    
    // PATCH request
    public Response patch(RequestSpecBuilder requestBuilder) {
        // Similar pattern with .patch()
    }
}
```

### Key Concepts:
1. **RequestSpecBuilder** - Builds the request specification (headers, body, params)
2. **Filter** - Intercepts request/response for logging
3. **Response** - Contains status code, body, headers, etc.

## 1.3 RequestSpecBuilder Deep Dive

### File: `utaf/src/main/java/com/siemens/cas/api/core/RequestBuilder.java`

```java
// Building requests from JSON configuration
public RequestSpecBuilder buildRequest(String jsonRequestDetails, ScenarioContext scenarioContext) {
    JSONObject jsonObject = JsonUtil.getJsonObjectFromString(jsonRequestDetails);
    RequestSpecBuilder builder = new RequestSpecBuilder();
    
    // Set Base URI
    if (null != jsonObject.get("baseUri")) {
        builder.setBaseUri(jsonObject.get("baseUri").toString());
    }
    
    // Set Base Path
    if (null != jsonObject.get("path")) {
        builder.setBasePath(jsonObject.get("path").toString());
    }
    
    // Set Headers
    if (null != jsonObject.get("headers")) {
        Map<String, String> headersData = (Map<String, String>) jsonObject.get("headers");
        builder.addHeaders(headersData);
    }
    
    // Set Query Parameters
    if (null != jsonObject.get("queryParams")) {
        builder.addParams((Map<String, String>) jsonObject.get("queryParams"));
    }
    
    // Set Form Parameters
    if (null != jsonObject.get("formParams")) {
        builder.addFormParams((Map<String, String>) jsonObject.get("formParams"));
    }
    
    // Set Request Body
    if (null != jsonObject.get("body")) {
        builder.setBody(jsonObject.get("body").toString());
    }
    
    return builder;
}
```

## 1.4 JSON Request Template Structure

### File: `src/test/resources/api/assetmanagement/create-asset.json`

```json
{
  "request": {
    "baseUri": "${dcBaseUri}",           // Variable substitution
    "path": "api/assets/${tenant}",      // Dynamic path
    "method": "post",                    // HTTP method
    "headers": {
      "Content-Type": "application/json",
      "Authorization": "Bearer ${accessToken}"  // Token from context
    },
    "queryParams": {
      "page": "1",
      "size": "10"
    },
    "body": {
      "name": "Asset_${uniqueId}",       // Dynamic unique name
      "description": "${testCaseId} Test Asset",
      "parentId": "${parentId}"
    }
  }
}
```

### Key Features:
- **Variable Substitution**: `${variableName}` gets replaced at runtime
- **ScenarioContext Integration**: Variables come from scenario context
- **Separation of Concerns**: Request details in JSON, logic in Java

## 1.5 Response Validation

### File: `utaf/src/main/java/com/siemens/cas/api/stepdefinitions/ResponseStepDefinitions.java`

```java
// Verify status code
@Then("The response status should be {int}")
public void verifiedStatusCodeAs(int expectedStatusCode) {
    assertEquals(expectedStatusCode, scenarioContext.getResponse().getStatusCode());
}

// Verify response contains expected data
@Then("The response should contains following data")
public void verifyExpectedData(List<Map<String, String>> expectedData) {
    JsonPath jsonPath = scenarioContext.getResponse().body().jsonPath();
    
    for (Map<String, String> data : expectedData) {
        String field = data.get("Field");           // e.g., "user.name"
        String condition = data.get("Condition");   // e.g., "match with"
        String expectedValue = data.get("ExpectedValue");
        
        String actualValue = jsonPath.getString(field);
        
        switch (condition) {
            case "match with":
                assertEquals(expectedValue, actualValue);
                break;
            case "contain":
                assertTrue(actualValue.contains(expectedValue));
                break;
        }
    }
}
```

## 1.6 🔬 Hands-On Exercise: RestAssured

Create a new file: `src/test/java/com/learning/RestAssuredExercise.java`

```java
package com.learning;

import io.restassured.RestAssured;
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.response.Response;
import org.junit.Test;

public class RestAssuredExercise {
    
    // Exercise 1: Basic GET Request
    @Test
    public void exercise1_basicGet() {
        Response response = RestAssured
            .given()
                .baseUri("https://jsonplaceholder.typicode.com")
            .when()
                .get("/posts/1")
            .then()
                .statusCode(200)
                .extract()
                .response();
        
        System.out.println("Response Body: " + response.getBody().asString());
        System.out.println("User ID: " + response.jsonPath().getInt("userId"));
    }
    
    // Exercise 2: POST Request with Body
    @Test
    public void exercise2_postRequest() {
        String requestBody = "{"
            + "\"title\": \"Test Post\","
            + "\"body\": \"This is test content\","
            + "\"userId\": 1"
            + "}";
        
        Response response = RestAssured
            .given()
                .baseUri("https://jsonplaceholder.typicode.com")
                .header("Content-Type", "application/json")
                .body(requestBody)
            .when()
                .post("/posts")
            .then()
                .statusCode(201)
                .extract()
                .response();
        
        System.out.println("Created Post ID: " + response.jsonPath().getInt("id"));
    }
    
    // Exercise 3: Using RequestSpecBuilder (like your framework)
    @Test
    public void exercise3_requestSpecBuilder() {
        RequestSpecBuilder builder = new RequestSpecBuilder();
        builder.setBaseUri("https://jsonplaceholder.typicode.com");
        builder.setBasePath("/users");
        builder.addQueryParam("_limit", "5");
        
        Response response = RestAssured
            .given(builder.build())
            .when()
            .get()
            .then()
            .statusCode(200)
            .extract()
            .response();
        
        System.out.println("Number of users: " + response.jsonPath().getList("$").size());
    }
}
```

---

# 📗 PART 2: Playwright Deep Dive

## What is Playwright?
Playwright is a **modern browser automation library** by Microsoft. It supports Chrome, Firefox, and WebKit with a single API.

## 2.1 Your Framework's Playwright Implementation

### File: `utaf/src/main/java/com/siemens/cas/ui/core/BrowserManager.java`

```java
public class BrowserManager {
    // ThreadLocal for parallel execution support
    private static final ThreadLocal<Playwright> playwright = ThreadLocal.withInitial(Playwright::create);
    private final ThreadLocal<Browser> browser = new ThreadLocal<>();
    private final ThreadLocal<BrowserContext> browserContext = new ThreadLocal<>();
    private final ThreadLocal<Page> page = new ThreadLocal<>();
    
    // Launch Chrome Browser
    private synchronized void launchChromeBrowser(String browserNumber) {
        log.info("Launching Chrome browser...");
        
        // Configure launch options
        BrowserType.LaunchOptions options = new BrowserType.LaunchOptions();
        options.setHeadless(false);         // Show browser window
        options.setChannel("chrome");        // Use Chrome browser
        options.setArgs(Arrays.asList("--start-maximized"));
        
        // Launch browser
        Browser browser = playwright.get().chromium().launch(options);
        
        // Create browser context (like incognito window)
        BrowserContext browserContext = browser.newContext(
            new Browser.NewContextOptions()
                .setViewportSize(null)       // Full window
                .setPermissions(List.of("clipboard-read", "clipboard-write"))
        );
        
        // Create new page/tab
        Page page = browserContext.newPage();
        
        // Store references
        setBrowser(browser);
        setBrowserContext(browserContext);
        setPage(page);
    }
    
    // Get current page
    public Page getPage() {
        return page.get();
    }
    
    // Close browser
    public synchronized void closeBrowser() {
        if (null != getPage()) getPage().close();
        if (null != getBrowserContext()) getBrowserContext().close();
        if (null != getBrowser()) getBrowser().close();
    }
}
```

## 2.2 BasePage - Playwright Actions

### File: `utaf/src/main/java/com/siemens/cas/ui/pages/BasePage.java`

```java
@Log4j2
public abstract class BasePage {
    
    // Get current page
    public Page getPage() {
        return BrowserManager.getInstance().getPage();
    }
    
    // ===================== NAVIGATION =====================
    
    public void navigate(String url) {
        log.info("Navigating to URL:" + url);
        getPage().navigate(url);
        waitForBrowserToLoad();
    }
    
    public void reload() {
        log.info("Reloading current page...");
        getPage().reload();
    }
    
    // ===================== CLICK ACTIONS =====================
    
    public void click(Locator locator) {
        waitForElementToBeClickable(locator);
        log.info("Clicking on element:" + locator);
        locator.click();
    }
    
    public void clickUsingJS(Locator locator) {
        log.info("Clicking using JavaScript on element:" + locator);
        locator.evaluate("element => element.click()");
    }
    
    public void doubleClick(Locator locator) {
        locator.dblclick();
    }
    
    public void rightClick(Locator locator) {
        locator.click(new Locator.ClickOptions().setButton(MouseButton.RIGHT));
    }
    
    // ===================== INPUT ACTIONS =====================
    
    public void enterText(Locator locator, String text) {
        waitForElementToBeVisible(locator);
        log.info("Entering text to element:" + locator);
        locator.fill("");        // Clear first
        locator.fill(text);      // Enter text
    }
    
    public void appendText(Locator locator, String text) {
        waitForElementToBeVisible(locator);
        locator.fill(text);      // Append without clearing
    }
    
    public void clearText(Locator locator) {
        locator.fill("");
    }
    
    // ===================== GET DATA =====================
    
    public String getText(Locator locator) {
        waitForElementToBeVisible(locator);
        return locator.textContent().trim();
    }
    
    public String getAttribute(Locator locator, String attributeName) {
        return locator.getAttribute(attributeName).trim();
    }
    
    public String getValue(Locator locator) {
        return locator.getAttribute("value").trim();
    }
    
    public String getCurrentUrl() {
        return getPage().url();
    }
    
    public String getPageTitle() {
        return getPage().title();
    }
    
    // ===================== ELEMENT STATE =====================
    
    public boolean isElementVisible(Locator locator) {
        return locator.isVisible();
    }
    
    public boolean isElementEnabled(Locator locator) {
        return locator.isEnabled();
    }
    
    public boolean isElementDisabled(Locator locator) {
        return locator.isDisabled();
    }
    
    public int getElementsCount(Locator locator) {
        return locator.count();
    }
    
    // ===================== CHECKBOX & RADIO =====================
    
    public void checkCheckbox(Locator locator) {
        if (!locator.isChecked()) {
            locator.check();
        }
    }
    
    public void uncheckCheckbox(Locator locator) {
        if (locator.isChecked()) {
            locator.uncheck();
        }
    }
    
    public boolean isCheckboxChecked(Locator locator) {
        return locator.isChecked();
    }
    
    // ===================== DROPDOWN =====================
    
    public void selectDropdownByValue(Locator locator, String value) {
        locator.selectOption(value);
    }
    
    public void selectDropdownByText(Locator locator, String text) {
        locator.selectOption(new SelectOption().setLabel(text));
    }
    
    // ===================== KEYBOARD ACTIONS =====================
    
    public void pressKey(Locator locator, KeyboardKeys key) {
        locator.focus();
        locator.press(key.getValue());
    }
    
    public void clearTextUsingKeyboard(Locator locator) {
        locator.press("Control+A");
        locator.press("Delete");
    }
    
    // ===================== SCROLLING =====================
    
    public void scrollToElement(Locator locator) {
        locator.scrollIntoViewIfNeeded();
    }
    
    public void scrollToTop() {
        getPage().evaluate("window.scrollTo(0, 0)");
    }
    
    public void scrollToBottom() {
        getPage().evaluate("window.scrollTo(0, document.body.scrollHeight)");
    }
    
    // ===================== DRAG & DROP =====================
    
    public void dragAndDrop(Locator source, Locator target) {
        source.dragTo(target);
    }
    
    // ===================== HOVER =====================
    
    public void hover(Locator locator) {
        locator.hover();
    }
    
    // ===================== ALERTS =====================
    
    public void acceptAlertBox(String promptText) {
        getPage().onceDialog(dialog -> {
            dialog.accept(promptText);
        });
    }
    
    public void dismissAlertBox() {
        getPage().onceDialog(dialog -> {
            dialog.dismiss();
        });
    }
    
    // ===================== WAITS =====================
    
    public void waitForElementToBeVisible(Locator locator) {
        locator.waitFor(new Locator.WaitForOptions()
            .setState(WaitForSelectorState.VISIBLE));
    }
    
    public void waitForElementToBeHidden(Locator locator) {
        locator.waitFor(new Locator.WaitForOptions()
            .setState(WaitForSelectorState.HIDDEN));
    }
    
    // ===================== SCREENSHOTS =====================
    
    public byte[] captureScreenshot() {
        return getPage().screenshot(new Page.ScreenshotOptions().setFullPage(true));
    }
}
```

## 2.3 Playwright Locator Strategies

```java
// Using Page to create locators
Page page = BrowserManager.getInstance().getPage();

// CSS Selector
Locator byCSS = page.locator("button.submit-btn");

// XPath
Locator byXPath = page.locator("xpath=//button[@id='submit']");

// Text Content
Locator byText = page.locator("text=Login");
Locator byExactText = page.locator("text='Login'");

// Role-based (Accessibility)
Locator byRole = page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Submit"));

// Test ID (Best Practice)
Locator byTestId = page.getByTestId("login-button");

// Placeholder
Locator byPlaceholder = page.getByPlaceholder("Enter email");

// Label
Locator byLabel = page.getByLabel("Username");

// Chained Locators
Locator chained = page.locator("form.login").locator("button.submit");

// Filter by text
Locator filtered = page.locator("button").filter(new Locator.FilterOptions().setHasText("Submit"));
```

## 2.4 🔬 Hands-On Exercise: Playwright

Create a new file: `src/test/java/com/learning/PlaywrightExercise.java`

```java
package com.learning;

import com.microsoft.playwright.*;
import org.junit.Test;
import java.nio.file.Paths;

public class PlaywrightExercise {
    
    @Test
    public void exercise1_basicNavigation() {
        try (Playwright playwright = Playwright.create()) {
            Browser browser = playwright.chromium().launch(
                new BrowserType.LaunchOptions().setHeadless(false)
            );
            Page page = browser.newPage();
            
            // Navigate to a website
            page.navigate("https://www.google.com");
            System.out.println("Page Title: " + page.title());
            
            // Take screenshot
            page.screenshot(new Page.ScreenshotOptions()
                .setPath(Paths.get("google_screenshot.png")));
            
            browser.close();
        }
    }
    
    @Test
    public void exercise2_formInteraction() {
        try (Playwright playwright = Playwright.create()) {
            Browser browser = playwright.chromium().launch(
                new BrowserType.LaunchOptions().setHeadless(false)
            );
            Page page = browser.newPage();
            
            page.navigate("https://the-internet.herokuapp.com/login");
            
            // Fill form fields
            page.locator("#username").fill("tomsmith");
            page.locator("#password").fill("SuperSecretPassword!");
            
            // Click login button
            page.locator("button[type='submit']").click();
            
            // Wait and verify
            page.waitForSelector(".flash.success");
            System.out.println("Login successful!");
            
            browser.close();
        }
    }
    
    @Test
    public void exercise3_locatorStrategies() {
        try (Playwright playwright = Playwright.create()) {
            Browser browser = playwright.chromium().launch(
                new BrowserType.LaunchOptions().setHeadless(false)
            );
            Page page = browser.newPage();
            page.navigate("https://the-internet.herokuapp.com/add_remove_elements/");
            
            // Click using CSS
            page.locator("button[onclick='addElement()']").click();
            
            // Click using text
            page.locator("text=Delete").click();
            
            // Click using XPath
            page.locator("//button[contains(text(),'Add Element')]").click();
            
            // Count elements
            int count = page.locator(".added-manually").count();
            System.out.println("Number of delete buttons: " + count);
            
            browser.close();
        }
    }
    
    @Test
    public void exercise4_waitingStrategies() {
        try (Playwright playwright = Playwright.create()) {
            Browser browser = playwright.chromium().launch(
                new BrowserType.LaunchOptions().setHeadless(false)
            );
            Page page = browser.newPage();
            page.navigate("https://the-internet.herokuapp.com/dynamic_loading/1");
            
            // Click start button
            page.locator("#start button").click();
            
            // Wait for element to appear
            Locator finishText = page.locator("#finish h4");
            finishText.waitFor();
            
            System.out.println("Text: " + finishText.textContent());
            
            browser.close();
        }
    }
}
```

---

# 📙 PART 3: Selenium Deep Dive

## What is Selenium?
Selenium WebDriver is a **traditional browser automation tool**. It's widely used but requires more setup than Playwright.

## 3.1 Your Framework's Selenium Implementation

### File: `utaf/src/main/java/com/siemens/cas/ui/core/DriverFactory.java`

```java
@Log4j2
public enum DriverFactory {
    INSTANCE;  // Singleton pattern
    
    private static Map<String, WebDriver> driversMap = new ConcurrentHashMap<>();
    private final ThreadLocal<WebDriver> testContexts = new ThreadLocal<>();
    
    // Get current driver
    public synchronized WebDriver getDriver() {
        return testContexts.get();
    }
    
    // Set driver
    public synchronized void setDriver(WebDriver driver) {
        testContexts.set(driver);
    }
    
    // Create Chrome Driver
    public WebDriver getChromeDriver() {
        String downloadFilepath = FileUtil.getDownloadPath();
        ChromeOptions options = new ChromeOptions();
        
        // Set download preferences
        HashMap<String, Object> chromePrefs = new HashMap<>();
        chromePrefs.put("profile.default_content_settings.popups", 0);
        chromePrefs.put("download.default_directory", downloadFilepath);
        options.setExperimentalOption("prefs", chromePrefs);
        
        // Additional arguments
        options.addArguments("--remote-allow-origins=*");
        
        // Linux/CI specific options
        if (isPlatformLinux()) {
            options.addArguments("--headless");
            options.addArguments("--no-sandbox");
            options.addArguments("window-size=2400x1800");
            options.addArguments("--disable-dev-shm-usage");
            options.setAcceptInsecureCerts(true);
        }
        
        ChromeDriverService driverService = ChromeDriverService.createDefaultService();
        return new ChromeDriver(driverService, options);
    }
    
    // Create Firefox Driver
    public WebDriver getFirefoxDriver() {
        FirefoxOptions options = new FirefoxOptions();
        if (isPlatformLinux()) {
            options.addArguments("--headless");
            options.addArguments("--no-sandbox");
        }
        return new FirefoxDriver(options);
    }
    
    // Create Edge Driver
    public WebDriver getMicrosoftEdgeDriver() {
        return new EdgeDriver();
    }
    
    // Create driver based on configuration
    public synchronized void createDriver(String windowId) {
        BrowserType browserType = Configuration.getInstance().getBrowserType();
        WebDriver driver = null;
        
        if (BrowserType.CHROME == browserType) {
            driver = getChromeDriver();
        } else if (BrowserType.FIREFOX == browserType) {
            driver = getFirefoxDriver();
        } else if (BrowserType.MICROSOFTEDGE == browserType) {
            driver = getMicrosoftEdgeDriver();
        }
        
        // Set timeouts
        driver.manage().timeouts().pageLoadTimeout(Duration.ofSeconds(90));
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(0));
        driver.manage().window().maximize();
        
        setDriver(driver);
        driversMap.put(windowId, driver);
    }
    
    // Quit driver
    public synchronized void quitDriver(String windowId) {
        if (testContexts.get() != null) {
            testContexts.get().manage().deleteAllCookies();
            testContexts.get().quit();
            testContexts.remove();
        }
        driversMap.remove(windowId);
    }
}
```

## 3.2 BasePageSelenium - Selenium Actions

### File: `utaf/src/main/java/com/siemens/cas/ui/pages/BasePageSelenium.java`

```java
@Log4j2
public abstract class BasePageSelenium {
    private static final int TEST_TIMEOUT = 90;
    private static final int POLLING_TIME = 2;
    
    // Get WebDriver
    public WebDriver getDriver() {
        return DriverFactory.INSTANCE.getDriver();
    }
    
    // Get WebDriverWait for explicit waits
    private WebDriverWait getWebDriverWait() {
        return new WebDriverWait(getDriver(), Duration.ofSeconds(TEST_TIMEOUT));
    }
    
    // Get FluentWait for custom polling
    private FluentWait<WebDriver> getFluentWait(int... timeout) {
        int waitTime = timeout.length > 0 ? timeout[0] : TEST_TIMEOUT;
        return new FluentWait<>(getDriver())
            .pollingEvery(Duration.ofSeconds(POLLING_TIME))
            .withTimeout(Duration.ofSeconds(waitTime))
            .ignoring(StaleElementReferenceException.class)
            .ignoring(NoSuchElementException.class);
    }
    
    // ===================== NAVIGATION =====================
    
    protected void go(String url) {
        getDriver().get(url);
    }
    
    public String getCurrentUrl() {
        return getDriver().getCurrentUrl();
    }
    
    // ===================== FIND ELEMENTS =====================
    
    protected WebElement findElement(By by, int... waitTime) {
        WebElement element = waitForElementToBeDisplayed(by, waitTime);
        if (element == null) {
            element = getDriver().findElement(by);
        }
        return element;
    }
    
    protected List<WebElement> findElements(By by, int... waitTime) {
        List<WebElement> elements = waitForElementsToBeDisplayed(by, waitTime);
        if (elements == null) {
            elements = getDriver().findElements(by);
        }
        return elements;
    }
    
    protected WebElement findVisibleElement(By by) {
        List<WebElement> elements = getDriver().findElements(by);
        for (WebElement element : elements) {
            if (element.isDisplayed()) {
                return element;
            }
        }
        throw new NoSuchElementException("Could not find visible element");
    }
    
    // ===================== CLICK ACTIONS =====================
    
    protected void click(By by, int... waitTime) {
        WebElement element = waitForElementToBeClickable(by, waitTime);
        element.click();
    }
    
    protected void clickUsingJavaScript(WebElement element) {
        waitForElementToBeClickable(element);
        getJavascriptExecutor().executeScript("arguments[0].click()", element);
    }
    
    protected void doubleClick(By by) {
        WebElement element = findElement(by);
        getActionsObject().doubleClick(element).perform();
    }
    
    protected void rightClick(By by) {
        WebElement element = findElement(by);
        getActionsObject().contextClick(element).perform();
    }
    
    // ===================== INPUT ACTIONS =====================
    
    protected void enterText(By by, String text) {
        WebElement element = waitForElementToBeDisplayed(by);
        element.clear();
        element.sendKeys(text);
    }
    
    protected void clearText(By by) {
        findElement(by).clear();
    }
    
    protected void appendText(By by, String text) {
        findElement(by).sendKeys(text);
    }
    
    // ===================== GET DATA =====================
    
    protected String getText(By by) {
        return findElement(by).getText();
    }
    
    protected String getAttribute(By by, String attribute) {
        return findElement(by).getAttribute(attribute);
    }
    
    protected String getValue(By by) {
        return findElement(by).getAttribute("value");
    }
    
    protected List<String> getElementsText(By by) {
        List<WebElement> elements = findElements(by);
        List<String> texts = new ArrayList<>();
        for (WebElement element : elements) {
            texts.add(element.getText());
        }
        return texts;
    }
    
    // ===================== ELEMENT STATE =====================
    
    protected boolean isElementDisplayed(By by) {
        try {
            return findElement(by).isDisplayed();
        } catch (NoSuchElementException e) {
            return false;
        }
    }
    
    protected boolean isElementEnabled(By by) {
        return findElement(by).isEnabled();
    }
    
    protected boolean isElementSelected(By by) {
        return findElement(by).isSelected();
    }
    
    // ===================== DROPDOWN =====================
    
    protected void selectDropdownByValue(By by, String value) {
        Select dropdown = new Select(findElement(by));
        dropdown.selectByValue(value);
    }
    
    protected void selectDropdownByText(By by, String text) {
        Select dropdown = new Select(findElement(by));
        dropdown.selectByVisibleText(text);
    }
    
    protected void selectDropdownByIndex(By by, int index) {
        Select dropdown = new Select(findElement(by));
        dropdown.selectByIndex(index);
    }
    
    // ===================== WAITS =====================
    
    protected WebElement waitForElementToBeDisplayed(By by, int... waitTime) {
        try {
            return getFluentWait(waitTime).until(
                ExpectedConditions.visibilityOfElementLocated(by)
            );
        } catch (TimeoutException e) {
            return null;
        }
    }
    
    protected WebElement waitForElementToBeClickable(By by, int... waitTime) {
        return getFluentWait(waitTime).until(
            ExpectedConditions.elementToBeClickable(by)
        );
    }
    
    protected void waitForElementToDisappear(By by, int... waitTime) {
        getFluentWait(waitTime).until(
            ExpectedConditions.invisibilityOfElementLocated(by)
        );
    }
    
    protected void waitForPageToLoad() {
        getFluentWait().until(driver -> 
            getJavascriptExecutor().executeScript("return document.readyState").equals("complete")
        );
    }
    
    // ===================== JAVASCRIPT EXECUTOR =====================
    
    private JavascriptExecutor getJavascriptExecutor() {
        return (JavascriptExecutor) getDriver();
    }
    
    protected void scrollToElement(By by) {
        WebElement element = findElement(by);
        getJavascriptExecutor().executeScript("arguments[0].scrollIntoView(true);", element);
    }
    
    protected void scrollToTop() {
        getJavascriptExecutor().executeScript("window.scrollTo(0, 0);");
    }
    
    protected void scrollToBottom() {
        getJavascriptExecutor().executeScript("window.scrollTo(0, document.body.scrollHeight);");
    }
    
    // ===================== ACTIONS CLASS =====================
    
    protected Actions getActionsObject() {
        return new Actions(getDriver());
    }
    
    protected void hoverOverElement(By by) {
        WebElement element = findElement(by);
        getActionsObject().moveToElement(element).perform();
    }
    
    protected void dragAndDrop(By source, By target) {
        WebElement sourceElement = findElement(source);
        WebElement targetElement = findElement(target);
        getActionsObject().dragAndDrop(sourceElement, targetElement).perform();
    }
    
    // ===================== WINDOW HANDLING =====================
    
    protected void switchToWindow(String windowHandle) {
        getDriver().switchTo().window(windowHandle);
    }
    
    protected void switchToFrame(By by) {
        WebElement frame = findElement(by);
        getDriver().switchTo().frame(frame);
    }
    
    protected void switchToDefaultContent() {
        getDriver().switchTo().defaultContent();
    }
    
    // ===================== ALERTS =====================
    
    protected void acceptAlert() {
        getDriver().switchTo().alert().accept();
    }
    
    protected void dismissAlert() {
        getDriver().switchTo().alert().dismiss();
    }
    
    protected String getAlertText() {
        return getDriver().switchTo().alert().getText();
    }
    
    // ===================== SCREENSHOTS =====================
    
    protected byte[] captureScreenshot() {
        return ((TakesScreenshot) getDriver()).getScreenshotAs(OutputType.BYTES);
    }
}
```

## 3.3 🔬 Hands-On Exercise: Selenium

Create a new file: `src/test/java/com/learning/SeleniumExercise.java`

```java
package com.learning;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.openqa.selenium.*;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.Select;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.time.Duration;
import java.util.List;

public class SeleniumExercise {
    private WebDriver driver;
    private WebDriverWait wait;
    
    @Before
    public void setUp() {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--remote-allow-origins=*");
        // options.addArguments("--headless"); // Uncomment for headless
        
        driver = new ChromeDriver(options);
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        wait = new WebDriverWait(driver, Duration.ofSeconds(30));
    }
    
    @After
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
    
    @Test
    public void exercise1_basicNavigation() {
        driver.get("https://www.google.com");
        System.out.println("Page Title: " + driver.getTitle());
        System.out.println("Current URL: " + driver.getCurrentUrl());
    }
    
    @Test
    public void exercise2_findElements() {
        driver.get("https://the-internet.herokuapp.com/login");
        
        // Find by ID
        WebElement username = driver.findElement(By.id("username"));
        username.sendKeys("tomsmith");
        
        // Find by Name
        WebElement password = driver.findElement(By.name("password"));
        password.sendKeys("SuperSecretPassword!");
        
        // Find by CSS
        WebElement button = driver.findElement(By.cssSelector("button[type='submit']"));
        button.click();
        
        // Find by XPath
        WebElement message = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.xpath("//div[@class='flash success']"))
        );
        System.out.println("Message: " + message.getText());
    }
    
    @Test
    public void exercise3_workingWithLists() {
        driver.get("https://the-internet.herokuapp.com/checkboxes");
        
        List<WebElement> checkboxes = driver.findElements(By.cssSelector("input[type='checkbox']"));
        System.out.println("Number of checkboxes: " + checkboxes.size());
        
        for (WebElement checkbox : checkboxes) {
            if (!checkbox.isSelected()) {
                checkbox.click();
            }
        }
    }
    
    @Test
    public void exercise4_explicitWaits() {
        driver.get("https://the-internet.herokuapp.com/dynamic_loading/1");
        
        // Click start
        driver.findElement(By.cssSelector("#start button")).click();
        
        // Wait for result
        WebElement result = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.cssSelector("#finish h4"))
        );
        
        System.out.println("Result: " + result.getText());
    }
    
    @Test
    public void exercise5_dropdown() {
        driver.get("https://the-internet.herokuapp.com/dropdown");
        
        Select dropdown = new Select(driver.findElement(By.id("dropdown")));
        
        // Select by visible text
        dropdown.selectByVisibleText("Option 1");
        System.out.println("Selected: " + dropdown.getFirstSelectedOption().getText());
        
        // Select by value
        dropdown.selectByValue("2");
        System.out.println("Selected: " + dropdown.getFirstSelectedOption().getText());
    }
    
    @Test
    public void exercise6_javascriptExecutor() {
        driver.get("https://the-internet.herokuapp.com/infinite_scroll");
        
        JavascriptExecutor js = (JavascriptExecutor) driver;
        
        // Scroll to bottom
        for (int i = 0; i < 5; i++) {
            js.executeScript("window.scrollTo(0, document.body.scrollHeight)");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        
        System.out.println("Scrolled to bottom 5 times");
    }
}
```

---

# 📕 PART 4: Comparison & Best Practices

## 4.1 Playwright vs Selenium Comparison

| Feature | Playwright | Selenium |
|---------|------------|----------|
| **Speed** | Faster (auto-waiting) | Slower (manual waits) |
| **Setup** | Single dependency | Multiple drivers needed |
| **Auto-waiting** | Built-in | Manual explicit waits |
| **Parallel execution** | Excellent | Needs configuration |
| **Browser support** | Chrome, Firefox, WebKit | All major browsers |
| **Mobile testing** | Emulation only | Appium integration |
| **Learning curve** | Moderate | Higher |
| **Community** | Growing | Very large |
| **CI/CD** | Excellent | Good |

## 4.2 When to Use What?

| Scenario | Recommended |
|----------|-------------|
| New project | **Playwright** |
| Legacy project with Selenium | Keep **Selenium** |
| Cross-browser testing | Both work |
| Speed is critical | **Playwright** |
| Mobile native apps | Selenium + Appium |
| Need WebKit (Safari) testing | **Playwright** |

## 4.3 Best Practices

### For Both:
```java
// 1. Use Page Object Model
public class LoginPage extends BasePage {
    private Locator usernameField = getPage().locator("#username");
    private Locator passwordField = getPage().locator("#password");
    private Locator loginButton = getPage().locator("#login-btn");
    
    public void login(String username, String password) {
        enterText(usernameField, username);
        enterText(passwordField, password);
        click(loginButton);
    }
}

// 2. Use meaningful locators (prefer test-id)
// Good: page.getByTestId("submit-button")
// Bad:  page.locator("body > div > div > form > button:nth-child(3)")

// 3. Avoid hardcoded waits
// Bad:  Thread.sleep(5000);
// Good: waitForElementToBeVisible(locator);

// 4. Use ScenarioContext for data sharing
scenarioContext.set("userId", responseUserId);
String savedUserId = scenarioContext.get("userId");
```

---

# 🎯 PART 5: Practice Projects

## Project 1: API Testing Practice

Create feature file: `src/test/java/com/learning/features/api_practice.feature`

```gherkin
@API @Learning
Feature: API Testing Practice

  @Exercise-1
  Scenario: Get users from JSONPlaceholder API
    Given I store value "https://jsonplaceholder.typicode.com" in variable "baseUri"
    When I call "get users" api using file "api/learning/get-users.json"
    Then The response status should be 200
    And The response should contains following data
      | Field       | Condition  | ExpectedValue |
      | [0].id      | match with | 1             |
      | [0].name    | contain    | Leanne        |

  @Exercise-2
  Scenario: Create a new post
    Given I store value "https://jsonplaceholder.typicode.com" in variable "baseUri"
    When I call "create post" api using file "api/learning/create-post.json"
    Then The response status should be 201
    And I store value of response field "id" in variable "postId"
    Then The response should contains following data
      | Field | Condition  | ExpectedValue    |
      | title | match with | Test Post Title  |
```

Create JSON file: `src/test/resources/api/learning/get-users.json`

```json
{
  "request": {
    "baseUri": "${baseUri}",
    "path": "/users",
    "method": "get",
    "headers": {
      "Content-Type": "application/json"
    }
  }
}
```

## Project 2: UI Testing Practice

Create feature file: `src/test/java/com/learning/features/ui_practice.feature`

```gherkin
@UI @Learning
Feature: UI Testing Practice

  @Exercise-UI-1
  Scenario: Login to the-internet app
    Given I launch browser window:"first" and navigate to "https://the-internet.herokuapp.com/login"
    When I enter "tomsmith" in username field
    And I enter "SuperSecretPassword!" in password field
    And I click on login button
    Then I should see success message

  @Exercise-UI-2  
  Scenario: Add and remove elements
    Given I launch browser window:"first" and navigate to "https://the-internet.herokuapp.com/add_remove_elements/"
    When I click on Add Element button 3 times
    Then I should see 3 Delete buttons
    When I click on Delete button
    Then I should see 2 Delete buttons
```

---

# 📚 Quick Reference Card

## RestAssured Quick Reference
```java
// GET
RestAssured.given().baseUri(url).get(path);

// POST with body
RestAssured.given().body(jsonBody).post(path);

// With headers
RestAssured.given().header("Authorization", "Bearer token").get();

// Extract response
Response response = RestAssured.given().get().then().extract().response();
response.getStatusCode();
response.jsonPath().getString("field");
```

## Playwright Quick Reference
```java
// Navigate
page.navigate("https://example.com");

// Find & Click
page.locator("button").click();
page.getByRole(AriaRole.BUTTON).click();
page.getByTestId("submit").click();

// Fill input
page.locator("input").fill("text");

// Get text
String text = page.locator("h1").textContent();

// Wait
page.waitForSelector(".element");
locator.waitFor();

// Screenshot
page.screenshot(new Page.ScreenshotOptions().setPath(Paths.get("screenshot.png")));
```

## Selenium Quick Reference
```java
// Navigate
driver.get("https://example.com");

// Find & Click
driver.findElement(By.id("button")).click();
driver.findElement(By.cssSelector(".btn")).click();
driver.findElement(By.xpath("//button")).click();

// Fill input
driver.findElement(By.id("input")).sendKeys("text");

// Get text
String text = driver.findElement(By.tagName("h1")).getText();

// Wait
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("element")));

// Screenshot
((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
```

---

**Happy Learning! 🚀**

Study the files in your framework, run the exercises, and you'll master these technologies!

