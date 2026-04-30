# 🏗️ Selenium Complete Hierarchy — Methods, Return Types & Flow

---

## VISUAL HIERARCHY

```
Selenium WebDriver
│
├── WebDriver (interface)
│   ├── ChromeDriver
│   ├── FirefoxDriver
│   ├── EdgeDriver
│   └── RemoteWebDriver
│
├── WebElement (interface)
│   └── Every element found by findElement()
│
├── WebDriver.Navigation
│   └── driver.navigate()
│
├── WebDriver.Options
│   └── driver.manage()
│       ├── WebDriver.Timeouts → manage().timeouts()
│       └── WebDriver.Window → manage().window()
│
├── WebDriver.TargetLocator
│   └── driver.switchTo()
│
├── Alert (interface)
│   └── driver.switchTo().alert()
│
├── Actions (class)
│   └── new Actions(driver)
│
├── Select (class)
│   └── new Select(element)
│
├── JavascriptExecutor (interface)
│   └── (JavascriptExecutor) driver
│
├── TakesScreenshot (interface)
│   └── (TakesScreenshot) driver
│
├── WebDriverWait (class)
│   └── new WebDriverWait(driver, Duration)
│       └── .until(ExpectedConditions.xxx())
│
└── By (abstract class)
    ├── By.id()
    ├── By.name()
    ├── By.cssSelector()
    ├── By.xpath()
    ├── By.className()
    ├── By.tagName()
    ├── By.linkText()
    └── By.partialLinkText()
```

---

# 1️⃣ WebDriver — Browser Control

```java
WebDriver driver = new ChromeDriver();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `driver.get(String url)` | `void` | Open URL (waits for page load) |
| `driver.getCurrentUrl()` | `String` | Get current page URL |
| `driver.getTitle()` | `String` | Get page title |
| `driver.getPageSource()` | `String` | Get full page HTML source |
| `driver.findElement(By locator)` | `WebElement` | Find first matching element |
| `driver.findElements(By locator)` | `List<WebElement>` | Find all matching elements |
| `driver.close()` | `void` | Close current tab/window only |
| `driver.quit()` | `void` | Close all windows + end session |
| `driver.getWindowHandle()` | `String` | Get current window handle ID |
| `driver.getWindowHandles()` | `Set<String>` | Get all open window handle IDs |

### Flow:
```
new ChromeDriver() → driver.get(url) → driver.findElement() → interact → driver.quit()
```

---

# 2️⃣ WebElement — Element Interaction

```java
WebElement element = driver.findElement(By.id("username"));
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `element.click()` | `void` | Click the element |
| `element.sendKeys(String text)` | `void` | Type text into element |
| `element.sendKeys(Keys.ENTER)` | `void` | Press keyboard key |
| `element.clear()` | `void` | Clear input field |
| `element.getText()` | `String` | Get visible text content |
| `element.getAttribute(String name)` | `String` | Get attribute value (href, value, class) |
| `element.getCssValue(String prop)` | `String` | Get CSS property (color, font-size) |
| `element.getTagName()` | `String` | Get HTML tag name (input, div, button) |
| `element.getSize()` | `Dimension` | Get width and height |
| `element.getLocation()` | `Point` | Get x, y coordinates on page |
| `element.getRect()` | `Rectangle` | Get size + location combined |
| `element.isDisplayed()` | `boolean` | Is element visible on page? |
| `element.isEnabled()` | `boolean` | Is element enabled (not disabled)? |
| `element.isSelected()` | `boolean` | Is checkbox/radio selected? |
| `element.submit()` | `void` | Submit the form (if inside form) |
| `element.findElement(By locator)` | `WebElement` | Find child element inside this element |
| `element.findElements(By locator)` | `List<WebElement>` | Find all child elements inside |
| `element.getShadowRoot()` | `SearchContext` | Get shadow DOM root (Selenium 4) |
| `element.getDomProperty(String name)` | `String` | Get DOM property (Selenium 4) |
| `element.getDomAttribute(String name)` | `String` | Get DOM attribute (Selenium 4) |
| `element.getAriaRole()` | `String` | Get ARIA role (Selenium 4) |
| `element.getAccessibleName()` | `String` | Get accessible name (Selenium 4) |

