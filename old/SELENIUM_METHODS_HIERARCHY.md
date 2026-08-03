# 🏗️ Selenium Complete Methods Hierarchy — Every Method, Return Type & Flow

---

## VISUAL HIERARCHY

```
Selenium WebDriver (Top Level)
│
├── WebDriver Setup
│   ├── WebDriverManager.chromedriver().setup()
│   ├── new ChromeDriver(options)          → WebDriver
│   ├── new FirefoxDriver(options)         → WebDriver
│   └── new EdgeDriver(options)            → WebDriver
│
├── WebDriver (MAIN interface — you use this always)
│   ├── driver.get(url)                    → void
│   ├── driver.findElement(By)             → WebElement
│   ├── driver.findElements(By)            → List<WebElement>
│   ├── driver.navigate()                  → Navigation
│   ├── driver.switchTo()                  → TargetLocator
│   ├── driver.manage()                    → Options
│   ├── driver.getWindowHandle()           → String
│   ├── driver.getWindowHandles()          → Set<String>
│   ├── driver.getCurrentUrl()             → String
│   ├── driver.getTitle()                  → String
│   ├── driver.getPageSource()             → String
│   ├── driver.close()                     → void
│   └── driver.quit()                      → void
│
├── By (8 Locator Strategies)
│   ├── By.id("x")
│   ├── By.name("x")
│   ├── By.className("x")
│   ├── By.tagName("x")
│   ├── By.cssSelector("x")
│   ├── By.xpath("x")
│   ├── By.linkText("x")
│   └── By.partialLinkText("x")
│
├── WebElement (element interactions)
│   ├── element.click()                    → void
│   ├── element.sendKeys(text)             → void
│   ├── element.clear()                    → void
│   ├── element.getText()                  → String
│   ├── element.getAttribute(name)         → String
│   ├── element.getCssValue(prop)          → String
│   ├── element.getTagName()               → String
│   ├── element.isDisplayed()              → boolean
│   ├── element.isEnabled()                → boolean
│   ├── element.isSelected()               → boolean
│   ├── element.getSize()                  → Dimension
│   ├── element.getLocation()              → Point
│   ├── element.getRect()                  → Rectangle
│   ├── element.findElement(By)            → WebElement
│   ├── element.findElements(By)           → List<WebElement>
│   ├── element.getShadowRoot()            → SearchContext (Selenium 4)
│   ├── element.getDomAttribute(name)      → String (Selenium 4)
│   ├── element.getDomProperty(name)       → String (Selenium 4)
│   ├── element.getAriaRole()              → String (Selenium 4)
│   ├── element.getAccessibleName()        → String (Selenium 4)
│   └── element.getScreenshotAs(type)      → File/byte[] (Selenium 4)
│
├── Navigation
│   ├── navigate().to(url)                 → void
│   ├── navigate().back()                  → void
│   ├── navigate().forward()               → void
│   └── navigate().refresh()               → void
│
├── TargetLocator (switchTo)
│   ├── switchTo().frame(index/name/el)    → WebDriver
│   ├── switchTo().defaultContent()        → WebDriver
│   ├── switchTo().parentFrame()           → WebDriver
│   ├── switchTo().alert()                 → Alert
│   ├── switchTo().window(handle)          → WebDriver
│   └── switchTo().newWindow(type)         → WebDriver (Selenium 4)
│
├── Alert
│   ├── alert.accept()                     → void
│   ├── alert.dismiss()                    → void
│   ├── alert.getText()                    → String
│   └── alert.sendKeys(text)               → void
│
├── Options (manage)
│   ├── manage().window()                  → Window
│   ├── manage().getCookies()              → Set<Cookie>
│   ├── manage().getCookieNamed(name)      → Cookie
│   ├── manage().addCookie(cookie)         → void
│   ├── manage().deleteCookieNamed(name)   → void
│   ├── manage().deleteAllCookies()        → void
│   └── manage().timeouts()                → Timeouts
│
├── Window
│   ├── window().maximize()                → void
│   ├── window().minimize()                → void
│   ├── window().fullscreen()              → void
│   ├── window().getSize()                 → Dimension
│   ├── window().setSize(dim)              → void
│   ├── window().getPosition()             → Point
│   └── window().setPosition(point)        → void
│
├── Timeouts
│   ├── implicitlyWait(Duration)           → Timeouts
│   ├── pageLoadTimeout(Duration)          → Timeouts
│   └── scriptTimeout(Duration)            → Timeouts
│
├── WebDriverWait + ExpectedConditions
│   ├── wait.until(condition)              → T
│   ├── visibilityOfElementLocated(By)     → WebElement
│   ├── elementToBeClickable(By)           → WebElement
│   ├── presenceOfElementLocated(By)       → WebElement
│   ├── invisibilityOfElementLocated(By)   → Boolean
│   ├── textToBePresentInElement(el, txt)  → Boolean
│   ├── titleContains(text)                → Boolean
│   ├── urlContains(text)                  → Boolean
│   ├── alertIsPresent()                   → Alert
│   ├── frameToBeAvailableAndSwitchToIt()  → WebDriver
│   └── numberOfWindowsToBe(n)             → Boolean
│
├── Actions (advanced user interactions)
│   ├── actions.moveToElement(el)          → Actions (hover)
│   ├── actions.click(el)                  → Actions
│   ├── actions.doubleClick(el)            → Actions
│   ├── actions.contextClick(el)           → Actions (right-click)
│   ├── actions.clickAndHold(el)           → Actions
│   ├── actions.release(el)                → Actions
│   ├── actions.dragAndDrop(src, tgt)      → Actions
│   ├── actions.dragAndDropBy(el, x, y)    → Actions
│   ├── actions.keyDown(key)               → Actions
│   ├── actions.keyUp(key)                 → Actions
│   ├── actions.sendKeys(keys)             → Actions
│   ├── actions.scrollToElement(el)        → Actions (Selenium 4)
│   ├── actions.scrollByAmount(x, y)       → Actions (Selenium 4)
│   ├── actions.pause(Duration)            → Actions
│   └── actions.perform()                  → void ⚠️ MUST CALL
│
├── Select (dropdown handling)
│   ├── select.selectByVisibleText(text)   → void
│   ├── select.selectByValue(value)        → void
│   ├── select.selectByIndex(index)        → void
│   ├── select.deselectAll()               → void
│   ├── select.deselectByVisibleText(text) → void
│   ├── select.deselectByValue(value)      → void
│   ├── select.deselectByIndex(index)      → void
│   ├── select.getOptions()                → List<WebElement>
│   ├── select.getAllSelectedOptions()      → List<WebElement>
│   ├── select.getFirstSelectedOption()    → WebElement
│   └── select.isMultiple()                → boolean
│
├── JavascriptExecutor
│   ├── js.executeScript(script, args)     → Object
│   └── js.executeAsyncScript(script, args)→ Object
│
├── TakesScreenshot
│   ├── getScreenshotAs(OutputType.FILE)   → File
│   ├── getScreenshotAs(OutputType.BYTES)  → byte[]
│   └── getScreenshotAs(OutputType.BASE64) → String
│
└── Relative Locators (Selenium 4)
    ├── with(By).above(el)
    ├── with(By).below(el)
    ├── with(By).toLeftOf(el)
    ├── with(By).toRightOf(el)
    └── with(By).near(el)
```

