# 🎯 Locators Complete Guide — Selenium & Playwright
### Hierarchy, Usage, Best Practices & Interview Questions

---

# SECTION 1: LOCATOR HIERARCHY & PRIORITY

## Selenium Locator Priority (Best → Worst)

```
1. By.id("username")              → Fastest, unique (BEST)
2. By.name("email")               → Fast, usually unique
3. By.cssSelector("#login .btn")  → Fast, flexible, powerful
4. By.xpath("//div[@id='x']")     → Slowest but most powerful
5. By.className("btn-primary")    → Simple but often not unique
6. By.tagName("input")            → Rarely unique
7. By.linkText("Click Here")      → Only for <a> tags
8. By.partialLinkText("Click")    → Partial match for <a> tags
```

## Playwright Locator Priority (Best → Worst)

```
1. page.getByTestId("login-btn")       → Best practice, stable
2. page.getByRole("button", {name})    → Accessibility-first
3. page.getByText("Submit")            → User-visible text
4. page.getByLabel("Username")         → Form fields
5. page.getByPlaceholder("Enter name") → Input placeholders
6. page.locator("#id")                 → CSS selector
7. page.locator("xpath=//div")         → XPath (last resort)
```

---

# SECTION 2: SELENIUM LOCATORS — COMPLETE REFERENCE

## 2.1 By.id

```java
// HTML: <input id="username" type="text">
WebElement element = driver.findElement(By.id("username"));
element.sendKeys("admin");
```

**When to use:** Always first choice if `id` attribute exists and is unique.
**Limitation:** Dynamic IDs like `id="field_38291"` change every load — avoid these.

---

## 2.2 By.name

```java
// HTML: <input name="email" type="email">
WebElement element = driver.findElement(By.name("email"));
```

**When to use:** When `id` is not available but `name` attribute is unique.

---

## 2.3 By.cssSelector (MOST IMPORTANT)

```java
// By ID
driver.findElement(By.cssSelector("#username"));

// By class
driver.findElement(By.cssSelector(".btn-primary"));

// By attribute
driver.findElement(By.cssSelector("[data-testid='login']"));
driver.findElement(By.cssSelector("input[type='email']"));
driver.findElement(By.cssSelector("[placeholder='Enter name']"));

// By tag + class
driver.findElement(By.cssSelector("button.submit-btn"));

// By tag + attribute
driver.findElement(By.cssSelector("input[name='password']"));

// Child combinator (direct child)
driver.findElement(By.cssSelector("div > span"));

// Descendant (any level deep)
driver.findElement(By.cssSelector("form input"));

// Adjacent sibling
driver.findElement(By.cssSelector("label + input"));

// Nth-child
driver.findElement(By.cssSelector("ul li:nth-child(3)"));
driver.findElement(By.cssSelector("tr:first-child"));
driver.findElement(By.cssSelector("tr:last-child"));

// Multiple classes
driver.findElement(By.cssSelector(".btn.btn-primary.large"));

// Contains in attribute
driver.findElement(By.cssSelector("[id*='partial']"));      // contains
driver.findElement(By.cssSelector("[id^='starts']"));       // starts with
driver.findElement(By.cssSelector("[id$='ends']"));         // ends with

// Multiple attributes
driver.findElement(By.cssSelector("input[type='text'][name='user']"));

// NOT selector
driver.findElement(By.cssSelector("input:not([disabled])"));
```

### CSS Selector Cheat Sheet

| Pattern | Meaning | Example |
|---------|---------|---------|
| `#id` | ID selector | `#username` |
| `.class` | Class selector | `.btn-primary` |
| `tag` | Tag selector | `input` |
| `[attr='val']` | Attribute equals | `[type='email']` |
| `[attr*='val']` | Attribute contains | `[id*='login']` |
| `[attr^='val']` | Attribute starts with | `[class^='btn']` |
| `[attr$='val']` | Attribute ends with | `[id$='_field']` |
| `parent > child` | Direct child | `div > span` |
| `ancestor descendant` | Any descendant | `form input` |
| `el + sibling` | Adjacent sibling | `h2 + p` |
| `el ~ sibling` | General sibling | `h2 ~ p` |
| `:nth-child(n)` | Nth child | `li:nth-child(2)` |
| `:first-child` | First child | `li:first-child` |
| `:last-child` | Last child | `li:last-child` |
| `:not(selector)` | Negation | `input:not([disabled])` |

