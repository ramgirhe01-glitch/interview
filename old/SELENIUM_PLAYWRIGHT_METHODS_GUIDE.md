# Selenium & Playwright — Hierarchy, Methods & Interview Guide

---

## 1. Class Hierarchy

### Selenium

```
DriverFactory (ThreadLocal<WebDriver>)
  └── BasePage (abstract - reusable actions: click, type, wait)
        ├── LoginPage
        ├── DashboardPage
        └── ...other Page Objects
```

### Playwright

```
PlaywrightFactory (ThreadLocal<Page>)
  └── BasePWPage (abstract - reusable actions)
        ├── PWLoginPage
        ├── PWDashboardPage
        └── ...other Page Objects
```

---

## 2. Selenium — How to Call Methods

### 2.1 DriverFactory

```java
// Initialize driver
DriverFactory.initDriver("chrome");

// Get driver anywhere in the framework
WebDriver driver = DriverFactory.getDriver();

// Navigate
DriverFactory.getDriver().get("https://example.com");

// Quit
DriverFactory.quitDriver();
```

### 2.2 BasePage — Reusable Actions (Most Used)

```java
// Every Page Object extends BasePage
public class LoginPage extends BasePage {

    @FindBy(id = "username") WebElement usernameField;
    @FindBy(id = "password") WebElement passwordField;
    @FindBy(id = "loginBtn") WebElement loginBtn;

    public LoginPage() {
        PageFactory.initElements(DriverFactory.getDriver(), this);
    }

    public void login(String user, String pass) {
        type(usernameField, user);       // BasePage method
        type(passwordField, pass);       // BasePage method
        click(loginBtn);                 // BasePage method
    }
}
```

### 2.3 BasePage Common Methods

```java
public abstract class BasePage {
    protected WebDriver driver = DriverFactory.getDriver();
    protected WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15));

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
        try { return element.isDisplayed(); } catch (Exception e) { return false; }
    }

    protected void selectDropdown(WebElement element, String value) {
        new Select(element).selectByVisibleText(value);
    }

    protected void jsClick(WebElement element) {
        ((JavascriptExecutor) driver).executeScript("arguments[0].click();", element);
    }

    protected void scrollTo(WebElement element) {
        ((JavascriptExecutor) driver).executeScript("arguments[0].scrollIntoView(true);", element);
    }

    protected byte[] takeScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }
}
```

### 2.4 Step Definition — How Page Objects Are Called

```java
public class LoginSteps {
    LoginPage loginPage = new LoginPage();
    DashboardPage dashboardPage = new DashboardPage();

    @Given("user navigates to {string}")
    public void navigateTo(String url) {
        DriverFactory.getDriver().get(url);
    }

    @When("user logs in with {string} and {string}")
    public void login(String user, String pass) {
        loginPage.login(user, pass);
    }

    @Then("user should see dashboard")
    public void verifyDashboard() {
        Assert.assertTrue(dashboardPage.isDashboardVisible());
    }
}
```

### 2.5 Most Used WebDriver Calls

```java
WebDriver driver = DriverFactory.getDriver();

// Navigation
driver.get(url);
driver.getCurrentUrl();
driver.getTitle();
driver.navigate().back();
driver.navigate().forward();
driver.navigate().refresh();

// Find & Interact
driver.findElement(By.id("x")).click();
driver.findElement(By.cssSelector(".x")).sendKeys("text");
driver.findElements(By.tagName("li")).size();

// Switch Context
driver.switchTo().frame("frameName");
driver.switchTo().frame(0);
driver.switchTo().defaultContent();
driver.switchTo().window(handle);
driver.switchTo().alert().accept();
driver.switchTo().alert().dismiss();
driver.switchTo().alert().getText();

// Window Handles
driver.getWindowHandle();
driver.getWindowHandles();

// JavaScript Execution
((JavascriptExecutor) driver).executeScript("arguments[0].click();", element);
((JavascriptExecutor) driver).executeScript("return document.title;");

// Actions Class
Actions actions = new Actions(driver);
actions.moveToElement(element).click().perform();
actions.doubleClick(element).perform();
actions.contextClick(element).perform();
actions.dragAndDrop(source, target).perform();
actions.sendKeys(Keys.ENTER).perform();
```