### Flow:
```
findElement(By.id("x")) → clear() → sendKeys("text") → click()
findElement(By.id("x")) → getText() → assert
findElement(By.id("x")) → isDisplayed() → boolean check
```

---

# 3️⃣ WebDriver.Navigation — Page Navigation

```java
WebDriver.Navigation nav = driver.navigate();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `nav.to(String url)` | `void` | Navigate to URL (same as driver.get) |
| `nav.to(URL url)` | `void` | Navigate to URL object |
| `nav.back()` | `void` | Go back (browser back button) |
| `nav.forward()` | `void` | Go forward (browser forward button) |
| `nav.refresh()` | `void` | Refresh current page |

### Flow:
```
driver.navigate().to(url) → do stuff → driver.navigate().back() → driver.navigate().forward()
```

---

# 4️⃣ WebDriver.Options — Browser Options (manage())

```java
WebDriver.Options options = driver.manage();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `options.timeouts()` | `WebDriver.Timeouts` | Access timeout settings |
| `options.window()` | `WebDriver.Window` | Access window settings |
| `options.getCookies()` | `Set<Cookie>` | Get all cookies |
| `options.getCookieNamed(String name)` | `Cookie` | Get specific cookie |
| `options.addCookie(Cookie cookie)` | `void` | Add a cookie |
| `options.deleteCookieNamed(String name)` | `void` | Delete specific cookie |
| `options.deleteAllCookies()` | `void` | Delete all cookies |
| `options.logs()` | `Logs` | Access browser logs |

---

## 4a. WebDriver.Timeouts

```java
WebDriver.Timeouts timeouts = driver.manage().timeouts();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `timeouts.implicitlyWait(Duration d)` | `WebDriver.Timeouts` | Wait before throwing NoSuchElement |
| `timeouts.pageLoadTimeout(Duration d)` | `WebDriver.Timeouts` | Max time for page to load |
| `timeouts.scriptTimeout(Duration d)` | `WebDriver.Timeouts` | Max time for JS execution |

### Usage:
```java
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
driver.manage().timeouts().pageLoadTimeout(Duration.ofSeconds(30));
driver.manage().timeouts().scriptTimeout(Duration.ofSeconds(15));
```

---

## 4b. WebDriver.Window

```java
WebDriver.Window window = driver.manage().window();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `window.maximize()` | `void` | Maximize browser window |
| `window.minimize()` | `void` | Minimize browser window |
| `window.fullscreen()` | `void` | Enter fullscreen mode |
| `window.getSize()` | `Dimension` | Get window width × height |
| `window.setSize(Dimension d)` | `void` | Set window size |
| `window.getPosition()` | `Point` | Get window position (x, y) |
| `window.setPosition(Point p)` | `void` | Set window position |

### Usage:
```java
driver.manage().window().maximize();
driver.manage().window().setSize(new Dimension(1920, 1080));
driver.manage().window().setSize(new Dimension(375, 812));  // mobile viewport
```

---

# 5️⃣ WebDriver.TargetLocator — Context Switching (switchTo())

```java
WebDriver.TargetLocator target = driver.switchTo();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `target.frame(String name)` | `WebDriver` | Switch to frame by name/ID |
| `target.frame(int index)` | `WebDriver` | Switch to frame by index (0-based) |
| `target.frame(WebElement el)` | `WebDriver` | Switch to frame by element |
| `target.parentFrame()` | `WebDriver` | Switch to parent frame (one level up) |
| `target.defaultContent()` | `WebDriver` | Switch back to main page (top) |
| `target.window(String handle)` | `WebDriver` | Switch to window/tab by handle |
| `target.newWindow(WindowType type)` | `WebDriver` | Open new window/tab (Selenium 4) |
| `target.alert()` | `Alert` | Switch to alert dialog |
| `target.activeElement()` | `WebElement` | Get currently focused element |

### Flow — Frames:
```
driver.switchTo().frame("frameName")  → interact inside frame → driver.switchTo().defaultContent()
```

### Flow — Windows:
```
String parent = driver.getWindowHandle();
// click link that opens new tab
Set<String> all = driver.getWindowHandles();
for (String handle : all) {
    if (!handle.equals(parent)) {
        driver.switchTo().window(handle);  // switch to new tab
    }
}
// do stuff in new tab
driver.close();                            // close new tab
driver.switchTo().window(parent);          // back to original
```

### Flow — New Window (Selenium 4):
```java
driver.switchTo().newWindow(WindowType.TAB);     // open new tab
driver.switchTo().newWindow(WindowType.WINDOW);  // open new window
```

---

# 6️⃣ Alert — Dialog Handling

```java
Alert alert = driver.switchTo().alert();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `alert.accept()` | `void` | Click OK/Accept button |
| `alert.dismiss()` | `void` | Click Cancel/Dismiss button |
| `alert.getText()` | `String` | Get alert message text |
| `alert.sendKeys(String text)` | `void` | Type into prompt dialog |

