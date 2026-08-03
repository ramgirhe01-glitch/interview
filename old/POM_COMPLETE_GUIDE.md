# 🏗️ Page Object Model (POM) Complete Guide — Design Pattern, Structure & Flow

---

## VISUAL HIERARCHY

```
POM Architecture
│
├── Test Layer (test classes)
│   ├── LoginTest.java
│   ├── DashboardTest.java
│   └── ProductTest.java
│
├── Page Layer (page objects)
│   ├── BasePage.java              → common methods (click, type, wait)
│   ├── LoginPage.java             → login page elements + actions
│   ├── DashboardPage.java         → dashboard elements + actions
│   └── ProductPage.java           → product page elements + actions
│
├── Component Layer (reusable components)
│   ├── HeaderComponent.java       → navbar, search, profile
│   ├── FooterComponent.java       → footer links
│   └── ModalComponent.java        → popup/modal dialogs
│
├── Utilities
│   ├── DriverFactory.java         → WebDriver creation/management
│   ├── ConfigReader.java          → read properties/config
│   ├── WaitUtils.java             → explicit wait helpers
│   └── ScreenshotUtils.java       → screenshot capture
│
└── Config
    ├── config.properties          → URLs, browser, timeouts
    └── testdata.json/xlsx         → test data
```

---

## POM FLOW

```
Test Class                    Page Object                  Web Page
─────────                     ───────────                  ────────
LoginTest                     LoginPage
  │                             │
  ├─ loginPage.enterUsername()──→ findElement(username)───→ types "admin"
  ├─ loginPage.enterPassword()──→ findElement(password)───→ types "pass123"
  ├─ loginPage.clickLogin()────→ findElement(loginBtn)───→ clicks button
  │                             │
  │                             └─ returns DashboardPage
  │                                    │
  └─ dashboardPage.getWelcome()───────→ findElement(msg)──→ "Welcome admin"
```

---

# 1️⃣ What is Page Object Model (POM)?

## Definition:
**POM is a design pattern** where each web page is represented by a Java class. The class contains:
- **Elements** — locators for page elements (private)
- **Methods** — actions you can perform on the page (public)

## Benefits:
```
✅ Separation of concerns — tests don't know about locators
✅ Reusability — same page object used by multiple tests
✅ Maintainability — locator changes in ONE place only
✅ Readability — tests read like plain English
✅ Reduces duplication — no repeated findElement() in tests
```

## Rules:
```
1. Page objects NEVER contain assertions (assertions live in tests)
2. Page methods return either:
   - The same page object (for actions staying on same page)
   - A NEW page object (for actions navigating to different page)
3. Elements are PRIVATE, methods are PUBLIC
4. No test logic in page objects
5. One page class per web page (or major section)
```

---

# 2️⃣ Project Structure

```
src/
├── main/java/
│   └── (empty or app code)
│
├── test/java/
│   ├── pages/                          ← Page Objects
│   │   ├── BasePage.java
│   │   ├── LoginPage.java
│   │   ├── DashboardPage.java
│   │   ├── ProductPage.java
│   │   ├── CartPage.java
│   │   └── CheckoutPage.java
│   │
│   ├── components/                     ← Reusable Components
│   │   ├── HeaderComponent.java
│   │   ├── FooterComponent.java
│   │   └── SearchComponent.java
│   │
│   ├── tests/                          ← Test Classes
│   │   ├── BaseTest.java
│   │   ├── LoginTest.java
│   │   ├── DashboardTest.java
│   │   └── ProductTest.java
│   │
│   ├── utils/                          ← Utilities
│   │   ├── DriverFactory.java
│   │   ├── ConfigReader.java
│   │   ├── WaitUtils.java
│   │   ├── ScreenshotUtils.java
│   │   └── TestDataReader.java
│   │
│   └── config/                         ← Configuration
│       ├── config.properties
│       └── testdata.json
│
└── test/resources/
    ├── testdata/
    └── screenshots/
```

---

# 3️⃣ BasePage — Parent of All Page Objects