---

## CREATION FLOW

```
WebDriverManager.chromedriver().setup()
    → WebDriver driver = new ChromeDriver(options)
        → driver.manage().window().maximize()
        → driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10))
        → driver.get("https://example.com")
        → WebElement el = driver.findElement(By.id("x"))
            → el.click()
            → el.sendKeys("text")
            → el.getText()
        → driver.quit()
```

---

# 1️⃣ WebDriver Setup — Entry Point

## Option 1: WebDriverManager (Recommended)
```java
WebDriverManager.chromedriver().setup();
WebDriver driver = new ChromeDriver();
```

## Option 2: Selenium Manager (Selenium 4.6+, Auto)
```java
// No setup needed! Selenium auto-downloads driver
WebDriver driver = new ChromeDriver();
```

## Option 3: Manual System Property
```java
System.setProperty("webdriver.chrome.driver", "/path/to/chromedriver");
WebDriver driver = new ChromeDriver();
```

## With Options:
```java
ChromeOptions options = new ChromeOptions();
options.addArguments("--headless");
options.addArguments("--start-maximized");
options.addArguments("--disable-notifications");
options.addArguments("--incognito");
options.addArguments("--window-size=1920,1080");
options.addArguments("--no-sandbox");
options.addArguments("--disable-dev-shm-usage");
options.addArguments("--disable-gpu");
options.addArguments("--remote-allow-origins=*");
options.setExperimentalOption("excludeSwitches", Arrays.asList("enable-automation"));
options.addExtensions(new File("ext.crx"));
options.setBinary("/path/to/chrome");

WebDriver driver = new ChromeDriver(options);
```

### All Browser Drivers:

| Browser | Driver | Options | WebDriverManager |
|---------|--------|---------|------------------|
| Chrome | `new ChromeDriver()` | `ChromeOptions` | `WebDriverManager.chromedriver().setup()` |
| Firefox | `new FirefoxDriver()` | `FirefoxOptions` | `WebDriverManager.firefoxdriver().setup()` |
| Edge | `new EdgeDriver()` | `EdgeOptions` | `WebDriverManager.edgedriver().setup()` |
| Safari | `new SafariDriver()` | `SafariOptions` | Not needed (built-in macOS) |

### RemoteWebDriver (Grid/Cloud):
```java
ChromeOptions options = new ChromeOptions();
WebDriver driver = new RemoteWebDriver(new URL("http://hub:4444/wd/hub"), options);
```

---

# 2️⃣ WebDriver — Core Interface

```java
WebDriver driver = new ChromeDriver();
```

## 2a. Navigation Methods

| Method | Return Type | Description |
|--------|-------------|-------------|
| `driver.get(String url)` | `void` | Navigate to URL (waits for page load) |
| `driver.getCurrentUrl()` | `String` | Get current URL |
| `driver.getTitle()` | `String` | Get page title |
| `driver.getPageSource()` | `String` | Get full HTML source |
| `driver.close()` | `void` | Close current tab/window only |
| `driver.quit()` | `void` | Close ALL tabs/windows + kill driver |

### ⚠️ `close()` vs `quit()`:
```
driver.close() → closes CURRENT window/tab only
driver.quit()  → closes ALL windows + ends WebDriver session + kills process
Always use quit() in @AfterMethod / finally block!
```

### Navigation Interface:
```java
driver.navigate().to("https://example.com");   // same as get()
driver.navigate().back();                       // browser back
driver.navigate().forward();                    // browser forward
driver.navigate().refresh();                    // refresh page
```

---

## 2b. Finding Elements — 8 Locator Strategies

```java
WebElement element = driver.findElement(By.id("username"));
List<WebElement> elements = driver.findElements(By.className("item"));
```

| Strategy | Syntax | HTML Example | Usage |
|----------|--------|-------------|-------|
| `By.id("x")` | Matches `id` attr | `<input id="x">` | `By.id("username")` |
| `By.name("x")` | Matches `name` attr | `<input name="x">` | `By.name("email")` |
| `By.className("x")` | Matches single class | `<div class="x y">` | `By.className("btn")` |
| `By.tagName("x")` | Matches HTML tag | `<input>` | `By.tagName("input")` |
| `By.cssSelector("x")` | CSS selector | any | `By.cssSelector("#form .btn")` |
| `By.xpath("x")` | XPath expression | any | `By.xpath("//button[@type='submit']")` |
| `By.linkText("x")` | Exact `<a>` text | `<a>Click Here</a>` | `By.linkText("Click Here")` |
| `By.partialLinkText("x")` | Partial `<a>` text | `<a>Click Here</a>` | `By.partialLinkText("Click")` |

### ⚠️ findElement vs findElements:
```
findElement(By)  → returns FIRST match, throws NoSuchElementException if not found
findElements(By) → returns List (empty if none), NEVER throws exception
```

### Locator Priority (Best → Worst):
```
1. By.id()              → fastest, unique
2. By.name()            → good for forms
3. By.cssSelector()     → flexible, fast, recommended
4. By.xpath()           → most powerful, slowest
5. By.className()       → single class only, fragile
6. By.linkText()        → links only
7. By.tagName()         → too generic, rarely useful alone
8. By.partialLinkText() → links only, ambiguous
```

### CSS Selector Cheat Sheet:
```css
#myId                          /* ID */
.myClass                       /* class */
input                          /* tag */
input.myClass                  /* tag + class */
input[type='text']             /* attribute equals */
input[name^='user']            /* starts with */
input[name$='name']            /* ends with */
input[name*='ern']             /* contains */
div > p                        /* direct child */
div p                          /* any descendant */
div + p                        /* adjacent sibling */
div ~ p                        /* general sibling */
ul li:first-child              /* first child */
ul li:last-child               /* last child */
ul li:nth-child(2)             /* 2nd child (1-based) */
ul li:nth-child(odd)           /* odd children */
:not(.disabled)                /* negation */
input:checked                  /* checked inputs */
input:enabled                  /* enabled inputs */
input:disabled                 /* disabled inputs */
[data-testid='login-btn']      /* data attribute */
```

