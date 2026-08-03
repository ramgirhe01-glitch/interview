# 🎯 Quick Interview Cheat Sheet - Selenium & Playwright

## 📌 One-Liners (Memorize These!)

### Selenium Quick Facts
```
Version in your framework: 4.25.0
Architecture: WebDriver Protocol → Browser Driver → Browser
Languages: Java, Python, C#, JavaScript, Ruby
Browsers: Chrome, Firefox, Edge, Safari, Opera
Components: WebDriver, IDE, Grid
```

### Playwright Quick Facts
```
Version in your framework: 1.52.0
Architecture: CDP/DevTools Protocol → Direct browser control
Languages: Java, JavaScript, Python, .NET
Browsers: Chromium, Firefox, WebKit
Key Feature: Auto-waiting (no explicit waits needed!)
```

---

## 🔥 Most Asked Interview Questions (Quick Answers)

### Q: What is the difference between findElement() and findElements()?
**A:** `findElement()` returns first WebElement or throws exception. `findElements()` returns List<WebElement> or empty list.

### Q: Types of waits in Selenium?
**A:** 
- **Implicit:** Global timeout for all elements
- **Explicit:** Wait for specific condition
- **Fluent:** Explicit + polling interval + ignore exceptions

### Q: What is Page Object Model?
**A:** Design pattern where each page is a class, elements are variables, actions are methods. Benefits: reusability, maintainability.

### Q: How to handle dynamic elements?
**A:** Use dynamic XPath/CSS with contains(), starts-with(), text(), following-sibling, parent axes.

### Q: What's new in Selenium 4?
**A:** W3C protocol, relative locators, Chrome DevTools Protocol, new window/tab management, upgraded Grid.

### Q: Selenium vs Playwright?
**A:** 
- Selenium: Mature, wide support, slower
- Playwright: Modern, auto-waiting, faster, better network control

### Q: What is Browser Context in Playwright?
**A:** Isolated browser session (like incognito) with separate cookies/storage. Enables parallel execution and test isolation.

### Q: How to handle alerts?
**A:** `Alert alert = driver.switchTo().alert(); alert.accept()/dismiss()/getText()/sendKeys()`

### Q: How to switch windows?
**A:** `String main = driver.getWindowHandle(); Set<String> all = driver.getWindowHandles(); driver.switchTo().window(handle);`

### Q: How to handle dropdowns?
**A:** `Select select = new Select(element); select.selectByVisibleText()/selectByValue()/selectByIndex()`

---

## 📝 Code Snippets (Copy-Paste Ready)

### Selenium Basic Setup
```java
ChromeOptions options = new ChromeOptions();
options.addArguments("--remote-allow-origins=*");
WebDriver driver = new ChromeDriver(options);
driver.manage().window().maximize();
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
```

### Playwright Basic Setup
```java
try (Playwright playwright = Playwright.create()) {
    Browser browser = playwright.chromium().launch(
        new BrowserType.LaunchOptions().setHeadless(false));
    BrowserContext context = browser.newContext();
    Page page = context.newPage();
    page.navigate("https://example.com");
}
```

### All Selenium Locators
```java
By.id("username")
By.name("password")
By.className("btn-primary")
By.tagName("input")
By.linkText("Click Here")
By.partialLinkText("Click")
By.cssSelector("#username")
By.xpath("//input[@id='username']")
```

### Playwright Recommended Locators
```java
page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Submit"))
page.getByTestId("submit-button")
page.getByLabel("Username")
page.getByPlaceholder("Enter email")
page.getByText("Click me")
page.locator("#username")  // fallback
```

### Explicit Wait (Selenium)
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
WebElement element = wait.until(
    ExpectedConditions.visibilityOfElementLocated(By.id("result"))
);
```

### Fluent Wait (Selenium)
```java
FluentWait<WebDriver> wait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofMillis(500))
    .ignoring(NoSuchElementException.class);