---

## 2.4 By.xpath (MOST POWERFUL)

```java
// Absolute XPath (NEVER use — fragile)
driver.findElement(By.xpath("/html/body/div[1]/form/input[2]"));

// Relative XPath (always use //)
driver.findElement(By.xpath("//input[@id='username']"));

// By attribute
driver.findElement(By.xpath("//input[@type='email']"));
driver.findElement(By.xpath("//button[@class='submit']"));
driver.findElement(By.xpath("//*[@data-testid='login']"));

// By text (CSS CAN'T DO THIS)
driver.findElement(By.xpath("//button[text()='Submit']"));
driver.findElement(By.xpath("//span[text()='Welcome']"));

// Contains text (partial match)
driver.findElement(By.xpath("//button[contains(text(),'Sub')]"));
driver.findElement(By.xpath("//*[contains(text(),'Welcome')]"));

// Contains attribute
driver.findElement(By.xpath("//input[contains(@id,'user')]"));
driver.findElement(By.xpath("//div[contains(@class,'active')]"));

// Starts-with
driver.findElement(By.xpath("//input[starts-with(@id,'field_')]"));

// AND / OR conditions
driver.findElement(By.xpath("//input[@type='text' and @name='user']"));
driver.findElement(By.xpath("//input[@type='text' or @type='email']"));

// Parent traversal (CSS CAN'T DO THIS)
driver.findElement(By.xpath("//span[text()='Email']/parent::div"));
driver.findElement(By.xpath("//input[@id='email']/.."));              // shorthand for parent

// Ancestor (CSS CAN'T DO THIS)
driver.findElement(By.xpath("//input[@id='email']/ancestor::form"));

// Following-sibling
driver.findElement(By.xpath("//label[text()='Email']/following-sibling::input"));

// Preceding-sibling
driver.findElement(By.xpath("//input[@id='pass']/preceding-sibling::label"));

// Child
driver.findElement(By.xpath("//div[@class='container']/child::span"));

// Index (position)
driver.findElement(By.xpath("(//button[@class='btn'])[1]"));    // first
driver.findElement(By.xpath("(//button[@class='btn'])[last()]"));// last
driver.findElement(By.xpath("(//input)[3]"));                    // third

// Normalize-space (handles extra whitespace)
driver.findElement(By.xpath("//button[normalize-space()='Submit']"));

// Multiple attributes combined
driver.findElement(By.xpath("//input[@type='text'][@placeholder='Name']"));
```

### XPath Axes — Visual

```
                    ancestor
                       ↑
  preceding-sibling ← SELF → following-sibling
                       ↓
                   descendant
                       ↓
                     child
```

### XPath Cheat Sheet

| Pattern | Meaning | Example |
|---------|---------|---------|
| `//tag` | Find anywhere | `//input` |
| `//tag[@attr='val']` | By attribute | `//input[@id='x']` |
| `//tag[text()='x']` | Exact text | `//button[text()='OK']` |
| `contains(@attr,'x')` | Attribute contains | `//div[contains(@class,'act')]` |
| `contains(text(),'x')` | Text contains | `//*[contains(text(),'Hel')]` |
| `starts-with(@attr,'x')` | Starts with | `//input[starts-with(@id,'f')]` |
| `/parent::tag` | Go to parent | `//span/parent::div` |
| `/ancestor::tag` | Go to ancestor | `//input/ancestor::form` |
| `/following-sibling::tag` | Next sibling | `//label/following-sibling::input` |
| `/preceding-sibling::tag` | Previous sibling | `//input/preceding-sibling::label` |
| `[1]`, `[last()]` | By position | `(//btn)[1]`, `(//btn)[last()]` |
| `and`, `or` | Conditions | `[@type='x' and @name='y']` |
| `not()` | Negation | `//input[not(@disabled)]` |
| `normalize-space()` | Trim whitespace | `[normalize-space()='text']` |

---

## 2.5 By.className

```java
// HTML: <button class="btn btn-primary large">
driver.findElement(By.className("btn-primary"));  // single class only!

// ❌ WRONG — can't use multiple classes
driver.findElement(By.className("btn btn-primary")); // ERROR!

// ✅ Use cssSelector for multiple classes
driver.findElement(By.cssSelector(".btn.btn-primary"));
```

---

## 2.6 By.tagName