### XPath Cheat Sheet:
```xpath
//input[@id='username']                 /* attribute equals */
//button[text()='Submit']               /* exact text */
//button[contains(text(),'Sub')]        /* partial text */
//div[contains(@class,'active')]        /* partial class */
//input[starts-with(@id,'user')]        /* starts with */
//input[@type='text' and @name='email'] /* AND */
//input[@type='text' or @type='email']  /* OR */
//div[@class='parent']/child::input    /* direct child */
//div[@class='parent']//input          /* any descendant */
//div[@class='parent']/..              /* parent */
//input[@id='x']/ancestor::form        /* ancestor */
//tr[1]                                 /* first (1-based) */
//tr[last()]                            /* last */
//tr[position()<=3]                     /* first 3 */
//td/following-sibling::td              /* next sibling */
//td/preceding-sibling::td             /* prev sibling */
//input[@id='x']/following::div         /* anything after */
//input[@id='x']/preceding::div        /* anything before */
(//input[@type='text'])[1]              /* first globally */
//div[not(@class='hidden')]             /* NOT */
//div[normalize-space(text())='Hello']  /* trim whitespace */
//*[local-name()='svg']                 /* SVG/namespace */
```

---

## 2c. Window & Tab Management

| Method | Return Type | Description |
|--------|-------------|-------------|
| `driver.getWindowHandle()` | `String` | Current window handle |
| `driver.getWindowHandles()` | `Set<String>` | All window handles |
| `driver.switchTo().window(handle)` | `WebDriver` | Switch to window |
| `driver.switchTo().newWindow(WindowType.TAB)` | `WebDriver` | Open new tab (Selenium 4) |
| `driver.switchTo().newWindow(WindowType.WINDOW)` | `WebDriver` | Open new window (Selenium 4) |

### Complete Window Handling Pattern:
```java
// 1. Store original handle
String originalWindow = driver.getWindowHandle();

// 2. Trigger new tab/window
driver.findElement(By.id("new-tab-link")).click();

// 3. Wait for new window
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.numberOfWindowsToBe(2));

// 4. Switch to new window
for (String handle : driver.getWindowHandles()) {
    if (!handle.equals(originalWindow)) {
        driver.switchTo().window(handle);
        break;
    }
}

// 5. Work in new window
System.out.println(driver.getTitle());

// 6. Close and return
driver.close();
driver.switchTo().window(originalWindow);
```

---

# 3️⃣ WebElement — Element Interactions

```java
WebElement element = driver.findElement(By.id("username"));
```

## 3a. Action Methods

| Method | Return Type | Description |
|--------|-------------|-------------|
| `element.click()` | `void` | Click the element |
| `element.sendKeys(CharSequence...)` | `void` | Type text or send keys |
| `element.clear()` | `void` | Clear input/textarea content |
| `element.submit()` | `void` | Submit enclosing form (⚠️ deprecated) |

### sendKeys Common Patterns:
```java
// Type text
element.sendKeys("Hello World");

// Press special keys
element.sendKeys(Keys.ENTER);
element.sendKeys(Keys.TAB);
element.sendKeys(Keys.ESCAPE);
element.sendKeys(Keys.BACK_SPACE);
element.sendKeys(Keys.DELETE);
element.sendKeys(Keys.SPACE);

// Keyboard shortcuts
element.sendKeys(Keys.chord(Keys.CONTROL, "a"));      // Select all
element.sendKeys(Keys.chord(Keys.CONTROL, "c"));      // Copy
element.sendKeys(Keys.chord(Keys.CONTROL, "v"));      // Paste
element.sendKeys(Keys.chord(Keys.CONTROL, "x"));      // Cut
element.sendKeys(Keys.chord(Keys.SHIFT, Keys.TAB));   // Shift+Tab

// Clear then type
element.clear();
element.sendKeys("new value");

// File upload (<input type="file">)
element.sendKeys("/absolute/path/to/file.txt");
// Multiple files
element.sendKeys("/path/file1.txt\n/path/file2.txt");
```

### All Keys Constants:
```java
Keys.ENTER, Keys.RETURN, Keys.TAB, Keys.ESCAPE, Keys.SPACE
Keys.BACK_SPACE, Keys.DELETE
Keys.ARROW_UP, Keys.ARROW_DOWN, Keys.ARROW_LEFT, Keys.ARROW_RIGHT
Keys.HOME, Keys.END, Keys.PAGE_UP, Keys.PAGE_DOWN
Keys.CONTROL, Keys.SHIFT, Keys.ALT, Keys.COMMAND  // Mac
Keys.F1, Keys.F2, ..., Keys.F12
Keys.INSERT, Keys.NUMPAD0-9
Keys.NULL  // release all modifier keys
```

---

## 3b. Reading Element State

| Method | Return Type | Description |
|--------|-------------|-------------|
| `element.getText()` | `String` | Get visible text content |
| `element.getAttribute(String name)` | `String` | Get attribute/property value |
| `element.getDomAttribute(String name)` | `String` | Get HTML attribute only (Selenium 4) |
| `element.getDomProperty(String name)` | `String` | Get JS property only (Selenium 4) |
| `element.getCssValue(String prop)` | `String` | Get computed CSS value |
| `element.getTagName()` | `String` | Get HTML tag name (lowercase) |
| `element.getSize()` | `Dimension` | Width + height |
| `element.getLocation()` | `Point` | X, Y position on page |
| `element.getRect()` | `Rectangle` | Size + position combined |
| `element.isDisplayed()` | `boolean` | Is visible on page? |
| `element.isEnabled()` | `boolean` | Is not disabled? |
| `element.isSelected()` | `boolean` | Is checkbox/radio checked? Or option selected? |
| `element.getAriaRole()` | `String` | ARIA role attribute (Selenium 4) |
| `element.getAccessibleName()` | `String` | Accessible name (Selenium 4) |

### ⚠️ getAttribute() Behavior:
```java
// For <input id="email" value="initial" type="text">
// User types "hello" in the field:

element.getAttribute("value");      // "hello" (current typed value — property)
element.getDomAttribute("value");   // "initial" (original HTML attribute)
element.getDomProperty("value");    // "hello" (current JS property)

// For boolean attributes:
// <input type="checkbox" checked>
element.getAttribute("checked");    // "true"
element.isSelected();               // true (preferred)
```