```java
package pages;

import org.openqa.selenium.*;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.*;
import java.time.Duration;

public class BasePage {

    protected WebDriver driver;
    protected WebDriverWait wait;

    // Constructor — initializes driver + PageFactory
    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);  // initialize @FindBy elements
    }

    // ===== COMMON ACTIONS =====

    protected void click(WebElement element) {
        waitForClickable(element);
        element.click();
    }

    protected void type(WebElement element, String text) {
        waitForVisible(element);
        element.clear();
        element.sendKeys(text);
    }

    protected String getText(WebElement element) {
        waitForVisible(element);
        return element.getText();
    }

    protected String getAttribute(WebElement element, String attr) {
        waitForVisible(element);
        return element.getAttribute(attr);
    }

    protected boolean isDisplayed(WebElement element) {
        try {
            return element.isDisplayed();
        } catch (NoSuchElementException | StaleElementReferenceException e) {
            return false;
        }
    }

    // ===== DROPDOWN =====

    protected void selectByVisibleText(WebElement element, String text) {
        waitForVisible(element);
        new Select(element).selectByVisibleText(text);
    }

    protected void selectByValue(WebElement element, String value) {
        waitForVisible(element);
        new Select(element).selectByValue(value);
    }

    protected String getSelectedText(WebElement element) {
        return new Select(element).getFirstSelectedOption().getText();
    }

    // ===== CHECKBOX / RADIO =====

    protected void checkCheckbox(WebElement element) {
        if (!element.isSelected()) {
            element.click();
        }
    }

    protected void uncheckCheckbox(WebElement element) {
        if (element.isSelected()) {
            element.click();
        }
    }

    // ===== WAITS =====

    protected WebElement waitForVisible(WebElement element) {
        return wait.until(ExpectedConditions.visibilityOf(element));
    }

    protected WebElement waitForClickable(WebElement element) {
        return wait.until(ExpectedConditions.elementToBeClickable(element));
    }

    protected void waitForInvisible(By locator) {
        wait.until(ExpectedConditions.invisibilityOfElementLocated(locator));
    }

    protected void waitForTextPresent(WebElement element, String text) {
        wait.until(ExpectedConditions.textToBePresentInElement(element, text));
    }

    protected void waitForUrl(String partialUrl) {
        wait.until(ExpectedConditions.urlContains(partialUrl));
    }

    // ===== JAVASCRIPT =====

    protected void jsClick(WebElement element) {
        ((JavascriptExecutor) driver).executeScript("arguments[0].click();", element);
    }

    protected void jsScrollTo(WebElement element) {
        ((JavascriptExecutor) driver).executeScript(
            "arguments[0].scrollIntoView(true);", element);
    }

    protected void jsType(WebElement element, String text) {
        ((JavascriptExecutor) driver).executeScript(
            "arguments[0].value='" + text + "';", element);
    }

    // ===== NAVIGATION =====

    protected String getCurrentUrl() {
        return driver.getCurrentUrl();
    }

    protected String getPageTitle() {
        return driver.getTitle();
    }

    // ===== FRAMES =====

    protected void switchToFrame(WebElement frameElement) {
        driver.switchTo().frame(frameElement);
    }

    protected void switchToDefaultContent() {
        driver.switchTo().defaultContent();
    }

    // ===== ALERTS =====

    protected String acceptAlert() {
        Alert alert = wait.until(ExpectedConditions.alertIsPresent());
        String text = alert.getText();
        alert.accept();
        return text;
    }

    protected String dismissAlert() {
        Alert alert = wait.until(ExpectedConditions.alertIsPresent());
        String text = alert.getText();
        alert.dismiss();
        return text;
    }

    // ===== ACTIONS =====

    protected void hover(WebElement element) {
        new org.openqa.selenium.interactions.Actions(driver)
            .moveToElement(element).perform();
    }

    protected void dragAndDrop(WebElement source, WebElement target) {
        new org.openqa.selenium.interactions.Actions(driver)
            .dragAndDrop(source, target).perform();
    }

    // ===== SCREENSHOT =====

    protected byte[] takeScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }
}
```

---

# 4️⃣ Page Objects — With @FindBy Annotations

## 4a. LoginPage