```java
// Find all links on a page
List<WebElement> links = driver.findElements(By.tagName("a"));
System.out.println("Total links: " + links.size());

// Find all rows in a table
List<WebElement> rows = driver.findElements(By.tagName("tr"));
```

---

## 2.7 By.linkText & By.partialLinkText

```java
// HTML: <a href="/about">About Us</a>
driver.findElement(By.linkText("About Us"));          // exact match
driver.findElement(By.partialLinkText("About"));      // partial match

// Only works for <a> tags!
```

---

## 2.8 Finding Multiple Elements

```java
List<WebElement> items = driver.findElements(By.cssSelector(".product-item"));
items.size();                        // count
items.get(0).click();                // first item
items.get(items.size()-1).click();   // last item

for (WebElement item : items) {
    System.out.println(item.getText());
}

// Check if element exists (no exception)
boolean exists = driver.findElements(By.id("popup")).size() > 0;
```

---

## 2.9 Relative Locators (Selenium 4+)

```java
import static org.openqa.selenium.support.locators.RelativeLocator.with;

// Find element near another element
WebElement passwordField = driver.findElement(
    with(By.tagName("input")).below(By.id("username"))
);

WebElement submitBtn = driver.findElement(
    with(By.tagName("button")).below(By.id("password"))
);

WebElement cancelBtn = driver.findElement(
    with(By.tagName("button")).toLeftOf(By.id("submit"))
);

// Available methods: above(), below(), toLeftOf(), toRightOf(), near()
```

---

## 2.10 Shadow DOM (Selenium)

```java
// Get shadow root
WebElement shadowHost = driver.findElement(By.cssSelector("#shadow-host"));
SearchContext shadowRoot = shadowHost.getShadowRoot();

// Find inside shadow DOM
WebElement innerElement = shadowRoot.findElement(By.cssSelector(".inner-class"));
innerElement.click();

// Nested shadow DOM
SearchContext outer = driver.findElement(By.id("outer")).getShadowRoot();
SearchContext inner = outer.findElement(By.id("inner")).getShadowRoot();
WebElement target = inner.findElement(By.cssSelector(".target"));
```

---

# SECTION 3: PLAYWRIGHT LOCATORS — COMPLETE REFERENCE

## 3.1 Built-in Locators (RECOMMENDED — Resilient)

```java
// By test ID (BEST practice — add data-testid in HTML)
page.getByTestId("login-button").click();
// HTML: <button data-testid="login-button">Login</button>

// By role (accessibility-first)
page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Submit")).click();
page.getByRole(AriaRole.HEADING, new Page.GetByRoleOptions().setName("Welcome")).isVisible();
page.getByRole(AriaRole.LINK, new Page.GetByRoleOptions().setName("Home")).click();
page.getByRole(AriaRole.TEXTBOX, new Page.GetByRoleOptions().setName("Email")).fill("a@b.com");
page.getByRole(AriaRole.CHECKBOX, new Page.GetByRoleOptions().setName("Agree")).check();

// By text
page.getByText("Welcome back").isVisible();
page.getByText("Submit", new Page.GetByTextOptions().setExact(true)).click();

// By label (form fields)
page.getByLabel("Username").fill("admin");
page.getByLabel("Password").fill("pass123");

// By placeholder
page.getByPlaceholder("Enter your email").fill("test@test.com");

// By alt text (images)
page.getByAltText("Company Logo").isVisible();

// By title attribute
page.getByTitle("Close dialog").click();
```

---

## 3.2 CSS & XPath Locators

```java
// CSS Selector
page.locator("#username").fill("admin");
page.locator(".btn-primary").click();
page.locator("[data-testid='login']").click();
page.locator("input[type='email']").fill("a@b.com");
page.locator("div.container > span.title").textContent();

// XPath
page.locator("xpath=//button[text()='Submit']").click();
page.locator("xpath=//input[@placeholder='Search']").fill("query");
page.locator("xpath=//span[contains(text(),'Welcome')]").isVisible();
```

---

## 3.3 Filtering Locators

```java
// Filter by text
page.locator(".product-card").filter(new Locator.FilterOptions().setHasText("iPhone")).click();

// Filter by NOT having text
page.locator(".item").filter(new Locator.FilterOptions().setHasNotText("Out of stock"));

// Filter by child locator
page.locator(".row").filter(new Locator.FilterOptions().setHas(page.locator(".status-active")));

// Chain filters
page.locator("tr")
    .filter(new Locator.FilterOptions().setHasText("John"))
    .locator("button.edit")
    .click();
```