### Common getCssValue() Properties:
```java
element.getCssValue("color");              // "rgba(0, 0, 0, 1)"
element.getCssValue("background-color");   // "rgba(255, 255, 255, 1)"
element.getCssValue("font-size");          // "14px"
element.getCssValue("font-weight");        // "700" (bold)
element.getCssValue("display");            // "block", "none", "flex"
element.getCssValue("visibility");         // "visible", "hidden"
element.getCssValue("border");             // "1px solid rgb(0,0,0)"
element.getCssValue("opacity");            // "1", "0.5"
```

---

## 3c. Finding Children

```java
WebElement parent = driver.findElement(By.id("parent"));
WebElement child = parent.findElement(By.className("child"));
List<WebElement> children = parent.findElements(By.tagName("li"));
```

### Table Iteration Pattern:
```java
WebElement table = driver.findElement(By.id("data-table"));
List<WebElement> rows = table.findElements(By.tagName("tr"));

for (WebElement row : rows) {
    List<WebElement> cells = row.findElements(By.tagName("td"));
    for (WebElement cell : cells) {
        System.out.print(cell.getText() + " | ");
    }
    System.out.println();
}

// Get specific cell
String value = rows.get(1).findElements(By.tagName("td")).get(2).getText();
```

---

## 3d. Shadow DOM (Selenium 4)

```java
WebElement shadowHost = driver.findElement(By.cssSelector("#shadow-host"));
SearchContext shadowRoot = shadowHost.getShadowRoot();
WebElement innerElement = shadowRoot.findElement(By.cssSelector("#inner-btn"));
innerElement.click();
```

---

# 4️⃣ TargetLocator — switchTo()

## 4a. Frames

| Method | Return Type | Description |
|--------|-------------|-------------|
| `switchTo().frame(int index)` | `WebDriver` | By index (0-based) |
| `switchTo().frame(String nameOrId)` | `WebDriver` | By name or id |
| `switchTo().frame(WebElement)` | `WebDriver` | By element reference |
| `switchTo().defaultContent()` | `WebDriver` | Back to main page (top) |
| `switchTo().parentFrame()` | `WebDriver` | Back to parent frame |

```java
// Enter frame
driver.switchTo().frame("frameName");
// OR
driver.switchTo().frame(driver.findElement(By.cssSelector("iframe.myFrame")));
// OR
driver.switchTo().frame(0);   // first frame

// Interact inside
driver.findElement(By.id("btn")).click();

// Back to main
driver.switchTo().defaultContent();

// Nested frames
driver.switchTo().frame("outer");
driver.switchTo().frame("inner");
driver.findElement(By.id("btn")).click();
driver.switchTo().parentFrame();           // back to outer
driver.switchTo().defaultContent();        // back to main
```

### Wait for Frame:
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt(By.id("myFrame")));
```

---

## 4b. Alerts

```java
// Trigger alert then switch
driver.findElement(By.id("alert-btn")).click();
Alert alert = driver.switchTo().alert();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `alert.accept()` | `void` | Click OK |
| `alert.dismiss()` | `void` | Click Cancel |
| `alert.getText()` | `String` | Get alert message |
| `alert.sendKeys(String)` | `void` | Type into prompt |

```java
// Simple alert
driver.findElement(By.id("alert-btn")).click();
Alert alert = driver.switchTo().alert();
System.out.println(alert.getText());
alert.accept();

// Confirm dialog
driver.findElement(By.id("confirm-btn")).click();
Alert confirm = driver.switchTo().alert();
confirm.dismiss();     // Cancel
// OR confirm.accept(); // OK

// Prompt dialog
driver.findElement(By.id("prompt-btn")).click();
Alert prompt = driver.switchTo().alert();
prompt.sendKeys("My input text");
prompt.accept();

// Wait for alert
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
Alert alert = wait.until(ExpectedConditions.alertIsPresent());
```

---

# 5️⃣ Options — manage()

## 5a. Window Management

```java
driver.manage().window().maximize();
driver.manage().window().minimize();
driver.manage().window().fullscreen();
driver.manage().window().setSize(new Dimension(1920, 1080));
driver.manage().window().setPosition(new Point(0, 0));

Dimension size = driver.manage().window().getSize();
Point position = driver.manage().window().getPosition();
```

## 5b. Cookies

| Method | Return Type | Description |
|--------|-------------|-------------|
| `manage().getCookies()` | `Set<Cookie>` | Get all cookies |
| `manage().getCookieNamed(name)` | `Cookie` | Get specific cookie |
| `manage().addCookie(cookie)` | `void` | Add cookie |
| `manage().deleteCookie(cookie)` | `void` | Delete specific cookie |
| `manage().deleteCookieNamed(name)` | `void` | Delete by name |
| `manage().deleteAllCookies()` | `void` | Delete all cookies |

```java
// Add
driver.manage().addCookie(new Cookie("token", "abc123"));

// Read
Cookie c = driver.manage().getCookieNamed("token");
String val = c.getValue();
String domain = c.getDomain();
String path = c.getPath();
Date expiry = c.getExpiry();

// Delete
driver.manage().deleteCookieNamed("token");
driver.manage().deleteAllCookies();

// Build cookie with all properties
Cookie cookie = new Cookie.Builder("name", "value")
    .domain(".example.com")
    .path("/")
    .expiresOn(new Date(System.currentTimeMillis() + 86400000))
    .isSecure(true)
    .isHttpOnly(true)
    .sameSite("Lax")
    .build();
```

## 5c. Timeouts

```java
// Implicit wait — applies to ALL findElement() calls
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));

// Page load timeout
driver.manage().timeouts().pageLoadTimeout(Duration.ofSeconds(30));

// Script timeout (for executeAsyncScript)
driver.manage().timeouts().scriptTimeout(Duration.ofSeconds(30));
```

### ⚠️ Implicit vs Explicit Wait — CRITICAL:
```
IMPLICIT WAIT:
  - Global setting, applies to EVERY findElement() call
  - Polls DOM repeatedly until element found or timeout
  - Set ONCE in setup
  - ⚠️ NEVER mix with explicit wait!

EXPLICIT WAIT (WebDriverWait):
  - Targeted, waits for SPECIFIC condition
  - More flexible: visibility, clickable, text present, etc.
  - ✅ PREFERRED approach

FLUENT WAIT:
  - Like explicit but with custom polling interval
  - Can ignore specific exceptions during polling
```

---

# 6️⃣ WebDriverWait & ExpectedConditions

```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
```

## All ExpectedConditions:

### Element Conditions:

| Condition | Returns | Description |
|-----------|---------|-------------|
| `presenceOfElementLocated(By)` | `WebElement` | Element exists in DOM (may be hidden) |
| `visibilityOfElementLocated(By)` | `WebElement` | Element visible on page |
| `visibilityOf(WebElement)` | `WebElement` | Known element becomes visible |
| `visibilityOfAllElementsLocatedBy(By)` | `List<WebElement>` | All matching visible |
| `visibilityOfAllElements(List<WebElement>)` | `List<WebElement>` | All given elements visible |
| `invisibilityOfElementLocated(By)` | `Boolean` | Element hidden or not in DOM |
| `invisibilityOf(WebElement)` | `Boolean` | Known element becomes hidden |
| `elementToBeClickable(By)` | `WebElement` | Visible + enabled |
| `elementToBeClickable(WebElement)` | `WebElement` | Known element clickable |
| `stalenessOf(WebElement)` | `Boolean` | Element becomes stale (re-rendered) |
| `presenceOfAllElementsLocatedBy(By)` | `List<WebElement>` | At least one element present |
| `numberOfElementsToBe(By, int)` | `List<WebElement>` | Exact count of elements |
| `numberOfElementsToBeMoreThan(By, int)` | `List<WebElement>` | More than N elements |
| `numberOfElementsToBeLessThan(By, int)` | `List<WebElement>` | Less than N elements |
| `presenceOfNestedElementLocatedBy(By, By)` | `WebElement` | Child inside parent |
| `presenceOfNestedElementsLocatedBy(By, By)` | `List<WebElement>` | Children inside parent |

### Text Conditions:

| Condition | Returns | Description |
|-----------|---------|-------------|
| `textToBePresentInElement(WebElement, String)` | `Boolean` | Element contains text |
| `textToBePresentInElementLocated(By, String)` | `Boolean` | Located element contains text |
| `textToBePresentInElementValue(By, String)` | `Boolean` | Input value contains text |
| `textToBe(By, String)` | `Boolean` | Element text equals exactly |

### Attribute Conditions:

| Condition | Returns | Description |
|-----------|---------|-------------|
| `attributeToBe(By, String, String)` | `Boolean` | Attribute equals value |
| `attributeContains(By, String, String)` | `Boolean` | Attribute contains value |
| `attributeToBeNotEmpty(WebElement, String)` | `Boolean` | Attribute is not empty |
| `domAttributeToBe(WebElement, String, String)` | `Boolean` | DOM attribute equals |
| `domPropertyToBe(WebElement, String, String)` | `Boolean` | DOM property equals |

### Selection Conditions:

| Condition | Returns | Description |
|-----------|---------|-------------|
| `elementToBeSelected(By)` | `Boolean` | Element is selected |
| `elementToBeSelected(WebElement)` | `Boolean` | Known element selected |
| `elementSelectionStateToBe(By, boolean)` | `Boolean` | Selection state matches |
| `elementSelectionStateToBe(WebElement, boolean)` | `Boolean` | Known element state |

### Page Conditions:

| Condition | Returns | Description |
|-----------|---------|-------------|
| `titleIs(String)` | `Boolean` | Exact title match |
| `titleContains(String)` | `Boolean` | Title contains text |
| `urlToBe(String)` | `Boolean` | Exact URL match |
| `urlContains(String)` | `Boolean` | URL contains text |
| `urlMatches(String regex)` | `Boolean` | URL matches regex |

### Frame & Alert Conditions:

| Condition | Returns | Description |
|-----------|---------|-------------|
| `frameToBeAvailableAndSwitchToIt(By)` | `WebDriver` | Frame ready, switches |
| `frameToBeAvailableAndSwitchToIt(String)` | `WebDriver` | Frame by name |
| `frameToBeAvailableAndSwitchToIt(int)` | `WebDriver` | Frame by index |
| `alertIsPresent()` | `Alert` | Alert dialog present |

### Window Conditions:

| Condition | Returns | Description |
|-----------|---------|-------------|
| `numberOfWindowsToBe(int)` | `Boolean` | Exact number of windows |

### Logical Conditions:

| Condition | Returns | Description |
|-----------|---------|-------------|
| `not(ExpectedCondition)` | `Boolean` | Negate condition |
| `and(ExpectedCondition...)` | `Boolean` | All true |
| `or(ExpectedCondition...)` | `Boolean` | Any true |

### JavaScript Conditions:

| Condition | Returns | Description |
|-----------|---------|-------------|
| `javaScriptThrowsNoExceptions(String)` | `Boolean` | JS runs without error |
| `jsReturnsValue(String)` | `Object` | JS returns non-null |

### Custom Wait Examples:
```java
// FluentWait with polling
Wait<WebDriver> fluentWait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofMillis(500))
    .ignoring(NoSuchElementException.class)
    .ignoring(StaleElementReferenceException.class)
    .withMessage("Element not found!");

WebElement el = fluentWait.until(d -> d.findElement(By.id("dynamic")));

// Custom lambda condition
wait.until(d -> {
    String text = d.findElement(By.id("status")).getText();
    return text.equals("Complete");
});

// Wait for page load via JS
wait.until(d -> ((JavascriptExecutor) d)
    .executeScript("return document.readyState").equals("complete"));

// Wait for jQuery AJAX
wait.until(d -> (Boolean) ((JavascriptExecutor) d)
    .executeScript("return jQuery.active == 0"));

// Wait for Angular
wait.until(d -> (Boolean) ((JavascriptExecutor) d)
    .executeScript("return window.getAllAngularTestabilities()" +
        ".findIndex(x=>!x.isStable()) === -1"));
```

---

# 7️⃣ Actions — Advanced Interactions

```java
Actions actions = new Actions(driver);
```

## All Actions Methods:

| Method | Return Type | Description |
|--------|-------------|-------------|
| `moveToElement(WebElement)` | `Actions` | Hover over element |
| `moveToElement(el, xOffset, yOffset)` | `Actions` | Hover with offset |
| `moveByOffset(x, y)` | `Actions` | Move mouse by offset |
| `click()` | `Actions` | Click at current position |
| `click(WebElement)` | `Actions` | Click specific element |
| `doubleClick()` | `Actions` | Double-click current |
| `doubleClick(WebElement)` | `Actions` | Double-click element |
| `contextClick()` | `Actions` | Right-click current |
| `contextClick(WebElement)` | `Actions` | Right-click element |
| `clickAndHold()` | `Actions` | Press mouse button |
| `clickAndHold(WebElement)` | `Actions` | Press on element |
| `release()` | `Actions` | Release mouse button |
| `release(WebElement)` | `Actions` | Release on element |
| `dragAndDrop(source, target)` | `Actions` | Drag and drop |
| `dragAndDropBy(el, x, y)` | `Actions` | Drag by pixel offset |
| `keyDown(Keys)` | `Actions` | Hold key down |
| `keyDown(WebElement, Keys)` | `Actions` | Hold key on element |
| `keyUp(Keys)` | `Actions` | Release key |
| `keyUp(WebElement, Keys)` | `Actions` | Release key on element |
| `sendKeys(CharSequence...)` | `Actions` | Send keys |
| `sendKeys(WebElement, CharSequence...)` | `Actions` | Send keys to element |
| `scrollToElement(WebElement)` | `Actions` | Scroll to element (Selenium 4) |
| `scrollByAmount(x, y)` | `Actions` | Scroll by pixels (Selenium 4) |
| `scrollFromOrigin(origin, x, y)` | `Actions` | Scroll from point (Selenium 4) |
| `pause(Duration)` | `Actions` | Wait between actions |
| `perform()` | `void` | **⚠️ EXECUTE — must call!** |
| `build()` | `Action` | Build without executing |