```java
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class LoginPage extends BasePage {

    // ===== ELEMENTS (private) =====

    @FindBy(id = "username")
    private WebElement usernameInput;

    @FindBy(id = "password")
    private WebElement passwordInput;

    @FindBy(id = "login-btn")
    private WebElement loginButton;

    @FindBy(css = ".error-message")
    private WebElement errorMessage;

    @FindBy(linkText = "Forgot Password?")
    private WebElement forgotPasswordLink;

    @FindBy(id = "remember-me")
    private WebElement rememberMeCheckbox;

    // ===== CONSTRUCTOR =====

    public LoginPage(WebDriver driver) {
        super(driver);
    }

    // ===== ACTIONS (public) =====

    public LoginPage enterUsername(String username) {
        type(usernameInput, username);
        return this;                           // return same page (fluent)
    }

    public LoginPage enterPassword(String password) {
        type(passwordInput, password);
        return this;
    }

    public DashboardPage clickLogin() {
        click(loginButton);
        return new DashboardPage(driver);      // returns NEW page object
    }

    public LoginPage clickLoginExpectingError() {
        click(loginButton);
        return this;                           // stay on same page (error case)
    }

    public ForgotPasswordPage clickForgotPassword() {
        click(forgotPasswordLink);
        return new ForgotPasswordPage(driver);
    }

    public LoginPage checkRememberMe() {
        checkCheckbox(rememberMeCheckbox);
        return this;
    }

    // ===== FLUENT LOGIN (combines multiple steps) =====

    public DashboardPage loginAs(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        return clickLogin();
    }

    // ===== GETTERS (for assertions in test) =====

    public String getErrorMessage() {
        return getText(errorMessage);
    }

    public boolean isErrorDisplayed() {
        return isDisplayed(errorMessage);
    }

    public boolean isLoginButtonEnabled() {
        return loginButton.isEnabled();
    }

    public String getUsernameValue() {
        return usernameInput.getAttribute("value");
    }
}
```

---

## 4b. DashboardPage

```java
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import java.util.List;
import java.util.stream.Collectors;

public class DashboardPage extends BasePage {

    @FindBy(css = ".welcome-message")
    private WebElement welcomeMessage;

    @FindBy(id = "search-input")
    private WebElement searchInput;

    @FindBy(id = "search-btn")
    private WebElement searchButton;

    @FindBy(css = ".product-card")
    private List<WebElement> productCards;

    @FindBy(css = ".product-card .product-name")
    private List<WebElement> productNames;

    @FindBy(id = "logout-btn")
    private WebElement logoutButton;

    @FindBy(id = "profile-link")
    private WebElement profileLink;

    @FindBy(css = ".notification-badge")
    private WebElement notificationBadge;

    public DashboardPage(WebDriver driver) {
        super(driver);
    }

    // ===== ACTIONS =====

    public DashboardPage searchProduct(String productName) {
        type(searchInput, productName);
        click(searchButton);
        return this;
    }

    public ProductPage clickProduct(String name) {
        for (WebElement card : productCards) {
            if (card.getText().contains(name)) {
                click(card);
                return new ProductPage(driver);
            }
        }
        throw new RuntimeException("Product not found: " + name);
    }

    public ProductPage clickProductByIndex(int index) {
        click(productCards.get(index));
        return new ProductPage(driver);
    }

    public LoginPage clickLogout() {
        click(logoutButton);
        return new LoginPage(driver);
    }

    public ProfilePage clickProfile() {
        click(profileLink);
        return new ProfilePage(driver);
    }

    // ===== GETTERS =====

    public String getWelcomeMessage() {
        return getText(welcomeMessage);
    }

    public int getProductCount() {
        return productCards.size();
    }

    public List<String> getAllProductNames() {
        return productNames.stream()
            .map(WebElement::getText)
            .collect(Collectors.toList());
    }

    public String getNotificationCount() {
        return getText(notificationBadge);
    }

    public boolean isWelcomeDisplayed() {
        return isDisplayed(welcomeMessage);
    }
}
```

---

## 4c. ProductPage