---

## 3.4 Locator Chaining & Traversal

```java
// Parent → Child (chaining)
page.locator(".form-group").locator("input").fill("text");

// Nth element
page.locator(".item").nth(0).click();       // first (0-indexed)
page.locator(".item").nth(2).click();       // third
page.locator(".item").first().click();      // first
page.locator(".item").last().click();       // last

// Count
int count = page.locator(".item").count();

// All elements
List<String> texts = page.locator(".item").allTextContents();
List<String> innerTexts = page.locator(".item").allInnerTexts();

// Visible only
page.locator(".btn").locator("visible=true").click();
```

---

## 3.5 Shadow DOM (Playwright)

```java
// Playwright pierces shadow DOM AUTOMATICALLY with CSS!
page.locator("#shadow-host .inner-element").click();  // just works!

// For complex cases
page.locator("#shadow-host").locator(".deep-element").fill("text");
```

**Key difference:** Selenium requires manual `.getShadowRoot()`. Playwright pierces automatically.

---

## 3.6 Frame Locators

```java
// Selenium way (switch context)
driver.switchTo().frame("frameName");
driver.findElement(By.id("btn")).click();
driver.switchTo().defaultContent();

// Playwright way (no context switch!)
page.frameLocator("#iframe").locator("#btn").click();
page.frameLocator("iframe[name='content']").getByRole(AriaRole.BUTTON, 
    new FrameLocator.GetByRoleOptions().setName("Submit")).click();

// Nested frames
page.frameLocator("#outer").frameLocator("#inner").locator("#btn").click();
```

---

## 3.7 Locator Assertions (Playwright)

```java
import static com.microsoft.playwright.assertions.PlaywrightAssertions.assertThat;

assertThat(page.locator("#msg")).isVisible();
assertThat(page.locator("#msg")).isHidden();
assertThat(page.locator("#msg")).hasText("Success");
assertThat(page.locator("#msg")).containsText("Suc");
assertThat(page.locator("#input")).hasValue("admin");
assertThat(page.locator("#btn")).isEnabled();
assertThat(page.locator("#btn")).isDisabled();
assertThat(page.locator("#check")).isChecked();
assertThat(page.locator(".item")).hasCount(5);
assertThat(page.locator("#link")).hasAttribute("href", "/home");
assertThat(page.locator("div")).hasClass("active");
```

---

# SECTION 4: COMPARISON TABLE

| Feature | Selenium | Playwright |
|---------|----------|------------|
| Fastest locator | `By.id` | `getByTestId` |
| Recommended | `By.cssSelector` | `getByRole` / `getByTestId` |
| Text-based | XPath only: `//btn[text()='x']` | `getByText("x")` |
| Parent traversal | XPath: `/parent::div` | Filter with `.locator("..")` |
| Shadow DOM | Manual: `.getShadowRoot()` | Automatic pierce |
| Frames | `switchTo().frame()` + `defaultContent()` | `frameLocator()` — no switch |
| Multiple elements | `findElements()` → List | `.count()`, `.nth()`, `.all()` |
| Stale element | ❌ StaleElementException | ✅ Never (auto re-query) |
| Auto-wait | ❌ Need explicit wait | ✅ Built-in |
| Relative locators | Selenium 4: `above()`, `below()` | Chaining + filter |

---

# SECTION 5: BEST PRACTICES

## Locator Strategy Priority

```
1. ID / data-testid     → Most stable, fastest
2. Name / Label         → Descriptive, stable
3. CSS Selector         → Flexible, fast
4. XPath               → Only when CSS can't do it (text, parent, axes)
5. LinkText            → Only for <a> tags
6. ClassName / TagName  → Avoid (not unique)
```

## When to Use XPath Over CSS

| Use XPath When | Example |
|----------------|---------|
| Find by text content | `//button[text()='Submit']` |
| Traverse to parent | `//input/parent::div` |
| Traverse to ancestor | `//span/ancestor::form` |
| Find preceding sibling | `//input/preceding-sibling::label` |
| Complex conditions | `//div[@class='x' and contains(text(),'y')]` |
| Position-based | `(//button)[last()]` |

## When to Use CSS Over XPath

