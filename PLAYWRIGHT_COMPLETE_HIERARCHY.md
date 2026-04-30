# 🏗️ Playwright Complete Hierarchy — Methods, Return Types & Flow

---

## VISUAL HIERARCHY

```
Playwright (Top Level)
│
├── Playwright (entry point)
│   ├── playwright.chromium()  → BrowserType
│   ├── playwright.firefox()   → BrowserType
│   └── playwright.webkit()    → BrowserType
│
├── BrowserType
│   └── browserType.launch()   → Browser
│
├── Browser
│   └── browser.newContext()   → BrowserContext
│       └── context.newPage()  → Page
│
├── Page (MAIN — you use this 90% of the time)
│   ├── page.locator()         → Locator
│   ├── page.getByTestId()     → Locator
│   ├── page.getByRole()       → Locator
│   ├── page.getByText()       → Locator
│   ├── page.getByLabel()      → Locator
│   ├── page.getByPlaceholder()→ Locator
│   ├── page.frameLocator()    → FrameLocator
│   ├── page.keyboard         → Keyboard
│   ├── page.mouse            → Mouse
│   └── page.request          → APIRequestContext
│
├── Locator (element reference — auto-waits, never stale)
│   ├── locator.click()
│   ├── locator.fill()
│   ├── locator.textContent()
│   ├── locator.filter()       → Locator
│   ├── locator.locator()      → Locator (chaining)
│   ├── locator.nth()          → Locator
│   ├── locator.first()        → Locator
│   └── locator.last()         → Locator
│
├── FrameLocator
│   └── frameLocator.locator() → Locator
│
├── Keyboard
│   ├── keyboard.press()
│   ├── keyboard.type()
│   └── keyboard.down() / up()
│
├── Mouse
│   ├── mouse.click()
│   ├── mouse.dblclick()
│   ├── mouse.move()
│   └── mouse.wheel()
│
├── Dialog (alert/confirm/prompt)
│   ├── dialog.accept()
│   ├── dialog.dismiss()
│   └── dialog.message()
│
├── FileChooser
│   └── fileChooser.setFiles()
│
├── Download
│   └── download.path() / download.saveAs()
│
├── APIRequestContext
│   ├── request.get()          → APIResponse
│   ├── request.post()         → APIResponse
│   ├── request.put()          → APIResponse
│   └── request.delete()       → APIResponse
│
└── Assertions (PlaywrightAssertions)
    ├── assertThat(locator).isVisible()
    ├── assertThat(locator).hasText()
    ├── assertThat(page).hasURL()
    └── assertThat(page).hasTitle()
```

---

## CREATION FLOW

```
Playwright.create()
    → playwright.chromium().launch()           → Browser
        → browser.newContext()                 → BrowserContext
            → context.newPage()                → Page
                → page.navigate(url)
                → page.locator("#id")          → Locator
                    → locator.click()
                    → locator.fill("text")
                → page.close()
            → context.close()
        → browser.close()
    → playwright.close()
```

---

# 1️⃣ Playwright — Entry Point

```java
Playwright playwright = Playwright.create();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `Playwright.create()` | `Playwright` | Create Playwright instance |
| `playwright.chromium()` | `BrowserType` | Get Chromium browser type |
| `playwright.firefox()` | `BrowserType` | Get Firefox browser type |
| `playwright.webkit()` | `BrowserType` | Get WebKit (Safari) browser type |
| `playwright.close()` | `void` | Close Playwright and free resources |

---

# 2️⃣ BrowserType — Launch Browsers

```java
BrowserType browserType = playwright.chromium();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `browserType.launch()` | `Browser` | Launch browser with defaults |
| `browserType.launch(options)` | `Browser` | Launch with custom options |
| `browserType.launchPersistentContext(path, options)` | `BrowserContext` | Launch with user data dir |
| `browserType.name()` | `String` | Get browser name ("chromium", "firefox", "webkit") |
| `browserType.connect(wsEndpoint)` | `Browser` | Connect to remote browser |

### Launch Options:

```java
Browser browser = playwright.chromium().launch(
    new BrowserType.LaunchOptions()
        .setHeadless(false)              // show browser UI
        .setSlowMo(100)                  // slow down actions by 100ms
        .setChannel("chrome")            // use installed Chrome instead of Chromium
        .setTimeout(30000)               // launch timeout
        .setArgs(Arrays.asList("--start-maximized"))
);
```

---

# 3️⃣ Browser — Browser Instance