```java
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class ProductPage extends BasePage {

    @FindBy(css = ".product-title")
    private WebElement productTitle;

    @FindBy(css = ".product-price")
    private WebElement productPrice;

    @FindBy(css = ".product-description")
    private WebElement productDescription;

    @FindBy(id = "quantity")
    private WebElement quantityInput;

    @FindBy(id = "add-to-cart")
    private WebElement addToCartButton;

    @FindBy(css = ".size-select")
    private WebElement sizeDropdown;

    @FindBy(css = ".success-message")
    private WebElement successMessage;

    @FindBy(css = ".cart-count")
    private WebElement cartCount;

    public ProductPage(WebDriver driver) {
        super(driver);
    }

    public ProductPage setQuantity(String qty) {
        type(quantityInput, qty);
        return this;
    }

    public ProductPage selectSize(String size) {
        selectByVisibleText(sizeDropdown, size);
        return this;
    }

    public ProductPage clickAddToCart() {
        click(addToCartButton);
        return this;
    }

    public CartPage goToCart() {
        click(cartCount);
        return new CartPage(driver);
    }

    // Fluent method
    public ProductPage addToCart(String size, String qty) {
        selectSize(size);
        setQuantity(qty);
        clickAddToCart();
        return this;
    }

    public String getProductTitle() {
        return getText(productTitle);
    }

    public String getProductPrice() {
        return getText(productPrice);
    }

    public String getSuccessMessage() {
        return getText(successMessage);
    }

    public String getCartCount() {
        return getText(cartCount);
    }
}
```

---

# 5️⃣ @FindBy Annotations — All Options

```java
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.FindBys;
import org.openqa.selenium.support.FindAll;
import org.openqa.selenium.support.CacheLookup;
```

## Single Locator:

```java
@FindBy(id = "username")                           // By.id
private WebElement usernameInput;

@FindBy(name = "email")                            // By.name
private WebElement emailInput;

@FindBy(className = "btn-primary")                 // By.className
private WebElement submitButton;

@FindBy(tagName = "input")                         // By.tagName
private List<WebElement> allInputs;

@FindBy(css = "#form .btn-submit")                 // By.cssSelector
private WebElement submitBtn;

@FindBy(xpath = "//button[@type='submit']")        // By.xpath
private WebElement submitXpath;

@FindBy(linkText = "Click Here")                   // By.linkText
private WebElement clickLink;

@FindBy(partialLinkText = "Click")                 // By.partialLinkText
private WebElement partialLink;
```

## @FindBys — AND condition (all must match, parent-child chain):
```java
// Finds element matching ALL conditions (chained)
// Equivalent to: By.cssSelector(".form-group input.email")
@FindBys({
    @FindBy(className = "form-group"),
    @FindBy(css = "input.email")
})
private WebElement emailField;
```

## @FindAll — OR condition (any can match):
```java
// Finds elements matching ANY condition (union)
@FindAll({
    @FindBy(css = ".error"),
    @FindBy(css = ".warning"),
    @FindBy(css = ".alert")
})
private List<WebElement> allMessages;
```

## @CacheLookup — Cache element (for static elements):
```java
// Element is looked up ONCE and cached (faster, but can cause StaleElement)
// ⚠️ Only use for elements that NEVER change (logo, static header, etc.)
@FindBy(id = "logo")
@CacheLookup
private WebElement logo;
```

## How @FindBy Works:
```
1. PageFactory.initElements(driver, this) is called in constructor
2. Creates PROXY objects for each @FindBy field
3. Elements are NOT looked up immediately — they use lazy loading
4. Element is found in DOM only when you FIRST interact with it
5. Every subsequent call re-finds the element (unless @CacheLookup)
```

---

# 6️⃣ Page Objects WITHOUT @FindBy (By Locators)

```java
package pages;

import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.*;
import java.time.Duration;

public class LoginPage {

    private WebDriver driver;
    private WebDriverWait wait;

    // Locators as By objects (not WebElements)
    private By usernameInput = By.id("username");
    private By passwordInput = By.id("password");
    private By loginButton   = By.id("login-btn");
    private By errorMessage  = By.cssSelector(".error-message");

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    public LoginPage enterUsername(String username) {
        driver.findElement(usernameInput).clear();
        driver.findElement(usernameInput).sendKeys(username);
        return this;
    }

    public LoginPage enterPassword(String password) {
        driver.findElement(passwordInput).clear();
        driver.findElement(passwordInput).sendKeys(password);
        return this;
    }

    public DashboardPage clickLogin() {
        wait.until(ExpectedConditions.elementToBeClickable(loginButton)).click();
        return new DashboardPage(driver);
    }

    public String getErrorMessage() {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(errorMessage)).getText();
    }

    public DashboardPage loginAs(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        return clickLogin();
    }
}
```