WebElement element = wait.until(driver -> driver.findElement(By.id("result")));
```

### Actions Class
```java
Actions actions = new Actions(driver);
actions.moveToElement(element).perform();
actions.dragAndDrop(source, target).perform();
actions.contextClick(element).perform();
actions.doubleClick(element).perform();
```

### JavaScript Executor
```java
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("arguments[0].click();", element);
js.executeScript("arguments[0].scrollIntoView(true);", element);
js.executeScript("window.scrollTo(0, document.body.scrollHeight);");
```

### Handle Alerts
```java
Alert alert = driver.switchTo().alert();
String text = alert.getText();
alert.accept();
alert.dismiss();
alert.sendKeys("input");
```

### Switch Windows
```java
String mainWindow = driver.getWindowHandle();
Set<String> allWindows = driver.getWindowHandles();
for (String window : allWindows) {
    if (!window.equals(mainWindow)) {
        driver.switchTo().window(window);
    }
}
```

### Handle Frames
```java
driver.switchTo().frame(0);
driver.switchTo().frame("frameName");
driver.switchTo().frame(element);
driver.switchTo().defaultContent();
```

### Dropdown Selection
```java
Select dropdown = new Select(driver.findElement(By.id("country")));
dropdown.selectByVisibleText("India");
dropdown.selectByValue("IND");
dropdown.selectByIndex(0);
```

### Take Screenshot
```java
TakesScreenshot ts = (TakesScreenshot) driver;
File srcFile = ts.getScreenshotAs(OutputType.FILE);
byte[] bytes = ts.getScreenshotAs(OutputType.BYTES);
String base64 = ts.getScreenshotAs(OutputType.BASE64);
```

### Playwright Screenshot
```java
page.screenshot(new Page.ScreenshotOptions()
    .setPath(Paths.get("screenshot.png"))
    .setFullPage(true));
```

### Playwright Network Interception
```java
page.route("**/api/**", route -> {
    route.fulfill(new Route.FulfillOptions()
        .setStatus(200)
        .setBody("{\"data\": \"test\"}"));
});
```

### Playwright Tracing
```java
context.tracing().start(new Tracing.StartOptions()
    .setScreenshots(true).setSnapshots(true));
// ... test steps ...
context.tracing().stop(new Tracing.StopOptions()
    .setPath(Paths.get("trace.zip")));
```

### Cucumber Scenario
```gherkin
Feature: Login
  Scenario: Successful login
    Given I am on the login page
    When I enter username "testuser"
    And I enter password "password123"
    And I click login button
    Then I should see the dashboard
```

### Cucumber Step Definition
```java
@Given("I am on the login page")
public void navigateToLogin() {
    driver.get("https://example.com/login");
}

@When("I enter username {string}")
public void enterUsername(String username) {
    driver.findElement(By.id("username")).sendKeys(username);
}
```

---

## 🎯 Your Framework Specifics

### Dependencies
```
Selenium: 4.25.0
Playwright: 1.52.0
Cucumber: 7.19.0
JUnit: 4.13.2
Rest-Assured: 5.5.0
Lombok: 1.18.34
```

### Gradle Commands
```bash
# Local test
./gradlew test

# CI test
./gradlew ciTest

# Run specific test
./gradlew test --tests SeleniumExercise
./gradlew test --tests PlaywrightExercise
```

### Framework Structure
```
src/main/java/com/siemens/xf/
├── ui/pages/ - Page Objects
├── ui/stepdefinitions/ - Cucumber steps
├── api/ - API helpers
└── utils/ - Utilities

src/test/java/com/learning/
├── SeleniumExercise.java - Selenium learning
└── PlaywrightExercise.java - Playwright learning
```

### Key Classes in Your Framework
```
BasePageSelenium - Base class for Selenium pages
BasePage - Base class for Playwright pages
DriverFactory - WebDriver management
LoginPage - Example page object
```

---

## 🚨 Common Mistakes to Avoid

### ❌ Don't Do This
```java
// Thread.sleep (unpredictable)
Thread.sleep(5000);

// Implicit + Explicit wait together (conflicts)
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));

// Hardcoded waits
for (int i = 0; i < 100; i++) {
    if (element.isDisplayed()) break;
    Thread.sleep(100);
}

// Not closing driver
// driver.quit() missing

// Locating same element repeatedly
for (int i = 0; i < 10; i++) {
    driver.findElement(By.id("btn")).click();
}
```

### ✅ Do This Instead
```java
// Explicit wait
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("element")));

// Use only explicit waits

// Fluent wait with conditions
FluentWait<WebDriver> wait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofMillis(500));

// Always quit in finally or @After
@After
public void tearDown() {
    if (driver != null) driver.quit();
}

// Store element reference
WebElement btn = driver.findElement(By.id("btn"));
for (int i = 0; i < 10; i++) {
    btn.click();
}
```

---

## 💡 Pro Tips

### 1. Locator Priority
```
1. ID (fastest, unique)
2. Name
3. CSS Selector (fast, flexible)
4. XPath (powerful but slower)
```

### 2. Debugging Tips
```java
// Print current URL
System.out.println(driver.getCurrentUrl());

// Print page source
System.out.println(driver.getPageSource());

// Element info
System.out.println(element.getTagName());
System.out.println(element.getAttribute("class"));
System.out.println(element.getText());