### Flow:
```
// Wait for alert first
wait.until(ExpectedConditions.alertIsPresent());
Alert alert = driver.switchTo().alert();
String text = alert.getText();    // read message
alert.accept();                   // click OK
// or
alert.dismiss();                  // click Cancel
```

### Three types:
```
1. Simple Alert    → alert.accept()
2. Confirm Alert   → alert.accept() or alert.dismiss()
3. Prompt Alert    → alert.sendKeys("text") → alert.accept()
```

---

# 7️⃣ Actions — Advanced User Interactions

```java
Actions actions = new Actions(driver);
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `actions.click(WebElement el)` | `Actions` | Click element |
| `actions.doubleClick(WebElement el)` | `Actions` | Double-click element |
| `actions.contextClick(WebElement el)` | `Actions` | Right-click element |
| `actions.moveToElement(WebElement el)` | `Actions` | Hover/mouse over element |
| `actions.moveToElement(el, x, y)` | `Actions` | Hover with offset |
| `actions.clickAndHold(WebElement el)` | `Actions` | Press and hold mouse |
| `actions.release()` | `Actions` | Release mouse button |
| `actions.dragAndDrop(source, target)` | `Actions` | Drag from source to target |
| `actions.dragAndDropBy(el, x, y)` | `Actions` | Drag by pixel offset |
| `actions.sendKeys(CharSequence keys)` | `Actions` | Press keyboard keys |
| `actions.keyDown(Keys key)` | `Actions` | Hold key down (Ctrl, Shift) |
| `actions.keyUp(Keys key)` | `Actions` | Release held key |
| `actions.scrollToElement(WebElement el)` | `Actions` | Scroll to element (Selenium 4) |
| `actions.scrollByAmount(x, y)` | `Actions` | Scroll by pixels (Selenium 4) |
| `actions.pause(Duration d)` | `Actions` | Pause between actions |
| `actions.perform()` | `void` | **EXECUTE all chained actions** |
| `actions.build()` | `Action` | Build without executing |

### ⚠️ IMPORTANT: Always end with `.perform()` — nothing happens without it!

### Flow — Hover:
```java
actions.moveToElement(menuItem).perform();
// submenu appears
driver.findElement(By.id("submenu-option")).click();
```

### Flow — Drag and Drop:
```java
WebElement source = driver.findElement(By.id("draggable"));
WebElement target = driver.findElement(By.id("droppable"));
actions.dragAndDrop(source, target).perform();
```

### Flow — Keyboard Shortcuts:
```java
// Ctrl+A (select all)
actions.keyDown(Keys.CONTROL).sendKeys("a").keyUp(Keys.CONTROL).perform();

// Ctrl+C (copy)
actions.keyDown(Keys.CONTROL).sendKeys("c").keyUp(Keys.CONTROL).perform();

// Shift+Click (multi-select)
actions.keyDown(Keys.SHIFT).click(element).keyUp(Keys.SHIFT).perform();
```

### Flow — Chain Multiple Actions:
```java
actions.moveToElement(element)
       .click()
       .sendKeys("text")
       .sendKeys(Keys.ENTER)
       .perform();