| Use CSS When | Example |
|--------------|---------|
| Simple attributes | `[data-testid='login']` |
| Class combinations | `.btn.primary.large` |
| Pseudo-selectors | `li:nth-child(3)`, `:first-child` |
| Speed matters | CSS is ~10-15% faster than XPath |
| Starts/ends with | `[id^='start']`, `[id$='end']` |

## Golden Rules

1. **Never use absolute XPath** — breaks with any DOM change
2. **Prefer attributes that don't change** — `data-testid`, `name`, `aria-label`
3. **Avoid index-based locators** — fragile when DOM order changes
4. **Keep locators short** — long chains = fragile
5. **Don't use auto-generated IDs** — `id="ember392"` changes every load
6. **Test your locator** — verify it returns exactly 1 element
7. **Use Page Object Model** — centralize locators, change in one place

---

# SECTION 6: REAL-WORLD LOCATOR PATTERNS

## Dynamic Tables

```java
// Selenium — find row by text, then click action button
String xpath = "//td[text()='John']/following-sibling::td/button[@class='edit']";
driver.findElement(By.xpath(xpath)).click();

// Playwright
page.locator("tr").filter(new Locator.FilterOptions().setHasText("John"))
    .locator("button.edit").click();
```

## Dropdowns (non-Select)

```java
// Selenium — custom dropdown
driver.findElement(By.cssSelector(".dropdown-trigger")).click();
driver.findElement(By.xpath("//li[text()='Option 2']")).click();

// Playwright
page.locator(".dropdown-trigger").click();
page.getByText("Option 2").click();
```

## Calendar/Date Picker

```java
// Selenium
driver.findElement(By.cssSelector(".calendar-trigger")).click();
driver.findElement(By.xpath("//td[@data-date='2026-04-29']")).click();

// Playwright
page.locator(".calendar-trigger").click();
page.locator("[data-date='2026-04-29']").click();
```

## Auto-complete / Search Suggestions

```java
// Selenium
driver.findElement(By.id("search")).sendKeys("java");
wait.until(ExpectedConditions.visibilityOfElementLocated(By.cssSelector(".suggestions")));
driver.findElement(By.xpath("//li[contains(text(),'JavaScript')]")).click();

// Playwright
page.fill("#search", "java");
page.locator(".suggestions").waitFor();
page.getByText("JavaScript").click();
```

## Tooltip / Hover Element

```java
// Selenium
Actions actions = new Actions(driver);
actions.moveToElement(driver.findElement(By.id("info-icon"))).perform();
String tooltip = driver.findElement(By.cssSelector(".tooltip-text")).getText();

// Playwright
page.hover("#info-icon");
String tooltip = page.locator(".tooltip-text").textContent();
```

## Dynamic ID Handling

```java
// HTML: <input id="field_83921_name"> — ID changes every load

// Selenium — use partial match
driver.findElement(By.cssSelector("[id*='_name']"));           // contains
driver.findElement(By.xpath("//input[contains(@id,'_name')]"));

// Playwright
page.locator("[id*='_name']").fill("text");
```

---

# SECTION 7: INTERVIEW QUESTIONS & ANSWERS (30 Questions)

---

## Q1: What locator strategies are available in Selenium?

> 8 strategies: `By.id`, `By.name`, `By.cssSelector`, `By.xpath`, `By.className`, `By.tagName`, `By.linkText`, `By.partialLinkText`.

---

## Q2: Which locator is fastest and why?

> `By.id` is fastest because browsers maintain an internal ID map (hash table) for O(1) lookup. CSS is next fastest. XPath is slowest because it traverses the DOM tree.

---

## Q3: Difference between CSS Selector and XPath?

| Feature | CSS Selector | XPath |
|---------|-------------|-------|
| Speed | Faster (~10-15%) | Slower |
| Direction | Top-down only | Bi-directional (parent, ancestor) |
| Text match | ❌ Can't match text | ✅ `text()`, `contains(text())` |
| Readability | Cleaner syntax | More verbose |
| Browser support | Native in all browsers | Inconsistent in IE |
| Pseudo-selectors | `:nth-child`, `:first-child` | `position()` |

---

## Q4: How do you find an element by text in Selenium?

> XPath only: `By.xpath("//button[text()='Submit']")` or `By.xpath("//*[contains(text(),'Submit')]")`
> CSS cannot match by text content.

---

## Q5: What is the difference between `findElement()` and `findElements()`?