### @FindBy vs By Locators — Comparison:

| Feature | @FindBy (PageFactory) | By Locators |
|---------|----------------------|-------------|
| Syntax | `@FindBy(id="x") WebElement el` | `By locator = By.id("x")` |
| Initialization | `PageFactory.initElements()` | Not needed |
| Element lookup | Lazy proxy (on first use) | `driver.findElement(by)` |
| Caching | `@CacheLookup` available | Manual caching if needed |
| Readability | Cleaner, annotation-based | More explicit |
| Flexibility | Less (no dynamic locators) | More (can build locators dynamically) |
| Stale Element | Proxy re-finds automatically | Must re-find manually |
| Recommended | ✅ Most teams prefer | ✅ Also valid approach |

---

# 7️⃣ Component Objects — Reusable Page Parts

```java
package components;

import org.openqa.selenium.*;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import pages.BasePage;

public class HeaderComponent extends BasePage {

    @FindBy(css = ".logo")
    private WebElement logo;

    @FindBy(id = "search-input")
    private WebElement searchInput;

    @FindBy(id = "search-btn")
    private WebElement searchButton;

    @FindBy(css = ".cart-icon")
    private WebElement cartIcon;

    @FindBy(css = ".cart-count")
    private WebElement cartCount;

    @FindBy(css = ".user-menu")
    private WebElement userMenu;

    @FindBy(css = ".user-menu .logout")
    private WebElement logoutOption;

    public HeaderComponent(WebDriver driver) {
        super(driver);
    }

    public void search(String query) {
        type(searchInput, query);
        click(searchButton);
    }

    public void clickCart() {
        click(cartIcon);
    }

    public String getCartCount() {
        return getText(cartCount);
    }

    public void logout() {
        hover(userMenu);
        click(logoutOption);
    }

    public boolean isLogoDisplayed() {
        return isDisplayed(logo);
    }
}
```

### Using Components in Page Objects:
```java
public class DashboardPage extends BasePage {

    private HeaderComponent header;

    @FindBy(css = ".welcome")
    private WebElement welcomeMsg;

    public DashboardPage(WebDriver driver) {
        super(driver);
        this.header = new HeaderComponent(driver);
    }

    public HeaderComponent header() {
        return header;
    }

    public String getWelcome() {
        return getText(welcomeMsg);
    }
}

// Usage in test:
dashboardPage.header().search("iPhone");
dashboardPage.header().logout();
String count = dashboardPage.header().getCartCount();
```

---

# 8️⃣ BaseTest — Test Setup/Teardown

```java
package tests;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.annotations.*;
import io.github.bonigarcia.wdm.WebDriverManager;
import pages.LoginPage;

public class BaseTest {

    protected WebDriver driver;
    protected LoginPage loginPage;

    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();

        ChromeOptions options = new ChromeOptions();
        options.addArguments("--start-maximized");
        options.addArguments("--disable-notifications");

        driver = new ChromeDriver(options);
        driver.manage().window().maximize();

        driver.get("https://example.com");
        loginPage = new LoginPage(driver);
    }

    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

---

# 9️⃣ Test Classes — Assertions Live Here

```java
package tests;

import org.testng.Assert;
import org.testng.annotations.Test;
import pages.DashboardPage;

public class LoginTest extends BaseTest {

    @Test
    public void testSuccessfulLogin() {
        DashboardPage dashboard = loginPage.loginAs("admin", "pass123");

        Assert.assertTrue(dashboard.isWelcomeDisplayed());
        Assert.assertEquals(dashboard.getWelcomeMessage(), "Welcome admin");
    }

    @Test
    public void testInvalidLogin() {
        loginPage.enterUsername("wrong")
                 .enterPassword("wrong")
                 .clickLoginExpectingError();

        Assert.assertTrue(loginPage.isErrorDisplayed());
        Assert.assertEquals(loginPage.getErrorMessage(), "Invalid credentials");
    }

    @Test
    public void testEmptyUsername() {
        loginPage.enterPassword("pass123")
                 .clickLoginExpectingError();

        Assert.assertEquals(loginPage.getErrorMessage(), "Username is required");
    }