```java
Browser browser = playwright.chromium().launch();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `browser.newContext()` | `BrowserContext` | Create new isolated context (like incognito) |
| `browser.newContext(options)` | `BrowserContext` | Context with custom settings |
| `browser.newPage()` | `Page` | Shortcut: creates context + page in one call |
| `browser.contexts()` | `List<BrowserContext>` | Get all open contexts |
| `browser.isConnected()` | `boolean` | Is browser still connected? |
| `browser.version()` | `String` | Get browser version |
| `browser.close()` | `void` | Close browser and all contexts/pages |

### Flow:
```
browser.newContext() → context.newPage() → interact → context.close() → browser.close()
// OR shortcut:
browser.newPage() → interact → browser.close()
```

---

# 4️⃣ BrowserContext — Isolated Session

```java
BrowserContext context = browser.newContext();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `context.newPage()` | `Page` | Create new page/tab in this context |
| `context.pages()` | `List<Page>` | Get all pages in this context |
| `context.cookies()` | `List<Cookie>` | Get all cookies |
| `context.addCookies(List<Cookie>)` | `void` | Add cookies |
| `context.clearCookies()` | `void` | Delete all cookies |
| `context.storageState()` | `String` | Get storage state (for auth reuse) |
| `context.setDefaultTimeout(ms)` | `void` | Set default timeout for all actions |
| `context.setDefaultNavigationTimeout(ms)` | `void` | Set navigation timeout |
| `context.grantPermissions(List<String>)` | `void` | Grant browser permissions |
| `context.setGeolocation(geo)` | `void` | Set fake geolocation |
| `context.route(pattern, handler)` | `void` | Intercept network requests |
| `context.unroute(pattern)` | `void` | Remove route handler |
| `context.setExtraHTTPHeaders(Map)` | `void` | Add headers to all requests |
| `context.tracing().start()` | `void` | Start tracing |
| `context.tracing().stop(options)` | `void` | Stop and save trace |
| `context.close()` | `void` | Close context and all its pages |

### Context Options:

```java
BrowserContext context = browser.newContext(
    new Browser.NewContextOptions()
        .setViewportSize(1920, 1080)             // viewport size
        .setLocale("en-US")                      // language
        .setTimezoneId("America/New_York")       // timezone
        .setGeolocation(40.7128, -74.0060)       // fake GPS
        .setPermissions(Arrays.asList("geolocation"))
        .setStorageStatePath(Paths.get("auth.json"))  // reuse login
        .setRecordVideoDir(Paths.get("videos/"))      // record video
        .setIgnoreHTTPSErrors(true)              // skip SSL errors
        .setUserAgent("Custom UA string")        // fake user agent
        .setHttpCredentials("user", "pass")      // basic auth
);
```

### Key Concept:
```
Each BrowserContext = isolated incognito session
  - Own cookies, storage, cache
  - Can run multiple contexts in parallel (parallel tests in same browser!)
  - No data shared between contexts
```

---

# 5️⃣ Page — MAIN INTERFACE (Use This 90% of the Time)

```java
Page page = context.newPage();
```

## 5a. Navigation

| Method | Return Type | Description |
|--------|-------------|-------------|
| `page.navigate(String url)` | `Response` | Go to URL (waits for load) |
| `page.navigate(url, options)` | `Response` | Navigate with timeout/waitUntil |
| `page.url()` | `String` | Get current URL |
| `page.title()` | `String` | Get page title |
| `page.content()` | `String` | Get full page HTML |
| `page.goBack()` | `Response` | Browser back button |
| `page.goForward()` | `Response` | Browser forward button |
| `page.reload()` | `Response` | Refresh page |
| `page.close()` | `void` | Close this tab/page |

### Navigate Options:
```java
page.navigate("https://example.com", new Page.NavigateOptions()
    .setTimeout(60000)
    .setWaitUntil(WaitUntilState.NETWORKIDLE)  // wait until no network activity
);
// WaitUntilState: LOAD, DOMCONTENTLOADED, NETWORKIDLE, COMMIT
```

---

## 5b. Finding Elements — Locator Methods

| Method | Return Type | Description |
|--------|-------------|-------------|
| `page.locator(String selector)` | `Locator` | CSS or XPath selector |
| `page.getByTestId(String testId)` | `Locator` | Find by `data-testid` attribute |
| `page.getByRole(AriaRole role)` | `Locator` | Find by ARIA role |
| `page.getByRole(role, options)` | `Locator` | Role + name/exact match |
| `page.getByText(String text)` | `Locator` | Find by visible text |
| `page.getByText(text, options)` | `Locator` | Text with exact match option |
| `page.getByLabel(String label)` | `Locator` | Find form field by label |
| `page.getByPlaceholder(String text)` | `Locator` | Find by placeholder text |
| `page.getByAltText(String alt)` | `Locator` | Find by alt attribute (images) |
| `page.getByTitle(String title)` | `Locator` | Find by title attribute |
| `page.frameLocator(String selector)` | `FrameLocator` | Access iframe content |

### Priority (Best → Worst):
```
1. getByTestId("login")          → Most stable
2. getByRole(BUTTON, {name})     → Accessibility-first
3. getByText("Submit")           → User-visible
4. getByLabel("Email")           → Form fields
5. getByPlaceholder("Enter...")  → Inputs
6. locator("#id")                → CSS
7. locator("xpath=//...")        → Last resort
```

---

## 5c. Direct Page Actions (without Locator)

| Method | Return Type | Description |
|--------|-------------|-------------|
| `page.click(String selector)` | `void` | Click element by selector |
| `page.dblclick(String selector)` | `void` | Double-click |
| `page.fill(String selector, String value)` | `void` | Clear + type into input |
| `page.type(String selector, String text)` | `void` | Type char by char (triggers events) |
| `page.press(String selector, String key)` | `void` | Press keyboard key on element |
| `page.check(String selector)` | `void` | Check checkbox |
| `page.uncheck(String selector)` | `void` | Uncheck checkbox |
| `page.selectOption(String sel, String val)` | `List<String>` | Select dropdown option |
| `page.hover(String selector)` | `void` | Mouse hover |
| `page.focus(String selector)` | `void` | Focus element |
| `page.setInputFiles(String sel, Path file)` | `void` | Upload file |
| `page.dragAndDrop(String src, String tgt)` | `void` | Drag and drop |