| `findElement()` | `findElements()` |
|-----------------|-------------------|
| Returns single `WebElement` | Returns `List<WebElement>` |
| Throws `NoSuchElementException` if not found | Returns empty list if not found |
| Returns first match | Returns all matches |

---

## Q6: How do you handle dynamic elements?

> 1. Use `contains()`: `//div[contains(@id,'partial')]`
> 2. Use `starts-with()`: `//input[starts-with(@id,'field_')]`
> 3. Use CSS partial match: `[id*='partial']`, `[id^='start']`, `[id$='end']`
> 4. Use stable parent + child relationship
> 5. Use explicit waits for elements that load dynamically

---

## Q7: What is the difference between absolute and relative XPath?

| Absolute XPath | Relative XPath |
|----------------|----------------|
| Starts with `/` | Starts with `//` |
| From root: `/html/body/div/input` | From anywhere: `//input[@id='x']` |
| Fragile — breaks with any DOM change | Resilient — finds anywhere in DOM |
| ❌ NEVER use in automation | ✅ Always use |

---

## Q8: How do you traverse to a parent element?

> **XPath:** `//input[@id='email']/parent::div` or `//input[@id='email']/..`
> **XPath ancestor:** `//input[@id='email']/ancestor::form`
> **CSS:** Cannot traverse to parent (only goes downward)
> **Playwright:** `page.locator("#email").locator("..")` 

---

## Q9: What are XPath axes?

> Axes define directions to traverse from the current node:
> - `parent::` — direct parent
> - `ancestor::` — all ancestors up to root
> - `child::` — direct children
> - `descendant::` — all descendants
> - `following-sibling::` — siblings after current
> - `preceding-sibling::` — siblings before current
> - `following::` — everything after in document order
> - `preceding::` — everything before in document order
> - `self::` — current node

---

## Q10: How do you find the nth element?

```java
// Selenium — XPath
driver.findElement(By.xpath("(//button[@class='btn'])[3]"));    // 3rd button (1-indexed)
driver.findElement(By.xpath("(//button[@class='btn'])[last()]"));// last button

// Selenium — CSS
driver.findElement(By.cssSelector(".btn:nth-child(3)"));
driver.findElement(By.cssSelector(".btn:last-child"));

// Selenium — findElements
driver.findElements(By.cssSelector(".btn")).get(2);  // 3rd (0-indexed)

// Playwright
page.locator(".btn").nth(2);     // 3rd (0-indexed)
page.locator(".btn").first();
page.locator(".btn").last();
```

---

## Q11: How do you handle Shadow DOM?

> **Selenium:** Manual navigation with `getShadowRoot()`
> ```java
> SearchContext shadow = driver.findElement(By.id("host")).getShadowRoot();
> shadow.findElement(By.cssSelector(".inner")).click();
> ```
>
> **Playwright:** Automatic — CSS locators pierce shadow DOM by default.
> ```java
> page.locator("#host .inner").click();  // just works!
> ```

---

## Q12: What is StaleElementReferenceException? How to handle?

> Occurs when the DOM changes after you found the element (page refresh, AJAX update, navigation).
>
> **Solutions:**
> 1. Re-find the element before interacting
> 2. Use explicit wait: `wait.until(ExpectedConditions.refreshed(visibilityOf(element)))`
> 3. Use try-catch with retry logic
> 4. **Playwright doesn't have this problem** — Locators auto-re-query every time

---

## Q13: What are Relative Locators in Selenium 4?

> Find elements based on visual position relative to another element:
> ```java
> with(By.tagName("input")).below(By.id("email"))
> with(By.tagName("input")).above(By.id("password"))
> with(By.tagName("button")).toRightOf(By.id("cancel"))
> with(By.tagName("button")).toLeftOf(By.id("submit"))
> with(By.tagName("input")).near(By.id("label"))
> ```

---

## Q14: How do you write XPath for a table to get specific cell data?

```java
// Get value from row 3, column 2
driver.findElement(By.xpath("//table[@id='data']//tr[3]/td[2]")).getText();

// Get row by cell text
driver.findElement(By.xpath("//td[text()='John']/parent::tr/td[3]")).getText();

// Get all values in a column
driver.findElements(By.xpath("//table[@id='data']//tr/td[2]"));
```

---

## Q15: How do you locate elements inside an iframe?

> **Selenium:**
> ```java
> driver.switchTo().frame("frameName");           // switch into
> driver.findElement(By.id("btn")).click();       // interact
> driver.switchTo().defaultContent();             // switch back
> ```
>
> **Playwright:**
> ```java
> page.frameLocator("#iframe").locator("#btn").click();  // no switch needed
> ```