### Common Patterns:
```java
// Hover
actions.moveToElement(menu).perform();

// Hover → click submenu
actions.moveToElement(menu)
       .pause(Duration.ofMillis(300))
       .moveToElement(submenuItem)
       .click()
       .perform();

// Right-click (context menu)
actions.contextClick(element).perform();

// Double-click
actions.doubleClick(element).perform();

// Drag and drop
actions.dragAndDrop(source, target).perform();

// Manual drag
actions.clickAndHold(source)
       .moveToElement(target)
       .release()
       .perform();

// Ctrl+A → Delete (clear field alternative)
actions.keyDown(Keys.CONTROL)
       .sendKeys("a")
       .keyUp(Keys.CONTROL)
       .sendKeys(Keys.DELETE)
       .perform();

// Shift+Click (multi-select)
actions.keyDown(Keys.SHIFT)
       .click(element)
       .keyUp(Keys.SHIFT)
       .perform();

// Scroll to element (Selenium 4)
actions.scrollToElement(element).perform();

// Scroll by amount (Selenium 4)
actions.scrollByAmount(0, 500).perform();

// Scroll from element (Selenium 4)
WheelInput.ScrollOrigin origin = WheelInput.ScrollOrigin.fromElement(element);
actions.scrollFromOrigin(origin, 0, 300).perform();

// Slider (drag by offset)
actions.clickAndHold(slider)
       .moveByOffset(100, 0)
       .release()
       .perform();
```

---

# 8️⃣ Select — Dropdown Handling

```java
Select select = new Select(driver.findElement(By.id("dropdown")));
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `selectByVisibleText(String)` | `void` | Select by displayed text |
| `selectByValue(String)` | `void` | Select by `value` attribute |
| `selectByIndex(int)` | `void` | Select by index (0-based) |
| `deselectAll()` | `void` | Deselect all (multi-select) |
| `deselectByVisibleText(String)` | `void` | Deselect by text |
| `deselectByValue(String)` | `void` | Deselect by value |
| `deselectByIndex(int)` | `void` | Deselect by index |
| `getOptions()` | `List<WebElement>` | ALL options |
| `getAllSelectedOptions()` | `List<WebElement>` | ALL selected options |
| `getFirstSelectedOption()` | `WebElement` | Currently selected |
| `isMultiple()` | `boolean` | Is multi-select? |

```java
Select dropdown = new Select(driver.findElement(By.id("country")));

// Select
dropdown.selectByVisibleText("India");
dropdown.selectByValue("IN");
dropdown.selectByIndex(3);

// Get selected
String selected = dropdown.getFirstSelectedOption().getText();

// Get all options
List<WebElement> options = dropdown.getOptions();
for (WebElement opt : options) {
    System.out.println(opt.getText() + " → " + opt.getAttribute("value"));
}

// Count options
int count = dropdown.getOptions().size();

// Multi-select
Select multi = new Select(driver.findElement(By.id("skills")));
multi.selectByVisibleText("Java");
multi.selectByVisibleText("Python");
List<WebElement> selected = multi.getAllSelectedOptions();
multi.deselectAll();
```

### ⚠️ Non-standard Dropdowns (not `<select>`):
```java
// For custom dropdowns (div/ul/li based):
driver.findElement(By.cssSelector(".dropdown-trigger")).click();  // open
driver.findElement(By.xpath("//li[text()='Option']")).click();    // select
// Cannot use Select class — only works with <select> tags!
```

---

# 9️⃣ JavascriptExecutor

```java
JavascriptExecutor js = (JavascriptExecutor) driver;
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `executeScript(String, Object...)` | `Object` | Execute synchronous JS |
| `executeAsyncScript(String, Object...)` | `Object` | Execute async JS (with callback) |

### Essential JS Commands:
```java
JavascriptExecutor js = (JavascriptExecutor) driver;

// --- SCROLLING ---
js.executeScript("window.scrollTo(0, document.body.scrollHeight)");     // bottom
js.executeScript("window.scrollTo(0, 0)");                              // top
js.executeScript("window.scrollBy(0, 500)");                            // down 500px
js.executeScript("window.scrollBy(0, -500)");                           // up 500px
js.executeScript("arguments[0].scrollIntoView(true);", element);        // to element (top)
js.executeScript("arguments[0].scrollIntoView(false);", element);       // to element (bottom)
js.executeScript("arguments[0].scrollIntoView({behavior:'smooth'});", element);

// --- CLICKING ---
js.executeScript("arguments[0].click();", element);                     // force click

// --- VALUES ---
js.executeScript("arguments[0].value='text';", element);                // set value
js.executeScript("arguments[0].setAttribute('value','text');", element);
String val = (String) js.executeScript("return arguments[0].value;", element);

// --- TEXT ---
String text = (String) js.executeScript("return arguments[0].textContent;", element);
String inner = (String) js.executeScript("return arguments[0].innerText;", element);
String html = (String) js.executeScript("return arguments[0].innerHTML;", element);

// --- PAGE INFO ---
String title = (String) js.executeScript("return document.title;");
String url = (String) js.executeScript("return window.location.href;");
String domain = (String) js.executeScript("return document.domain;");
Long height = (Long) js.executeScript("return document.body.scrollHeight;");

// --- DOM MANIPULATION ---
js.executeScript("arguments[0].removeAttribute('readonly');", element);
js.executeScript("arguments[0].removeAttribute('disabled');", element);
js.executeScript("arguments[0].setAttribute('style','border:3px solid red');", element);
js.executeScript("arguments[0].style.display='block';", element);
js.executeScript("arguments[0].remove();", element);

// --- PAGE LOAD STATUS ---
String state = (String) js.executeScript("return document.readyState;");   // "complete"

// --- WINDOW ---
js.executeScript("window.open('https://url.com','_blank');");
js.executeScript("window.close();");
js.executeScript("window.stop();");                                        // stop loading

// --- ALERTS ---
js.executeScript("alert('Hello');");
js.executeScript("confirm('Sure?');");

// --- LOCAL/SESSION STORAGE ---
js.executeScript("localStorage.setItem('key','value');");
String val2 = (String) js.executeScript("return localStorage.getItem('key');");
js.executeScript("localStorage.removeItem('key');");
js.executeScript("localStorage.clear();");
js.executeScript("sessionStorage.setItem('key','value');");

// --- COMPUTED STYLE ---
String color = (String) js.executeScript(
    "return window.getComputedStyle(arguments[0]).color;", element);

// --- SHADOW DOM (alternative) ---
WebElement shadowEl = (WebElement) js.executeScript(
    "return arguments[0].shadowRoot.querySelector('#inner');", shadowHost);
```