    @Test
    public void testLoginAndLogout() {
        DashboardPage dashboard = loginPage.loginAs("admin", "pass123");
        Assert.assertTrue(dashboard.isWelcomeDisplayed());

        loginPage = dashboard.clickLogout();
        Assert.assertTrue(loginPage.isLoginButtonEnabled());
    }
}
```

```java
package tests;

import org.testng.Assert;
import org.testng.annotations.Test;
import pages.DashboardPage;
import pages.ProductPage;

public class ProductTest extends BaseTest {

    @Test
    public void testAddProductToCart() {
        DashboardPage dashboard = loginPage.loginAs("admin", "pass123");

        ProductPage productPage = dashboard.clickProduct("iPhone 15");
        productPage.addToCart("Large", "2");

        Assert.assertEquals(productPage.getSuccessMessage(), "Added to cart");
        Assert.assertEquals(productPage.getCartCount(), "2");
    }

    @Test
    public void testSearchProduct() {
        DashboardPage dashboard = loginPage.loginAs("admin", "pass123");
        dashboard.searchProduct("MacBook");

        Assert.assertTrue(dashboard.getProductCount() > 0);
        Assert.assertTrue(dashboard.getAllProductNames().stream()
            .anyMatch(name -> name.contains("MacBook")));
    }
}
```

---

# 🔟 POM with Cucumber (BDD)

## Step Definitions:
```java
package stepdefs;

import io.cucumber.java.en.*;
import org.testng.Assert;
import pages.*;
import utils.DriverFactory;

public class LoginSteps {

    private LoginPage loginPage;
    private DashboardPage dashboardPage;

    @Given("I am on the login page")
    public void iAmOnLoginPage() {
        loginPage = new LoginPage(DriverFactory.getDriver());
    }

    @When("I login with username {string} and password {string}")
    public void iLogin(String username, String password) {
        dashboardPage = loginPage.loginAs(username, password);
    }

    @Then("I should see the welcome message {string}")
    public void iShouldSeeWelcome(String expected) {
        Assert.assertEquals(dashboardPage.getWelcomeMessage(), expected);
    }

    @Then("I should see error message {string}")
    public void iShouldSeeError(String expected) {
        Assert.assertEquals(loginPage.getErrorMessage(), expected);
    }

    @When("I login with invalid credentials")
    public void iLoginWithInvalidCreds() {
        loginPage.enterUsername("wrong")
                 .enterPassword("wrong")
                 .clickLoginExpectingError();
    }
}
```

## Feature File:
```gherkin
Feature: Login functionality

  Scenario: Successful login
    Given I am on the login page
    When I login with username "admin" and password "pass123"
    Then I should see the welcome message "Welcome admin"

  Scenario: Invalid login
    Given I am on the login page
    When I login with invalid credentials
    Then I should see error message "Invalid credentials"

  Scenario Outline: Login with multiple users
    Given I am on the login page
    When I login with username "<username>" and password "<password>"
    Then I should see the welcome message "<welcome>"

    Examples:
      | username | password | welcome       |
      | admin    | pass123  | Welcome admin |
      | user1    | pass456  | Welcome user1 |
```

---

# 1️⃣1️⃣ DriverFactory — Thread-Safe WebDriver

```java
package utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxDriver;
import io.github.bonigarcia.wdm.WebDriverManager;

public class DriverFactory {

    // ThreadLocal for parallel execution safety
    private static ThreadLocal<WebDriver> driver = new ThreadLocal<>();

    public static WebDriver getDriver() {
        return driver.get();
    }

    public static void initDriver(String browser) {
        WebDriver webDriver;

        switch (browser.toLowerCase()) {
            case "firefox":
                WebDriverManager.firefoxdriver().setup();
                webDriver = new FirefoxDriver();
                break;
            case "chrome":
            default:
                WebDriverManager.chromedriver().setup();
                ChromeOptions options = new ChromeOptions();
                options.addArguments("--start-maximized");
                options.addArguments("--disable-notifications");
                webDriver = new ChromeDriver(options);
                break;
        }

        webDriver.manage().window().maximize();
        driver.set(webDriver);
    }