---

## Q16: How do you validate if an element exists without throwing exception?

```java
// Selenium
boolean exists = driver.findElements(By.id("popup")).size() > 0;

// Selenium with try-catch
try {
    driver.findElement(By.id("popup"));
    return true;
} catch (NoSuchElementException e) {
    return false;
}

// Playwright
boolean visible = page.locator("#popup").isVisible();
int count = page.locator("#popup").count();
```

---

## Q17: What is the difference between `isDisplayed()`, `isEnabled()`, `isSelected()`?

| Method | Checks | Used for |
|--------|--------|----------|
| `isDisplayed()` | Element visible on page | Any element |
| `isEnabled()` | Element not disabled | Buttons, inputs |
| `isSelected()` | Element selected/checked | Checkboxes, radio buttons, options |

---

## Q18: How do you handle elements that are present but not visible?

```java
// Wait for visibility
wait.until(ExpectedConditions.visibilityOf(element));

// Use JavaScript to check
Boolean isVisible = (Boolean) ((JavascriptExecutor) driver)
    .executeScript("return arguments[0].offsetParent !== null;", element);

// Scroll into view first
((JavascriptExecutor) driver).executeScript("arguments[0].scrollIntoView(true);", element);
element.click();

// Playwright — auto-scrolls before interacting
page.locator("#hidden-btn").click();  // scrolls + waits + clicks
```

---

## Q19: What is `getByRole()` in Playwright and why is it preferred?

> `getByRole()` finds elements by their ARIA role — how assistive technologies see the page.
> 
> **Why preferred:**
> 1. Tests verify accessibility alongside functionality
> 2. Resilient — doesn't depend on CSS classes or IDs that change
> 3. Matches user intent ("click the Submit button" → `getByRole(BUTTON, {name: "Submit"})`)
>
> Common roles: `BUTTON`, `TEXTBOX`, `LINK`, `HEADING`, `CHECKBOX`, `RADIO`, `COMBOBOX`, `LIST`, `LISTITEM`

---

## Q20: How do you find elements with multiple conditions?

```java
// Selenium — CSS
driver.findElement(By.cssSelector("input[type='text'][name='email']"));

// Selenium — XPath with AND
driver.findElement(By.xpath("//input[@type='text' and @name='email']"));

// Selenium — XPath with OR
driver.findElement(By.xpath("//input[@type='text' or @type='email']"));

// Playwright — chaining
page.locator("input[type='text'][name='email']").fill("test");

// Playwright — filter
page.locator("input").filter(new Locator.FilterOptions().setHasText("email"));
```

---

## Q21: What is `data-testid` and why should developers add it?

> A custom HTML attribute (`data-testid="login-btn"`) added specifically for test automation.
>
> **Benefits:**
> 1. Never changes during refactoring (unlike class names)
> 2. Clear intent — "this is for testing"
> 3. Doesn't affect styling or functionality
> 4. Easy to find: `[data-testid='login-btn']`
> 5. Both Selenium and Playwright support it natively

---

## Q22: How do you handle auto-complete/suggestion dropdowns?

```java
// Selenium
driver.findElement(By.id("search")).sendKeys("java");
wait.until(ExpectedConditions.visibilityOfElementLocated(By.cssSelector(".suggestion-list")));
driver.findElement(By.xpath("//li[text()='JavaScript']")).click();

// Playwright
page.fill("#search", "java");
page.locator(".suggestion-list").waitFor();
page.getByText("JavaScript").click();
```

---

## Q23: How do you verify text of all elements in a list?

```java
// Selenium
List<WebElement> items = driver.findElements(By.cssSelector(".menu-item"));
List<String> texts = items.stream().map(WebElement::getText).collect(Collectors.toList());
Assert.assertEquals(texts, Arrays.asList("Home", "About", "Contact"));

// Playwright
List<String> texts = page.locator(".menu-item").allTextContents();
assertThat(page.locator(".menu-item")).hasText(new String[]{"Home", "About", "Contact"});
```

---

## Q24: What is the difference between `text()` and `.` in XPath?