### ⚠️ Prefer Locator API over direct page actions:
```java
// ❌ Old way (still works)
page.click("#btn");

// ✅ Preferred way (better chaining, filtering, assertions)
page.locator("#btn").click();
```

---

## 5d. Reading Page State

| Method | Return Type | Description |
|--------|-------------|-------------|
| `page.textContent(String selector)` | `String` | Get text (includes hidden text) |
| `page.innerText(String selector)` | `String` | Get visible text only |
| `page.innerHTML(String selector)` | `String` | Get inner HTML |
| `page.getAttribute(String sel, String attr)` | `String` | Get attribute value |
| `page.inputValue(String selector)` | `String` | Get input field value |
| `page.isVisible(String selector)` | `boolean` | Is element visible? (no wait) |
| `page.isEnabled(String selector)` | `boolean` | Is element enabled? |
| `page.isChecked(String selector)` | `boolean` | Is checkbox checked? |
| `page.isHidden(String selector)` | `boolean` | Is element hidden? |

---

## 5e. Waiting

| Method | Return Type | Description |
|--------|-------------|-------------|
| `page.waitForSelector(String sel)` | `ElementHandle` | Wait for element in DOM |
| `page.waitForSelector(sel, options)` | `ElementHandle` | Wait with state (visible/hidden/attached) |
| `page.waitForURL(String pattern)` | `void` | Wait for URL to match |
| `page.waitForLoadState()` | `void` | Wait for page load complete |
| `page.waitForLoadState(state)` | `void` | LOAD / DOMCONTENTLOADED / NETWORKIDLE |
| `page.waitForResponse(String url)` | `Response` | Wait for specific API response |
| `page.waitForResponse(predicate)` | `Response` | Wait with custom condition |
| `page.waitForRequest(String url)` | `Request` | Wait for specific API request |
| `page.waitForTimeout(double ms)` | `void` | Hard wait (⚠️ avoid in production) |
| `page.waitForCondition(BooleanSupplier)` | `void` | Wait for custom condition |
| `page.waitForFunction(String js)` | `JSHandle` | Wait for JS expression to be truthy |
| `page.waitForPopup(Runnable)` | `Page` | Wait for new page/popup |

### Wait Options:
```java
page.waitForSelector("#loaded", new Page.WaitForSelectorOptions()
    .setState(WaitForSelectorState.VISIBLE)   // VISIBLE, HIDDEN, ATTACHED, DETACHED
    .setTimeout(10000)
);

page.waitForURL("**/dashboard", new Page.WaitForURLOptions().setTimeout(15000));

page.waitForLoadState(LoadState.NETWORKIDLE);
```

### ⚠️ Key Point: Most waits are AUTOMATIC in Playwright!
```java
page.locator("#btn").click();      // auto-waits for: visible, stable, enabled, no overlay
page.locator("#input").fill("x");  // auto-waits for: visible, enabled, editable
page.locator("#msg").textContent();// auto-waits for: attached to DOM
```

---

## 5f. Dialogs (Alerts/Confirms/Prompts)

```java
// Must register handler BEFORE the action triggers the dialog
page.onDialog(dialog -> {
    System.out.println(dialog.message());   // get alert text
    dialog.accept();                         // click OK
    // or dialog.dismiss();                  // click Cancel
    // or dialog.accept("input text");       // for prompts
});
page.click("#trigger-alert");
```

| Dialog Method | Return Type | Description |
|---------------|-------------|-------------|
| `dialog.accept()` | `void` | Click OK |
| `dialog.accept(String text)` | `void` | Enter text + OK (prompt) |
| `dialog.dismiss()` | `void` | Click Cancel |
| `dialog.message()` | `String` | Get dialog message |
| `dialog.type()` | `String` | "alert", "confirm", "prompt", "beforeunload" |
| `dialog.defaultValue()` | `String` | Default prompt input value |

### Key Difference from Selenium:
```
Selenium:  action → switchTo().alert() → accept()     (AFTER)
Playwright: page.onDialog(handler) → action            (BEFORE — event-based)
```

---

## 5g. Screenshots & Videos

| Method | Return Type | Description |
|--------|-------------|-------------|
| `page.screenshot()` | `byte[]` | Full page screenshot as bytes |
| `page.screenshot(options)` | `byte[]` | Screenshot with options |

### Options:
```java
// Full page screenshot
page.screenshot(new Page.ScreenshotOptions()
    .setPath(Paths.get("screenshot.png"))
    .setFullPage(true)                        // entire scrollable page
);

// Specific area
page.screenshot(new Page.ScreenshotOptions()
    .setClip(10, 10, 500, 300)               // x, y, width, height
);

// Element screenshot
page.locator("#chart").screenshot(new Locator.ScreenshotOptions()
    .setPath(Paths.get("chart.png"))
);

// Video recording (set at context level)
BrowserContext context = browser.newContext(new Browser.NewContextOptions()
    .setRecordVideoDir(Paths.get("videos/"))
    .setRecordVideoSize(1280, 720)
);
```

---

## 5h. JavaScript Execution

| Method | Return Type | Description |
|--------|-------------|-------------|
| `page.evaluate(String expression)` | `Object` | Execute JS, return result |
| `page.evaluate(expression, arg)` | `Object` | Execute JS with argument |
| `page.evaluateHandle(expression)` | `JSHandle` | Execute JS, return handle |

