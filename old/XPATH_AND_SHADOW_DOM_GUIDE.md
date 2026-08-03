# Complete Guide: XPath & Shadow DOM Element Locators

## Table of Contents
1. [XPath Fundamentals](#1-xpath-fundamentals)
2. [XPath Axes (Relationships)](#2-xpath-axes-relationships)
3. [XPath Functions](#3-xpath-functions)
4. [Advanced XPath Patterns](#4-advanced-xpath-patterns)
5. [Shadow DOM - What & Why](#5-shadow-dom---what--why)
6. [Finding Shadow Elements - Selenium](#6-finding-shadow-elements---selenium)
7. [Finding Shadow Elements - Playwright](#7-finding-shadow-elements---playwright)
8. [Real-World Examples](#8-real-world-examples)
9. [Chrome DevTools Tricks](#9-chrome-devtools-tricks)
10. [Cheat Sheet](#10-cheat-sheet)

---

## 1. XPath Fundamentals

### What is XPath?
XPath is a query language to navigate and select elements from an HTML/XML document tree.

### Two Types of XPath

| Type | Syntax | Example | Pros/Cons |
|------|--------|---------|-----------|
| **Absolute** | Starts with `/` | `/html/body/div[1]/form/input` | ❌ Fragile, breaks easily |
| **Relative** | Starts with `//` | `//input[@id='username']` | ✅ Flexible, recommended |

### Basic Syntax

```
//tagName[@attribute='value']
```

| Part | Meaning |
|------|---------|
| `//` | Search anywhere in the document |
| `tagName` | HTML tag (div, input, button, span, a, etc.) |
| `@attribute` | HTML attribute (id, class, name, type, href, etc.) |
| `'value'` | The attribute's value |

### Core XPath Examples

```
HTML:
<div class="login-form">
  <input id="username" type="text" name="user" placeholder="Enter username">
  <input id="password" type="password" name="pass">
  <button class="btn btn-primary" data-testid="login-btn">Sign In</button>
  <a href="/forgot">Forgot Password?</a>
  <span class="error-msg">Invalid credentials</span>
</div>
```

| What to Find | XPath | Explanation |
|-------------|-------|-------------|
| By ID | `//input[@id='username']` | Find input with id="username" |
| By Name | `//input[@name='user']` | Find input with name="user" |
| By Class | `//button[@class='btn btn-primary']` | Exact class match |
| By Type | `//input[@type='password']` | Find password field |
| By Text | `//button[text()='Sign In']` | Exact text match |
| By Partial Text | `//a[contains(text(),'Forgot')]` | Text contains "Forgot" |
| By Placeholder | `//input[@placeholder='Enter username']` | By placeholder text |
| By data-testid | `//button[@data-testid='login-btn']` | By custom attribute |
| By href | `//a[@href='/forgot']` | By link URL |
| Any attribute | `//*[@id='username']` | `*` matches any tag |

---

## 2. XPath Axes (Relationships)

Axes let you navigate the DOM tree **relative to a known element**.

### DOM Tree Visualization

```
         <html>                    ← ancestor
           │
         <body>                    ← ancestor
           │
    ┌──────┼──────┐
    │      │      │
  <div>  <div>  <div>             ← preceding-sibling / following-sibling
           │
    ┌──────┼──────┐
    │      │      │
  <label> <input> <button>         ← child
    │      SELF      │
  <span>           <svg>           ← descendant
```

### All Axes with Examples

#### 1. `parent` — Go UP one level

```
HTML:
<div class="form-group">
  <input id="email">
</div>

XPath: //input[@id='email']/parent::div
Result: Selects the <div class="form-group">
```

#### 2. `child` — Go DOWN one level

```
HTML:
<ul id="menu">
  <li>Home</li>
  <li>About</li>
</ul>

XPath: //ul[@id='menu']/child::li
Result: Selects both <li> elements (direct children only)
```

#### 3. `ancestor` — Go UP to ANY parent/grandparent

```
HTML:
<div class="card">
  <div class="card-body">
    <div class="form-group">
      <input id="name">
    </div>
  </div>
</div>

XPath: //input[@id='name']/ancestor::div[@class='card']
Result: Selects the outermost <div class="card">
Skips intermediate divs and goes straight to "card"
```

#### 4. `descendant` — Go DOWN to ANY child/grandchild

```
HTML:
<div id="container">
  <div>
    <div>
      <span class="price">$99</span>
    </div>
  </div>
</div>

XPath: //div[@id='container']/descendant::span[@class='price']
Result: Finds <span> at any depth inside container
Same as: //div[@id='container']//span[@class='price']
```

#### 5. `following-sibling` — Next elements at SAME level

```
HTML:
<div class="row">
  <label>Email</label>
  <input id="email">
  <span class="error">Required</span>
</div>

XPath: //label[text()='Email']/following-sibling::input
Result: Selects <input id="email"> (next sibling after label)

XPath: //label[text()='Email']/following-sibling::span
Result: Selects <span class="error">
```

#### 6. `preceding-sibling` — Previous elements at SAME level

```
HTML:
<tr>
  <td>John</td>
  <td>Admin</td>
  <td><button>Delete</button></td>
</tr>

XPath: //button[text()='Delete']/preceding-sibling::td[1]
Result: Selects <td>Admin</td> (1st td before the button's parent)

USE CASE: Find the name in same row as Delete button
XPath: //button[text()='Delete']/ancestor::tr/td[1]
Result: <td>John</td>
```

#### 7. `following` — ALL elements AFTER in entire document

```
XPath: //div[@id='header']/following::input[1]
Result: First <input> anywhere after the header div
```

#### 8. `preceding` — ALL elements BEFORE in entire document

```
XPath: //div[@id='footer']/preceding::input[1]
Result: Last <input> anywhere before the footer div
```

### Axes Summary Diagram

```
                    ancestor
                       ↑
                       |
preceding-sibling ← SELF → following-sibling
                       |
                       ↓
                   descendant

preceding ← (before in doc)    (after in doc) → following
```

---

## 3. XPath Functions

### Text Functions

```xpath
# Exact text match
//button[text()='Submit']

# Contains text (partial match) — MOST USED
//button[contains(text(),'Sub')]

# Starts with
//div[starts-with(@class,'btn-')]

# Normalize space (trims whitespace)
//button[normalize-space(text())='Sign In']

# String length
//input[string-length(@value) > 0]
```

### Contains — Most Powerful Function

```xpath
# Contains in attribute
//div[contains(@class,'error')]          # class contains "error"
//a[contains(@href,'login')]             # href contains "login"
//input[contains(@placeholder,'Enter')]  # placeholder contains "Enter"

# Contains in text
//span[contains(text(),'Success')]
//p[contains(.,'Welcome')]               # . means entire text content

# Contains with multiple classes (class="btn btn-primary btn-lg")
//button[contains(@class,'btn-primary')]  # Matches even with other classes
```

### Boolean Operators (and / or / not)

```xpath
# AND — both conditions must match
//input[@type='text' and @name='email']
//div[@class='alert' and contains(text(),'Error')]

# OR — either condition matches
//input[@type='text' or @type='email']
//button[text()='Submit' or text()='Save']

# NOT — exclude matches
//input[not(@type='hidden')]
//div[not(contains(@class,'disabled'))]
//li[not(@style='display:none')]

# Combined
//input[@type='text' and not(@disabled) and contains(@class,'form')]
```

### Position Functions

```xpath
# First element
(//div[@class='card'])[1]

# Last element
(//div[@class='card'])[last()]

# Second to last
(//div[@class='card'])[last()-1]

# Position range
//ul/li[position() >= 2 and position() <= 5]

# First child of type
//table//tr[1]/td[1]     # First row, first column of table

# Nth child
//ul[@id='list']/li[3]   # Third <li> in the list
```

---

## 4. Advanced XPath Patterns

### Pattern 1: Dynamic Tables — Find Cell by Row Content

```
HTML:
<table>
  <tr><td>John</td><td>Admin</td><td><button>Edit</button></td></tr>
  <tr><td>Jane</td><td>User</td><td><button>Edit</button></td></tr>
</table>
```

```xpath
# Click Edit button in John's row
//tr[td[text()='John']]//button[text()='Edit']

# Get role of Jane
//tr[td[text()='Jane']]/td[2]

# Find row where any cell contains "Admin"
//tr[td[contains(text(),'Admin')]]
```

### Pattern 2: Dynamic IDs / Attributes

```xpath
# ID changes every time: id="input_283746"
//input[starts-with(@id,'input_')]
//input[contains(@id,'input')]

# Class is dynamically generated
//div[contains(@class,'MuiButton') and contains(@class,'primary')]

# data-* attributes (most stable for testing)
//*[@data-testid='submit-btn']
//*[@data-cy='login-form']
//*[@data-automation='user-table']
```

### Pattern 3: Sibling-Based Navigation (No Direct Locator)

```
HTML:
<div class="form-group">
  <label>Username</label>
  <input type="text">         ← No id, no name, nothing unique!
</div>
<div class="form-group">
  <label>Password</label>
  <input type="password">     ← Same problem
</div>
```

```xpath
# Find input next to "Username" label
//label[text()='Username']/following-sibling::input

# Find input inside same parent as "Password" label
//label[text()='Password']/parent::div//input

# More robust: ancestor axis
//label[text()='Username']/ancestor::div[@class='form-group']//input
```

### Pattern 4: Index-Based (When All Else Fails)

```xpath
# First matching element
(//input[@type='text'])[1]

# Third button on page
(//button)[3]

# Last row of table
//table//tr[last()]

# Second dropdown
(//select[@class='form-control'])[2]
```

### Pattern 5: Handling SVG Elements

```xpath
# SVG elements need namespace or *
//*[local-name()='svg']
//*[local-name()='svg']//*[local-name()='path']

# Or with namespace wildcard
//*[name()='svg']
```

### Pattern 6: Multi-Condition Complex XPath

```xpath
# Find enabled, visible submit button with specific text
//button[
  text()='Submit' 
  and not(@disabled) 
  and not(contains(@class,'hidden'))
  and ancestor::form[@id='loginForm']
]

# Find link in navigation that is currently active
//nav//a[contains(@class,'active') and contains(@href,'dashboard')]

# Find error message near email field
//input[@name='email']/ancestor::div[contains(@class,'form-group')]//span[contains(@class,'error')]
```

---

## 5. Shadow DOM — What & Why

### What is Shadow DOM?

Shadow DOM is a web standard that creates an **encapsulated mini-DOM** inside an element. The inner elements are **hidden from normal CSS selectors and XPath**.

```
Regular DOM:                    Shadow DOM:
┌──────────────┐               ┌──────────────────────┐
│ <div>        │               │ <custom-element>     │
│   <span>     │               │   #shadow-root       │ ← Boundary!
│   <input>    │               │     <div>            │
│ </div>       │               │       <input>        │ ← Hidden!
│              │               │       <button>       │ ← Hidden!
│ Normal DOM   │               │     </div>           │
│ XPath works! │               │   #end shadow-root   │
└──────────────┘               │ </custom-element>     │
                               │ XPath CANNOT reach!   │
                               └──────────────────────┘
```

### How to Identify Shadow DOM in DevTools

1. Open **Chrome DevTools** (F12)
2. Look for `#shadow-root (open)` or `#shadow-root (closed)` in the Elements tab

```
▼ <my-app>
    ▼ #shadow-root (open)          ← This is Shadow DOM!
        <div class="container">
          <input id="search">
          ▼ <nested-component>
              ▼ #shadow-root (open)  ← Nested Shadow DOM!
                  <button>Click</button>
```

### Why Normal Locators Fail

```java
// ❌ These WILL NOT WORK for shadow elements
driver.findElement(By.xpath("//input[@id='search']"));       // FAILS
driver.findElement(By.cssSelector("my-app input#search"));   // FAILS
driver.findElement(By.id("search"));                         // FAILS
```

---

## 6. Finding Shadow Elements — Selenium

### Method 1: `getShadowRoot()` (Selenium 4+) ✅ Recommended

```java
// Single level shadow DOM
WebElement shadowHost = driver.findElement(By.cssSelector("my-app"));
SearchContext shadowRoot = shadowHost.getShadowRoot();
WebElement input = shadowRoot.findElement(By.cssSelector("input#search"));
input.sendKeys("Hello");
```

### Method 2: Nested Shadow DOM

```
HTML Structure:
<my-app>
  #shadow-root
    <user-panel>
      #shadow-root
        <login-form>
          #shadow-root
            <input id="username">
```

```java
// Navigate through each shadow boundary
SearchContext shadow1 = driver.findElement(By.cssSelector("my-app")).getShadowRoot();
SearchContext shadow2 = shadow1.findElement(By.cssSelector("user-panel")).getShadowRoot();
SearchContext shadow3 = shadow2.findElement(By.cssSelector("login-form")).getShadowRoot();
WebElement username = shadow3.findElement(By.cssSelector("input#username"));
username.sendKeys("admin");
```

### Method 3: JavaScript Executor (Works for ALL Selenium versions)

```java
// Single shadow root
WebElement element = (WebElement) ((JavascriptExecutor) driver).executeScript(
    "return document.querySelector('my-app')" +
    ".shadowRoot.querySelector('input#search')"
);
element.sendKeys("Hello");

// Nested shadow roots
WebElement nested = (WebElement) ((JavascriptExecutor) driver).executeScript(
    "return document.querySelector('my-app')" +
    ".shadowRoot.querySelector('user-panel')" +
    ".shadowRoot.querySelector('login-form')" +
    ".shadowRoot.querySelector('input#username')"
);
```

### Method 4: Reusable Shadow DOM Utility Class

```java
package com.framework.utils;

import org.openqa.selenium.*;
import java.util.List;

public class ShadowDomHelper {

    private final WebDriver driver;

    public ShadowDomHelper(WebDriver driver) {
        this.driver = driver;
    }

    /**
     * Get shadow root of an element
     */
    public SearchContext getShadowRoot(String hostCssSelector) {
        WebElement host = driver.findElement(By.cssSelector(hostCssSelector));
        return host.getShadowRoot();
    }

    /**
     * Get shadow root from a parent shadow context
     */
    public SearchContext getShadowRoot(SearchContext parentShadow, String hostCssSelector) {
        WebElement host = parentShadow.findElement(By.cssSelector(hostCssSelector));
        return host.getShadowRoot();
    }

    /**
     * Find element inside single shadow root
     * Usage: findInShadow("my-app", "input#search")
     */
    public WebElement findInShadow(String shadowHostCss, String elementCss) {
        SearchContext shadow = getShadowRoot(shadowHostCss);
        return shadow.findElement(By.cssSelector(elementCss));
    }

    /**
     * Find element inside nested shadow roots
     * Usage: findInNestedShadow("input#username", "my-app", "user-panel", "login-form")
     */
    public WebElement findInNestedShadow(String elementCss, String... shadowHostCssPath) {
        SearchContext current = driver;
        for (String hostCss : shadowHostCssPath) {
            WebElement host;
            if (current instanceof WebDriver) {
                host = ((WebDriver) current).findElement(By.cssSelector(hostCss));
            } else {
                host = ((SearchContext) current).findElement(By.cssSelector(hostCss));
            }
            current = host.getShadowRoot();
        }
        return current.findElement(By.cssSelector(elementCss));
    }

    /**
     * Find multiple elements inside shadow root
     */
    public List<WebElement> findAllInShadow(String shadowHostCss, String elementCss) {
        SearchContext shadow = getShadowRoot(shadowHostCss);
        return shadow.findElements(By.cssSelector(elementCss));
    }

    /**
     * Click element inside shadow DOM
     */
    public void clickInShadow(String shadowHostCss, String elementCss) {
        findInShadow(shadowHostCss, elementCss).click();
    }

    /**
     * Type text into shadow DOM element
     */
    public void typeInShadow(String shadowHostCss, String elementCss, String text) {
        WebElement element = findInShadow(shadowHostCss, elementCss);
        element.clear();
        element.sendKeys(text);
    }

    /**
     * Get text from shadow DOM element
     */
    public String getTextFromShadow(String shadowHostCss, String elementCss) {
        return findInShadow(shadowHostCss, elementCss).getText();
    }

    /**
     * JavaScript approach - works for closed shadow roots too
     */
    public WebElement findByJS(String jsQuery) {
        return (WebElement) ((JavascriptExecutor) driver).executeScript(
            "return " + jsQuery
        );
    }

    /**
     * Check if element exists in shadow DOM
     */
    public boolean existsInShadow(String shadowHostCss, String elementCss) {
        try {
            findInShadow(shadowHostCss, elementCss);
            return true;
        } catch (NoSuchElementException e) {
            return false;
        }
    }

    /**
     * Wait for element in shadow DOM
     */
    public WebElement waitForShadowElement(String shadowHostCss, String elementCss, int timeoutSeconds) {
        long end = System.currentTimeMillis() + (timeoutSeconds * 1000L);
        while (System.currentTimeMillis() < end) {
            try {
                WebElement el = findInShadow(shadowHostCss, elementCss);
                if (el.isDisplayed()) return el;
            } catch (Exception ignored) {}
            try { Thread.sleep(500); } catch (InterruptedException ignored) {}
        }
        throw new TimeoutException("Shadow element not found: " + elementCss + " in " + shadowHostCss);
    }
}
```

### Usage in Page Object

```java
public class MyAppPage extends BasePage {

    private ShadowDomHelper shadow;

    public MyAppPage() {
        super();
        this.shadow = new ShadowDomHelper(driver);
    }

    public void searchProduct(String query) {
        // Single shadow: <my-app> → #shadow-root → <input#search>
        shadow.typeInShadow("my-app", "input#search", query);
        shadow.clickInShadow("my-app", "button.search-btn");
    }

    public void login(String user, String pass) {
        // Nested shadow: my-app → user-panel → login-form → inputs
        WebElement username = shadow.findInNestedShadow(
            "input#username", "my-app", "user-panel", "login-form"
        );
        username.sendKeys(user);

        WebElement password = shadow.findInNestedShadow(
            "input#password", "my-app", "user-panel", "login-form"
        );
        password.sendKeys(pass);

        WebElement loginBtn = shadow.findInNestedShadow(
            "button#login", "my-app", "user-panel", "login-form"
        );
        loginBtn.click();
    }

    public String getErrorMessage() {
        return shadow.getTextFromShadow("my-app", "span.error-msg");
    }
}
```

### Usage in Step Definition

```java
@When("I search for {string} in the app")
public void searchInApp(String query) {
    MyAppPage page = new MyAppPage();
    page.searchProduct(query);
}
```

---

## 7. Finding Shadow Elements — Playwright

Playwright handles Shadow DOM **automatically** — no special code needed!

### Playwright Auto-Pierces Shadow DOM ✅

```java
// Playwright automatically searches inside shadow roots!
page.locator("input#search").fill("Hello");           // Just works!
page.locator("button.search-btn").click();             // Just works!
page.locator("my-app input#username").fill("admin");   // Just works!
```

### Explicit Shadow Piercing with `>>` operator

```java
// Use >> to explicitly pierce shadow boundaries
page.locator("my-app >> input#search").fill("Hello");

// Nested shadow piercing
page.locator("my-app >> user-panel >> login-form >> input#username").fill("admin");

// CSS piercing combinators
page.locator("my-app >> css=input.search-field").fill("query");
```

### Playwright Locator Examples for Shadow DOM

```java
// By role (auto-pierces shadow)
page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Submit")).click();

// By text (auto-pierces shadow)
page.getByText("Welcome back").isVisible();

// By test id (auto-pierces shadow)
page.getByTestId("login-btn").click();

// By placeholder (auto-pierces shadow)
page.getByPlaceholder("Enter username").fill("admin");

// Chained locators through shadow
page.locator("my-app").locator("input#search").fill("test");
```

### Playwright Shadow DOM Page Object

```java
public class PWMyAppPage {
    private final Page page;

    public PWMyAppPage(Page page) {
        this.page = page;
    }

    // No shadow DOM handling needed — Playwright auto-pierces!
    public void search(String query) {
        page.getByPlaceholder("Search products").fill(query);
        page.getByRole(AriaRole.BUTTON, 
            new Page.GetByRoleOptions().setName("Search")).click();
    }

    public void login(String user, String pass) {
        page.getByLabel("Username").fill(user);
        page.getByLabel("Password").fill(pass);
        page.getByTestId("login-btn").click();
    }
}
```

---

## 8. Real-World Examples

### Example 1: Siemens Xcelerator UI Components (Shadow DOM)

```
<siemens-header>
  #shadow-root (open)
    <nav class="main-nav">
      <div class="user-menu">
        <button class="profile-btn">John Doe</button>
        <ul class="dropdown">
          <li><a href="/settings">Settings</a></li>
          <li><a href="/logout">Logout</a></li>
        </ul>
      </div>
    </nav>
```

**Selenium:**
```java
ShadowDomHelper shadow = new ShadowDomHelper(driver);
shadow.clickInShadow("siemens-header", "button.profile-btn");
shadow.clickInShadow("siemens-header", "a[href='/logout']");
```

**Playwright:**
```java
page.locator("siemens-header >> button.profile-btn").click();
page.locator("siemens-header >> a[href='/logout']").click();
// OR simply:
page.getByText("Logout").click();
```

### Example 2: Complex Table with Dynamic Data

```
HTML:
<table id="users">
  <thead><tr><th>Name</th><th>Email</th><th>Role</th><th>Actions</th></tr></thead>
  <tbody>
    <tr><td>John</td><td>john@test.com</td><td>Admin</td><td><button>Edit</button><button>Delete</button></td></tr>
    <tr><td>Jane</td><td>jane@test.com</td><td>User</td><td><button>Edit</button><button>Delete</button></td></tr>
  </tbody>
</table>
```

```xpath
# Edit button for John
//table[@id='users']//tr[td[text()='John']]//button[text()='Edit']

# Email of user with role "Admin"
//table[@id='users']//tr[td[text()='Admin']]/td[2]

# Delete button in row containing "jane@test.com"
//tr[td[contains(text(),'jane@test.com')]]//button[text()='Delete']

# All usernames (column 1)
//table[@id='users']//tbody/tr/td[1]

# Count rows
count(//table[@id='users']//tbody/tr)

# Last row's name
//table[@id='users']//tbody/tr[last()]/td[1]
```

### Example 3: Dropdowns / Select Menus

```xpath
# Standard <select>
//select[@id='country']/option[@value='US']

# Custom dropdown (div-based)
//div[@class='dropdown']//li[text()='United States']

# Searchable dropdown
//div[contains(@class,'select')]//input[@type='search']

# Material UI dropdown
//div[@role='listbox']//div[@role='option' and text()='India']

# PrimeNG dropdown
//p-dropdown[@formcontrolname='country']//li[contains(@aria-label,'US')]
```

### Example 4: Modals / Dialogs

```xpath
# Modal title
//div[contains(@class,'modal')]//h5[contains(@class,'modal-title')]

# Close button in modal
//div[contains(@class,'modal')]//button[contains(@class,'close')]

# Confirm button in dialog
//div[@role='dialog']//button[text()='Confirm']

# Input inside modal
//div[contains(@class,'modal-body')]//input[@name='email']
```

### Example 5: Angular / React Components

```xpath
# Angular Material input
//mat-form-field//input[@formcontrolname='username']

# React component with data-testid
//*[@data-testid='user-profile-card']

# Angular CDK overlay (dropdowns/modals)
//div[contains(@class,'cdk-overlay')]//button[text()='OK']

# MUI (Material UI) components
//div[contains(@class,'MuiTextField')]//input
//button[contains(@class,'MuiButton') and contains(text(),'Save')]
```

---

## 9. Chrome DevTools Tricks

### Find XPath of Any Element

1. **Right-click** element → **Inspect**
2. In Elements tab, **right-click** the highlighted element
3. **Copy** → **Copy XPath** (absolute) or **Copy full XPath**

### Test XPath in Console

Press **F12** → **Console** tab:

```javascript
// Test XPath — returns matching elements
$x("//button[text()='Submit']")

// Test CSS selector
$$("button.submit-btn")

// Test if shadow root exists
document.querySelector('my-app').shadowRoot

// Find element inside shadow
document.querySelector('my-app').shadowRoot.querySelector('input#search')

// Nested shadow
document.querySelector('my-app').shadowRoot
  .querySelector('user-panel').shadowRoot
  .querySelector('input#username')
```

### Test XPath in Elements Tab

1. Press **F12** → **Elements** tab
2. Press **Ctrl+F** (search bar appears at bottom)
3. Type your XPath: `//button[text()='Submit']`
4. It shows **matches count** and highlights elements

### Identify Shadow DOM

```javascript
// Check if element has shadow root
document.querySelector('my-component').shadowRoot  // returns shadow root or null

// List all elements with shadow roots on page
document.querySelectorAll('*').forEach(el => {
  if (el.shadowRoot) console.log(el.tagName, el);
});
```

### Copy Unique Selector

1. Right-click element in Elements tab
2. **Copy** → **Copy selector** (CSS)
3. **Copy** → **Copy JS path** (full JavaScript path including shadow roots!)

---

## 10. Cheat Sheet

### XPath Quick Reference

| Task | XPath |
|------|-------|
| By ID | `//tag[@id='val']` |
| By class | `//tag[@class='val']` |
| Partial class | `//tag[contains(@class,'val')]` |
| By text | `//tag[text()='exact']` |
| Partial text | `//tag[contains(text(),'partial')]` |
| By attribute | `//tag[@attr='val']` |
| Any tag | `//*[@id='val']` |
| Parent | `//child/parent::tag` |
| Ancestor | `//child/ancestor::tag[@attr='val']` |
| Following sibling | `//tag/following-sibling::tag2` |
| Preceding sibling | `//tag/preceding-sibling::tag2` |
| Child | `//parent/child::tag` |
| Descendant | `//parent//descendant::tag` or `//parent//tag` |
| First match | `(//tag)[1]` |
| Last match | `(//tag)[last()]` |
| AND condition | `//tag[@a='1' and @b='2']` |
| OR condition | `//tag[@a='1' or @a='2']` |
| NOT condition | `//tag[not(@disabled)]` |
| Starts with | `//tag[starts-with(@id,'pre_')]` |
| Contains attr | `//tag[contains(@href,'login')]` |
| Table cell by row | `//tr[td[text()='Name']]//button` |
| Normalize space | `//tag[normalize-space()='text']` |

### Shadow DOM Quick Reference

| Task | Selenium 4 | Playwright |
|------|------------|------------|
| Single shadow | `host.getShadowRoot().findElement(By.css("sel"))` | `page.locator("sel")` (auto) |
| Nested shadow | Chain `.getShadowRoot()` calls | `page.locator("host1 >> host2 >> sel")` |
| JS approach | `executeScript("return el.shadowRoot.querySelector()")` | Not needed |
| Wait for shadow el | Custom wait loop | `page.locator("sel").waitFor()` |
| Click in shadow | `shadowRoot.findElement().click()` | `page.locator("sel").click()` |
| Type in shadow | `shadowRoot.findElement().sendKeys()` | `page.locator("sel").fill()` |
| Check exists | try-catch `NoSuchElementException` | `page.locator("sel").isVisible()` |

### When to Use What?

| Situation | Use |
|-----------|-----|
| Element has unique `id` | `By.id("val")` — fastest |
| Element has unique `name` | `By.name("val")` |
| Element has `data-testid` | `By.cssSelector("[data-testid='val']")` |
| Need text-based search | XPath: `//tag[text()='val']` |
| Need parent/ancestor navigation | XPath axes |
| Need sibling navigation | XPath: `following-sibling` / `preceding-sibling` |
| Dynamic ID/class | XPath: `contains()`, `starts-with()` |
| Shadow DOM (Selenium) | `getShadowRoot()` + CSS selectors |
| Shadow DOM (Playwright) | Just use normal locators — auto-pierces! |
| Complex table lookup | XPath: `//tr[td[text()='X']]//button` |
| Multiple conditions | XPath: `and` / `or` / `not` |