// Is element displayed/enabled/selected?
System.out.println(element.isDisplayed());
System.out.println(element.isEnabled());
System.out.println(element.isSelected());
```

### 3. XPath Axes
```
//button[@id='submit']                    - Self
//button[@id='submit']/parent::form       - Parent
//button[@id='submit']/following-sibling::div  - Next sibling
//button[@id='submit']/preceding-sibling::input - Previous sibling
//form//button                             - Descendant
//label[text()='Username']/following::input[1]  - Following
```

### 4. CSS Selector Tricks
```css
#username              - ID
.btn-primary           - Class
input[type='text']     - Attribute
input[type^='te']      - Starts with
input[type$='xt']      - Ends with
input[type*='ex']      - Contains
div > input            - Direct child
div input              - Descendant
div + input            - Next sibling
div ~ input            - Following sibling
input:nth-child(2)     - Nth child
input:first-child      - First child
input:last-child       - Last child
```

---

## 📊 When to Use What?

### Use Selenium When:
- ✅ Testing legacy applications
- ✅ Wide browser support needed (Safari, IE)
- ✅ Large existing Selenium codebase
- ✅ Using cloud providers (BrowserStack, Sauce Labs)
- ✅ Team has Selenium expertise

### Use Playwright When:
- ✅ New project starting
- ✅ Modern web applications (React, Angular, Vue)
- ✅ Need network interception
- ✅ Need fast execution
- ✅ Built-in debugging tools needed
- ✅ API + UI testing combined

---

## 🎤 Interview Day Checklist

### Before Interview:
- [ ] Review this cheat sheet
- [ ] Practice live coding on the-internet.herokuapp.com
- [ ] Understand your framework architecture
- [ ] Be ready to explain POM
- [ ] Know wait strategies

### Common Live Coding Tasks:
1. **Login automation** - Basic scenario
2. **Handle dynamic table** - Extract data
3. **Handle dropdowns** - Select by different methods
4. **Wait for element** - Implement proper wait
5. **Take screenshot** - On failure
6. **Handle multiple windows** - Switch and verify

### Questions to Ask Interviewer:
1. What automation tools do you currently use?
2. What is the test coverage percentage?
3. How is CI/CD implemented?
4. What are the main challenges in your automation?
5. What frameworks/patterns do you follow?

---

## 🔍 Framework Analysis Command

```bash
# To understand your framework before interview
cd C:\XceleratorTest_Automation\XceleratorTestAutomation

# View dependencies
cat build.gradle
cat utaf\build.gradle

# Count test files
dir /s /b *.feature
dir /s /b *Test*.java

# View reports
start reports\cucumber-html-reports\overview-features.html

# Run learning exercises
.\gradlew test --tests SeleniumExercise
.\gradlew test --tests PlaywrightExercise
```

---

## 📚 Last Minute Revision

### Top 10 Must-Know Concepts:
1. ✅ WebDriver architecture
2. ✅ Locator strategies (8 types)
3. ✅ Wait types (Implicit, Explicit, Fluent)
4. ✅ Page Object Model
5. ✅ Handling alerts, frames, windows
6. ✅ Actions class
7. ✅ JavaScript Executor
8. ✅ Selenium 4 features
9. ✅ Playwright auto-waiting
10. ✅ CI/CD integration

### Top 5 Common Failures and Solutions:
1. **NoSuchElementException** → Use explicit wait
2. **StaleElementReferenceException** → Re-find element or use FluentWait
3. **ElementClickInterceptedException** → Wait for element, use JS click
4. **TimeoutException** → Increase timeout or check locator
5. **ElementNotInteractableException** → Wait for visibility/enabled

---

## 🎯 Practice Sites

1. **The Internet** - https://the-internet.herokuapp.com
   - All types of challenges
   - Great for practice

2. **DemoQA** - https://demoqa.com
   - Real-world scenarios
   - Elements, forms, alerts, windows

3. **SauceDemo** - https://www.saucedemo.com
   - E-commerce flow
   - Good for POM practice

4. **Automation Practice** - http://automationpractice.com
   - Complete e-commerce site
   - End-to-end testing

---

## 🚀 Final Tips

1. **Be confident** - You have a working framework
2. **Explain your approach** - Don't just code silently
3. **Ask clarifying questions** - Show analytical thinking
4. **Use best practices** - Even in live coding
5. **Admit if you don't know** - But show willingness to learn

---

## 📞 Quick Help

**Your Framework Location:**
```
C:\XceleratorTest_Automation\XceleratorTestAutomation
```

**Learning Files:**
```
src\test\java\com\learning\SeleniumExercise.java
src\test\java\com\learning\PlaywrightExercise.java
```

**Documentation:**
```
README.md
QUICK_REFERENCE.md
docs\LEARNING_GUIDE.md
```

---

**Good Luck! 🎯 You got this! 💪**

Remember: The best answer is the one that solves the problem efficiently and is maintainable!