### Common Usages:
```java
// Get value
String title = (String) page.evaluate("document.title");

// Scroll
page.evaluate("window.scrollTo(0, document.body.scrollHeight)");

// Get computed style
String color = (String) page.evaluate(
    "el => getComputedStyle(el).color", page.locator("#btn")
);

// Check element property
boolean checked = (boolean) page.evaluate(
    "el => el.checked", page.locator("#checkbox")
);

// Modify DOM
page.evaluate("document.getElementById('x').remove()");

// Return complex data
Map<String, Object> rect = (Map<String, Object>) page.evaluate(
    "el => el.getBoundingClientRect().toJSON()", page.locator("#box")
);
```

---

## 5i. Network Interception (Routing)

| Method | Return Type | Description |
|--------|-------------|-------------|
| `page.route(String pattern, handler)` | `void` | Intercept matching requests |
| `page.unroute(String pattern)` | `void` | Remove route handler |
| `page.onRequest(handler)` | `void` | Listen to all requests |
| `page.onResponse(handler)` | `void` | Listen to all responses |

### Route Handler Methods:

| Method | Return Type | Description |
|--------|-------------|-------------|
| `route.fulfill(options)` | `void` | Respond with fake data (mock) |
| `route.abort()` | `void` | Block the request |
| `route.continue_()` | `void` | Let request continue normally |
| `route.request()` | `Request` | Get request details |

### Usages:
```java
// Mock API response
page.route("**/api/users", route -> {
    route.fulfill(new Route.FulfillOptions()
        .setStatus(200)
        .setContentType("application/json")
        .setBody("[{\"name\":\"John\"}]")
    );
});

// Block images (speed up tests)
page.route("**/*.{png,jpg,jpeg,gif}", Route::abort);

// Modify request headers
page.route("**/api/**", route -> {
    Map<String, String> headers = new HashMap<>(route.request().headers());
    headers.put("Authorization", "Bearer token123");
    route.continue_(new Route.ResumeOptions().setHeaders(headers));
});

// Wait for specific API response
Response response = page.waitForResponse("**/api/data", () -> {
    page.click("#load-data");
});
String body = response.text();
int status = response.status();
```

---

## 5j. Multiple Pages/Tabs

```java
// Wait for new page from action
Page newPage = context.waitForPage(() -> {
    page.click("#open-new-tab");    // action that opens new tab
});
newPage.waitForLoadState();
newPage.locator("#input").fill("text");
newPage.close();

// Get all pages
List<Page> pages = context.pages();

// Popup handling
Page popup = page.waitForPopup(() -> {
    page.click("#open-popup");
});
popup.waitForLoadState();
popup.locator("#btn").click();
```

---

## 5k. Keyboard & Mouse

### Keyboard:
```java
Keyboard keyboard = page.keyboard();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `keyboard.press(String key)` | `void` | Press + release key |
| `keyboard.type(String text)` | `void` | Type text character by character |
| `keyboard.down(String key)` | `void` | Hold key down |
| `keyboard.up(String key)` | `void` | Release held key |
| `keyboard.insertText(String text)` | `void` | Insert text (no key events) |

```java
// Press Enter
page.keyboard().press("Enter");

// Keyboard shortcut
page.keyboard().press("Control+a");    // select all
page.keyboard().press("Control+c");    // copy
page.keyboard().press("Meta+v");       // paste (Mac: Meta, Win: Control)

// Hold shift and press
page.keyboard().down("Shift");
page.keyboard().press("ArrowDown");
page.keyboard().press("ArrowDown");
page.keyboard().up("Shift");

// Type slowly
page.keyboard().type("Hello World", new Keyboard.TypeOptions().setDelay(100));
```

### Key Names:
```
Enter, Tab, Escape, Backspace, Delete, Space
ArrowUp, ArrowDown, ArrowLeft, ArrowRight
Home, End, PageUp, PageDown
Control, Shift, Alt, Meta
F1-F12
```

### Mouse:
```java
Mouse mouse = page.mouse();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `mouse.click(double x, double y)` | `void` | Click at coordinates |
| `mouse.dblclick(double x, double y)` | `void` | Double-click at coordinates |
| `mouse.move(double x, double y)` | `void` | Move to coordinates |
| `mouse.down()` | `void` | Press mouse button |
| `mouse.up()` | `void` | Release mouse button |
| `mouse.wheel(double deltaX, double deltaY)` | `void` | Scroll |

```java
// Drag manually
page.mouse().move(100, 100);
page.mouse().down();
page.mouse().move(300, 300);
page.mouse().up();

// Scroll down
page.mouse().wheel(0, 500);
```

---

## 5l. File Upload & Download

### Upload:
```java
// Simple file upload
page.locator("#file-input").setInputFiles(Paths.get("file.txt"));

// Multiple files
page.locator("#file-input").setInputFiles(new Path[] {
    Paths.get("file1.txt"),
    Paths.get("file2.txt")
});

// Clear file selection
page.locator("#file-input").setInputFiles(new Path[0]);

// Non-input file upload (file chooser dialog)
FileChooser fileChooser = page.waitForFileChooser(() -> {
    page.click("#upload-btn");
});
fileChooser.setFiles(Paths.get("file.txt"));
```