---

# 🔟 TakesScreenshot

```java
// Full page
File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(src, new File("screenshot.png"));

// As bytes (for Cucumber/reports)
byte[] bytes = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);

// As Base64
String base64 = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BASE64);

// Element screenshot (Selenium 4)
File elSrc = element.getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(elSrc, new File("element.png"));
```

---

# 1️⃣1️⃣ Relative Locators (Selenium 4)

```java
import static org.openqa.selenium.support.locators.RelativeLocator.with;
```

| Method | Description |
|--------|-------------|
| `with(By).above(element)` | Find element above reference |
| `with(By).below(element)` | Find element below reference |
| `with(By).toLeftOf(element)` | Find element to the left |
| `with(By).toRightOf(element)` | Find element to the right |
| `with(By).near(element)` | Find element nearby (50px default) |
| `with(By).near(element, distance)` | Find within distance |

```java
WebElement password = driver.findElement(By.id("password"));

// Find input above password field
WebElement email = driver.findElement(with(By.tagName("input")).above(password));

// Find button below password field
WebElement login = driver.findElement(with(By.tagName("button")).below(password));

// Find label to left of input
WebElement label = driver.findElement(with(By.tagName("label")).toLeftOf(emailInput));

// Chain
WebElement el = driver.findElement(
    with(By.tagName("input")).below(header).toRightOf(label)
);
```

---

# 1️⃣2️⃣ Complete Flow Diagrams

## Flow 1: Basic Login Test
```
WebDriverManager.chromedriver().setup()
    → new ChromeDriver()
    → driver.manage().window().maximize()
    → driver.get("https://example.com/login")
    → driver.findElement(By.id("username")).sendKeys("admin")
    → driver.findElement(By.id("password")).sendKeys("pass123")
    → driver.findElement(By.id("login-btn")).click()
    → wait.until(urlContains("/dashboard"))
    → assertEquals("Dashboard", driver.getTitle())
    → driver.quit()
```

## Flow 2: Frame → Interact → Back
```
driver.switchTo().frame("frameName")
    → driver.findElement(By.id("input")).sendKeys("text")
    → driver.findElement(By.id("btn")).click()
    → driver.switchTo().defaultContent()
```

## Flow 3: Alert Handling
```
driver.findElement(By.id("trigger")).click()
    → Alert alert = driver.switchTo().alert()
    → String msg = alert.getText()
    → alert.accept()  // or alert.dismiss()
```

## Flow 4: New Tab Handling
```
String original = driver.getWindowHandle()
    → element.click()  // opens new tab
    → wait.until(numberOfWindowsToBe(2))
    → for(handle : getWindowHandles()) → switchTo().window(newHandle)
    → // interact
    → driver.close()
    → driver.switchTo().window(original)
```

## Flow 5: Dropdown
```
Select dropdown = new Select(findElement(By.id("dd")))
    → dropdown.selectByVisibleText("Option")
    → String val = dropdown.getFirstSelectedOption().getText()
```

## Flow 6: Drag & Drop
```
Actions actions = new Actions(driver)
    → actions.dragAndDrop(source, target).perform()
```

## Flow 7: File Upload
```
driver.findElement(By.id("file-input")).sendKeys("C:\\path\\file.txt")
```

## Flow 8: Explicit Wait → Act
```
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10))
    → WebElement el = wait.until(visibilityOfElementLocated(By.id("msg")))
    → String text = el.getText()
```

## Flow 9: JS Scroll → Screenshot
```
JavascriptExecutor js = (JavascriptExecutor) driver
    → js.executeScript("arguments[0].scrollIntoView(true);", element)
    → File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE)
    → FileUtils.copyFile(src, new File("screenshot.png"))
```

## Flow 10: Table Data Extraction
```
WebElement table = findElement(By.id("table"))
    → List<WebElement> rows = table.findElements(By.tagName("tr"))
    → for each row → findElements(By.tagName("td")) → getText()
```

---

# 1️⃣3️⃣ Common Exceptions & Fixes

| Exception | Cause | Fix |
|-----------|-------|-----|
| `NoSuchElementException` | Element not in DOM | Add explicit wait, check locator |
| `StaleElementReferenceException` | DOM re-rendered, ref invalid | Re-find element after page change |
| `ElementNotInteractableException` | Element exists but hidden/off-screen | Wait for visible, scroll into view |
| `ElementClickInterceptedException` | Another element overlaps | Wait, scroll, or JS click |
| `TimeoutException` | Wait condition never met | Increase timeout, check condition logic |
| `NoSuchFrameException` | Frame not found | Wait for frame, verify name/id |
| `NoAlertPresentException` | Alert not open yet | `wait.until(alertIsPresent())` |
| `NoSuchWindowException` | Window/tab closed | Verify handle before switching |
| `InvalidSelectorException` | Bad CSS/XPath syntax | Test in browser DevTools first |
| `WebDriverException` | Driver/browser crash | Update driver, check compatibility |
| `SessionNotCreatedException` | Version mismatch | Use WebDriverManager or Selenium Manager |
| `InvalidArgumentException` | Bad argument (e.g., null URL) | Validate inputs before passing |
| `MoveTargetOutOfBoundsException` | Element outside viewport | Scroll first, then interact |
| `UnhandledAlertException` | Unexpected alert blocking | Handle or dismiss alert first |
| `InsecureCertificateException` | SSL certificate issue | `options.setAcceptInsecureCerts(true)` |

---

# 1️⃣4️⃣ Quick Reference — All Methods in One Place

