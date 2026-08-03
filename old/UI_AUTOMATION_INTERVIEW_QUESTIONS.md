# UI Automation Interview Questions — Complete Guide

## Table of Contents
1. [Selenium Fundamentals](#1-selenium-fundamentals)
2. [WebDriver & Browser Internals](#2-webdriver--browser-internals)
3. [Locator Strategies](#3-locator-strategies)
4. [Waits & Synchronization](#4-waits--synchronization)
5. [Page Object Model (POM)](#5-page-object-model-pom)
6. [Frames, Windows & Alerts](#6-frames-windows--alerts)
7. [Actions & Advanced Interactions](#7-actions--advanced-interactions)
8. [Shadow DOM & Dynamic Elements](#8-shadow-dom--dynamic-elements)
9. [Cucumber BDD](#9-cucumber-bdd)
10. [Playwright](#10-playwright)
11. [Framework Design & Architecture](#11-framework-design--architecture)
12. [TestNG / JUnit](#12-testng--junit)
13. [CI/CD & Reporting](#13-cicd--reporting)
14. [Performance & Optimization](#14-performance--optimization)
15. [Scenario-Based / Coding Questions](#15-scenario-based--coding-questions)
16. [Tricky / Tough Questions](#16-tricky--tough-questions)

---

## 1. Selenium Fundamentals

### Q1: What is Selenium and what are its components?
**Answer:**
Selenium is an open-source browser automation tool suite with 4 components:

| Component | Purpose |
|-----------|---------|
| **Selenium WebDriver** | Core API to control browsers programmatically |
| **Selenium IDE** | Record & playback browser plugin (Chrome/Firefox) |
| **Selenium Grid** | Run tests in parallel on multiple machines/browsers |
| **Selenium Manager** | (New in Selenium 4.6+) Auto-manages browser drivers |

---

### Q2: What is the difference between Selenium 3 and Selenium 4?

| Feature | Selenium 3 | Selenium 4 |
|---------|-----------|-----------|
| Protocol | JSON Wire Protocol | W3C WebDriver Protocol (native) |
| Driver Manager | Needed WebDriverManager or manual download | Built-in Selenium Manager |
| Chrome DevTools | Not supported | CDP (Chrome DevTools Protocol) support |
| Relative Locators | Not available | `RelativeLocator.with()` — above, below, near, toLeftOf, toRightOf |
| Window/Tab | Workaround with JS | `driver.switchTo().newWindow(WindowType.TAB)` |
| Element Screenshot | Not available | `element.getScreenshotAs(OutputType.FILE)` |
| Shadow DOM | JavaScript only | `element.getShadowRoot()` native support |
| Grid | Hub + Node architecture | Standalone, Hub-Node, or Fully Distributed |
| Selenium IDE | Limited | Better export, resilient locators |

---

### Q3: What are the different types of locators in Selenium? Which is fastest?

**8 Locator Types (fastest → slowest):**

| Priority | Locator | Syntax | Speed |
|----------|---------|--------|-------|
| 1 | **ID** | `By.id("login")` | ⚡ Fastest |
| 2 | **Name** | `By.name("username")` | ⚡ Fast |
| 3 | **CSS Selector** | `By.cssSelector("input.form-control")` | ⚡ Fast |
| 4 | **Class Name** | `By.className("btn-primary")` | 🔵 Medium |
| 5 | **Tag Name** | `By.tagName("input")` | 🔵 Medium |
| 6 | **Link Text** | `By.linkText("Click Here")` | 🔵 Medium |
| 7 | **Partial Link Text** | `By.partialLinkText("Click")` | 🟡 Slower |
| 8 | **XPath** | `By.xpath("//input[@id='login']")` | 🔴 Slowest |

**Why ID is fastest:** Browser uses `document.getElementById()` which is O(1) hash lookup.
**Why XPath is slowest:** Browser must traverse the DOM tree.

---

### Q4: What is the difference between `findElement()` and `findElements()`?

| Feature | `findElement()` | `findElements()` |
|---------|-----------------|-------------------|
| Return Type | Single `WebElement` | `List<WebElement>` |
| No Match | Throws `NoSuchElementException` | Returns **empty list** (no exception) |
| Multiple Matches | Returns **first** match | Returns **all** matches |
| Use Case | Click a button, type text | Count elements, iterate table rows |

```java
// findElement — throws exception if not found
WebElement btn = driver.findElement(By.id("submit"));

// findElements — returns empty list if not found (safe)
List<WebElement> items = driver.findElements(By.className("item"));
if (items.isEmpty()) {
    System.out.println("No items found");
}
```

---

### Q5: What is the difference between `close()` and `quit()`?

| Method | Behavior |
|--------|----------|
| `driver.close()` | Closes **current** browser window/tab only |
| `driver.quit()` | Closes **all** windows + ends WebDriver session |

**Best Practice:** Always use `quit()` in `@After` hook to prevent memory leaks.

---

### Q6: What are the different exceptions in Selenium?

| Exception | When It Occurs |
|-----------|----------------|
| `NoSuchElementException` | Element not found in DOM |
| `StaleElementReferenceException` | Element was in DOM but page refreshed/changed |
| `ElementNotInteractableException` | Element exists but hidden/disabled/overlapped |
| `TimeoutException` | Wait condition not met within timeout |
| `ElementClickInterceptedException` | Another element covering the target |
| `NoSuchWindowException` | Window/tab was closed |
| `NoSuchFrameException` | Frame doesn't exist |
| `InvalidSelectorException` | Bad XPath/CSS syntax |
| `SessionNotCreatedException` | Browser/driver version mismatch |
| `WebDriverException` | Generic base exception |
| `NoAlertPresentException` | No alert/confirm/prompt dialog |
| `MoveTargetOutOfBoundsException` | Actions move to invalid coordinates |

---

### Q7: How do you handle `StaleElementReferenceException`?

**Why it happens:** DOM was modified after you found the element (page refresh, AJAX update, navigation).

```java
// Solution 1: Re-find the element
driver.findElement(By.id("btn")).click();  // StaleElement? → find again
driver.findElement(By.id("btn")).click();  // Fresh reference

// Solution 2: Explicit wait for staleness + re-find
wait.until(ExpectedConditions.stalenessOf(oldElement));
WebElement freshElement = driver.findElement(By.id("btn"));

// Solution 3: Retry mechanism
public void clickWithRetry(By locator, int maxRetries) {
    for (int i = 0; i < maxRetries; i++) {
        try {
            driver.findElement(locator).click();
            return;
        } catch (StaleElementReferenceException e) {
            if (i == maxRetries - 1) throw e;
        }
    }
}

// Solution 4: Use ExpectedConditions
wait.until(ExpectedConditions.elementToBeClickable(By.id("btn"))).click();
```

---

### Q8: What is the difference between `getText()`, `getAttribute()`, and `getCssValue()`?

```java
// HTML: <input id="email" value="john@test.com" class="form-input" style="color: red;">
//         Visible Label Text
// </input>

element.getText();                    // "Visible Label Text" — visible text on page
element.getAttribute("value");        // "john@test.com" — input field value
element.getAttribute("class");        // "form-input" — any HTML attribute
element.getCssValue("color");         // "rgba(255, 0, 0, 1)" — computed CSS property
element.getDomAttribute("value");     // Selenium 4: DOM attribute (not property)
element.getDomProperty("value");      // Selenium 4: JavaScript property
```

---

### Q9: How do you take a screenshot in Selenium?

```java
// Full page screenshot
File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(src, new File("screenshots/page.png"));

// Element screenshot (Selenium 4)
WebElement element = driver.findElement(By.id("chart"));
File src = element.getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(src, new File("screenshots/element.png"));

// As bytes (for Cucumber report attachment)
byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
scenario.attach(screenshot, "image/png", "failure-screenshot");
```

---

### Q10: How do you handle dropdowns in Selenium?

```java
// Standard HTML <select> dropdown
Select dropdown = new Select(driver.findElement(By.id("country")));
dropdown.selectByVisibleText("India");
dropdown.selectByValue("IN");
dropdown.selectByIndex(2);

// Get selected option
String selected = dropdown.getFirstSelectedOption().getText();

// Get all options
List<WebElement> options = dropdown.getOptions();

// Multi-select
dropdown.selectByVisibleText("Option1");
dropdown.selectByVisibleText("Option2");
dropdown.deselectAll();

// Custom dropdown (not <select> — div/ul based)
driver.findElement(By.cssSelector(".dropdown-toggle")).click();       // Open
driver.findElement(By.xpath("//li[text()='India']")).click();         // Select

// Searchable dropdown
WebElement search = driver.findElement(By.cssSelector(".search-input"));
search.sendKeys("Ind");
driver.findElement(By.xpath("//li[contains(text(),'India')]")).click();
```

---

## 2. WebDriver & Browser Internals

### Q11: How does Selenium WebDriver communicate with the browser?

```
┌──────────┐    HTTP/JSON     ┌──────────────┐    DevTools/     ┌─────────┐
│ Test Code │ ──────────────> │ Browser      │ ──────────────> │ Browser │
│ (Java)   │  W3C WebDriver  │ Driver       │  Protocol       │ (Chrome)│
│          │  Protocol       │ (chromedriver)│                 │         │
│          │ <────────────── │              │ <────────────── │         │
│          │   Response      │              │   Result        │         │
└──────────┘                 └──────────────┘                 └─────────┘
```

1. Test code sends HTTP request (POST/GET) to browser driver
2. Browser driver translates to browser-native commands
3. Browser executes action and returns result
4. Driver sends HTTP response back to test code

---

### Q12: What is the difference between `driver.get()` and `driver.navigate().to()`?

| Feature | `driver.get(url)` | `driver.navigate().to(url)` |
|---------|-------------------|----------------------------|
| Waits for page load | ✅ Yes | ✅ Yes |
| Behavior | Same as typing URL in address bar | Same behavior |
| Extra Methods | None | `.back()`, `.forward()`, `.refresh()` |
| Cookies/Session | Maintained | Maintained |

**Practical difference is minimal.** Use `navigate()` when you also need back/forward/refresh.

```java
driver.get("https://google.com");

driver.navigate().to("https://google.com");
driver.navigate().back();
driver.navigate().forward();
driver.navigate().refresh();
```

---

### Q13: How do you handle cookies in Selenium?

```java
// Get all cookies
Set<Cookie> cookies = driver.manage().getCookies();

// Get specific cookie
Cookie cookie = driver.manage().getCookieNamed("session_id");

// Add cookie
Cookie newCookie = new Cookie("token", "abc123");
driver.manage().addCookie(newCookie);

// Delete specific cookie
driver.manage().deleteCookieNamed("session_id");

// Delete all cookies
driver.manage().deleteAllCookies();
```

---

### Q14: What are the different ways to maximize/resize a browser window?

```java
driver.manage().window().maximize();
driver.manage().window().minimize();
driver.manage().window().fullscreen();
driver.manage().window().setSize(new Dimension(1920, 1080));
driver.manage().window().setPosition(new Point(0, 0));

// Headless with specific viewport
ChromeOptions options = new ChromeOptions();
options.addArguments("--headless=new", "--window-size=1920,1080");
```

---

## 3. Locator Strategies

### Q15: What are Relative Locators in Selenium 4?

```java
// Find element ABOVE another element
WebElement emailLabel = driver.findElement(
    RelativeLocator.with(By.tagName("label")).above(By.id("password"))
);

// Find element BELOW
WebElement rememberMe = driver.findElement(
    RelativeLocator.with(By.tagName("input")).below(By.id("password"))
);

// Find element to LEFT OF
WebElement firstName = driver.findElement(
    RelativeLocator.with(By.tagName("input")).toLeftOf(By.id("lastName"))
);

// Find element to RIGHT OF
WebElement lastName = driver.findElement(
    RelativeLocator.with(By.tagName("input")).toRightOf(By.id("firstName"))
);

// Find element NEAR (within 50px)
WebElement label = driver.findElement(
    RelativeLocator.with(By.tagName("label")).near(By.id("email"))
);

// Chained
WebElement target = driver.findElement(
    RelativeLocator.with(By.tagName("button"))
        .below(By.id("password"))
        .toRightOf(By.id("cancel"))
);
```

---

### Q16: CSS Selector vs XPath — When to use which?

| Criteria | CSS Selector | XPath |
|----------|-------------|-------|
| Speed | ⚡ Faster | 🔴 Slower |
| Traverse UP (parent) | ❌ Cannot | ✅ `parent::`, `ancestor::` |
| Traverse DOWN | ✅ Yes | ✅ Yes |
| Text-based search | ❌ Cannot | ✅ `text()`, `contains(text())` |
| Sibling navigation | Limited (+ ~) | ✅ `following-sibling`, `preceding-sibling` |
| Shadow DOM | ✅ Works inside `getShadowRoot()` | ❌ Doesn't work in shadow |
| Readability | 🔵 Cleaner | 🟡 Verbose |
| Browser support | All browsers consistent | Slight browser differences |

**Rule of thumb:**
- Use **CSS** for forward/downward traversal → faster
- Use **XPath** when you need text search, parent/ancestor navigation, or complex conditions

---

### Q17: Write XPath for these scenarios:

```xpath
# 1. Button with exact text
//button[text()='Submit']

# 2. Input next to label "Email"
//label[text()='Email']/following-sibling::input

# 3. Element with multiple classes
//div[contains(@class,'btn') and contains(@class,'primary')]

# 4. Nth row in table
//table[@id='data']//tbody/tr[3]

# 5. Edit button in row containing "John"
//tr[td[text()='John']]//button[text()='Edit']

# 6. Dynamic ID
//input[starts-with(@id,'user_')]
//input[contains(@id,'email')]

# 7. Element that is NOT disabled
//button[not(@disabled)]

# 8. Last element in a list
(//ul[@id='menu']/li)[last()]

# 9. Parent div of an input
//input[@id='email']/parent::div

# 10. Grandparent with specific class
//input[@id='email']/ancestor::div[@class='form-container']
```

---

## 4. Waits & Synchronization

### Q18: What are the types of waits in Selenium? ⭐ Most Asked

**Three types:**

#### 1. Implicit Wait (Global)
```java
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
// Applies to ALL findElement calls
// Polls DOM every 500ms until element found or timeout
// Set ONCE, applies everywhere
```

#### 2. Explicit Wait (Conditional)
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15));

// Wait for specific condition
WebElement btn = wait.until(ExpectedConditions.elementToBeClickable(By.id("submit")));
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("result")));
wait.until(ExpectedConditions.invisibilityOfElementLocated(By.id("loader")));
wait.until(ExpectedConditions.titleContains("Dashboard"));
wait.until(ExpectedConditions.urlContains("/home"));
wait.until(ExpectedConditions.alertIsPresent());
wait.until(ExpectedConditions.numberOfElementsToBe(By.className("item"), 5));
wait.until(ExpectedConditions.textToBePresentInElementLocated(By.id("msg"), "Success"));
wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt("myFrame"));
wait.until(ExpectedConditions.stalenessOf(oldElement));
```

#### 3. Fluent Wait (Custom polling)
```java
Wait<WebDriver> fluentWait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofMillis(250))         // Custom polling interval
    .ignoring(NoSuchElementException.class)        // Ignore specific exceptions
    .ignoring(StaleElementReferenceException.class)
    .withMessage("Element not found after 30s");   // Custom message

WebElement element = fluentWait.until(d -> d.findElement(By.id("dynamic-element")));
```

#### Comparison Table

| Feature | Implicit | Explicit | Fluent |
|---------|----------|----------|--------|
| Scope | Global (all elements) | Specific element/condition | Specific element/condition |
| Polling | 500ms (fixed) | 500ms (default) | Customizable |
| Condition | Element exists only | Any ExpectedCondition | Any custom condition |
| Exception Handling | No | No | Ignore specific exceptions |
| Best For | Simple apps | Most scenarios ✅ | Complex/flaky scenarios |

#### ❌ Why NOT to mix Implicit + Explicit?
```
Implicit = 10s, Explicit = 15s
If element not found → waits up to 10 + 15 = 25s (unpredictable!)
```
**Best Practice:** Use **Explicit Wait only**, set Implicit to 0.

---

### Q19: What is `Thread.sleep()` and why should you avoid it?

```java
Thread.sleep(5000); // ❌ BAD — Always waits 5 seconds even if element is ready in 1s
```

| | `Thread.sleep()` | Explicit Wait |
|---|---|---|
| Waits | Fixed time always | Only until condition met |
| If element ready in 1s | Still waits 5s | Returns in 1s ✅ |
| If element needs 6s | Fails (5s timeout) | Can set 10s timeout ✅ |
| Dynamic | ❌ No | ✅ Yes |
| Use in production | ❌ Never | ✅ Always |

**Only acceptable use:** Debugging or waiting for a non-DOM event (file download, email delivery).

---

### Q20: How do you wait for a page to fully load?

```java
// Method 1: Wait for document.readyState
wait.until(d -> ((JavascriptExecutor) d)
    .executeScript("return document.readyState").equals("complete"));

// Method 2: Wait for jQuery AJAX calls to finish
wait.until(d -> (Boolean) ((JavascriptExecutor) d)
    .executeScript("return jQuery.active == 0"));

// Method 3: Wait for Angular
wait.until(d -> (Boolean) ((JavascriptExecutor) d)
    .executeScript("return window.getAllAngularTestabilities().findIndex(t => !t.isStable()) === -1"));

// Method 4: Wait for specific element that appears after load
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("dashboard")));

// Method 5: Wait for spinner/loader to disappear
wait.until(ExpectedConditions.invisibilityOfElementLocated(By.className("spinner")));
```

---

## 5. Page Object Model (POM)

### Q21: What is Page Object Model? Why use it?

**POM** is a design pattern where each web page has a corresponding Java class containing:
- **Locators** (elements on that page)
- **Methods** (actions on those elements)

**Benefits:**
- ✅ Code reusability — same page used across tests
- ✅ Maintainability — if UI changes, update ONE class
- ✅ Readability — test reads like business logic
- ✅ Separation of concerns — locators separate from test logic

```java
// ❌ Without POM — locators scattered in test
@Test
public void loginTest() {
    driver.findElement(By.id("username")).sendKeys("admin");
    driver.findElement(By.id("password")).sendKeys("pass");
    driver.findElement(By.id("loginBtn")).click();
}

// ✅ With POM — clean, reusable
@Test
public void loginTest() {
    loginPage.enterUsername("admin");
    loginPage.enterPassword("pass");
    loginPage.clickLogin();
}
```

---

### Q22: What is Page Factory? Difference from POM?

| Feature | Page Object Model | Page Factory |
|---------|-------------------|-------------|
| Locator Style | `By.id("username")` | `@FindBy(id = "username")` |
| Initialization | Manual `driver.findElement()` | `PageFactory.initElements(driver, this)` |
| Lazy Loading | No | Yes (elements found when first used) |
| Cache | No | `@CacheLookup` annotation |

```java
public class LoginPage {
    @FindBy(id = "username")
    private WebElement usernameInput;

    @FindBy(css = "button.login-btn")
    private WebElement loginButton;

    @CacheLookup  // Element cached — won't re-find each time
    @FindBy(id = "logo")
    private WebElement logo;

    public LoginPage(WebDriver driver) {
        PageFactory.initElements(driver, this);
    }

    public void login(String user, String pass) {
        usernameInput.sendKeys(user);
        loginButton.click();
    }
}
```

**⚠️ `@CacheLookup` pitfall:** Don't use on dynamic elements (AJAX-loaded) — causes `StaleElementReferenceException`.

---

### Q23: Write a complete POM framework structure.

```java
// === BasePage.java ===
public abstract class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(15));
        PageFactory.initElements(driver, this);
    }

    protected void click(WebElement el) {
        wait.until(ExpectedConditions.elementToBeClickable(el)).click();
    }

    protected void type(WebElement el, String text) {
        wait.until(ExpectedConditions.visibilityOf(el));
        el.clear();
        el.sendKeys(text);
    }

    protected String getText(WebElement el) {
        return wait.until(ExpectedConditions.visibilityOf(el)).getText();
    }
}

// === LoginPage.java ===
public class LoginPage extends BasePage {
    @FindBy(id = "username") private WebElement usernameInput;
    @FindBy(id = "password") private WebElement passwordInput;
    @FindBy(id = "loginBtn") private WebElement loginBtn;

    public LoginPage(WebDriver driver) { super(driver); }

    public DashboardPage login(String user, String pass) {
        type(usernameInput, user);
        type(passwordInput, pass);
        click(loginBtn);
        return new DashboardPage(driver);  // Return next page
    }
}

// === DashboardPage.java ===
public class DashboardPage extends BasePage {
    @FindBy(css = ".welcome-msg") private WebElement welcomeMsg;

    public DashboardPage(WebDriver driver) { super(driver); }

    public String getWelcomeMessage() {
        return getText(welcomeMsg);
    }
}

// === Test (Step Definition) ===
@When("user logs in as {string} with {string}")
public void login(String user, String pass) {
    LoginPage loginPage = new LoginPage(driver);
    DashboardPage dashboard = loginPage.login(user, pass);
    String msg = dashboard.getWelcomeMessage();
    assertThat(msg).contains("Welcome");
}
```

---

## 6. Frames, Windows & Alerts

### Q24: How do you handle iFrames?

```java
// Switch by index
driver.switchTo().frame(0);

// Switch by name or ID
driver.switchTo().frame("frameName");

// Switch by WebElement
WebElement frameElement = driver.findElement(By.cssSelector("iframe#myFrame"));
driver.switchTo().frame(frameElement);

// Switch back to main page
driver.switchTo().defaultContent();

// Switch to parent frame (one level up in nested frames)
driver.switchTo().parentFrame();

// Wait for frame and switch
wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt("frameName"));

// Nested frames
driver.switchTo().frame("outerFrame");
driver.switchTo().frame("innerFrame");
// Do work inside inner frame
driver.switchTo().defaultContent();  // Back to main
```

---

### Q25: How do you handle multiple windows/tabs?

```java
// Get current window handle
String mainWindow = driver.getWindowHandle();

// Click link that opens new tab
driver.findElement(By.linkText("Open New Tab")).click();

// Get all window handles
Set<String> allWindows = driver.getWindowHandles();

// Switch to new window
for (String handle : allWindows) {
    if (!handle.equals(mainWindow)) {
        driver.switchTo().window(handle);
        break;
    }
}

// Do work in new window
System.out.println(driver.getTitle());

// Close new window & switch back
driver.close();  // Close current
driver.switchTo().window(mainWindow);

// Selenium 4: Open new tab/window directly
driver.switchTo().newWindow(WindowType.TAB);
driver.switchTo().newWindow(WindowType.WINDOW);
```

---

### Q26: How do you handle JavaScript alerts?

```java
// Simple Alert (OK button)
Alert alert = driver.switchTo().alert();
String alertText = alert.getText();
alert.accept();      // Click OK

// Confirm Alert (OK + Cancel)
Alert confirm = driver.switchTo().alert();
confirm.dismiss();   // Click Cancel
// OR
confirm.accept();    // Click OK

// Prompt Alert (Text input + OK/Cancel)
Alert prompt = driver.switchTo().alert();
prompt.sendKeys("My input text");
prompt.accept();

// Wait for alert
wait.until(ExpectedConditions.alertIsPresent());
```

---

## 7. Actions & Advanced Interactions

### Q27: What is the Actions class? Give examples.

```java
Actions actions = new Actions(driver);

// Hover / Mouse Over
actions.moveToElement(menuItem).perform();

// Double Click
actions.doubleClick(element).perform();

// Right Click (Context Click)
actions.contextClick(element).perform();

// Drag and Drop
actions.dragAndDrop(source, target).perform();
// OR
actions.clickAndHold(source).moveToElement(target).release().perform();

// Keyboard Actions
actions.keyDown(Keys.CONTROL).click(element1).click(element2).keyUp(Keys.CONTROL).perform();
actions.sendKeys(Keys.ENTER).perform();
actions.keyDown(Keys.SHIFT).sendKeys("hello").keyUp(Keys.SHIFT).perform(); // "HELLO"

// Scroll to Element (Selenium 4)
actions.scrollToElement(element).perform();
actions.scrollByAmount(0, 500).perform();  // Scroll down 500px

// Chain multiple actions
actions.moveToElement(menu)
       .pause(Duration.ofMillis(500))
       .click(submenuItem)
       .perform();
```

---

### Q28: How do you handle file upload and download?

```java
// === FILE UPLOAD ===
// Method 1: sendKeys to input[type="file"]
WebElement uploadInput = driver.findElement(By.cssSelector("input[type='file']"));
uploadInput.sendKeys("C:\\files\\test.pdf");

// Method 2: Robot class (for OS-level dialogs)
Robot robot = new Robot();
StringSelection filePath = new StringSelection("C:\\files\\test.pdf");
Toolkit.getDefaultToolkit().getSystemClipboard().setContents(filePath, null);
robot.keyPress(KeyEvent.VK_CONTROL);
robot.keyPress(KeyEvent.VK_V);
robot.keyRelease(KeyEvent.VK_V);
robot.keyRelease(KeyEvent.VK_CONTROL);
robot.keyPress(KeyEvent.VK_ENTER);
robot.keyRelease(KeyEvent.VK_ENTER);

// === FILE DOWNLOAD ===
// Set Chrome download directory
HashMap<String, Object> prefs = new HashMap<>();
prefs.put("download.default_directory", "C:\\downloads");
prefs.put("download.prompt_for_download", false);
ChromeOptions options = new ChromeOptions();
options.setExperimentalOption("prefs", prefs);
WebDriver driver = new ChromeDriver(options);

// Wait for file download
File file = new File("C:\\downloads\\report.pdf");
wait.until(d -> file.exists());
```

---

### Q29: How do you execute JavaScript in Selenium?

```java
JavascriptExecutor js = (JavascriptExecutor) driver;

// Scroll to bottom
js.executeScript("window.scrollTo(0, document.body.scrollHeight)");

// Scroll to element
js.executeScript("arguments[0].scrollIntoView(true);", element);

// Click hidden element
js.executeScript("arguments[0].click();", element);

// Set value (bypass readonly)
js.executeScript("arguments[0].value = 'new value';", element);

// Remove readonly attribute
js.executeScript("arguments[0].removeAttribute('readonly');", element);

// Get page title
String title = (String) js.executeScript("return document.title;");

// Highlight element (for debugging)
js.executeScript("arguments[0].style.border='3px solid red'", element);

// Wait for page load
js.executeScript("return document.readyState").equals("complete");

// Open new tab
js.executeScript("window.open('https://google.com', '_blank');");
```

---

## 8. Shadow DOM & Dynamic Elements

### Q30: How do you handle Shadow DOM elements?

```java
// Selenium 4 — getShadowRoot()
WebElement host = driver.findElement(By.cssSelector("my-component"));
SearchContext shadowRoot = host.getShadowRoot();
WebElement innerElement = shadowRoot.findElement(By.cssSelector("input#search"));

// Nested Shadow DOM
SearchContext shadow1 = driver.findElement(By.cssSelector("app-root")).getShadowRoot();
SearchContext shadow2 = shadow1.findElement(By.cssSelector("app-header")).getShadowRoot();
WebElement navBtn = shadow2.findElement(By.cssSelector("button.nav-toggle"));

// JavaScript approach (works for closed shadow roots)
WebElement el = (WebElement) ((JavascriptExecutor) driver).executeScript(
    "return document.querySelector('my-component').shadowRoot.querySelector('input#search')"
);

// ⚠️ IMPORTANT: XPath does NOT work inside Shadow DOM. Only CSS selectors work.
```

---

### Q31: How do you handle dynamic elements (elements that change on each page load)?

```java
// 1. Use partial attribute match
driver.findElement(By.xpath("//input[contains(@id, 'email')]"));
driver.findElement(By.xpath("//input[starts-with(@id, 'user_')]"));
driver.findElement(By.cssSelector("input[id*='email']"));
driver.findElement(By.cssSelector("input[id^='user_']"));

// 2. Use stable parent + relative path
driver.findElement(By.xpath("//div[@class='login-form']//input[1]"));

// 3. Use text content
driver.findElement(By.xpath("//button[text()='Submit']"));

// 4. Use data-testid (ask developers to add)
driver.findElement(By.cssSelector("[data-testid='submit-btn']"));

// 5. Explicit wait for dynamic element
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("dynamic-content")));
```

---

## 9. Cucumber BDD

### Q32: What is Cucumber and BDD?

**BDD (Behavior-Driven Development):** Write tests in plain English that non-technical stakeholders can read.

**Cucumber Components:**

| Component | File Type | Purpose |
|-----------|-----------|---------|
| Feature File | `.feature` | Business scenarios in Gherkin |
| Step Definitions | `.java` | Code behind each Gherkin step |
| Hooks | `.java` | `@Before` / `@After` setup/teardown |
| Runner | `.java` | Configures and runs tests |
| Tags | `@smoke` | Filter which tests to run |

---

### Q33: What is the difference between `Scenario` and `Scenario Outline`?

```gherkin
# Scenario — runs ONCE with fixed data
Scenario: Login with valid credentials
  Given user enters "admin" and "pass123"
  Then user should see dashboard

# Scenario Outline — runs MULTIPLE times with different data
Scenario Outline: Login with multiple users
  Given user enters "<username>" and "<password>"
  Then user should see "<result>"

  Examples:
    | username | password | result    |
    | admin    | pass123  | dashboard |
    | user1    | wrong    | error     |
    | ""       | ""       | error     |
# Runs 3 times — once per row
```

---

### Q34: What are Cucumber Hooks? What is the execution order?

```java
@Before(order = 0)   // Runs FIRST (lowest order)
public void setup() { }

@Before(order = 1)   // Runs SECOND
public void openBrowser() { }

@After(order = 1)    // Runs FIRST in after (highest order first!)
public void takeScreenshot() { }

@After(order = 0)    // Runs LAST in after
public void closeBrowser() { }

// Conditional hooks with tags
@Before("@ui")
public void beforeUI() { }

@Before("@api")
public void beforeAPI() { }

@After("@ui and not @playwright")
public void afterSeleniumUI() { }
```

**Execution order:**
```
@Before (order 0) → @Before (order 1) → @Given → @When → @Then → @After (order 1) → @After (order 0)
```

---

### Q35: What is the difference between `Background` and `@Before` hook?

| Feature | `Background` | `@Before` Hook |
|---------|-------------|----------------|
| Written in | `.feature` file (Gherkin) | `.java` file |
| Visible to | Business/non-technical people | Developers only |
| Scope | Per feature file | All scenarios (or filtered by tags) |
| Purpose | Common Given steps | Technical setup (driver, config) |

```gherkin
# Background — visible in feature file
Background:
  Given user is logged in
  And user is on dashboard page

Scenario: View profile
  When user clicks profile link
  Then profile page should display
```

---

### Q36: How do you share data between Cucumber steps?

```java
// Method 1: ScenarioContext (your framework uses this)
scenarioContext.set("userId", "42");
String id = scenarioContext.get("userId");

// Method 2: Instance variables in same step definition class
public class LoginSteps {
    private String token;  // Shared within this class only

    @Given("I get auth token")
    public void getToken() { token = "abc123"; }

    @Then("I use the token")
    public void useToken() { System.out.println(token); }
}

// Method 3: Dependency Injection (PicoContainer/Spring)
// Constructor injection — same object shared across ALL step def classes
public class StepDef1 {
    private SharedState state;
    public StepDef1(SharedState state) { this.state = state; }
}
public class StepDef2 {
    private SharedState state;
    public StepDef2(SharedState state) { this.state = state; }  // Same instance!
}
```

---

### Q37: What are Cucumber Tags? How do you use them?

```gherkin
@smoke @ui @regression
Feature: Login

  @P1 @sanity
  Scenario: Valid login
    ...

  @P2 @negative
  Scenario: Invalid login
    ...
```

**Run by tags:**
```bash
# Single tag
./gradlew test -Dcucumber.filter.tags="@smoke"

# AND
./gradlew test -Dcucumber.filter.tags="@smoke and @ui"

# OR
./gradlew test -Dcucumber.filter.tags="@smoke or @regression"

# NOT
./gradlew test -Dcucumber.filter.tags="not @skip"

# Combined
./gradlew test -Dcucumber.filter.tags="@ui and (@smoke or @P1) and not @broken"
```

---

## 10. Playwright

### Q38: Selenium vs Playwright — Key Differences?

| Feature | Selenium | Playwright |
|---------|----------|------------|
| Language Support | Java, Python, C#, Ruby, JS | Java, Python, C#, JS/TS |
| Speed | 🟡 Moderate | ⚡ Faster (direct CDP) |
| Auto-Wait | ❌ Manual waits needed | ✅ Built-in auto-wait |
| Shadow DOM | Manual `getShadowRoot()` | ✅ Auto-pierces |
| Parallel | Needs Grid or TestNG | Built-in browser contexts |
| Network Intercept | Limited (proxy) | ✅ Native `route()` API |
| Video Recording | Third-party tools | ✅ Built-in |
| Trace Viewer | No | ✅ Built-in debugging tool |
| iFrames | Manual `switchTo().frame()` | `frameLocator()` — no switching |
| Multiple tabs | Manual handle switching | Direct `page` objects |
| API Testing | Need RestAssured | Built-in `APIRequestContext` |
| Headless | `ChromeOptions("--headless")` | Default is headless |
| Protocol | W3C WebDriver (HTTP) | CDP/DevTools (WebSocket) — faster |
| Community | Very large, mature | Growing fast |

---

### Q39: What are Playwright locator strategies?

```java
// Recommended locators (priority order)
page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Submit"));
page.getByText("Welcome back");
page.getByLabel("Username");
page.getByPlaceholder("Enter your email");
page.getByTestId("login-btn");          // data-testid attribute
page.getByTitle("Close dialog");
page.getByAltText("Company Logo");

// CSS / XPath (still available)
page.locator("css=button.submit");
page.locator("xpath=//button[text()='Submit']");
page.locator("#username");               // CSS shorthand
page.locator("button:has-text('Submit')");

// Chained / Filtered
page.locator("div.card").filter(new Locator.FilterOptions().setHasText("John")).locator("button");
page.locator("tr").filter(new Locator.FilterOptions().setHas(page.locator("td:text('Admin')"))).locator("button.edit");

// Nth element
page.locator("div.card").nth(2);
page.locator("div.card").first();
page.locator("div.card").last();
```

---

### Q40: How does Playwright handle auto-waiting?

```java
// Playwright auto-waits for:
// ✅ Element to be attached to DOM
// ✅ Element to be visible
// ✅ Element to be stable (not animating)
// ✅ Element to be enabled
// ✅ Element to receive events

// No explicit waits needed!
page.locator("#submit").click();           // Auto-waits until clickable
page.locator("#username").fill("admin");   // Auto-waits until editable

// Assertions also auto-wait (with timeout)
assertThat(page.locator(".success")).isVisible();
assertThat(page.locator(".success")).hasText("Done");
assertThat(page).hasURL("https://example.com/dashboard");

// Custom wait
page.locator("#result").waitFor();
page.locator("#result").waitFor(new Locator.WaitForOptions().setState(WaitForSelectorState.VISIBLE).setTimeout(10000));
page.waitForURL("**/dashboard");
page.waitForLoadState(LoadState.NETWORKIDLE);
```

---

## 11. Framework Design & Architecture

### Q41: How did you design your automation framework?

**Answer Template:**

> "Our framework follows a **Hybrid BDD framework** using:
> - **Cucumber** for BDD scenarios
> - **Selenium/Playwright** for UI automation
> - **RestAssured** for API automation
> - **Page Object Model** for maintainability
> - **ScenarioContext** (via PicoContainer DI) for data sharing between steps
> - **Gradle** as build tool
> - **JUnit 5** as test runner
> - **Cucumber Reporting + Extent Reports** for reporting
> - **Selenium Grid / BrowserStack** for cross-browser testing
> - **Jenkins/GitHub Actions** for CI/CD
> - **ThreadLocal** for parallel execution safety"

**Architecture:**
```
Feature Files → Step Definitions → Page Objects → WebDriver/Playwright
                      ↕
               ScenarioContext (shared data)
                      ↕
               Config / TestData / API Helper
```

---

### Q42: How do you handle parallel execution?

```java
// ThreadLocal WebDriver — each thread gets its own driver
public class DriverFactory {
    private static ThreadLocal<WebDriver> driver = new ThreadLocal<>();

    public static WebDriver getDriver() { return driver.get(); }

    public static void setDriver(WebDriver d) { driver.set(d); }

    public static void quitDriver() {
        if (driver.get() != null) {
            driver.get().quit();
            driver.remove();   // Prevent memory leak!
        }
    }
}
```

```properties
# junit-platform.properties
cucumber.execution.parallel.enabled=true
cucumber.execution.parallel.config.strategy=fixed
cucumber.execution.parallel.config.fixed.parallelism=4
```

---

### Q43: How do you handle test data in your framework?

| Source | Use Case | Example |
|--------|----------|---------|
| **Feature file** | Small, scenario-specific data | DataTable, Examples |
| **JSON file** | API request templates | `api/create-user.json` |
| **Properties file** | Environment config, credentials | `staging.properties` |
| **Excel file** | Large data sets | Test data for data-driven |
| **Database** | Dynamic/live data | JDBC queries |
| **ScenarioContext** | Runtime data between steps | `set("token", value)` |
| **System properties** | CI/CD parameters | `-Denv=staging` |

---

### Q44: How do you handle test failures and retries?

```java
// Cucumber: Rerun failed scenarios
// Step 1: Generate rerun file
@ConfigurationParameter(key = PLUGIN_PROPERTY_NAME, 
    value = "rerun:reports/rerun.txt")

// Step 2: Create rerun runner
@ConfigurationParameter(key = FEATURES_PROPERTY_NAME, 
    value = "@reports/rerun.txt")   // @ reads from rerun file

// Retry with custom annotation
public void retryOnFailure(Runnable action, int maxRetries) {
    for (int i = 0; i < maxRetries; i++) {
        try {
            action.run();
            return;
        } catch (Exception e) {
            if (i == maxRetries - 1) throw e;
            System.out.println("Retry " + (i + 1) + " of " + maxRetries);
        }
    }
}
```

---

## 12. TestNG / JUnit

### Q45: JUnit 4 vs JUnit 5 for Cucumber?

| Feature | JUnit 4 + Cucumber | JUnit 5 + Cucumber |
|---------|-------------------|-------------------|
| Runner | `@RunWith(Cucumber.class)` | `@Suite` + `@IncludeEngines("cucumber")` |
| Annotations | `@CucumberOptions(...)` | `@ConfigurationParameter(...)` |
| Parallel | Limited | Built-in via `junit-platform.properties` |
| Config File | In annotation | `junit-platform.properties` |
| Dependency | `cucumber-junit` | `cucumber-junit-platform-engine` |

---

## 13. CI/CD & Reporting

### Q46: How do you integrate Selenium tests with CI/CD?

```
┌──────────────┐     ┌──────────────┐     ┌────────────────┐
│ Git Push /   │ ──> │ Jenkins /    │ ──> │ Run Tests      │
│ PR Created   │     │ GitHub Actions│    │ (Gradle/Maven) │
└──────────────┘     └──────────────┘     └───────┬────────┘
                                                   │
                           ┌───────────────────────┤
                           ▼                       ▼
                    ┌──────────────┐     ┌──────────────────┐
                    │ Reports      │     │ Notifications    │
                    │ (HTML/JSON)  │     │ (Slack/Email)    │
                    └──────────────┘     └──────────────────┘
```

**Typical CI command:**
```bash
./gradlew test -Dcucumber.filter.tags="@smoke" -Dbrowser=chrome-headless -Denv=staging
```

---

### Q47: What types of reports do you generate?

| Report | Plugin | Output |
|--------|--------|--------|
| Cucumber HTML | `html:reports/cucumber.html` | Basic HTML |
| Cucumber JSON | `json:reports/cucumber.json` | For downstream tools |
| Extent Report | `extentreports-cucumber7-adapter` | Rich HTML with charts |
| Cucumber Reporting | `net.masterthought:cucumber-reporting` | Dashboard with trends |
| Allure | `io.qameta.allure` | Interactive report |

---

## 14. Performance & Optimization

### Q48: How do you speed up Selenium tests?

1. **Parallel execution** — Run scenarios in parallel with JUnit 5
2. **Headless mode** — `--headless=new` (30-50% faster)
3. **Explicit waits only** — Remove `Thread.sleep()` and implicit waits
4. **Reuse browser session** — Don't restart browser between scenarios (when safe)
5. **API shortcuts** — Login via API, then set cookies instead of UI login
6. **Minimize locator usage** — Cache frequently used elements
7. **Use CSS over XPath** — CSS selectors are faster
8. **Optimize page loads** — Disable images/CSS for non-visual tests
9. **Use Selenium Grid** — Distribute across machines
10. **Clean test data** — Avoid stale data causing retries

```java
// Login via API instead of UI (saves ~5 seconds per test)
Response response = given().formParam("user", "admin").formParam("pass", "123")
    .post("/api/login");
Cookie cookie = new Cookie("session", response.getCookie("session"));
driver.manage().addCookie(cookie);
driver.navigate().refresh();
```

---

## 15. Scenario-Based / Coding Questions

### Q49: How would you automate a scenario where a table has 100+ rows and you need to find a specific row?

```java
// ❌ Slow — finds all rows, loops in Java
List<WebElement> rows = driver.findElements(By.xpath("//table//tr"));
for (WebElement row : rows) { ... }

// ✅ Fast — let XPath do the filtering
WebElement targetRow = driver.findElement(
    By.xpath("//table//tr[td[text()='John Doe']]")
);
WebElement editBtn = targetRow.findElement(By.xpath(".//button[text()='Edit']"));
editBtn.click();

// ✅ If paginated — combine with loop
while (true) {
    List<WebElement> matches = driver.findElements(
        By.xpath("//tr[td[text()='John Doe']]"));
    if (!matches.isEmpty()) {
        matches.get(0).findElement(By.xpath(".//button")).click();
        break;
    }
    WebElement nextBtn = driver.findElement(By.cssSelector("button.next"));
    if (!nextBtn.isEnabled()) break;
    nextBtn.click();
    wait.until(ExpectedConditions.stalenessOf(matches.get(0)));
}
```

---

### Q50: How do you handle a canvas or map element?

```java
// Canvas — no DOM elements inside, must use coordinates
Actions actions = new Actions(driver);
WebElement canvas = driver.findElement(By.id("myCanvas"));

// Click at specific position
actions.moveToElement(canvas, 100, 200).click().perform();

// Draw on canvas
actions.moveToElement(canvas, 50, 50)
       .clickAndHold()
       .moveByOffset(200, 100)
       .release()
       .perform();

// Verify canvas content via JavaScript
String pixelColor = (String) js.executeScript(
    "var canvas = document.getElementById('myCanvas');" +
    "var ctx = canvas.getContext('2d');" +
    "var pixel = ctx.getImageData(100, 200, 1, 1).data;" +
    "return 'rgb(' + pixel[0] + ',' + pixel[1] + ',' + pixel[2] + ')';");
```

---

### Q51: How do you handle authentication popups (HTTP Basic Auth)?

```java
// Method 1: URL with credentials
driver.get("https://username:password@example.com/secure");

// Method 2: Selenium 4 CDP (Chrome DevTools Protocol)
((HasAuthentication) driver).register(
    UsernameAndPassword.of("admin", "password123")
);
driver.get("https://example.com/secure");

// Method 3: AutoIT / Robot class for OS-level popup
Robot robot = new Robot();
robot.keyPress(KeyEvent.VK_TAB);
// ... type username/password
```

---

## 16. Tricky / Tough Questions

### Q52: What happens if you call `findElement()` before the page loads?

`NoSuchElementException` if no implicit wait is set. With implicit wait, it polls the DOM for the specified duration. **Best practice:** Use explicit wait.

---

### Q53: Can you find an element using multiple attributes?

```java
// XPath
driver.findElement(By.xpath("//input[@type='text' and @name='email']"));

// CSS
driver.findElement(By.cssSelector("input[type='text'][name='email']"));
```

---

### Q54: How to check if an element is present but NOT visible?

```java
// Present in DOM but hidden
List<WebElement> elements = driver.findElements(By.id("hiddenEl"));
boolean inDOM = !elements.isEmpty();                    // true
boolean visible = !elements.isEmpty() && elements.get(0).isDisplayed();  // false
```

---

### Q55: What is the difference between `isDisplayed()`, `isEnabled()`, and `isSelected()`?

| Method | Checks | Used For |
|--------|--------|----------|
| `isDisplayed()` | Element visible on page | Hidden/shown elements |
| `isEnabled()` | Element not disabled/grayed out | Buttons, inputs |
| `isSelected()` | Checkbox/radio is checked | Checkboxes, radio buttons, options |

```java
WebElement btn = driver.findElement(By.id("submit"));
btn.isDisplayed();  // true = visible on screen
btn.isEnabled();    // true = not grayed out / disabled
btn.isSelected();   // true = checked (for checkbox/radio)
```

---

### Q56: How to scroll to an element?

```java
// Method 1: JavaScript
((JavascriptExecutor) driver).executeScript("arguments[0].scrollIntoView(true);", element);

// Method 2: Actions (Selenium 4)
new Actions(driver).scrollToElement(element).perform();

// Method 3: Scroll by amount
new Actions(driver).scrollByAmount(0, 1000).perform();

// Method 4: Keys
element.sendKeys(Keys.PAGE_DOWN);
```

---

### Q57: How do you handle lazy-loaded / infinite scroll pages?

```java
JavascriptExecutor js = (JavascriptExecutor) driver;
long lastHeight = (long) js.executeScript("return document.body.scrollHeight");

while (true) {
    js.executeScript("window.scrollTo(0, document.body.scrollHeight)");
    Thread.sleep(2000); // Wait for new content to load

    long newHeight = (long) js.executeScript("return document.body.scrollHeight");
    if (newHeight == lastHeight) break;  // No more content
    lastHeight = newHeight;
}
```

---

### Q58: `driver.getTitle()` vs `driver.getCurrentUrl()` — when do they fail?

Both return empty string or stale data if called **before page loads**. Always wait:

```java
wait.until(ExpectedConditions.titleContains("Dashboard"));
wait.until(ExpectedConditions.urlContains("/dashboard"));
```

---

### Q59: How do you handle CAPTCHA in automation?

| Approach | How |
|----------|-----|
| **Disable in test env** | Ask dev to disable CAPTCHA in staging ✅ Best |
| **Bypass via API** | Use API to get session, skip CAPTCHA |
| **Test key** | Google reCAPTCHA provides test site keys |
| **Cookie injection** | Set auth cookie to bypass login + CAPTCHA |
| **Third-party service** | 2Captcha, Anti-Captcha (not recommended) |

**You should NEVER automate CAPTCHA solving in production.**

---

### Q60: Tell me about a challenging automation problem you solved.

**Answer Template:**

> **Situation:** "We had a dashboard with complex Shadow DOM components inside web components. Normal Selenium locators couldn't reach elements inside shadow roots."
>
> **Task:** "I needed to automate end-to-end tests for user management which involved nested shadow DOM (3 levels deep)."
>
> **Action:** "I created a reusable `ShadowDomHelper` utility class with methods like `findInNestedShadow()` that chains `getShadowRoot()` calls. For Playwright tests, I leveraged its auto-piercing capability with the `>>` operator. I also added wait mechanisms since shadow DOM elements load asynchronously."
>
> **Result:** "This reduced our shadow DOM test code by 60%, and the utility was adopted by other teams. Test execution time improved by 40% after switching to Playwright for shadow-heavy components."

---

## Quick Interview Prep Checklist

| Topic | Must-Know Questions |
|-------|-------------------|
| Selenium Basics | Q1-Q10 |
| Locators & XPath | Q15-Q17 |
| Waits (most asked!) | Q18-Q20 |
| POM | Q21-Q23 |
| Frames/Windows/Alerts | Q24-Q26 |
| Actions | Q27-Q29 |
| Shadow DOM | Q30-Q31 |
| Cucumber | Q32-Q37 |
| Playwright | Q38-Q40 |
| Framework Design | Q41-Q44 |
| Scenario-Based | Q49-Q57 |
| Tricky | Q52-Q60 |