### Download:
```java
Download download = page.waitForDownload(() -> {
    page.click("#download-btn");
});
String filename = download.suggestedFilename();  // "report.pdf"
download.saveAs(Paths.get("downloads/" + filename));
Path tempPath = download.path();                 // temp file path
```

---

# 6️⃣ Locator — Element Reference (CORE API)

```java
Locator locator = page.locator("#username");
```

## 6a. Actions (All auto-wait!)

| Method | Return Type | Description |
|--------|-------------|-------------|
| `locator.click()` | `void` | Click (waits for visible + enabled + stable) |
| `locator.click(options)` | `void` | Click with options (force, position, modifiers) |
| `locator.dblclick()` | `void` | Double-click |
| `locator.fill(String value)` | `void` | Clear + type (for inputs) |
| `locator.type(String text)` | `void` | Type character by character |
| `locator.press(String key)` | `void` | Press keyboard key |
| `locator.clear()` | `void` | Clear input field |
| `locator.check()` | `void` | Check checkbox |
| `locator.uncheck()` | `void` | Uncheck checkbox |
| `locator.setChecked(boolean)` | `void` | Set checkbox state |
| `locator.selectOption(String value)` | `List<String>` | Select dropdown by value |
| `locator.selectOption(new SelectOption().setLabel("x"))` | `List<String>` | Select by label |
| `locator.hover()` | `void` | Mouse hover |
| `locator.focus()` | `void` | Focus the element |
| `locator.blur()` | `void` | Remove focus |
| `locator.tap()` | `void` | Touch tap (mobile) |
| `locator.setInputFiles(Path)` | `void` | Upload file |
| `locator.selectText()` | `void` | Select all text in input |
| `locator.scrollIntoViewIfNeeded()` | `void` | Scroll element into view |
| `locator.highlight()` | `void` | Highlight element (debugging) |
| `locator.dragTo(Locator target)` | `void` | Drag to another element |
| `locator.dispatchEvent(String type)` | `void` | Trigger DOM event |

### Click Options:
```java
locator.click(new Locator.ClickOptions()
    .setButton(MouseButton.RIGHT)          // right-click
    .setClickCount(2)                      // double-click
    .setModifiers(Arrays.asList(KeyboardModifier.CONTROL))  // Ctrl+click
    .setForce(true)                        // skip actionability checks
    .setPosition(10, 20)                   // click at offset
    .setTimeout(5000)                      // custom timeout
);
```

---

## 6b. Reading State

| Method | Return Type | Description |
|--------|-------------|-------------|
| `locator.textContent()` | `String` | Get all text (including hidden) |
| `locator.innerText()` | `String` | Get visible text only |
| `locator.innerHTML()` | `String` | Get inner HTML |
| `locator.getAttribute(String name)` | `String` | Get attribute value |
| `locator.inputValue()` | `String` | Get input/textarea value |
| `locator.isVisible()` | `boolean` | Is visible? (instant, no wait) |
| `locator.isHidden()` | `boolean` | Is hidden? |
| `locator.isEnabled()` | `boolean` | Is enabled? |
| `locator.isDisabled()` | `boolean` | Is disabled? |
| `locator.isChecked()` | `boolean` | Is checked? |
| `locator.isEditable()` | `boolean` | Is editable? |
| `locator.count()` | `int` | Number of matched elements |
| `locator.allTextContents()` | `List<String>` | Text of ALL matched elements |
| `locator.allInnerTexts()` | `List<String>` | Inner text of ALL elements |
| `locator.boundingBox()` | `BoundingBox` | Get x, y, width, height |

---

## 6c. Filtering & Chaining

| Method | Return Type | Description |
|--------|-------------|-------------|
| `locator.filter(options)` | `Locator` | Filter by text/child locator |
| `locator.locator(String sel)` | `Locator` | Find child within this element |
| `locator.nth(int index)` | `Locator` | Get nth element (0-based) |
| `locator.first()` | `Locator` | Get first element |
| `locator.last()` | `Locator` | Get last element |
| `locator.and(Locator other)` | `Locator` | Match BOTH locators |
| `locator.or(Locator other)` | `Locator` | Match EITHER locator |

### Filter Examples:
```java
// Filter by text
page.locator(".product").filter(new Locator.FilterOptions().setHasText("iPhone")).click();

// Filter by NOT having text
page.locator(".item").filter(new Locator.FilterOptions().setHasNotText("Out of stock"));

// Filter by child element
page.locator("tr").filter(new Locator.FilterOptions()
    .setHas(page.locator(".status-active"))
).click();

// Filter by NOT having child
page.locator("tr").filter(new Locator.FilterOptions()
    .setHasNot(page.locator(".disabled"))
);

// Chain: find row with "John" → click Edit button in that row
page.locator("tr")
    .filter(new Locator.FilterOptions().setHasText("John"))
    .locator("button.edit")
    .click();

// AND — element must match both
page.locator("button").and(page.getByText("Submit")).click();

// OR — match either
page.locator("#save-btn").or(page.locator("#update-btn")).click();
```

---

## 6d. Locator Screenshots

```java
page.locator("#chart").screenshot(new Locator.ScreenshotOptions()
    .setPath(Paths.get("element.png"))
);

// As bytes (for reports)
byte[] bytes = page.locator("#chart").screenshot();
```