```

---

# 8️⃣ Select — Dropdown Handling

```java
Select select = new Select(driver.findElement(By.id("dropdown")));
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `select.selectByVisibleText(String text)` | `void` | Select by display text |
| `select.selectByValue(String value)` | `void` | Select by `value` attribute |
| `select.selectByIndex(int index)` | `void` | Select by position (0-based) |
| `select.deselectAll()` | `void` | Deselect all (multi-select only) |
| `select.deselectByVisibleText(String)` | `void` | Deselect by text |
| `select.deselectByValue(String)` | `void` | Deselect by value |
| `select.deselectByIndex(int)` | `void` | Deselect by index |
| `select.getOptions()` | `List<WebElement>` | Get all options |
| `select.getAllSelectedOptions()` | `List<WebElement>` | Get all selected options |
| `select.getFirstSelectedOption()` | `WebElement` | Get currently selected option |
| `select.isMultiple()` | `boolean` | Is it a multi-select dropdown? |

### Flow:
```java
Select dropdown = new Select(driver.findElement(By.id("country")));
dropdown.selectByVisibleText("India");

// Get selected value
String selected = dropdown.getFirstSelectedOption().getText();

// Get all options
List<WebElement> options = dropdown.getOptions();
for (WebElement opt : options) {
    System.out.println(opt.getText());
}
```

### ⚠️ Only works for `<select>` HTML tag. For custom dropdowns:
```java
driver.findElement(By.cssSelector(".custom-dropdown")).click();
driver.findElement(By.xpath("//li[text()='Option']")).click();
```

---

# 9️⃣ JavascriptExecutor — Execute JavaScript

```java
JavascriptExecutor js = (JavascriptExecutor) driver;
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `js.executeScript(String script, Object... args)` | `Object` | Execute JS synchronously |
| `js.executeAsyncScript(String script, Object... args)` | `Object` | Execute JS asynchronously |

### Common Usages:

```java
JavascriptExecutor js = (JavascriptExecutor) driver;

// Click (when normal click fails)
js.executeScript("arguments[0].click();", element);

// Scroll to element
js.executeScript("arguments[0].scrollIntoView(true);", element);

// Scroll to bottom
js.executeScript("window.scrollTo(0, document.body.scrollHeight);");

// Scroll to top
js.executeScript("window.scrollTo(0, 0);");

// Scroll by pixels
js.executeScript("window.scrollBy(0, 500);");

// Get page title
String title = (String) js.executeScript("return document.title;");

// Get element text (hidden elements)
String text = (String) js.executeScript("return arguments[0].textContent;", element);

// Set value (bypasses events)
js.executeScript("arguments[0].value='admin';", element);

// Remove attribute
js.executeScript("arguments[0].removeAttribute('disabled');", element);

// Highlight element (for debugging)
js.executeScript("arguments[0].style.border='3px solid red';", element);

// Check page load status
String readyState = (String) js.executeScript("return document.readyState;");

// Open new tab
js.executeScript("window.open('https://google.com', '_blank');");

// Get element attribute
String href = (String) js.executeScript("return arguments[0].getAttribute('href');", element);

// Trigger hidden file upload
js.executeScript("document.getElementById('fileInput').style.display='block';");
```

### Flow:
```
Cast driver to JavascriptExecutor → executeScript("JS code", args) → cast return value
```

---

# 🔟 TakesScreenshot — Capture Screenshots

```java
TakesScreenshot ts = (TakesScreenshot) driver;
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `ts.getScreenshotAs(OutputType.FILE)` | `File` | Screenshot as file |
| `ts.getScreenshotAs(OutputType.BYTES)` | `byte[]` | Screenshot as byte array |
| `ts.getScreenshotAs(OutputType.BASE64)` | `String` | Screenshot as Base64 string |

### Usage:

```java
// Save to file
File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(src, new File("screenshot.png"));

// For Cucumber report
byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
scenario.attach(screenshot, "image/png", "failure-screenshot");

// Element screenshot (Selenium 4)
WebElement element = driver.findElement(By.id("chart"));
File elementShot = element.getScreenshotAs(OutputType.FILE);
```

---

# 1️⃣1️⃣ WebDriverWait & ExpectedConditions — Explicit Waits