### 2.6 Most Used Waits

```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15));

wait.until(ExpectedConditions.visibilityOf(element));
wait.until(ExpectedConditions.elementToBeClickable(element));
wait.until(ExpectedConditions.invisibilityOf(loader));
wait.until(ExpectedConditions.urlContains("/dashboard"));
wait.until(ExpectedConditions.titleContains("Home"));
wait.until(ExpectedConditions.textToBePresentInElement(el, "Success"));
wait.until(ExpectedConditions.presenceOfElementLocated(By.id("x")));
wait.until(ExpectedConditions.numberOfElementsToBe(By.css(".item"), 5));
wait.until(ExpectedConditions.alertIsPresent());
wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt("frame"));
```

---

## 3. Playwright — How to Call Methods

### 3.1 PlaywrightFactory

```java
// Initialize
PlaywrightFactory.initBrowser("chromium");

// Get page anywhere
Page page = PlaywrightFactory.getPage();

// Close
PlaywrightFactory.closeBrowser();
```

### 3.2 Page Object

```java
public class PWLoginPage {
    private Page page;

    public PWLoginPage() {
        this.page = PlaywrightFactory.getPage();
    }

    public void login(String user, String pass) {
        page.fill("#username", user);
        page.fill("#password", pass);
        page.click("#loginBtn");
    }

    public String getErrorMessage() {
        return page.textContent(".error-msg");
    }
}
```

### 3.3 Most Used Page Calls

```java
Page page = PlaywrightFactory.getPage();

// Navigation
page.navigate(url);
page.url();
page.title();
page.goBack();
page.goForward();
page.reload();

// Actions (auto-wait built-in)
page.fill("#username", "admin");          // clear + type
page.click("#loginBtn");
page.dblclick("#item");
page.check("#checkbox");
page.uncheck("#checkbox");
page.selectOption("#dropdown", "value");
page.hover(".menu-item");
page.type("#search", "text");             // types character by character
page.press("#input", "Enter");

// Reading
page.textContent(".message");
page.innerText(".message");
page.innerHTML(".container");
page.getAttribute("#link", "href");
page.inputValue("#username");
page.isVisible(".element");
page.isEnabled("#btn");
page.isChecked("#checkbox");

// Locator API (PREFERRED)
Locator btn = page.locator("#loginBtn");
btn.click();
btn.fill("text");
btn.isVisible();
btn.count();
btn.first().click();
btn.nth(2).click();
btn.allTextContents();
page.locator("text=Submit").click();
page.locator("[data-testid='login']").click();

// Waiting
page.waitForSelector(".loaded");
page.waitForURL("**/dashboard");
page.waitForLoadState();
page.waitForResponse("**/api/login");
page.waitForTimeout(1000);                // hard wait (avoid)

// Frames
page.frameLocator("#iframe").locator("#btn").click();

// Screenshot
page.screenshot(new Page.ScreenshotOptions().setPath(Paths.get("screenshot.png")));
page.locator("#element").screenshot();

// Evaluate JS
page.evaluate("document.title");
```

---

## 4. Side-by-Side Comparison

| Action | Selenium | Playwright |
|---|---|---|
| Navigate | `driver.get(url)` | `page.navigate(url)` |
| Click | `element.click()` | `page.click("#id")` |
| Type | `element.clear(); element.sendKeys("x")` | `page.fill("#id", "x")` |
| Get text | `element.getText()` | `page.textContent("#id")` |
| Get URL | `driver.getCurrentUrl()` | `page.url()` |
| Is visible | `element.isDisplayed()` | `page.isVisible("#id")` |
| Wait for element | `wait.until(visibilityOf(el))` | `page.waitForSelector("#id")` |
| Dropdown | `new Select(el).selectByVisibleText("x")` | `page.selectOption("#id", "x")` |
| Frame | `driver.switchTo().frame("f")` | `page.frameLocator("#f")` |
| Screenshot | `((TakesScreenshot)driver).getScreenshotAs(...)` | `page.screenshot()` |
| JS execute | `((JavascriptExecutor)driver).executeScript(...)` | `page.evaluate(...)` |
| Hover | `new Actions(driver).moveToElement(el).perform()` | `page.hover("#id")` |
| Drag & drop | `new Actions(driver).dragAndDrop(s, t).perform()` | `page.dragAndDrop("#src", "#tgt")` |

