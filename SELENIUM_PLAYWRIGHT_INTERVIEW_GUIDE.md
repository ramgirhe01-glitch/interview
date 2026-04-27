# 🎯 SELENIUM & PLAYWRIGHT - Complete Interview Guide

## 📋 Table of Contents
1. [Framework Overview](#framework-overview)
2. [Selenium in Your Framework](#selenium-in-your-framework)
3. [Playwright in Your Framework](#playwright-in-your-framework)
4. [Top 50 Interview Questions & Answers](#top-50-interview-questions--answers)
5. [Practical Coding Challenges](#practical-coding-challenges)

---

## 🏗️ Framework Overview

### Dependencies Used
```gradle
// From utaf/build.gradle
- Selenium: 4.25.0 (selenium-java)
- Playwright: 1.52.0 (com.microsoft.playwright)
- Cucumber: 7.19.0 (BDD Framework)
- JUnit: 4.13.2 (Test Runner)
- Rest-Assured: 5.5.0 (API Testing)
```

### Framework Architecture
```
Your Framework Structure:
├── Selenium WebDriver (Web UI Automation)
│   ├── BasePageSelenium.java (Base methods)
│   ├── DriverFactory (Driver management)
│   └── Page Object Model implementation
├── Playwright (Modern Web Automation)
│   ├── BasePage.java
│   └── BrowserManager
├── Cucumber (BDD)
└── Test Data Management
```

---

## 🔥 Selenium in Your Framework

### Key Selenium Concepts Implemented

#### 1. **Locator Strategies**
```java
// All 8 locator types used in your framework:
By.id("username")
By.name("password")
By.className("btn-login")
By.tagName("input")
By.linkText("Login")
By.partialLinkText("Log")
By.cssSelector("button[type='submit']")
By.xpath("//button[@type='submit']")
```

#### 2. **Waits (Critical for Interviews)**
```java
// Implicit Wait
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));

// Explicit Wait (WebDriverWait)
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("result")));

// Fluent Wait (Used in your BasePageSelenium)
FluentWait<WebDriver> fluentWait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofMillis(500))
    .ignoring(NoSuchElementException.class)
    .ignoring(StaleElementReferenceException.class);
```

#### 3. **Browser Configuration**
```java
ChromeOptions options = new ChromeOptions();
options.addArguments("--remote-allow-origins=*");
options.addArguments("--headless");  // For CI/CD
options.addArguments("--disable-gpu");
options.addArguments("--no-sandbox");
driver = new ChromeDriver(options);
```

#### 4. **Common Actions**
```java
// Form Interactions
element.sendKeys("text");
element.clear();
element.click();
element.submit();

// Dropdown
Select dropdown = new Select(element);
dropdown.selectByVisibleText("Option 1");
dropdown.selectByValue("value");
dropdown.selectByIndex(0);

// JavaScript Executor
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("window.scrollTo(0, document.body.scrollHeight)");
js.executeScript("arguments[0].click()", element);

// Actions Class (Mouse & Keyboard)
Actions actions = new Actions(driver);
actions.moveToElement(element).perform();
actions.dragAndDrop(source, target).perform();
actions.keyDown(Keys.CONTROL).sendKeys("a").keyUp(Keys.CONTROL).perform();
```

#### 5. **Window Handling**
```java
// Get window handles
String mainWindow = driver.getWindowHandle();
Set<String> allWindows = driver.getWindowHandles();

// Switch windows
driver.switchTo().window(windowHandle);

// Frames
driver.switchTo().frame("frameName");
driver.switchTo().defaultContent();
```

#### 6. **Alert Handling**
```java
Alert alert = driver.switchTo().alert();
alert.getText();
alert.accept();
alert.dismiss();
alert.sendKeys("text");
```

---

## 🚀 Playwright in Your Framework

### Key Playwright Concepts

#### 1. **Modern Locators (Recommended)**
```java
// Role-based (Most resilient)
page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Login"));

// Test ID (Best for testing)
page.getByTestId("submit-button");

// Label
page.getByLabel("Username");

// Placeholder
page.getByPlaceholder("Enter username");

// Text
page.getByText("Click me");

// Traditional selectors still work
page.locator("#username");
page.locator("xpath=//button");
```

#### 2. **Auto-Waiting (Huge Advantage)**
```java
// Playwright automatically waits for:
// - Element to be attached to DOM
// - Element to be visible
// - Element to be stable (not animating)
// - Element to receive events
// - Element to be enabled

page.locator("#submit").click();  // No explicit wait needed!
```

#### 3. **Browser Context (Isolation)**
```java
Browser browser = playwright.chromium().launch();

// Each context is isolated (like incognito)
BrowserContext context1 = browser.newContext();
BrowserContext context2 = browser.newContext();

// Configure context
BrowserContext context = browser.newContext(
    new Browser.NewContextOptions()
        .setViewportSize(1920, 1080)
        .setLocale("en-US")
        .setTimezoneId("America/New_York")
        .setPermissions(Arrays.asList("geolocation"))
);
```

#### 4. **Powerful Features**
```java
// Network interception
page.route("**/api/**", route -> {
    route.fulfill(new Route.FulfillOptions().setStatus(200).setBody("{}"));
});

// Screenshots
page.screenshot(new Page.ScreenshotOptions().setPath(Paths.get("screenshot.png")));
element.screenshot();  // Element-level screenshot

// Tracing (for debugging)
context.tracing().start(new Tracing.StartOptions().setScreenshots(true));
// ... test steps ...
context.tracing().stop(new Tracing.StopOptions().setPath(Paths.get("trace.zip")));
// View with: npx playwright show-trace trace.zip

// Multiple elements
locator.count();
locator.first();
locator.last();
locator.nth(2);
locator.allTextContents();
```

---

## 📝 Top 50 Interview Questions & Answers

### **Selenium Basics (Q1-15)**

#### Q1: What is Selenium? What are its components?
**Answer:** 
Selenium is an open-source automation framework for web applications. Components:
1. **Selenium WebDriver** - API for browser automation
2. **Selenium IDE** - Record and playback tool
3. **Selenium Grid** - Parallel execution on multiple machines

**In your framework:** You use Selenium WebDriver 4.25.0 with ChromeDriver.

---

#### Q2: What is WebDriver? How is it different from Selenium RC?
**Answer:**
- **WebDriver:** Directly communicates with browser through browser's native support
- **Selenium RC (deprecated):** Used JavaScript injection and proxy server
- WebDriver is faster, more reliable, and supports headless mode

---

#### Q3: Explain different types of locators in Selenium.
**Answer:**
```java
1. ID - By.id("username") - Fastest, unique
2. Name - By.name("password")
3. ClassName - By.className("btn-primary")
4. TagName - By.tagName("input")
5. LinkText - By.linkText("Login")
6. PartialLinkText - By.partialLinkText("Log")
7. CSS Selector - By.cssSelector("#username") - Fast, flexible
8. XPath - By.xpath("//input[@id='username']") - Most powerful
```

**Priority:** ID > Name > CSS > XPath
**In your framework:** All locators are used in BasePageSelenium.java

---

#### Q4: What's the difference between findElement() and findElements()?
**Answer:**
```java
// findElement() - Returns first matching WebElement, throws NoSuchElementException
WebElement element = driver.findElement(By.id("btn"));

// findElements() - Returns List<WebElement>, returns empty list if not found
List<WebElement> elements = driver.findElements(By.className("item"));
```

---

#### Q5: Explain Implicit vs Explicit vs Fluent Wait.
**Answer:**
```java
// 1. IMPLICIT WAIT - Global timeout for findElement
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
// Drawback: Applied to all elements, cannot customize

// 2. EXPLICIT WAIT - Wait for specific condition
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("result")));
// Advantage: Custom condition, better control

// 3. FLUENT WAIT - Explicit wait + polling + exception ignore
FluentWait<WebDriver> wait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofMillis(500))
    .ignoring(NoSuchElementException.class);
// Advantage: Custom polling, ignore specific exceptions
```

**In your framework:** BasePageSelenium uses FluentWait for reliability.

---

#### Q6: How to handle dropdowns in Selenium?
**Answer:**
```java
WebElement dropdown = driver.findElement(By.id("country"));
Select select = new Select(dropdown);

// Three ways to select
select.selectByVisibleText("India");
select.selectByValue("IND");
select.selectByIndex(0);

// Get options
List<WebElement> options = select.getOptions();

// Get selected
WebElement selected = select.getFirstSelectedOption();

// Check if multi-select
boolean isMultiple = select.isMultiple();
```

---

#### Q7: How to handle alerts/pop-ups in Selenium?
**Answer:**
```java
// Switch to alert
Alert alert = driver.switchTo().alert();

// Get text
String text = alert.getText();

// Accept (OK)
alert.accept();

// Dismiss (Cancel)
alert.dismiss();

// Send input (prompt)
alert.sendKeys("Input text");

// Wait for alert
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
Alert alert = wait.until(ExpectedConditions.alertIsPresent());
```

---

#### Q8: How to switch between windows/tabs?
**Answer:**
```java
// Get current window handle
String mainWindow = driver.getWindowHandle();

// Get all window handles
Set<String> allWindows = driver.getWindowHandles();

// Switch to specific window
for (String window : allWindows) {
    if (!window.equals(mainWindow)) {
        driver.switchTo().window(window);
        // Perform actions
    }
}

// Close current window and switch back
driver.close();
driver.switchTo().window(mainWindow);
```

**In your framework:** Used in XshareZelx tests for multi-window scenarios.

---

#### Q9: How to handle frames/iframes?
**Answer:**
```java
// Switch by index
driver.switchTo().frame(0);

// Switch by name or ID
driver.switchTo().frame("frameName");

// Switch by WebElement
WebElement frameElement = driver.findElement(By.id("myframe"));
driver.switchTo().frame(frameElement);

// Switch back to main content
driver.switchTo().defaultContent();

// Switch to parent frame
driver.switchTo().parentFrame();
```

---

#### Q10: What is JavascriptExecutor? When to use it?
**Answer:**
```java
JavascriptExecutor js = (JavascriptExecutor) driver;

// Use cases:
// 1. Click when element is hidden/overlapped
js.executeScript("arguments[0].click();", element);

// 2. Scroll to element
js.executeScript("arguments[0].scrollIntoView(true);", element);

// 3. Scroll to bottom
js.executeScript("window.scrollTo(0, document.body.scrollHeight);");

// 4. Set value directly
js.executeScript("arguments[0].value='text';", element);

// 5. Get page height
Long height = (Long) js.executeScript("return document.body.scrollHeight;");
```

**When to use:** When normal WebDriver methods fail due to element state.

---

#### Q11: What is Actions class? Provide examples.
**Answer:**
```java
Actions actions = new Actions(driver);

// Mouse hover
actions.moveToElement(element).perform();

// Drag and drop
actions.dragAndDrop(sourceElement, targetElement).perform();

// Right click
actions.contextClick(element).perform();

// Double click
actions.doubleClick(element).perform();

// Keyboard actions
actions.keyDown(Keys.CONTROL).sendKeys("a").keyUp(Keys.CONTROL).perform();

// Complex chain
actions.moveToElement(menu)
       .click()
       .moveToElement(submenu)
       .click()
       .perform();
```

**In your framework:** Used in SeleniumExercise.java for hover tests.

---

#### Q12: How to take screenshots in Selenium?
**Answer:**
```java
// Cast driver to TakesScreenshot
TakesScreenshot ts = (TakesScreenshot) driver;

// As File
File srcFile = ts.getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(srcFile, new File("screenshot.png"));

// As Bytes (for reports)
byte[] screenshot = ts.getScreenshotAs(OutputType.BYTES);

// As Base64 (for HTML embedding)
String base64 = ts.getScreenshotAs(OutputType.BASE64);

// Element screenshot (Selenium 4+)
WebElement element = driver.findElement(By.id("logo"));
File file = element.getScreenshotAs(OutputType.FILE);
```

**In your framework:** Screenshots are captured on test failure automatically.

---

#### Q13: What is StaleElementReferenceException? How to handle it?
**Answer:**
**Cause:** Element was found in DOM but DOM changed (page refresh, AJAX update).

```java
// Solution 1: Re-find element
WebElement element = driver.findElement(By.id("button"));
element.click();
// ... DOM changes ...
element = driver.findElement(By.id("button")); // Re-find
element.click();

// Solution 2: Use FluentWait
FluentWait<WebDriver> wait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofMillis(500))
    .ignoring(StaleElementReferenceException.class);

// Solution 3: Retry mechanism
boolean clicked = false;
int attempts = 0;
while(attempts < 3) {
    try {
        driver.findElement(By.id("button")).click();
        clicked = true;
        break;
    } catch(StaleElementReferenceException e) {
        attempts++;
    }
}
```

**In your framework:** FluentWait ignores this exception.

---

#### Q14: Explain Page Object Model (POM).
**Answer:**
**Page Object Model** is a design pattern where:
- Each web page is represented as a Java class
- Page elements are defined as variables
- Actions are defined as methods

```java
// Example from your framework
public class LoginPage extends BasePageSelenium {
    
    // Locators
    private By usernameField = By.id("username");
    private By passwordField = By.id("password");
    private By loginButton = By.cssSelector("button[type='submit']");
    
    // Actions
    public void enterUsername(String username) {
        driver.findElement(usernameField).sendKeys(username);
    }
    
    public void enterPassword(String password) {
        driver.findElement(passwordField).sendKeys(password);
    }
    
    public void clickLogin() {
        driver.findElement(loginButton).click();
    }
    
    public void login(String user, String pass) {
        enterUsername(user);
        enterPassword(pass);
        clickLogin();
    }
}
```

**Benefits:**
- Code reusability
- Easy maintenance
- Readable tests
- Separation of concerns

**In your framework:** All page classes extend BasePageSelenium.

---

#### Q15: What is Page Factory in Selenium?
**Answer:**
Page Factory is an extension of POM using @FindBy annotations:

```java
public class LoginPage {
    
    WebDriver driver;
    
    // Initialize elements
    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }
    
    // Find elements using annotations
    @FindBy(id = "username")
    WebElement usernameField;
    
    @FindBy(id = "password")
    WebElement passwordField;
    
    @FindBy(css = "button[type='submit']")
    WebElement loginButton;
    
    // Actions
    public void login(String user, String pass) {
        usernameField.sendKeys(user);
        passwordField.sendKeys(pass);
        loginButton.click();
    }
}
```

**Note:** Your framework uses traditional POM, not Page Factory.

---

### **Selenium Advanced (Q16-30)**

#### Q16: How to handle dynamic elements?
**Answer:**
```java
// Dynamic ID: id="username_12345"
// Use contains, starts-with, ends-with

// XPath strategies
By.xpath("//input[contains(@id, 'username')]")
By.xpath("//input[starts-with(@id, 'user')]")
By.xpath("//button[text()='Login']")

// CSS strategies
By.cssSelector("input[id^='user']")  // starts-with
By.cssSelector("input[id$='name']")  // ends-with
By.cssSelector("input[id*='user']")  // contains

// Following sibling
By.xpath("//label[text()='Username']/following-sibling::input")

// Parent
By.xpath("//button[@id='save']/parent::form")
```

---

#### Q17: What is Selenium Grid? How does it work?
**Answer:**
**Selenium Grid** enables parallel execution across:
- Multiple browsers
- Multiple OS
- Multiple machines

**Components:**
- **Hub:** Central point, receives test requests
- **Nodes:** Machines that execute tests

```java
// Connect to Grid
DesiredCapabilities cap = new DesiredCapabilities();
cap.setBrowserName("chrome");
cap.setPlatform(Platform.WINDOWS);

WebDriver driver = new RemoteWebDriver(
    new URL("http://localhost:4444/wd/hub"), cap
);
```

**Setup:**
```bash
# Start Hub
java -jar selenium-server.jar hub

# Start Node
java -jar selenium-server.jar node --hub http://localhost:4444
```

---

#### Q18: How to handle SSL certificate errors?
**Answer:**
```java
// Chrome
ChromeOptions options = new ChromeOptions();
options.setAcceptInsecureCerts(true);
options.addArguments("--ignore-certificate-errors");
WebDriver driver = new ChromeDriver(options);

// Firefox
FirefoxOptions options = new FirefoxOptions();
options.setAcceptInsecureCerts(true);
WebDriver driver = new FirefoxDriver(options);
```

---

#### Q19: How to handle CAPTCHA in automation?
**Answer:**
**CAPTCHA is designed to prevent automation**, but solutions:

1. **Disable CAPTCHA** in test environment (Best practice)
2. **Use test CAPTCHA** with known response
3. **Mock CAPTCHA** with automation-friendly alternative
4. **Manual intervention** (pause script)
5. **Third-party services** (2Captcha, Anti-Captcha)

```java
// Best approach: Environment-based
if (env.equals("test")) {
    // CAPTCHA disabled
} else {
    // Handle CAPTCHA
}
```

**Important:** Never automate real CAPTCHA - violates terms of service.

---

#### Q20: Explain different browser options.
**Answer:**
```java
// CHROME OPTIONS
ChromeOptions options = new ChromeOptions();
options.addArguments("--headless");                    // Headless mode
options.addArguments("--disable-gpu");                 // Disable GPU
options.addArguments("--no-sandbox");                  // For Docker/Linux
options.addArguments("--disable-dev-shm-usage");       // Overcome limited resource
options.addArguments("--window-size=1920,1080");       // Set window size
options.addArguments("--disable-notifications");       // Disable notifications
options.addArguments("--disable-popup-blocking");      // Disable popups
options.addArguments("--incognito");                   // Private mode
options.addArguments("--start-maximized");             // Maximize window

// Set download directory
Map<String, Object> prefs = new HashMap<>();
prefs.put("download.default_directory", "/path/to/download");
options.setExperimentalOption("prefs", prefs);

// Set user agent
options.addArguments("user-agent=Mozilla/5.0...");

// Disable images
prefs.put("profile.managed_default_content_settings.images", 2);
options.setExperimentalOption("prefs", prefs);
```

**In your framework:** ChromeOptions configured in setup methods.

---

#### Q21: How to upload files in Selenium?
**Answer:**
```java
// Method 1: sendKeys to file input
WebElement uploadElement = driver.findElement(By.id("fileUpload"));
uploadElement.sendKeys("C:\\path\\to\\file.txt");

// Method 2: For hidden inputs, use JavaScript
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("document.getElementById('fileUpload').style.display='block';");
uploadElement.sendKeys("C:\\path\\to\\file.txt");

// Method 3: Robot class (for OS dialog)
Robot robot = new Robot();
StringSelection filePath = new StringSelection("C:\\path\\to\\file.txt");
Toolkit.getDefaultToolkit().getSystemClipboard().setContents(filePath, null);
robot.keyPress(KeyEvent.VK_CONTROL);
robot.keyPress(KeyEvent.VK_V);
robot.keyRelease(KeyEvent.VK_V);
robot.keyRelease(KeyEvent.VK_CONTROL);
robot.keyPress(KeyEvent.VK_ENTER);
robot.keyRelease(KeyEvent.VK_ENTER);
```

---

#### Q22: How to download files in Selenium?
**Answer:**
```java
// Set download directory
ChromeOptions options = new ChromeOptions();
Map<String, Object> prefs = new HashMap<>();
prefs.put("download.default_directory", "C:\\Downloads");
prefs.put("download.prompt_for_download", false);
prefs.put("safebrowsing.enabled", false);
options.setExperimentalOption("prefs", prefs);

WebDriver driver = new ChromeDriver(options);

// Click download link
driver.findElement(By.linkText("Download")).click();

// Wait for file to download
File downloadedFile = new File("C:\\Downloads\\file.pdf");
int timeout = 30;
while (!downloadedFile.exists() && timeout > 0) {
    Thread.sleep(1000);
    timeout--;
}

// Verify download
assertTrue(downloadedFile.exists());
```

---

#### Q23: What is Desired Capabilities?
**Answer:**
**DesiredCapabilities** set browser properties for RemoteWebDriver.

```java
DesiredCapabilities cap = new DesiredCapabilities();

// Browser
cap.setBrowserName("chrome");
cap.setVersion("120");
cap.setPlatform(Platform.WINDOWS);

// Chrome specific
ChromeOptions options = new ChromeOptions();
options.addArguments("--headless");
cap.setCapability(ChromeOptions.CAPABILITY, options);

// Remote WebDriver
WebDriver driver = new RemoteWebDriver(
    new URL("http://localhost:4444/wd/hub"), cap
);
```

**Note:** In Selenium 4, use browser-specific Options classes.

---

#### Q24: How to handle cookies in Selenium?
**Answer:**
```java
// Get all cookies
Set<Cookie> cookies = driver.manage().getCookies();

// Get specific cookie
Cookie cookie = driver.manage().getCookieNamed("sessionId");

// Add cookie
Cookie newCookie = new Cookie("user", "testuser");
driver.manage().addCookie(newCookie);

// Delete cookie
driver.manage().deleteCookieNamed("sessionId");

// Delete all cookies
driver.manage().deleteAllCookies();

// Cookie details
System.out.println(cookie.getName());
System.out.println(cookie.getValue());
System.out.println(cookie.getDomain());
System.out.println(cookie.getPath());
System.out.println(cookie.getExpiry());
```

---

#### Q25: Explain relative locators in Selenium 4.
**Answer:**
**Relative Locators** find elements based on relationship with other elements.

```java
import static org.openqa.selenium.support.locators.RelativeLocator.with;

// Above
driver.findElement(with(By.tagName("input"))
    .above(driver.findElement(By.id("password"))));

// Below
driver.findElement(with(By.tagName("button"))
    .below(driver.findElement(By.id("username"))));

// To left of
driver.findElement(with(By.tagName("label"))
    .toLeftOf(driver.findElement(By.id("username"))));

// To right of
driver.findElement(with(By.tagName("span"))
    .toRightOf(driver.findElement(By.id("username"))));

// Near (within 50 pixels)
driver.findElement(with(By.tagName("button"))
    .near(driver.findElement(By.id("submit"))));

// Combine multiple
driver.findElement(with(By.tagName("input"))
    .below(driver.findElement(By.id("label1")))
    .above(driver.findElement(By.id("label2"))));
```

---

#### Q26: What's new in Selenium 4?
**Answer:**
**Major changes:**

1. **W3C WebDriver Protocol** - Standardized communication
2. **Relative Locators** - Find elements by position
3. **Chrome DevTools Protocol** - Network interception, geolocation
4. **Better Window/Tab Management**
5. **Upgraded Selenium Grid** - GraphQL, observability
6. **Deprecated** - DesiredCapabilities, FindsBy

```java
// New window/tab management
driver.switchTo().newWindow(WindowType.TAB);
driver.switchTo().newWindow(WindowType.WINDOW);

// Chrome DevTools Protocol
DevTools devTools = ((ChromeDriver) driver).getDevTools();
devTools.createSession();

// Network interception
devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));
devTools.addListener(Network.requestWillBeSent(), request -> {
    System.out.println("Request: " + request.getRequest().getUrl());
});

// Mock geolocation
devTools.send(Emulation.setGeolocationOverride(
    Optional.of(51.5074), 
    Optional.of(-0.1278), 
    Optional.of(100)
));
```

---

#### Q27: How to handle browser logs?
**Answer:**
```java
// Get browser console logs
LogEntries logs = driver.manage().logs().get(LogType.BROWSER);
for (LogEntry log : logs) {
    System.out.println(log.getLevel() + " " + log.getMessage());
}

// Enable logging
LoggingPreferences logPrefs = new LoggingPreferences();
logPrefs.enable(LogType.BROWSER, Level.ALL);
logPrefs.enable(LogType.PERFORMANCE, Level.ALL);

ChromeOptions options = new ChromeOptions();
options.setCapability(CapabilityType.LOGGING_PREFS, logPrefs);

// Performance logs (network)
LogEntries perfLogs = driver.manage().logs().get(LogType.PERFORMANCE);
```

---

#### Q28: How to handle Authentication pop-ups?
**Answer:**
```java
// Method 1: Pass credentials in URL
driver.get("https://username:password@example.com");

// Method 2: Alert (if basic auth)
Alert alert = driver.switchTo().alert();
alert.sendKeys("username");
alert.sendKeys(Keys.TAB);
alert.sendKeys("password");
alert.accept();

// Method 3: Chrome DevTools Protocol (Selenium 4)
((ChromeDriver) driver).getDevTools().createSession();
((ChromeDriver) driver).getDevTools().send(
    Network.setExtraHTTPHeaders(
        ImmutableMap.of("Authorization", "Basic " + 
        Base64.getEncoder().encodeToString("username:password".getBytes()))
    )
);
```

---

#### Q29: Explain TestNG annotations used with Selenium.
**Answer:**
```java
public class TestClass {
    
    WebDriver driver;
    
    @BeforeSuite
    public void beforeSuite() {
        // Runs once before all tests in suite
        System.out.println("Setup environment");
    }
    
    @BeforeTest
    public void beforeTest() {
        // Runs before each <test> in testng.xml
    }
    
    @BeforeClass
    public void beforeClass() {
        // Runs once before class
        System.setProperty("webdriver.chrome.driver", "path");
    }
    
    @BeforeMethod
    public void beforeMethod() {
        // Runs before each @Test method
        driver = new ChromeDriver();
        driver.manage().window().maximize();
    }
    
    @Test(priority = 1)
    public void testLogin() {
        // Test case
    }
    
    @Test(priority = 2, dependsOnMethods = {"testLogin"})
    public void testDashboard() {
        // Depends on testLogin
    }
    
    @AfterMethod
    public void afterMethod() {
        // Runs after each test
        driver.quit();
    }
    
    @AfterClass
    public void afterClass() {
        // Cleanup
    }
}
```

**In your framework:** Uses JUnit (@Before, @After, @Test).

---

#### Q30: How to run tests in parallel with Selenium?
**Answer:**
```xml
<!-- testng.xml -->
<suite name="ParallelSuite" parallel="tests" thread-count="3">
    <test name="ChromeTest">
        <parameter name="browser" value="chrome"/>
        <classes>
            <class name="TestClass"/>
        </classes>
    </test>
    <test name="FirefoxTest">
        <parameter name="browser" value="firefox"/>
        <classes>
            <class name="TestClass"/>
        </classes>
    </test>
</suite>
```

```java
// Use ThreadLocal for WebDriver
public class DriverManager {
    private static ThreadLocal<WebDriver> driver = new ThreadLocal<>();
    
    public static WebDriver getDriver() {
        return driver.get();
    }
    
    public static void setDriver(WebDriver driverInstance) {
        driver.set(driverInstance);
    }
    
    public static void quitDriver() {
        driver.get().quit();
        driver.remove();
    }
}
```

---

### **Playwright Questions (Q31-45)**

#### Q31: What is Playwright? How is it different from Selenium?
**Answer:**
**Playwright** is a modern automation framework by Microsoft.

**Key Differences:**
| Feature | Selenium | Playwright |
|---------|----------|------------|
| Architecture | WebDriver protocol | CDP/DevTools Protocol |
| Auto-waiting | Manual explicit waits | Automatic built-in |
| Browser support | Chrome, Firefox, Safari, Edge | Chromium, Firefox, WebKit |
| Speed | Slower | Faster |
| Network control | Limited | Full control |
| Browser context | Not built-in | Built-in isolation |
| Screenshots | Basic | Advanced (full page, element) |
| Tracing | Not available | Built-in trace viewer |
| Setup | Complex | Simple |

**In your framework:** Playwright 1.52.0 used alongside Selenium.

---

#### Q32: What is Browser Context in Playwright?
**Answer:**
**BrowserContext** is like an incognito session - isolated environment.

```java
Browser browser = playwright.chromium().launch();

// Create contexts
BrowserContext context1 = browser.newContext();
BrowserContext context2 = browser.newContext();

// Different cookies, storage, sessions
Page page1 = context1.newPage();
Page page2 = context2.newPage();

// Configure context
BrowserContext context = browser.newContext(
    new Browser.NewContextOptions()
        .setViewportSize(1920, 1080)
        .setLocale("en-US")
        .setTimezoneId("America/New_York")
        .setGeolocation(37.7749, -122.4194)
        .setPermissions(Arrays.asList("geolocation"))
        .setColorScheme(ColorScheme.DARK)
);
```

**Benefits:**
- Test isolation
- Parallel execution
- Different user sessions
- Mock different devices

---

#### Q33: Explain auto-waiting in Playwright.
**Answer:**
**Playwright automatically waits for elements** before performing actions:

```java
// No explicit wait needed!
page.locator("#submit").click();  

// Playwright automatically waits for:
// 1. Element to be attached to DOM
// 2. Element to be visible
// 3. Element to be stable (not animating)
// 4. Element to receive events
// 5. Element to be enabled

// Custom timeout
page.locator("#slow-element").click(new Locator.ClickOptions()
    .setTimeout(60000));

// Wait for specific state
page.locator("#element").waitFor(new Locator.WaitForOptions()
    .setState(WaitForSelectorState.VISIBLE)
    .setTimeout(30000));
```

**This eliminates most StaleElementReferenceException issues!**

---

#### Q34: What are the recommended locator strategies in Playwright?
**Answer:**
**Priority Order:**

1. **getByRole** - Most resilient
2. **getByTestId** - Best for testing
3. **getByLabel** - Forms
4. **getByPlaceholder** - Inputs
5. **getByText** - Text content
6. **CSS/XPath** - Last resort

```java
// 1. By Role (Recommended)
page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions()
    .setName("Submit"));

// 2. By Test ID (Add data-testid attribute)
page.getByTestId("submit-button");

// 3. By Label
page.getByLabel("Username");

// 4. By Placeholder
page.getByPlaceholder("Enter your email");

// 5. By Text
page.getByText("Click here");

// 6. CSS/XPath (fallback)
page.locator("#submit");
page.locator("xpath=//button[@type='submit']");
```

**In your framework:** PlaywrightExercise.java demonstrates all strategies.

---

#### Q35: How to handle multiple elements in Playwright?
**Answer:**
```java
Locator items = page.locator(".item");

// Count
int count = items.count();

// First, last, nth
items.first().click();
items.last().click();
items.nth(2).click();

// All text contents
List<String> allTexts = items.allTextContents();

// Iterate
for (int i = 0; i < items.count(); i++) {
    System.out.println(items.nth(i).textContent());
}

// Filter
Locator activeItems = page.locator(".item")
    .filter(new Locator.FilterOptions()
        .setHasText("Active"));

// Has
Locator parentWithChild = page.locator(".parent")
    .filter(new Locator.FilterOptions()
        .setHas(page.locator("button")));
```

---

#### Q36: How to take screenshots in Playwright?
**Answer:**
```java
// Full page screenshot
page.screenshot(new Page.ScreenshotOptions()
    .setPath(Paths.get("screenshot.png"))
    .setFullPage(true));

// Element screenshot
page.locator("#logo").screenshot(new Locator.ScreenshotOptions()
    .setPath(Paths.get("logo.png")));

// Screenshot to buffer
byte[] buffer = page.screenshot();

// Mask elements (hide sensitive data)
page.screenshot(new Page.ScreenshotOptions()
    .setPath(Paths.get("masked.png"))
    .setMask(Arrays.asList(
        page.locator("#password"),
        page.locator("#ssn")
    )));

// Clip (specific area)
page.screenshot(new Page.ScreenshotOptions()
    .setPath(Paths.get("clip.png"))
    .setClip(100, 100, 500, 300));  // x, y, width, height
```

---

#### Q37: What is Trace Viewer in Playwright?
**Answer:**
**Trace Viewer** is a debugging tool that records test execution.

```java
// Start tracing
context.tracing().start(new Tracing.StartOptions()
    .setScreenshots(true)
    .setSnapshots(true)
    .setSources(true));

// Test steps
page.navigate("https://example.com");
page.click("#login");

// Stop and save trace
context.tracing().stop(new Tracing.StopOptions()
    .setPath(Paths.get("trace.zip")));

// View trace
// npx playwright show-trace trace.zip
```

**Features:**
- DOM snapshots at each action
- Screenshots
- Network logs
- Console logs
- Timeline
- Source code

**In your framework:** Used in PlaywrightExercise.java for debugging.

---

#### Q38: How to handle network requests in Playwright?
**Answer:**
```java
// Listen to all requests
page.onRequest(request -> {
    System.out.println("Request: " + request.url());
    System.out.println("Method: " + request.method());
});

// Listen to responses
page.onResponse(response -> {
    System.out.println("Response: " + response.url());
    System.out.println("Status: " + response.status());
});

// Route and modify requests
page.route("**/api/users", route -> {
    // Block request
    route.abort();
    
    // Or modify and continue
    route.continue_(new Route.ContinueOptions()
        .setUrl("https://modified-url.com"));
    
    // Or mock response
    route.fulfill(new Route.FulfillOptions()
        .setStatus(200)
        .setBody("{\"name\": \"Test User\"}"));
});

// Wait for specific request
page.waitForRequest("**/api/login", () -> {
    page.click("#submit");
});

// Wait for response
Response response = page.waitForResponse("**/api/data", () -> {
    page.click("#load");
});
```

---

#### Q39: How to handle file downloads in Playwright?
**Answer:**
```java
// Method 1: Wait for download
Download download = page.waitForDownload(() -> {
    page.click("#download-button");
});

// Get download details
String fileName = download.suggestedFilename();
String path = download.path();

// Save to specific location
download.saveAs(Paths.get("C:\\Downloads\\" + fileName));

// Delete after test
download.delete();

// Method 2: Listen to downloads
page.onDownload(download -> {
    System.out.println("Downloaded: " + download.suggestedFilename());
});
```

---

#### Q40: How to handle file uploads in Playwright?
**Answer:**
```java
// Single file
page.locator("#fileUpload").setInputFiles(Paths.get("file.txt"));

// Multiple files
page.locator("#multiUpload").setInputFiles(new Path[] {
    Paths.get("file1.txt"),
    Paths.get("file2.txt")
});

// From buffer
page.locator("#upload").setInputFiles(
    new FilePayload("file.txt", "text/plain", "File content".getBytes())
);

// Clear files
page.locator("#upload").setInputFiles(new Path[0]);

// File chooser (for dialog)
FileChooser fileChooser = page.waitForFileChooser(() -> {
    page.click("#uploadButton");
});
fileChooser.setFiles(Paths.get("file.txt"));
```

---

#### Q41: How to handle authentication in Playwright?
**Answer:**
```java
// Method 1: HTTP credentials
BrowserContext context = browser.newContext(
    new Browser.NewContextOptions()
        .setHttpCredentials("username", "password")
);

// Method 2: Storage state (save/reuse session)
// Login once and save
context.storageState(new BrowserContext.StorageStateOptions()
    .setPath(Paths.get("auth.json")));

// Reuse in other tests
BrowserContext context = browser.newContext(
    new Browser.NewContextOptions()
        .setStorageState(Paths.get("auth.json"))
);

// Method 3: Add cookies
List<Cookie> cookies = Arrays.asList(
    new Cookie("session", "abc123")
        .setDomain("example.com")
        .setPath("/")
);
context.addCookies(cookies);
```

---

#### Q42: How to emulate mobile devices in Playwright?
**Answer:**
```java
// Use device descriptor
Browser browser = playwright.chromium().launch();
BrowserContext context = browser.newContext(
    new Browser.NewContextOptions()
        .setUserAgent("Mozilla/5.0 (iPhone...)")
        .setViewportSize(375, 667)
        .setDeviceScaleFactor(2)
        .setIsMobile(true)
        .setHasTouch(true)
);

// Or use Playwright devices
import com.microsoft.playwright.options.Device;

BrowserContext context = browser.newContext(
    playwright.devices().get("iPhone 12")
);

// Available devices
// iPhone 12, iPhone 13, Pixel 5, Galaxy S8, iPad Pro, etc.
```

---

#### Q43: Explain Playwright's codegen feature.
**Answer:**
**Codegen** auto-generates test scripts by recording actions.

```bash
# Generate Java code
mvn exec:java -e -D exec.mainClass=com.microsoft.playwright.CLI -D exec.args="codegen example.com"

# With authentication
mvn exec:java -e -D exec.mainClass=com.microsoft.playwright.CLI -D exec.args="codegen --save-storage=auth.json example.com"

# With device emulation
mvn exec:java -e -D exec.mainClass=com.microsoft.playwright.CLI -D exec.args="codegen --device='iPhone 12' example.com"
```

**Generated code:**
```java
public class Example {
    public static void main(String[] args) {
        try (Playwright playwright = Playwright.create()) {
            Browser browser = playwright.chromium().launch(new BrowserType.LaunchOptions()
                .setHeadless(false));
            BrowserContext context = browser.newContext();
            Page page = context.newPage();
            page.navigate("https://example.com");
            page.click("#login");
            page.fill("#username", "test");
            page.fill("#password", "pass");
            page.click("button:has-text('Submit')");
        }
    }
}
```

---

#### Q44: How to handle frames in Playwright?
**Answer:**
```java
// Get frame by name
Frame frame = page.frame("frameName");

// Get frame by URL
Frame frame = page.frameByUrl("https://example.com/frame");

// Frame locator (better approach)
FrameLocator frameLocator = page.frameLocator("#myFrame");
frameLocator.locator("#button").click();

// Nested frames
page.frameLocator("#outerFrame")
    .frameLocator("#innerFrame")
    .locator("#button")
    .click();

// Wait for frame
page.waitForFrameNavigation(() -> {
    page.click("#loadFrame");
});

// All frames
List<Frame> frames = page.frames();
```

---

#### Q45: What are Playwright Assertions?
**Answer:**
**Playwright has built-in assertions** with auto-retry.

```java
import static com.microsoft.playwright.assertions.PlaywrightAssertions.*;

// Element assertions
assertThat(page.locator("#submit")).isVisible();
assertThat(page.locator("#submit")).isEnabled();
assertThat(page.locator("#result")).containsText("Success");
assertThat(page.locator("#title")).hasText("Welcome");
assertThat(page.locator("#input")).hasValue("Test");
assertThat(page.locator("#count")).hasCount(5);

// Page assertions
assertThat(page).hasTitle("Home Page");
assertThat(page).hasURL("https://example.com");

// Custom timeout
assertThat(page.locator("#slow"))
    .isVisible(new LocatorAssertions.IsVisibleOptions()
        .setTimeout(60000));

// NOT assertions
assertThat(page.locator("#hidden")).not().isVisible();
assertThat(page.locator("#disabled")).not().isEnabled();
```

**Advantage:** Auto-retry until condition is met (no manual waits).

---

### **Framework & BDD Questions (Q46-50)**

#### Q46: Explain Cucumber and BDD framework.
**Answer:**
**BDD (Behavior Driven Development)** uses natural language for tests.

**Components:**
- **Feature files (.feature)** - Test scenarios in Gherkin
- **Step Definitions (.java)** - Implementation
- **Runner** - Execute tests

```gherkin
# login.feature
Feature: Login functionality

  Scenario: Successful login
    Given I am on the login page
    When I enter username "testuser"
    And I enter password "password123"
    And I click login button
    Then I should see the dashboard
    And I should see welcome message "Welcome testuser"
```

```java
// Step Definitions
public class LoginSteps {
    
    @Given("I am on the login page")
    public void navigateToLogin() {
        driver.get("https://example.com/login");
    }
    
    @When("I enter username {string}")
    public void enterUsername(String username) {
        driver.findElement(By.id("username")).sendKeys(username);
    }
    
    @Then("I should see the dashboard")
    public void verifyDashboard() {
        assertTrue(driver.getCurrentUrl().contains("dashboard"));
    }
}
```

**In your framework:** Cucumber 7.19.0 with Picocontainer for DI.

---

#### Q47: How is your test framework structured?
**Answer:**
Based on your framework:

```
Framework Architecture:
├── src/main/java/com/siemens/xf
│   ├── api/ - API helper classes
│   ├── config/ - Configuration management
│   ├── constants/ - Constants
│   ├── ui/
│   │   ├── pages/ - Page Object classes
│   │   └── stepdefinitions/ - Cucumber step definitions
│   └── utils/ - Utility classes
├── src/test/java
│   ├── runners/ - CIRunner, LocalRunner
│   └── learning/ - SeleniumExercise, PlaywrightExercise
├── utaf/ - Core automation framework
├── features/ - Cucumber feature files
├── build.gradle - Dependencies
└── reports/ - Test reports
```

**Key Components:**
- **Page Objects:** LoginPage, DevConsolePage, AdminConsolePage
- **Step Definitions:** Login steps, navigation steps
- **Base Classes:** BasePageSelenium, BasePage
- **Utilities:** DriverFactory, configuration readers
- **Reports:** Cucumber HTML reports, screenshots

---

#### Q48: How do you manage test data in your framework?
**Answer:**
**Best practices implemented:**

```java
// 1. Properties files
// config.properties
env=test
base.url=https://test.example.com
timeout=30

// 2. Load properties
Properties props = new Properties();
props.load(new FileInputStream("config.properties"));
String url = props.getProperty("base.url");

// 3. Environment-based
String env = System.getProperty("env", "test");
if (env.equals("prod")) {
    baseUrl = "https://example.com";
} else {
    baseUrl = "https://test.example.com";
}

// 4. Data from Excel/CSV
// Use Apache POI or OpenCSV

// 5. Cucumber Examples (Data-driven)
Scenario Outline: Login with multiple users
    Given I am on login page
    When I login with "<username>" and "<password>"
    Then I should see "<message>"

Examples:
    | username | password | message |
    | user1    | pass1    | Success |
    | user2    | pass2    | Success |

// 6. JSON test data
String json = new String(Files.readAllBytes(Paths.get("testdata.json")));
JSONObject data = new JSONObject(json);
```

**In your framework:** Properties files in XceleratorTestAutomationConfig/.

---

#### Q49: How do you handle test reporting?
**Answer:**
**Multiple reporting mechanisms:**

```java
// 1. Cucumber Reports (Your framework)
@CucumberOptions(
    features = "src/test/resources/features",
    glue = "stepdefinitions",
    plugin = {
        "pretty",
        "html:reports/cucumber-html-reports",
        "json:reports/cucumber.json",
        "xml:reports/cucumber.xml"
    }
)

// 2. Cucumber-Reporting dependency
import net.masterthought.cucumber.Configuration;
import net.masterthought.cucumber.ReportBuilder;

Configuration config = new Configuration(new File("reports"), "ProjectName");
config.setBuildNumber("1.0");
config.addClassifications("Platform", "Windows");
config.addClassifications("Browser", "Chrome");

List<String> jsonFiles = Arrays.asList("reports/cucumber.json");
ReportBuilder reportBuilder = new ReportBuilder(jsonFiles, config);
reportBuilder.generateReports();

// 3. Screenshots on failure
@After
public void tearDown(Scenario scenario) {
    if (scenario.isFailed()) {
        byte[] screenshot = ((TakesScreenshot) driver)
            .getScreenshotAs(OutputType.BYTES);
        scenario.attach(screenshot, "image/png", "Screenshot");
    }
    driver.quit();
}

// 4. Extent Reports (alternative)
ExtentReports extent = new ExtentReports();
ExtentSparkReporter spark = new ExtentSparkReporter("ExtentReport.html");
extent.attachReporter(spark);

ExtentTest test = extent.createTest("LoginTest");
test.pass("Test passed");
test.log(Status.PASS, "Screenshot", 
    MediaEntityBuilder.createScreenCaptureFromPath("screenshot.png").build());
extent.flush();
```

**Your framework reports:** cucumber-html-reports/, OverviewReport.json

---

#### Q50: How do you handle CI/CD integration?
**Answer:**
**Best practices for CI/CD:**

```java
// 1. Headless browser
ChromeOptions options = new ChromeOptions();
options.addArguments("--headless");
options.addArguments("--no-sandbox");
options.addArguments("--disable-dev-shm-usage");

// 2. Gradle task (Your framework has ciTest)
task ciTest(type: Test) {
    maxHeapSize = "2048m"
    systemProperties System.getProperties()
    ignoreFailures = false
    include '**/CIRunner.class'
}

// 3. Jenkins Pipeline
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/repo.git'
            }
        }
        stage('Build') {
            steps {
                sh './gradlew clean build'
            }
        }
        stage('Test') {
            steps {
                sh './gradlew ciTest'
            }
        }
        stage('Report') {
            steps {
                cucumber reportTitle: 'Test Report',
                         fileIncludePattern: '**/cucumber.json'
            }
        }
    }
    post {
        always {
            junit '**/test-results/**/*.xml'
            archiveArtifacts 'reports/**'
        }
    }
}

// 4. GitHub Actions
name: Test Suite
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK
        uses: actions/setup-java@v2
        with:
          java-version: '11'
      - name: Run tests
        run: ./gradlew ciTest
      - name: Upload reports
        uses: actions/upload-artifact@v2
        with:
          name: test-reports
          path: reports/

// 5. Docker
FROM openjdk:11
RUN apt-get update && apt-get install -y \
    wget unzip chromium chromium-driver
COPY . /app
WORKDIR /app
CMD ["./gradlew", "ciTest"]
```

**In your framework:** run_ci.py and ciTest gradle task for CI execution.

---

## 🛠️ Practical Coding Challenges

### Challenge 1: Dynamic Table Handling
```java
// Extract all rows from a dynamic table
public List<Map<String, String>> extractTableData(WebDriver driver) {
    List<Map<String, String>> tableData = new ArrayList<>();
    
    // Get headers
    List<WebElement> headers = driver.findElements(
        By.xpath("//table//th"));
    List<String> headerNames = headers.stream()
        .map(WebElement::getText)
        .collect(Collectors.toList());
    
    // Get all rows
    List<WebElement> rows = driver.findElements(
        By.xpath("//table//tbody//tr"));
    
    for (WebElement row : rows) {
        Map<String, String> rowData = new HashMap<>();
        List<WebElement> cells = row.findElements(By.tagName("td"));
        
        for (int i = 0; i < cells.size(); i++) {
            rowData.put(headerNames.get(i), cells.get(i).getText());
        }
        tableData.add(rowData);
    }
    
    return tableData;
}
```

### Challenge 2: Wait for AJAX Call
```java
public void waitForAjaxToComplete(WebDriver driver) {
    JavascriptExecutor js = (JavascriptExecutor) driver;
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
    
    // Wait for jQuery
    wait.until(driver -> {
        return (Boolean) js.executeScript(
            "return typeof jQuery != 'undefined' && jQuery.active == 0"
        );
    });
    
    // Wait for document ready
    wait.until(driver -> {
        return js.executeScript("return document.readyState").equals("complete");
    });
}
```

### Challenge 3: Drag and Drop
```java
public void dragAndDrop(WebDriver driver, By source, By target) {
    WebElement sourceElement = driver.findElement(source);
    WebElement targetElement = driver.findElement(target);
    
    // Method 1: Actions class
    Actions actions = new Actions(driver);
    actions.dragAndDrop(sourceElement, targetElement).perform();
    
    // Method 2: JavaScript (if Actions fails)
    JavascriptExecutor js = (JavascriptExecutor) driver;
    js.executeScript(
        "function createEvent(typeOfEvent) {\n" +
        "    var event = document.createEvent('CustomEvent');\n" +
        "    event.initCustomEvent(typeOfEvent, true, true, null);\n" +
        "    return event;\n" +
        "}\n" +
        "var src = arguments[0], tgt = arguments[1];\n" +
        "src.dispatchEvent(createEvent('dragstart'));\n" +
        "tgt.dispatchEvent(createEvent('drop'));\n",
        sourceElement, targetElement
    );
}
```

### Challenge 4: Scroll to Element
```java
public void scrollToElement(WebDriver driver, WebElement element) {
    JavascriptExecutor js = (JavascriptExecutor) driver;
    
    // Method 1: scrollIntoView
    js.executeScript("arguments[0].scrollIntoView(true);", element);
    
    // Method 2: Scroll by pixel
    int y = element.getLocation().getY();
    js.executeScript("window.scrollTo(0, " + y + ")");
    
    // Method 3: Actions class
    Actions actions = new Actions(driver);
    actions.moveToElement(element).perform();
}
```

### Challenge 5: Wait for Element Clickable with Retry
```java
public void clickWithRetry(WebDriver driver, By locator, int maxAttempts) {
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    
    for (int attempt = 0; attempt < maxAttempts; attempt++) {
        try {
            WebElement element = wait.until(
                ExpectedConditions.elementToBeClickable(locator)
            );
            element.click();
            return;  // Success
        } catch (StaleElementReferenceException | 
                 ElementClickInterceptedException e) {
            System.out.println("Retry attempt: " + (attempt + 1));
            if (attempt == maxAttempts - 1) {
                throw e;  // Failed after all retries
            }
        }
    }
}
```

---

## 📚 Quick Reference

### Selenium vs Playwright Comparison
```
Use Selenium when:
✅ Need wide browser support (Safari, IE)
✅ Large existing Selenium codebase
✅ Team familiar with Selenium
✅ Cloud providers (BrowserStack, Sauce Labs)

Use Playwright when:
✅ Need fast execution
✅ Network control required
✅ Modern web apps (SPAs)
✅ Built-in tracing needed
✅ Starting new project
```

### Your Framework Usage
```java
// Selenium pages
public class MyPage extends BasePageSelenium {
    // Use WebDriver
}

// Playwright pages  
public class MyPage extends BasePage {
    // Use Playwright Page
}
```

---

## 🎓 Study Tips

1. **Practice on real sites:**
   - https://the-internet.herokuapp.com
   - https://demoqa.com
   - https://www.saucedemo.com

2. **Run your framework's learning exercises:**
   ```bash
   # Selenium exercises
   ./gradlew test --tests SeleniumExercise
   
   # Playwright exercises
   ./gradlew test --tests PlaywrightExercise
   ```

3. **Master these concepts:**
   - Page Object Model
   - Different wait strategies
   - Dynamic element handling
   - Framework architecture

4. **Common interview scenarios:**
   - Handle dynamic tables
   - Handle alerts/popups
   - Upload/download files
   - Parallel execution
   - CI/CD integration

---

## 🚀 Next Steps

1. Study your framework's code:
   - `src/main/java/com/siemens/xf/ui/pages/`
   - `src/main/java/com/siemens/xf/ui/stepdefinitions/`

2. Practice with learning files:
   - `src/test/java/com/learning/SeleniumExercise.java`
   - `src/test/java/com/learning/PlaywrightExercise.java`

3. Review framework configuration:
   - `build.gradle` - Dependencies
   - `utaf/build.gradle` - Core framework

Good luck with your interviews! 🎯