```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15));
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `wait.until(ExpectedCondition<T> condition)` | `T` | Wait until condition is true, returns result |
| `wait.withMessage(String msg)` | `WebDriverWait` | Custom timeout error message |
| `wait.pollingEvery(Duration d)` | `WebDriverWait` | Check frequency (default 500ms) |
| `wait.ignoring(Class exception)` | `WebDriverWait` | Ignore specific exceptions while waiting |

### All ExpectedConditions:

| Condition | Returns | Waits for |
|-----------|---------|-----------|
| `visibilityOf(element)` | `WebElement` | Element is visible |
| `visibilityOfElementLocated(By)` | `WebElement` | Located element is visible |
| `invisibilityOf(element)` | `Boolean` | Element disappears |
| `invisibilityOfElementLocated(By)` | `Boolean` | Located element disappears |
| `elementToBeClickable(element)` | `WebElement` | Element is visible + enabled |
| `elementToBeClickable(By)` | `WebElement` | Located element is clickable |
| `presenceOfElementLocated(By)` | `WebElement` | Element exists in DOM (may not be visible) |
| `presenceOfAllElementsLocatedBy(By)` | `List<WebElement>` | All elements present |
| `textToBePresentInElement(el, text)` | `Boolean` | Element contains text |
| `textToBePresentInElementValue(el, text)` | `Boolean` | Input value contains text |
| `titleIs(String)` | `Boolean` | Page title equals |
| `titleContains(String)` | `Boolean` | Page title contains |
| `urlToBe(String)` | `Boolean` | URL exactly matches |
| `urlContains(String)` | `Boolean` | URL contains string |
| `alertIsPresent()` | `Alert` | Alert dialog is shown |
| `frameToBeAvailableAndSwitchToIt(name)` | `WebDriver` | Frame is loaded + switches to it |
| `numberOfElementsToBe(By, int)` | `List<WebElement>` | Exactly N elements exist |
| `numberOfElementsToBeMoreThan(By, int)` | `List<WebElement>` | More than N elements |
| `numberOfElementsToBeLessThan(By, int)` | `List<WebElement>` | Less than N elements |
| `stalenessOf(element)` | `Boolean` | Element is no longer in DOM |
| `elementToBeSelected(element)` | `Boolean` | Checkbox/radio is selected |
| `attributeToBe(el, attr, val)` | `Boolean` | Attribute has expected value |
| `attributeContains(el, attr, val)` | `Boolean` | Attribute contains value |
| `numberOfWindowsToBe(int)` | `Boolean` | N windows/tabs are open |
| `jsReturnsValue(String script)` | `Object` | JS returns non-null value |

### Flow:
```java
// Wait → Find → Interact
WebElement el = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("btn")));
el.click();

// Wait for page transition
wait.until(ExpectedConditions.urlContains("/dashboard"));

// Wait for loading spinner to disappear
wait.until(ExpectedConditions.invisibilityOfElementLocated(By.cssSelector(".spinner")));

// Fluent Wait (custom polling + ignore exceptions)
Wait<WebDriver> fluentWait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofMillis(200))
    .ignoring(NoSuchElementException.class)
    .ignoring(StaleElementReferenceException.class);

WebElement el = fluentWait.until(d -> d.findElement(By.id("dynamic")));
```

---

# 1️⃣2️⃣ By — Locator Strategies

```java
// All static methods return By object
By locator = By.id("username");
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `By.id(String id)` | `By` | Find by id attribute |
| `By.name(String name)` | `By` | Find by name attribute |
| `By.cssSelector(String css)` | `By` | Find by CSS selector |
| `By.xpath(String xpath)` | `By` | Find by XPath expression |
| `By.className(String className)` | `By` | Find by class (single class only) |
| `By.tagName(String tag)` | `By` | Find by HTML tag |
| `By.linkText(String text)` | `By` | Find `<a>` by exact link text |
| `By.partialLinkText(String text)` | `By` | Find `<a>` by partial link text |

---

# 1️⃣3️⃣ ChromeOptions / FirefoxOptions — Browser Configuration