    public static void quitDriver() {
        if (driver.get() != null) {
            driver.get().quit();
            driver.remove();
        }
    }
}
```

---

# 1️⃣2️⃣ ConfigReader — Properties File

### config.properties:
```properties
browser=chrome
base.url=https://example.com
implicit.wait=10
explicit.wait=15
page.load.timeout=30
headless=false
username=admin
password=pass123
```

### ConfigReader.java:
```java
package utils;

import java.io.*;
import java.util.Properties;

public class ConfigReader {

    private static Properties properties;

    static {
        try {
            properties = new Properties();
            FileInputStream fis = new FileInputStream("src/test/resources/config.properties");
            properties.load(fis);
            fis.close();
        } catch (IOException e) {
            throw new RuntimeException("Config file not found!", e);
        }
    }

    public static String get(String key) {
        return properties.getProperty(key);
    }

    public static String getBrowser() {
        return get("browser");
    }

    public static String getBaseUrl() {
        return get("base.url");
    }

    public static int getImplicitWait() {
        return Integer.parseInt(get("implicit.wait"));
    }

    public static int getExplicitWait() {
        return Integer.parseInt(get("explicit.wait"));
    }

    public static boolean isHeadless() {
        return Boolean.parseBoolean(get("headless"));
    }
}
```

---

# 1️⃣3️⃣ Advanced Patterns

## Pattern 1: Fluent Page Object (Method Chaining)
```java
// Enables: loginPage.enterUsername("x").enterPassword("y").clickLogin()
public LoginPage enterUsername(String username) {
    type(usernameInput, username);
    return this;  // ← return this for chaining
}
```

## Pattern 2: Page Navigation Returns
```java
// Action stays on same page → return this
public LoginPage clickLoginExpectingError() {
    click(loginButton);
    return this;
}

// Action navigates to new page → return new PageObject
public DashboardPage clickLogin() {
    click(loginButton);
    return new DashboardPage(driver);
}
```

## Pattern 3: Loadable Component (verify page loaded)
```java
public class LoginPage extends BasePage {

    public LoginPage(WebDriver driver) {
        super(driver);
        waitForPageLoad();
    }

    private void waitForPageLoad() {
        waitForVisible(loginButton);
        // OR: waitForUrl("/login");
        // OR: waitForTextPresent(heading, "Login");
    }
}
```

## Pattern 4: Generic Type for Fluent Methods
```java
@SuppressWarnings("unchecked")
public abstract class BasePage<T extends BasePage<T>> {

    protected void type(WebElement el, String text) {
        el.clear();
        el.sendKeys(text);
    }

    // Now all child pages can chain without casting
    public T enterField(WebElement el, String text) {
        type(el, text);
        return (T) this;
    }
}

public class LoginPage extends BasePage<LoginPage> {
    // enterField() now returns LoginPage, not BasePage
}
```

## Pattern 5: Dynamic Elements
```java
public class ProductPage extends BasePage {

    // Use method instead of @FindBy for dynamic locators
    public WebElement getProductByName(String name) {
        return driver.findElement(
            By.xpath("//div[@class='product' and contains(text(),'" + name + "')]")
        );
    }

    public void clickProductByName(String name) {
        click(getProductByName(name));
    }

    // Parameterized locator
    private By productLocator(String name) {
        return By.xpath("//div[@class='product'][contains(.,'" + name + "')]");
    }
}
```

## Pattern 6: Table Handling Page Object
```java
public class TablePage extends BasePage {

    @FindBy(css = "#data-table tbody tr")
    private List<WebElement> rows;

    public TablePage(WebDriver driver) {
        super(driver);
    }

    public int getRowCount() {
        return rows.size();
    }

    public String getCellValue(int row, int col) {
        return rows.get(row)
            .findElements(By.tagName("td"))
            .get(col)
            .getText();
    }

    public WebElement getRowByText(String text) {
        return rows.stream()
            .filter(row -> row.getText().contains(text))
            .findFirst()
            .orElseThrow(() -> new RuntimeException("Row not found: " + text));
    }

    public void clickActionInRow(String rowText, String buttonClass) {
        getRowByText(rowText)
            .findElement(By.cssSelector("." + buttonClass))
            .click();
    }