---

## 6e. Auto-Wait Behavior

Every Locator action **automatically waits** for:

| Action | Waits for |
|--------|-----------|
| `click()` | Visible, stable, enabled, receives events, no overlay |
| `fill()` | Visible, enabled, editable |
| `check()` | Visible, enabled, unchecked (skips if already checked) |
| `textContent()` | Element attached to DOM |
| `isVisible()` | ⚠️ NO WAIT — instant check |
| `isEnabled()` | ⚠️ NO WAIT — instant check |

### Override auto-wait:
```java
// Force click (skip all checks)
locator.click(new Locator.ClickOptions().setForce(true));

// Custom timeout
locator.click(new Locator.ClickOptions().setTimeout(5000));

// Wait for specific state explicitly
locator.waitFor();                    // wait for visible (default)
locator.waitFor(new Locator.WaitForOptions().setState(WaitForSelectorState.HIDDEN));
```

---

# 7️⃣ FrameLocator — iFrame Handling

```java
FrameLocator frame = page.frameLocator("#iframe");
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `frame.locator(String selector)` | `Locator` | Find element inside frame |
| `frame.getByTestId(String)` | `Locator` | Find by test ID in frame |
| `frame.getByRole(role, options)` | `Locator` | Find by role in frame |
| `frame.getByText(String text)` | `Locator` | Find by text in frame |
| `frame.getByLabel(String label)` | `Locator` | Find by label in frame |
| `frame.frameLocator(String)` | `FrameLocator` | Access nested frame |
| `frame.first()` | `FrameLocator` | First matching frame |
| `frame.last()` | `FrameLocator` | Last matching frame |
| `frame.nth(int index)` | `FrameLocator` | Nth matching frame |

### Usage:
```java
// Simple frame
page.frameLocator("#my-iframe").locator("#btn").click();

// Frame by name
page.frameLocator("iframe[name='content']").getByText("Submit").click();

// Nested frames
page.frameLocator("#outer").frameLocator("#inner").locator("#btn").click();

// Multiple frames — access specific one
page.frameLocator("iframe").nth(2).locator("#btn").click();
```

### Key Difference from Selenium:
```
Selenium:  switchTo().frame() → interact → switchTo().defaultContent()  (manual context switch)
Playwright: frameLocator().locator().click()                             (one line, no switch)
```

---

# 8️⃣ Assertions — PlaywrightAssertions

```java
import static com.microsoft.playwright.assertions.PlaywrightAssertions.assertThat;
```

## Locator Assertions:

| Assertion | Description |
|-----------|-------------|
| `assertThat(locator).isVisible()` | Element is visible |
| `assertThat(locator).isHidden()` | Element is hidden |
| `assertThat(locator).isEnabled()` | Element is enabled |
| `assertThat(locator).isDisabled()` | Element is disabled |
| `assertThat(locator).isChecked()` | Checkbox is checked |
| `assertThat(locator).isEditable()` | Element is editable |
| `assertThat(locator).isEmpty()` | Input is empty |
| `assertThat(locator).isFocused()` | Element has focus |
| `assertThat(locator).hasText(String)` | Has exact text |
| `assertThat(locator).containsText(String)` | Contains text |
| `assertThat(locator).hasValue(String)` | Input has value |
| `assertThat(locator).hasValues(String[])` | Multi-select values |
| `assertThat(locator).hasAttribute(name, val)` | Has attribute with value |
| `assertThat(locator).hasClass(String)` | Has CSS class |
| `assertThat(locator).hasCSS(prop, val)` | Has CSS property value |
| `assertThat(locator).hasCount(int)` | N elements match |
| `assertThat(locator).hasId(String)` | Has specific ID |

## Page Assertions:

| Assertion | Description |
|-----------|-------------|
| `assertThat(page).hasURL(String)` | URL matches (supports regex) |
| `assertThat(page).hasTitle(String)` | Title matches |

## Negation:
```java
assertThat(locator).not().isVisible();
assertThat(locator).not().hasText("Error");
assertThat(page).not().hasURL("**/login");
```

## Custom Timeout:
```java
assertThat(locator).hasText("Loaded", new LocatorAssertions.HasTextOptions().setTimeout(10000));
```

### Key Point: Assertions AUTO-RETRY until timeout (default 5s)!
```java
// This will keep checking for up to 5 seconds:
assertThat(page.locator("#status")).hasText("Complete");
// If text changes to "Complete" within 5s → passes
// If still not "Complete" after 5s → fails
```

---

# 9️⃣ APIRequestContext — API Testing from Browser Context

```java
APIRequestContext request = page.request();
// OR standalone:
APIRequestContext request = playwright.request().newContext();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `request.get(String url)` | `APIResponse` | GET request |
| `request.get(url, options)` | `APIResponse` | GET with headers/params |
| `request.post(String url)` | `APIResponse` | POST request |
| `request.post(url, options)` | `APIResponse` | POST with body/headers |
| `request.put(String url)` | `APIResponse` | PUT request |
| `request.patch(String url)` | `APIResponse` | PATCH request |
| `request.delete(String url)` | `APIResponse` | DELETE request |
| `request.head(String url)` | `APIResponse` | HEAD request |
| `request.dispose()` | `void` | Clean up |

### APIResponse Methods:

| Method | Return Type | Description |
|--------|-------------|-------------|
| `response.status()` | `int` | HTTP status code |
| `response.statusText()` | `String` | Status text ("OK", "Not Found") |
| `response.text()` | `String` | Response body as string |
| `response.body()` | `byte[]` | Response body as bytes |
| `response.json()` | `Object` | Parse JSON response |
| `response.headers()` | `Map<String,String>` | Response headers |
| `response.ok()` | `boolean` | Status 200-299? |
| `response.url()` | `String` | Final URL (after redirects) |

### Usage:
```java
// GET
APIResponse response = request.get("https://api.example.com/users");
int status = response.status();
String body = response.text();

// POST with JSON body
APIResponse response = request.post("https://api.example.com/users",
    RequestOptions.create()
        .setHeader("Content-Type", "application/json")
        .setData(Map.of("name", "John", "email", "john@test.com"))
);

// With authentication (cookies from browser context!)
APIRequestContext authedRequest = page.request();
APIResponse response = authedRequest.get("/api/profile");  // uses page's cookies!
```

---

# 🔟 Tracing — Debug Test Failures

```java
// Start tracing
context.tracing().start(new Tracing.StartOptions()
    .setScreenshots(true)
    .setSnapshots(true)
    .setSources(true)
);

// ... run test ...

// Stop and save trace
context.tracing().stop(new Tracing.StopOptions()
    .setPath(Paths.get("trace.zip"))
);

// View trace: npx playwright show-trace trace.zip
```

---

# 1️⃣1️⃣ Complete Flow Diagrams

## Flow 1: Basic Test

```
Playwright.create()
    → playwright.chromium().launch()
    → browser.newPage()
    → page.navigate("https://url.com")
    → page.getByLabel("Username").fill("admin")
    → page.getByLabel("Password").fill("pass123")
    → page.getByRole(BUTTON, {name: "Login"}).click()
    → assertThat(page).hasURL("**/dashboard")
    → page.close()
    → browser.close()
    → playwright.close()
```

## Flow 2: Locator → Filter → Act

```
page.locator("tr")                              → all rows
    .filter(hasText("John"))                    → rows containing "John"
    .locator("button.edit")                     → Edit button in that row
    .click()                                    → click it
```

## Flow 3: Frame Interaction

```
page.frameLocator("#iframe")                    → enter iframe
    .locator("#username")                       → find element
    .fill("admin")                              → type into it
// No switchTo/defaultContent needed!
```

## Flow 4: Dialog (Alert) Handling

```
page.onDialog(dialog -> dialog.accept())        → register handler FIRST
    → page.click("#trigger-alert")              → then trigger the alert
```

## Flow 5: New Tab/Window

```
Page newPage = context.waitForPage(() -> {
    page.click("#open-new-tab");                → action opens new tab
});
newPage.waitForLoadState();                     → wait for load
newPage.locator("#btn").click();                → interact
newPage.close();                                → close
```

## Flow 6: File Upload

```
page.locator("#file-input").setInputFiles(Paths.get("file.txt"))  → simple upload
// OR for non-input:
FileChooser fc = page.waitForFileChooser(() -> page.click("#upload-btn"));
fc.setFiles(Paths.get("file.txt"));
```

## Flow 7: File Download

```
Download dl = page.waitForDownload(() -> page.click("#download-btn"));
dl.saveAs(Paths.get("downloads/" + dl.suggestedFilename()));
```

## Flow 8: Mock API

```
page.route("**/api/users", route -> {
    route.fulfill(status: 200, body: "[{...}]")
});
page.navigate(url);                             → app calls mocked API
```

## Flow 9: Network Wait

```
Response apiResponse = page.waitForResponse("**/api/data", () -> {
    page.click("#load-data");                   → triggers API call
});
int status = apiResponse.status();
String body = apiResponse.text();
```

## Flow 10: Auth State Reuse (Login Once)

```
// Save auth state after login:
context.storageState(new BrowserContext.StorageStateOptions()
    .setPath(Paths.get("auth.json")));

// Reuse in other tests:
BrowserContext context = browser.newContext(new Browser.NewContextOptions()
    .setStorageStatePath(Paths.get("auth.json"))
);
// Already logged in! No need to login again.
```

---

# 1️⃣2️⃣ Method Chaining Quick Reference