> **Key Difference**: Playwright auto-waits before every action. Selenium requires explicit `WebDriverWait`.

---

## 5. Interview Questions & Answers

### Selenium

**Q1: What is the difference between `findElement()` and `findElements()`?**
- `findElement()` → returns first matching `WebElement`, throws `NoSuchElementException` if not found.
- `findElements()` → returns `List<WebElement>`, returns empty list if not found (no exception).

**Q2: What are the different types of waits in Selenium?**
- **Implicit Wait**: `driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));` — applies globally to all findElement calls.
- **Explicit Wait**: `new WebDriverWait(driver, Duration.ofSeconds(15)).until(...)` — waits for a specific condition.
- **Fluent Wait**: Explicit wait + custom polling interval + ignore exceptions.
- **Thread.sleep()**: Hard wait — **avoid in frameworks**.

**Q3: How do you handle dynamic elements?**
- Use explicit waits with `ExpectedConditions`.
- Use dynamic XPath/CSS: `//div[contains(@id,'partial')]`
- Use `WebDriverWait` + `presenceOfElementLocated`.

**Q4: How do you handle alerts?**
```java
Alert alert = driver.switchTo().alert();
alert.getText();
alert.accept();
alert.dismiss();
alert.sendKeys("text");
```

**Q5: How do you handle multiple windows/tabs?**
```java
String parent = driver.getWindowHandle();
Set<String> all = driver.getWindowHandles();
for (String handle : all) {
    if (!handle.equals(parent)) {
        driver.switchTo().window(handle);
    }
}
// switch back
driver.switchTo().window(parent);
```

**Q6: How do you handle frames?**
```java
driver.switchTo().frame("name");       // by name
driver.switchTo().frame(0);            // by index
driver.switchTo().frame(element);      // by WebElement
driver.switchTo().defaultContent();    // back to main
driver.switchTo().parentFrame();       // one level up
```

**Q7: What is PageFactory? Why use it?**
- `PageFactory.initElements(driver, this)` initializes `@FindBy` annotated elements lazily.
- Cleaner code, separation of locators from actions.

**Q8: How do you handle stale element exception?**
- Re-find the element before interacting.
- Use explicit wait: `wait.until(ExpectedConditions.stalenessOf(element))` then re-locate.
- Use try-catch with retry logic.

**Q9: What is the difference between `driver.close()` and `driver.quit()`?**
- `close()` → closes current window only.
- `quit()` → closes all windows and ends the WebDriver session.

**Q10: How do you scroll in Selenium?**
```java
// Scroll to element
((JavascriptExecutor) driver).executeScript("arguments[0].scrollIntoView(true);", element);
// Scroll by pixels
((JavascriptExecutor) driver).executeScript("window.scrollBy(0, 500);");
// Scroll to bottom
((JavascriptExecutor) driver).executeScript("window.scrollTo(0, document.body.scrollHeight);");
```

**Q11: How do you handle dropdowns?**
```java
Select select = new Select(driver.findElement(By.id("dropdown")));
select.selectByVisibleText("Option 1");
select.selectByValue("opt1");
select.selectByIndex(0);
select.getOptions();               // all options
select.getFirstSelectedOption();   // currently selected
```

**Q12: How do you upload a file in Selenium?**
```java
driver.findElement(By.id("upload")).sendKeys("C:\\path\\to\\file.txt");
```

**Q13: What is ThreadLocal in DriverFactory? Why?**
- `ThreadLocal<WebDriver>` ensures each thread (parallel test) gets its own driver instance.
- Prevents thread interference in parallel execution.