```java
// ===== SETUP =====
WebDriverManager.chromedriver().setup();
ChromeOptions options = new ChromeOptions();
options.addArguments("--headless", "--start-maximized");
WebDriver driver = new ChromeDriver(options);
driver.manage().window().maximize();
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
driver.manage().timeouts().pageLoadTimeout(Duration.ofSeconds(30));

// ===== NAVIGATION =====
driver.get(url);                               // void
driver.getCurrentUrl();                        // String
driver.getTitle();                             // String
driver.getPageSource();                        // String
driver.navigate().to(url);                     // void
driver.navigate().back();                      // void
driver.navigate().forward();                   // void
driver.navigate().refresh();                   // void

// ===== FINDING ELEMENTS =====
driver.findElement(By.id("x"));               // WebElement
driver.findElement(By.name("x"));             // WebElement
driver.findElement(By.className("x"));        // WebElement
driver.findElement(By.tagName("x"));          // WebElement
driver.findElement(By.cssSelector("x"));      // WebElement
driver.findElement(By.xpath("x"));            // WebElement
driver.findElement(By.linkText("x"));         // WebElement
driver.findElement(By.partialLinkText("x"));  // WebElement
driver.findElements(By.cssSelector("x"));     // List<WebElement>

// ===== ELEMENT ACTIONS =====
element.click();                               // void
element.sendKeys("text");                      // void
element.sendKeys(Keys.ENTER);                  // void
element.sendKeys(Keys.chord(Keys.CTRL, "a")); // void
element.clear();                               // void
element.submit();                              // void (deprecated)

// ===== ELEMENT STATE =====
element.getText();                             // String
element.getAttribute("href");                  // String
element.getDomAttribute("value");              // String (Sel 4)
element.getDomProperty("value");               // String (Sel 4)
element.getCssValue("color");                  // String
element.getTagName();                          // String
element.getSize();                             // Dimension
element.getLocation();                         // Point
element.getRect();                             // Rectangle
element.isDisplayed();                         // boolean
element.isEnabled();                           // boolean
element.isSelected();                          // boolean
element.getAriaRole();                         // String (Sel 4)
element.getAccessibleName();                   // String (Sel 4)
element.getShadowRoot();                       // SearchContext (Sel 4)
element.getScreenshotAs(OutputType.FILE);      // File (Sel 4)

// ===== CHILD ELEMENTS =====
element.findElement(By.cssSelector(".child")); // WebElement
element.findElements(By.tagName("li"));        // List<WebElement>

// ===== FRAMES =====
driver.switchTo().frame("name");               // WebDriver
driver.switchTo().frame(0);                    // WebDriver
driver.switchTo().frame(element);              // WebDriver
driver.switchTo().parentFrame();               // WebDriver
driver.switchTo().defaultContent();            // WebDriver

// ===== ALERTS =====
Alert alert = driver.switchTo().alert();       // Alert
alert.getText();                               // String
alert.accept();                                // void
alert.dismiss();                               // void
alert.sendKeys("text");                        // void

// ===== WINDOWS =====
driver.getWindowHandle();                      // String
driver.getWindowHandles();                     // Set<String>
driver.switchTo().window(handle);              // WebDriver
driver.switchTo().newWindow(WindowType.TAB);   // WebDriver (Sel 4)
driver.switchTo().newWindow(WindowType.WINDOW);// WebDriver (Sel 4)

// ===== WINDOW SIZE =====
driver.manage().window().maximize();           // void
driver.manage().window().minimize();           // void
driver.manage().window().fullscreen();         // void
driver.manage().window().getSize();            // Dimension
driver.manage().window().setSize(dim);         // void
driver.manage().window().getPosition();        // Point
driver.manage().window().setPosition(point);   // void

// ===== COOKIES =====
driver.manage().getCookies();                  // Set<Cookie>
driver.manage().getCookieNamed("x");           // Cookie
driver.manage().addCookie(cookie);             // void
driver.manage().deleteCookieNamed("x");        // void
driver.manage().deleteAllCookies();            // void

// ===== WAITS =====
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("x")));
wait.until(ExpectedConditions.elementToBeClickable(By.id("x")));
wait.until(ExpectedConditions.presenceOfElementLocated(By.id("x")));
wait.until(ExpectedConditions.invisibilityOfElementLocated(By.id("x")));
wait.until(ExpectedConditions.textToBePresentInElementLocated(By.id("x"), "text"));
wait.until(ExpectedConditions.titleContains("text"));
wait.until(ExpectedConditions.urlContains("text"));
wait.until(ExpectedConditions.alertIsPresent());
wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt("name"));
wait.until(ExpectedConditions.numberOfWindowsToBe(2));
wait.until(ExpectedConditions.stalenessOf(element));
wait.until(d -> d.findElement(By.id("x")).getText().equals("Done"));

// ===== ACTIONS =====
Actions actions = new Actions(driver);
actions.moveToElement(el).perform();           // hover
actions.doubleClick(el).perform();             // double-click
actions.contextClick(el).perform();            // right-click
actions.clickAndHold(el).release(tgt).perform(); // drag
actions.dragAndDrop(src, tgt).perform();       // drag & drop
actions.keyDown(Keys.SHIFT).click(el).keyUp(Keys.SHIFT).perform();
actions.scrollToElement(el).perform();         // Selenium 4
actions.scrollByAmount(0, 500).perform();      // Selenium 4

// ===== SELECT (DROPDOWN) =====
Select select = new Select(element);
select.selectByVisibleText("x");              // void
select.selectByValue("x");                    // void
select.selectByIndex(0);                      // void
select.getOptions();                          // List<WebElement>
select.getFirstSelectedOption();              // WebElement
select.getAllSelectedOptions();               // List<WebElement>
select.isMultiple();                          // boolean
select.deselectAll();                         // void

// ===== JAVASCRIPT =====
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("return document.title;");    // Object
js.executeScript("arguments[0].click();", el); // Object
js.executeScript("window.scrollTo(0, document.body.scrollHeight);");
js.executeScript("arguments[0].scrollIntoView(true);", el);

// ===== SCREENSHOT =====
((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
element.getScreenshotAs(OutputType.FILE);      // Selenium 4

// ===== RELATIVE LOCATORS (Selenium 4) =====
driver.findElement(with(By.tagName("input")).above(el));
driver.findElement(with(By.tagName("input")).below(el));
driver.findElement(with(By.tagName("label")).toLeftOf(el));
driver.findElement(with(By.tagName("button")).toRightOf(el));
driver.findElement(with(By.tagName("div")).near(el));

// ===== CLEANUP =====
driver.close();                                // void (current tab)
driver.quit();                                 // void (all + kill)
```

---

*Complete Selenium Methods Hierarchy with every method, return type, and flow. Bookmark this for daily use.*
*Last updated: April 30, 2026*