    public List<String> getColumnValues(int colIndex) {
        return rows.stream()
            .map(row -> row.findElements(By.tagName("td")).get(colIndex).getText())
            .collect(Collectors.toList());
    }
}
```

---

# 1️⃣4️⃣ POM Anti-Patterns (What NOT to Do)

```java
// ❌ BAD: Assertions in page object
public class LoginPage {
    public void verifyErrorMessage(String expected) {
        Assert.assertEquals(errorMsg.getText(), expected);  // ❌ NO!
    }
}

// ✅ GOOD: Return data, assert in test
public class LoginPage {
    public String getErrorMessage() {
        return getText(errorMessage);  // ✅ return data
    }
}
// Test: Assert.assertEquals(loginPage.getErrorMessage(), "Invalid");

// ───────────────────────────────────────

// ❌ BAD: Exposing WebElements
public class LoginPage {
    public WebElement getUsernameField() {
        return usernameInput;  // ❌ exposes implementation
    }
}

// ✅ GOOD: Expose actions/data only
public class LoginPage {
    public LoginPage enterUsername(String text) {
        type(usernameInput, text);  // ✅ encapsulated
        return this;
    }
}

// ───────────────────────────────────────

// ❌ BAD: Test logic in page object
public class LoginPage {
    public void loginAndVerifyDashboard() {
        type(username, "admin");
        type(password, "pass");
        click(loginBtn);
        Assert.assertTrue(driver.getCurrentUrl().contains("/dashboard"));  // ❌
    }
}

// ✅ GOOD: Separate concerns
// Page: provides actions
// Test: contains assertions and logic

// ───────────────────────────────────────

// ❌ BAD: Thread.sleep() in page object
public void clickLogin() {
    click(loginButton);
    Thread.sleep(3000);  // ❌ NEVER
}

// ✅ GOOD: Explicit waits
public DashboardPage clickLogin() {
    click(loginButton);
    waitForUrl("/dashboard");  // ✅
    return new DashboardPage(driver);
}

// ───────────────────────────────────────

// ❌ BAD: Hardcoded test data in page object
public void login() {
    type(usernameInput, "admin");      // ❌ hardcoded
    type(passwordInput, "password");    // ❌ hardcoded
}

// ✅ GOOD: Accept parameters
public DashboardPage loginAs(String username, String password) {
    type(usernameInput, username);      // ✅ parameterized
    type(passwordInput, password);
    return clickLogin();
}
```

---

# 1️⃣5️⃣ Complete POM Flow Summary

```
┌─────────────────────────────────────────────────────────────┐
│                     TEST CLASS                               │
│  LoginTest extends BaseTest                                  │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ @Test testSuccessfulLogin()                            │ │
│  │   DashboardPage dash = loginPage.loginAs("admin","p"); │ │
│  │   Assert.assertEquals(dash.getWelcome(), "Welcome");   │ │
│  └────────────┬───────────────────────────┬───────────────┘ │
│               │                           │                  │
│  ┌────────────▼──────────┐   ┌───────────▼──────────────┐  │
│  │     LoginPage         │   │    DashboardPage          │  │
│  │  (Page Object)        │   │    (Page Object)          │  │
│  │                       │   │                           │  │
│  │ @FindBy(id="user")    │   │ @FindBy(css=".welcome")   │  │
│  │ usernameInput         │   │ welcomeMessage             │  │
│  │                       │   │                           │  │
│  │ enterUsername(text)    │   │ getWelcomeMessage()       │  │
│  │ enterPassword(text)   │   │ searchProduct(text)       │  │
│  │ clickLogin()          │──→│ clickLogout()             │  │
│  │ loginAs(user, pass)   │   │                           │  │
│  └────────────┬──────────┘   └───────────────────────────┘  │
│               │                                              │
│  ┌────────────▼──────────┐                                  │
│  │     BasePage          │                                  │
│  │  (Common Methods)     │                                  │
│  │                       │                                  │
│  │ click(el)             │                                  │
│  │ type(el, text)        │                                  │
│  │ getText(el)           │                                  │
│  │ waitForVisible(el)    │                                  │
│  │ waitForClickable(el)  │                                  │
│  │ jsClick(el)           │                                  │
│  │ hover(el)             │                                  │
│  └───────────────────────┘                                  │
└─────────────────────────────────────────────────────────────┘
```

---

*Complete POM Guide with every pattern, example, and best practice. Bookmark this for daily use.*
*Last updated: April 30, 2026*