**Q14: How do you take a screenshot on failure?**
```java
byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
// In Cucumber: scenario.attach(screenshot, "image/png", "failure");
```

**Q15: What locator strategies do you use (in order of preference)?**
1. `By.id` — fastest, unique
2. `By.cssSelector` — fast, flexible
3. `By.xpath` — most powerful, handles text/parent traversal
4. `By.name`, `By.className`, `By.linkText` — situational

---

### Playwright

**Q1: What is the main advantage of Playwright over Selenium?**
- **Auto-wait**: Every action waits for the element to be actionable (visible, stable, enabled).
- Built-in support for frames, multiple tabs, network interception.
- Faster execution — communicates via WebSocket (not HTTP like Selenium).

**Q2: What is the Locator API?**
- `page.locator("#id")` returns a `Locator` — a lazy reference that re-queries every time.
- Prevents stale element issues (no StaleElementReferenceException).
- Preferred over `page.querySelector()`.

**Q3: How does Playwright handle frames?**
```java
// No switchTo needed!
page.frameLocator("#iframe").locator("#button").click();
```

**Q4: How does Playwright handle multiple pages/tabs?**
```java
Page newPage = context.waitForPage(() -> {
    page.click("#openNewTab");
});
newPage.waitForLoadState();
newPage.fill("#input", "text");
```

**Q5: How do you intercept network requests in Playwright?**
```java
page.route("**/api/login", route -> {
    route.fulfill(new Route.FulfillOptions()
        .setStatus(200)
        .setBody("{\"token\": \"fake\"}"));
});
```

**Q6: How do you wait in Playwright?**
- **Auto-wait** handles most cases.
- `page.waitForSelector(".element")` — wait for DOM element.
- `page.waitForURL("**/dashboard")` — wait for navigation.
- `page.waitForResponse("**/api/data")` — wait for API response.
- `page.waitForLoadState()` — wait for page load.

**Q7: What browsers does Playwright support?**
- Chromium, Firefox, WebKit (Safari engine) — all bundled, no separate driver needed.

**Q8: How do you run tests in headed/headless mode?**
```java
Browser browser = playwright.chromium().launch(
    new BrowserType.LaunchOptions().setHeadless(false)  // headed
);
```

**Q9: What is BrowserContext in Playwright?**
- Isolated browser session (like incognito).
- Each context has its own cookies, storage, cache.
- Enables parallel tests in the same browser instance.

**Q10: How do you handle file upload in Playwright?**
```java
page.setInputFiles("#upload", Paths.get("file.txt"));
```

---

### Comparison Interview Questions

**Q: When would you choose Playwright over Selenium?**
| Factor | Selenium | Playwright |
|---|---|---|
| Auto-wait | ❌ Manual | ✅ Built-in |
| Speed | Slower (HTTP) | Faster (WebSocket) |
| Browser install | Manual drivers | Auto-bundled |
| Parallel | Via Grid/TestNG | Via BrowserContext |
| Network mock | ❌ Needs proxy | ✅ Built-in |
| Community | Huge, mature | Growing fast |
| Language support | Java, Python, C#, JS, Ruby | Java, Python, C#, JS |
| Legacy browser | ✅ IE support | ❌ Modern only |

**Q: What is the biggest pain point in Selenium that Playwright solves?**
- Flaky tests due to timing — Playwright's auto-wait eliminates most `WebDriverWait` boilerplate.
- No need for driver management (ChromeDriver, GeckoDriver) — Playwright bundles browsers.
- Native frame/tab handling without `switchTo()`.

---

## 6. Quick Cheat Sheet — Copy & Use

### Selenium (5 most used lines)
```java
WebDriver driver = DriverFactory.getDriver();
driver.get("https://example.com");
new WebDriverWait(driver, Duration.ofSeconds(10)).until(ExpectedConditions.visibilityOf(el));
element.click();
element.sendKeys("text");
```

### Playwright (5 most used lines)
```java
Page page = PlaywrightFactory.getPage();
page.navigate("https://example.com");
page.fill("#username", "admin");
page.click("#loginBtn");
page.textContent(".message");
```

---

*Last updated: April 28, 2026*