```java
ChromeOptions options = new ChromeOptions();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `options.addArguments(String... args)` | `ChromeOptions` | Add browser arguments |
| `options.setHeadless(boolean)` | `ChromeOptions` | Run without UI (deprecated, use arg) |
| `options.addExtensions(File ext)` | `ChromeOptions` | Add browser extension |
| `options.setExperimentalOption(k, v)` | `ChromeOptions` | Set experimental options |
| `options.setBinary(String path)` | `ChromeOptions` | Set browser binary path |
| `options.setCapability(String k, Object v)` | `void` | Set capability |

### Common Arguments:
```java
ChromeOptions options = new ChromeOptions();
options.addArguments("--headless=new");           // headless mode
options.addArguments("--start-maximized");        // maximize
options.addArguments("--incognito");              // private mode
options.addArguments("--disable-notifications");  // block popups
options.addArguments("--disable-gpu");            // for CI
options.addArguments("--window-size=1920,1080");  // set size
options.addArguments("--no-sandbox");             // for Docker
options.addArguments("--disable-dev-shm-usage"); // for Docker

// Set download directory
Map<String, Object> prefs = new HashMap<>();
prefs.put("download.default_directory", "C:\\downloads");
options.setExperimentalOption("prefs", prefs);

WebDriver driver = new ChromeDriver(options);
```

---

# 1️⃣4️⃣ Keys — Keyboard Constants

```java
element.sendKeys(Keys.ENTER);
```

| Constant | Description |
|----------|-------------|
| `Keys.ENTER` / `Keys.RETURN` | Enter key |
| `Keys.TAB` | Tab key |
| `Keys.ESCAPE` | Escape key |
| `Keys.BACK_SPACE` | Backspace |
| `Keys.DELETE` | Delete key |
| `Keys.SPACE` | Spacebar |
| `Keys.CONTROL` | Ctrl key |
| `Keys.SHIFT` | Shift key |
| `Keys.ALT` | Alt key |
| `Keys.ARROW_UP/DOWN/LEFT/RIGHT` | Arrow keys |
| `Keys.HOME` | Home key |
| `Keys.END` | End key |
| `Keys.PAGE_UP` / `Keys.PAGE_DOWN` | Page up/down |
| `Keys.F1` to `Keys.F12` | Function keys |

### Common Combos:
```java
element.sendKeys(Keys.CONTROL + "a");          // Select all
element.sendKeys(Keys.CONTROL + "c");          // Copy
element.sendKeys(Keys.CONTROL + "v");          // Paste
element.sendKeys(Keys.chord(Keys.CONTROL, "a")); // Alternative syntax
```

---

# 1️⃣5️⃣ Complete Flow Diagrams

## Flow 1: Basic Test

```
ChromeOptions → new ChromeDriver(options)
    → driver.manage().window().maximize()
    → driver.manage().timeouts().implicitlyWait(10s)
    → driver.get("https://url.com")
    → driver.findElement(By.id("user")).sendKeys("admin")
    → driver.findElement(By.id("pass")).sendKeys("pass123")
    → driver.findElement(By.id("login")).click()
    → assert driver.getTitle().equals("Dashboard")
    → driver.quit()
```

## Flow 2: Wait + Interact

```
new WebDriverWait(driver, 15s)
    → wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("x")))
    → returns WebElement
    → element.click() / element.getText() / etc.
```

## Flow 3: Frame Handling

```
driver.switchTo().frame("frameName")
    → driver.findElement(By.id("innerBtn")).click()
    → driver.switchTo().defaultContent()
```

## Flow 4: Multiple Windows

```
driver.getWindowHandle() → save as "parent"
    → click link (opens new tab)
    → driver.getWindowHandles() → iterate
    → driver.switchTo().window(newHandle)
    → interact in new tab
    → driver.close()
    → driver.switchTo().window(parent)
```

## Flow 5: Alert

```
action triggers alert
    → wait.until(ExpectedConditions.alertIsPresent())
    → driver.switchTo().alert()
    → alert.getText() / alert.accept() / alert.dismiss()
```

## Flow 6: Actions (Hover → Click submenu)

```
new Actions(driver)
    → .moveToElement(menu)
    → .perform()
    → findElement(submenuItem).click()
```

## Flow 7: Dropdown

```
new Select(driver.findElement(By.id("dropdown")))
    → select.selectByVisibleText("Option")
    → select.getFirstSelectedOption().getText()
```

## Flow 8: JavaScript Execution

```
(JavascriptExecutor) driver
    → js.executeScript("arguments[0].scrollIntoView(true);", element)
    → js.executeScript("arguments[0].click();", element)