```java
// ===== SETUP =====
Playwright playwright = Playwright.create();
Browser browser = playwright.chromium().launch();                     // Browser
BrowserContext context = browser.newContext();                        // BrowserContext
Page page = context.newPage();                                       // Page

// ===== NAVIGATION =====
page.navigate(url);                        // Response
page.url();                                // String
page.title();                              // String
page.content();                            // String
page.goBack();                             // Response
page.goForward();                          // Response
page.reload();                             // Response

// ===== FINDING ELEMENTS =====
page.locator("#css");                      // Locator
page.locator("xpath=//div");               // Locator
page.getByTestId("login");                 // Locator
page.getByRole(AriaRole.BUTTON);           // Locator
page.getByText("Submit");                  // Locator
page.getByLabel("Email");                  // Locator
page.getByPlaceholder("Enter...");         // Locator
page.getByAltText("Logo");                 // Locator
page.getByTitle("Close");                  // Locator
page.frameLocator("#iframe");              // FrameLocator

// ===== LOCATOR ACTIONS =====
locator.click();                           // void
locator.dblclick();                        // void
locator.fill("text");                      // void
locator.type("text");                      // void
locator.press("Enter");                    // void
locator.clear();                           // void
locator.check();                           // void
locator.uncheck();                         // void
locator.selectOption("value");             // List<String>
locator.hover();                           // void
locator.focus();                           // void
locator.blur();                            // void
locator.dragTo(targetLocator);             // void
locator.setInputFiles(path);               // void
locator.scrollIntoViewIfNeeded();          // void
locator.highlight();                       // void

// ===== LOCATOR READING =====
locator.textContent();                     // String
locator.innerText();                       // String
locator.innerHTML();                       // String
locator.getAttribute("href");              // String
locator.inputValue();                      // String
locator.isVisible();                       // boolean
locator.isHidden();                        // boolean
locator.isEnabled();                       // boolean
locator.isDisabled();                      // boolean
locator.isChecked();                       // boolean
locator.isEditable();                      // boolean
locator.count();                           // int
locator.allTextContents();                 // List<String>
locator.allInnerTexts();                   // List<String>
locator.boundingBox();                     // BoundingBox

// ===== LOCATOR FILTERING =====
locator.filter(hasText("x"));             // Locator
locator.filter(has(childLocator));         // Locator
locator.locator(".child");                 // Locator
locator.nth(0);                            // Locator
locator.first();                           // Locator
locator.last();                            // Locator
locator.and(otherLocator);                 // Locator
locator.or(otherLocator);                  // Locator

// ===== WAITING =====
page.waitForSelector("#el");               // ElementHandle
page.waitForURL("**/dashboard");           // void
page.waitForLoadState();                   // void
page.waitForResponse("**/api");            // Response
page.waitForRequest("**/api");             // Request
page.waitForTimeout(1000);                 // void
page.waitForCondition(booleanSupplier);    // void
page.waitForFunction("js expression");     // JSHandle
locator.waitFor();                         // void

// ===== KEYBOARD & MOUSE =====
page.keyboard().press("Enter");            // void
page.keyboard().type("text");              // void
page.keyboard().down("Shift");             // void
page.keyboard().up("Shift");               // void
page.mouse().click(x, y);                  // void
page.mouse().move(x, y);                   // void
page.mouse().wheel(0, 500);                // void

// ===== SCREENSHOTS =====
page.screenshot();                         // byte[]
page.screenshot(options);                  // byte[]
locator.screenshot();                      // byte[]

// ===== JAVASCRIPT =====
page.evaluate("document.title");           // Object
page.evaluate("el => el.value", locator);  // Object

// ===== NETWORK =====
page.route("**/api/**", handler);          // void
page.unroute("**/api/**");                 // void
page.request().get(url);                   // APIResponse
page.request().post(url, options);         // APIResponse

// ===== CLEANUP =====
page.close();                              // void
context.close();                           // void
browser.close();                           // void
playwright.close();                        // void
```

---

# 1️⃣3️⃣ Selenium vs Playwright — Same Task Comparison

| Task | Selenium | Playwright |
|------|----------|------------|
| Setup | `new ChromeDriver()` | `playwright.chromium().launch().newPage()` |
| Navigate | `driver.get(url)` | `page.navigate(url)` |
| Find by ID | `driver.findElement(By.id("x"))` | `page.locator("#x")` |
| Find by text | `By.xpath("//btn[text()='x']")` | `page.getByText("x")` |
| Find by role | ❌ Not available | `page.getByRole(BUTTON, {name})` |
| Click | `element.click()` | `locator.click()` |
| Type | `element.clear(); element.sendKeys("x")` | `locator.fill("x")` |
| Get text | `element.getText()` | `locator.textContent()` |
| Get URL | `driver.getCurrentUrl()` | `page.url()` |
| Is visible | `element.isDisplayed()` | `locator.isVisible()` |
| Wait visible | `wait.until(visibilityOf(el))` | Automatic! |
| Dropdown | `new Select(el).selectByVisibleText("x")` | `locator.selectOption("x")` |
| Frame | `switchTo().frame() → act → defaultContent()` | `frameLocator().locator().act()` |
| Alert | `switchTo().alert().accept()` | `page.onDialog(d -> d.accept())` |
| New tab | Manual handle switching | `context.waitForPage(() -> click)` |
| Hover | `new Actions(driver).moveToElement(el).perform()` | `locator.hover()` |
| Drag | `new Actions(driver).dragAndDrop(s,t).perform()` | `locator.dragTo(target)` |
| Upload | `element.sendKeys("path")` | `locator.setInputFiles(path)` |
| Download | Complex (no built-in) | `page.waitForDownload(() -> click)` |
| Screenshot | `((TakesScreenshot)driver).getScreenshotAs(...)` | `page.screenshot()` |
| JS execute | `((JavascriptExecutor)driver).executeScript(...)` | `page.evaluate(...)` |
| Mock API | ❌ Not possible | `page.route(pattern, handler)` |
| Shadow DOM | Manual `.getShadowRoot()` | Automatic (CSS pierces) |
| Stale element | ❌ StaleElementException | ✅ Never happens |
| Parallel | Needs Grid/TestNG threads | BrowserContext isolation |
| Auth reuse | ❌ Manual cookies | `storageState()` save/load |

---

*Complete Playwright hierarchy with every method, return type, and flow. Bookmark this for daily use.*
*Last updated: April 30, 2026*