```java
// text() — matches direct text node only
By.xpath("//div[text()='Hello']")  
// Matches: <div>Hello</div>
// Does NOT match: <div><span>Hello</span></div>

// . (dot) — matches concatenated text of element and all descendants
By.xpath("//div[.='Hello']")
// Matches: <div>Hello</div>
// Also matches: <div><span>Hello</span></div>

// contains with text()
By.xpath("//*[contains(text(),'Hello')]")  // direct text only

// contains with .
By.xpath("//*[contains(.,'Hello')]")       // includes child text
```

---

## Q25: How do you handle elements that load asynchronously (AJAX)?

```java
// Selenium — explicit wait
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("data-loaded")));
wait.until(ExpectedConditions.textToBePresentInElementLocated(By.id("status"), "Complete"));
wait.until(ExpectedConditions.numberOfElementsToBeMoreThan(By.cssSelector(".item"), 0));

// Playwright — auto-wait (default 30s)
page.locator("#data-loaded").click();     // waits automatically
page.locator("#status").waitFor();        // explicit if needed
page.waitForResponse("**/api/data");      // wait for API call to complete
```

---

## Q26: What is `normalize-space()` in XPath?

> Strips leading/trailing whitespace and collapses internal spaces.
> ```java
> // HTML: <button>   Submit   </button>
> By.xpath("//button[text()='Submit']")            // ❌ Fails (extra spaces)
> By.xpath("//button[normalize-space()='Submit']") // ✅ Works
> ```

---

## Q27: How do you locate elements in a responsive/mobile view?

> Same locators work, but:
> 1. Some elements are hidden on mobile (hamburger menu replaces navbar)
> 2. Use `isDisplayed()` / `isVisible()` to check current state
> 3. May need to click hamburger menu first to reveal elements
> 4. Use `driver.manage().window().setSize(new Dimension(375, 812))` for mobile viewport
> 5. Playwright: `page.setViewportSize(375, 812)`

---

## Q28: How do you create a custom/dynamic XPath at runtime?

```java
// Parameterized XPath
public WebElement getMenuByName(String name) {
    return driver.findElement(By.xpath("//li[text()='" + name + "']"));
}

// String.format approach
String xpath = String.format("//button[@id='%s' and text()='%s']", id, text);
driver.findElement(By.xpath(xpath));

// Playwright
page.locator(String.format("[data-testid='%s']", testId)).click();
```

---

## Q29: What is the Playwright Locator vs ElementHandle?

| Locator | ElementHandle |
|---------|--------------|
| Lazy — re-queries every time | Eager — points to one DOM node |
| ✅ Never stale | ❌ Can become stale |
| Auto-waits | No auto-wait |
| Recommended | Deprecated/Legacy |

> Always use `page.locator()`, never `page.querySelector()`.

---

## Q30: How would you locate a button that has no ID, no name, no unique class?

```java
// Strategy 1: Use text
By.xpath("//button[text()='Submit']")

// Strategy 2: Use parent/sibling relationship
By.xpath("//div[@class='form-footer']/button")

// Strategy 3: Use other attributes
By.cssSelector("button[type='submit']")

// Strategy 4: Use position (last resort)
By.xpath("(//button)[3]")

// Strategy 5: Ask developer to add data-testid (BEST)
By.cssSelector("[data-testid='submit-btn']")

// Playwright
page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Submit")).click();
```

---

# SECTION 8: QUICK REFERENCE CARD

## Selenium — Top 10 Locator Patterns

```java
By.id("unique-id")
By.cssSelector("[data-testid='x']")
By.cssSelector("#parent .child")
By.cssSelector("input[type='text']")
By.xpath("//button[text()='Submit']")
By.xpath("//div[contains(@class,'active')]")
By.xpath("//label[text()='Email']/following-sibling::input")
By.xpath("//input[@id='x']/parent::div")
By.cssSelector("[id*='partial']")
By.xpath("(//button[@class='btn'])[1]")
```

## Playwright — Top 10 Locator Patterns

```java
page.getByTestId("login-btn")
page.getByRole(AriaRole.BUTTON, opts.setName("Submit"))
page.getByText("Welcome")
page.getByLabel("Username")
page.getByPlaceholder("Enter email")
page.locator("#id")
page.locator("[data-testid='x']")
page.locator("tr").filter(opts.setHasText("John")).locator(".edit")
page.locator(".item").nth(2)
page.frameLocator("#iframe").locator("#btn")
```

---

*30 interview questions + complete syntax reference. Master these and you'll handle any locator question.*
*Last updated: April 29, 2026*