```

## Flow 9: Screenshot on Failure

```
(TakesScreenshot) driver
    → ts.getScreenshotAs(OutputType.BYTES)
    → scenario.attach(bytes, "image/png", "name")
```

---

# 1️⃣6️⃣ Method Chaining Quick Reference

```java
// Everything from driver
driver.get(url);                                    // void
driver.findElement(By.id("x"));                     // WebElement
driver.findElements(By.css(".x"));                  // List<WebElement>
driver.getCurrentUrl();                             // String
driver.getTitle();                                  // String
driver.getPageSource();                             // String
driver.getWindowHandle();                           // String
driver.getWindowHandles();                          // Set<String>
driver.close();                                     // void
driver.quit();                                      // void

// From driver.navigate()
driver.navigate().to(url);                          // void
driver.navigate().back();                           // void
driver.navigate().forward();                        // void
driver.navigate().refresh();                        // void

// From driver.manage()
driver.manage().window().maximize();                // void
driver.manage().timeouts().implicitlyWait(dur);     // Timeouts
driver.manage().deleteAllCookies();                 // void
driver.manage().getCookies();                       // Set<Cookie>

// From driver.switchTo()
driver.switchTo().frame("name");                    // WebDriver
driver.switchTo().defaultContent();                 // WebDriver
driver.switchTo().window(handle);                   // WebDriver
driver.switchTo().alert();                          // Alert
driver.switchTo().activeElement();                  // WebElement
driver.switchTo().newWindow(WindowType.TAB);        // WebDriver

// From element
element.click();                                    // void
element.sendKeys("text");                           // void
element.clear();                                    // void
element.getText();                                  // String
element.getAttribute("href");                       // String
element.isDisplayed();                              // boolean
element.isEnabled();                                // boolean
element.isSelected();                               // boolean
element.getTagName();                               // String
element.getSize();                                  // Dimension
element.getLocation();                              // Point
element.findElement(By.css(".child"));              // WebElement
element.getShadowRoot();                            // SearchContext

// From Actions
new Actions(driver).moveToElement(el).click().perform();  // void
new Actions(driver).dragAndDrop(src, tgt).perform();      // void
new Actions(driver).doubleClick(el).perform();            // void
new Actions(driver).contextClick(el).perform();           // void

// From Select
new Select(el).selectByVisibleText("x");            // void
new Select(el).getFirstSelectedOption();            // WebElement
new Select(el).getOptions();                        // List<WebElement>

// From JavascriptExecutor
((JavascriptExecutor) driver).executeScript("...");  // Object

// From TakesScreenshot
((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);   // File
((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);  // byte[]

// From WebDriverWait
new WebDriverWait(driver, Duration.ofSeconds(10))
    .until(ExpectedConditions.visibilityOf(el));     // WebElement
```

---

# 1️⃣7️⃣ Exception Hierarchy

```
WebDriverException (parent of all)
├── NoSuchElementException         → Element not found
├── StaleElementReferenceException → Element no longer in DOM
├── ElementNotInteractableException→ Element exists but can't interact
├── ElementClickInterceptedException → Another element covers it
├── TimeoutException               → Wait timed out
├── NoSuchFrameException           → Frame not found
├── NoSuchWindowException          → Window/tab not found
├── NoAlertPresentException        → No alert to switch to
├── InvalidSelectorException       → Bad locator syntax
├── SessionNotCreatedException     → Browser/driver version mismatch
└── UnhandledAlertException        → Unexpected alert blocking action
```

### How to handle each:

| Exception | Solution |
|-----------|----------|
| `NoSuchElementException` | Use explicit wait, check locator |
| `StaleElementReferenceException` | Re-find element, use wait |
| `ElementNotInteractableException` | Scroll into view, wait for visibility |
| `ElementClickInterceptedException` | Wait for overlay to disappear, use JS click |
| `TimeoutException` | Increase wait time, check if element actually loads |
| `NoSuchFrameException` | Wait for frame, check frame name/index |
| `SessionNotCreatedException` | Update driver to match browser version |

---

*Complete Selenium hierarchy with every method, return type, and flow. Bookmark this for daily use.*
*Last updated: April 29, 2026*

