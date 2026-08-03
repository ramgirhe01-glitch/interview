# 🌐 JavaScript Complete Interview Guide — SDET Focus
### Ram Girhe | 3 YOE | Exercises + Interview Questions + Deep Concepts

---

# PART 1: JAVASCRIPT CORE CONCEPTS

---

## 1. Variables & Data Types

```javascript
// var (function-scoped, hoisted) — AVOID
var x = 10;

// let (block-scoped, reassignable) — USE
let name = "Ram";
name = "Shyam"; // OK

// const (block-scoped, not reassignable) — PREFER
const PI = 3.14;
// PI = 3.15; // ❌ TypeError

// But const objects/arrays CAN be mutated
const user = { name: "Ram" };
user.name = "Shyam"; // OK — mutating property, not reassigning

// Data Types
// Primitives: string, number, boolean, null, undefined, symbol, bigint
// Reference: object, array, function

typeof "hello"     // "string"
typeof 42          // "number"
typeof true        // "boolean"
typeof undefined   // "undefined"
typeof null        // "object" ← famous JS bug!
typeof []          // "object"
typeof {}          // "object"
Array.isArray([])  // true
```

---

## 2. == vs === (Equality)

```javascript
// == (loose equality) — type coercion
5 == "5"       // true ← string "5" coerced to number
0 == false     // true
null == undefined  // true
"" == false    // true

// === (strict equality) — no coercion
5 === "5"      // false
0 === false    // false
null === undefined  // false

// RULE: ALWAYS use === in production code
```

---

## 3. Functions

```javascript
// Function declaration (hoisted)
function add(a, b) {
    return a + b;
}

// Function expression (not hoisted)
const subtract = function(a, b) {
    return a - b;
};

// Arrow function (ES6)
const multiply = (a, b) => a * b;
const square = x => x * x;        // Single param: no parens needed
const greet = () => "Hello!";       // No params

// Default parameters
function createUser(name, role = "user") {
    return { name, role };
}

// Rest parameters
function sum(...numbers) {
    return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4); // 10

// Spread operator
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]
const obj1 = { a: 1 };
const obj2 = { ...obj1, b: 2 }; // { a: 1, b: 2 }
```

---

## 4. Closures (VERY commonly asked)

```javascript
function outer() {
    let count = 0;  // private variable
    return function inner() {
        count++;
        return count;
    };
}

const counter = outer();
counter(); // 1
counter(); // 2
counter(); // 3
// count is "enclosed" in inner function — closure!

// Practical use: Factory function
function createLogger(prefix) {
    return function(message) {
        console.log(`[${prefix}] ${message}`);
    };
}
const apiLog = createLogger("API");
apiLog("Request sent");  // [API] Request sent
apiLog("Response: 200"); // [API] Response: 200
```

**Interview Q:** *What is a closure?*
> A closure is a function that remembers and accesses variables from its outer scope, even after the outer function has returned. It "closes over" the variables.

---

## 5. `this` keyword

```javascript
// In global scope: window (browser) or global (Node)
console.log(this); // window

// In object method: the object
const user = {
    name: "Ram",
    greet() {
        console.log(this.name); // "Ram"
    }
};

// Arrow function: inherits `this` from parent scope
const user2 = {
    name: "Ram",
    greet: () => {
        console.log(this.name); // undefined! Arrow doesn't have own `this`
    },
    greetDelayed() {
        setTimeout(() => {
            console.log(this.name); // "Ram" — arrow inherits from greetDelayed
        }, 100);
    }
};

// bind, call, apply
function greet(greeting) {
    console.log(`${greeting}, ${this.name}`);
}
const person = { name: "Ram" };
greet.call(person, "Hello");     // "Hello, Ram"
greet.apply(person, ["Hello"]);  // "Hello, Ram"
const bound = greet.bind(person);
bound("Hello");                   // "Hello, Ram"
```

---

## 6. Promises & Async/Await (CRITICAL for SDET)

### Callbacks (old way):
```javascript
function getData(callback) {
    setTimeout(() => {
        callback("Data loaded");
    }, 1000);
}
getData((data) => console.log(data));
```

### Promises:
```javascript
function getData() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const success = true;
            if (success) resolve("Data loaded");
            else reject(new Error("Failed to load"));
        }, 1000);
    });
}

getData()
    .then(data => console.log(data))       // "Data loaded"
    .catch(err => console.error(err))
    .finally(() => console.log("Done"));

// Promise.all — parallel, fail if ANY fails
Promise.all([fetch(url1), fetch(url2), fetch(url3)])
    .then(responses => console.log("All done"))
    .catch(err => console.log("One failed"));

// Promise.allSettled — parallel, get all results
Promise.allSettled([fetch(url1), fetch(url2)])
    .then(results => {
        results.forEach(r => console.log(r.status)); // "fulfilled" or "rejected"
    });

// Promise.race — first to resolve/reject wins
Promise.race([fetch(url1), fetch(url2)])
    .then(fastest => console.log(fastest));
```

### Async/Await (modern, preferred):
```javascript
async function fetchUser(id) {
    try {
        const response = await fetch(`/api/users/${id}`);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const user = await response.json();
        return user;
    } catch (error) {
        console.error("Failed:", error.message);
        throw error;
    }
}

// Parallel async calls
async function fetchAll() {
    const [users, orders] = await Promise.all([
        fetch("/api/users").then(r => r.json()),
        fetch("/api/orders").then(r => r.json())
    ]);
    return { users, orders };
}
```

---

## 7. Array Methods (asked in EVERY JS interview)

```javascript
const nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// map — transform each element
const doubled = nums.map(n => n * 2);        // [2, 4, 6, ...]

// filter — keep elements matching condition
const evens = nums.filter(n => n % 2 === 0);  // [2, 4, 6, 8, 10]

// reduce — combine into single value
const sum = nums.reduce((acc, n) => acc + n, 0);  // 55

// find — first match
const firstEven = nums.find(n => n % 2 === 0);    // 2

// findIndex
const idx = nums.findIndex(n => n > 5);            // 5

// some — at least one matches?
nums.some(n => n > 9);    // true

// every — all match?
nums.every(n => n > 0);   // true

// includes
nums.includes(5);          // true

// forEach — iterate (no return value)
nums.forEach(n => console.log(n));

// flat — flatten nested arrays
[[1,2], [3,4], [5]].flat();  // [1, 2, 3, 4, 5]

// flatMap — map + flatten
["hello world", "foo bar"].flatMap(s => s.split(" "));
// ["hello", "world", "foo", "bar"]

// sort (CAREFUL: sorts as strings by default!)
[10, 1, 21, 2].sort();              // [1, 10, 2, 21] ❌
[10, 1, 21, 2].sort((a, b) => a - b); // [1, 2, 10, 21] ✅

// Chaining
const result = nums
    .filter(n => n % 2 === 0)
    .map(n => n * n)
    .reduce((acc, n) => acc + n, 0);  // 4+16+36+64+100 = 220
```

---

## 8. Objects & Destructuring

```javascript
// Object shorthand
const name = "Ram", age = 25;
const user = { name, age };  // { name: "Ram", age: 25 }

// Destructuring
const { name: userName, age: userAge, city = "Unknown" } = user;

// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first=1, second=2, rest=[3,4,5]

// Object methods
const keys = Object.keys(user);      // ["name", "age"]
const values = Object.values(user);  // ["Ram", 25]
const entries = Object.entries(user); // [["name","Ram"], ["age",25]]
const merged = Object.assign({}, obj1, obj2);
const frozen = Object.freeze(user);  // Cannot modify

// Optional chaining (?.)
const street = user?.address?.street ?? "N/A";

// Nullish coalescing (??)
const val = null ?? "default";   // "default"
const val2 = 0 ?? "default";    // 0 (only null/undefined trigger ??)
const val3 = 0 || "default";    // "default" (|| treats 0 as falsy)
```

---

## 9. ES6+ Classes

```javascript
class Animal {
    #id;  // Private field
    
    constructor(name, sound) {
        this.name = name;
        this.sound = sound;
        this.#id = Math.random();
    }
    
    speak() {
        return `${this.name} says ${this.sound}`;
    }
    
    get info() {    // Getter
        return `${this.name} (${this.sound})`;
    }
    
    set nickname(value) {  // Setter
        if (!value) throw new Error("Nickname required");
        this._nickname = value;
    }
    
    static create(name) {  // Static method
        return new Animal(name, "...");
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name, "Woof");
        this.breed = breed;
    }
    
    speak() {  // Override
        return `${this.name} barks: ${this.sound}!`;
    }
}

const dog = new Dog("Buddy", "Labrador");
console.log(dog.speak());  // "Buddy barks: Woof!"
console.log(dog.info);     // "Buddy (Woof)"
console.log(dog instanceof Animal);  // true
```

---

## 10. Event Loop & Asynchronous JS

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve().then(() => console.log("3"));

console.log("4");

// Output: 1, 4, 3, 2
// Why? Call stack first (1, 4), then microtask queue (Promise: 3),
// then macrotask queue (setTimeout: 2)
```

**Interview Q:** *Explain the Event Loop.*
> JS is single-threaded. The event loop processes:
> 1. **Call Stack** — synchronous code
> 2. **Microtask Queue** — Promises, queueMicrotask
> 3. **Macrotask Queue** — setTimeout, setInterval, I/O
> Microtasks always run before macrotasks.

---

## 11. Error Handling

```javascript
// try-catch-finally
try {
    const data = JSON.parse(invalidJson);
} catch (error) {
    console.error(`Error: ${error.message}`);
    console.error(`Stack: ${error.stack}`);
} finally {
    console.log("Cleanup");
}

// Custom Error
class ApiError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.name = "ApiError";
        this.statusCode = statusCode;
    }
}

throw new ApiError("Not Found", 404);

// Async error handling
async function fetchData() {
    try {
        const res = await fetch("/api/data");
        if (!res.ok) throw new ApiError("API failed", res.status);
        return await res.json();
    } catch (err) {
        if (err instanceof ApiError) console.log(`API Error: ${err.statusCode}`);
        else throw err;  // re-throw unexpected errors
    }
}
```

---

# PART 2: JAVASCRIPT FOR TEST AUTOMATION (Playwright)

---

## Playwright Basics
```javascript
const { test, expect } = require('@playwright/test');

test.describe('User Management', () => {
    
    test.beforeEach(async ({ page }) => {
        await page.goto('https://app.example.com/login');
    });

    test('should login successfully', async ({ page }) => {
        await page.fill('#username', 'ram@test.com');
        await page.fill('#password', 'password123');
        await page.click('#loginBtn');
        
        await expect(page).toHaveURL('/dashboard');
        await expect(page.locator('.welcome-msg')).toContainText('Welcome, Ram');
    });

    test('should show error for invalid login', async ({ page }) => {
        await page.fill('#username', 'wrong@test.com');
        await page.fill('#password', 'wrong');
        await page.click('#loginBtn');
        
        await expect(page.locator('.error')).toBeVisible();
        await expect(page.locator('.error')).toHaveText('Invalid credentials');
    });
});

// API testing with Playwright
test('API: Create user', async ({ request }) => {
    const response = await request.post('/api/users', {
        data: { name: 'Ram', email: 'ram@test.com' },
        headers: { 'Authorization': 'Bearer token123' }
    });
    
    expect(response.ok()).toBeTruthy();
    expect(response.status()).toBe(201);
    
    const body = await response.json();
    expect(body.name).toBe('Ram');
    expect(body.id).toBeDefined();
});

// Network interception
test('mock API response', async ({ page }) => {
    await page.route('**/api/users', route => {
        route.fulfill({
            status: 200,
            body: JSON.stringify([{ id: 1, name: 'Mock User' }])
        });
    });
    
    await page.goto('/users');
    await expect(page.locator('.user-name')).toHaveText('Mock User');
});
```

---

## Node.js API Testing (axios + jest/mocha)
```javascript
const axios = require('axios');

const BASE_URL = 'https://api.example.com';
let token;

// Setup: Get auth token
beforeAll(async () => {
    const res = await axios.post(`${BASE_URL}/auth/token`, {
        grant_type: 'client_credentials',
        client_id: 'test-client'
    });
    token = res.data.access_token;
});

// GET test
test('GET /users returns list', async () => {
    const res = await axios.get(`${BASE_URL}/users`, {
        headers: { Authorization: `Bearer ${token}` }
    });
    
    expect(res.status).toBe(200);
    expect(Array.isArray(res.data)).toBe(true);
    expect(res.data.length).toBeGreaterThan(0);
    expect(res.data[0]).toHaveProperty('id');
    expect(res.data[0]).toHaveProperty('name');
});

// POST + GET chain
test('Create and verify user', async () => {
    // Create
    const createRes = await axios.post(`${BASE_URL}/users`, 
        { name: 'Ram', email: `ram_${Date.now()}@test.com` },
        { headers: { Authorization: `Bearer ${token}` } }
    );
    expect(createRes.status).toBe(201);
    const userId = createRes.data.id;
    
    // Verify
    const getRes = await axios.get(`${BASE_URL}/users/${userId}`, {
        headers: { Authorization: `Bearer ${token}` }
    });
    expect(getRes.data.name).toBe('Ram');
    
    // Cleanup
    await axios.delete(`${BASE_URL}/users/${userId}`, {
        headers: { Authorization: `Bearer ${token}` }
    });
});
```

---

# PART 3: JAVASCRIPT EXERCISES

---

### E1: Debounce function (classic interview question)
```javascript
function debounce(func, delay) {
    let timer;
    return function(...args) {
        clearTimeout(timer);
        timer = setTimeout(() => func.apply(this, args), delay);
    };
}
// Usage: Input search — only fires API after user stops typing for 300ms
const search = debounce((query) => fetch(`/api/search?q=${query}`), 300);
```

### E2: Flatten nested array
```javascript
function flatten(arr) {
    return arr.reduce((acc, item) => 
        acc.concat(Array.isArray(item) ? flatten(item) : item), []);
}
flatten([1, [2, [3, [4]]]]); // [1, 2, 3, 4]
```

### E3: Deep clone an object
```javascript
// Simple (doesn't handle functions, dates, undefined)
const clone1 = JSON.parse(JSON.stringify(obj));

// Proper deep clone
function deepClone(obj) {
    if (obj === null || typeof obj !== 'object') return obj;
    if (Array.isArray(obj)) return obj.map(deepClone);
    return Object.fromEntries(
        Object.entries(obj).map(([key, val]) => [key, deepClone(val)])
    );
}
```

### E4: Group array of objects by property
```javascript
function groupBy(arr, key) {
    return arr.reduce((groups, item) => {
        const group = item[key];
        groups[group] = groups[group] || [];
        groups[group].push(item);
        return groups;
    }, {});
}
const people = [
    { name: "Ram", city: "Pune" },
    { name: "Shyam", city: "Mumbai" },
    { name: "Sita", city: "Pune" }
];
groupBy(people, "city");
// { Pune: [{Ram}, {Sita}], Mumbai: [{Shyam}] }
```

### E5: Implement Promise.all
```javascript
function promiseAll(promises) {
    return new Promise((resolve, reject) => {
        const results = [];
        let completed = 0;
        promises.forEach((promise, index) => {
            Promise.resolve(promise).then(value => {
                results[index] = value;
                completed++;
                if (completed === promises.length) resolve(results);
            }).catch(reject);
        });
    });
}
```

### E6: Memoize function
```javascript
function memoize(fn) {
    const cache = new Map();
    return function(...args) {
        const key = JSON.stringify(args);
        if (cache.has(key)) return cache.get(key);
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}
const expensiveCalc = memoize((n) => { /* heavy computation */ return n * n; });
```

### E7: Retry API call with exponential backoff
```javascript
async function retryWithBackoff(fn, maxRetries = 3, baseDelay = 1000) {
    for (let attempt = 0; attempt < maxRetries; attempt++) {
        try {
            return await fn();
        } catch (error) {
            if (attempt === maxRetries - 1) throw error;
            const delay = baseDelay * Math.pow(2, attempt);
            console.log(`Attempt ${attempt + 1} failed. Retrying in ${delay}ms...`);
            await new Promise(resolve => setTimeout(resolve, delay));
        }
    }
}

// Usage
const data = await retryWithBackoff(() => fetch('/api/flaky-endpoint'));
```

---

# PART 4: TOP 35 JAVASCRIPT INTERVIEW QUESTIONS

---

| # | Question | Key Answer |
|---|----------|------------|
| 1 | var vs let vs const? | var: function-scoped, hoisted. let: block-scoped. const: block-scoped, not reassignable |
| 2 | == vs ===? | == coerces types. === strict (no coercion). Always use === |
| 3 | What is hoisting? | Declarations moved to top. var=undefined, let/const=TDZ, functions=full hoist |
| 4 | What is a closure? | Function that remembers outer scope variables even after outer function returns |
| 5 | What is `this`? | Depends on call context: object method=object, arrow=parent scope, global=window |
| 6 | What is the event loop? | Processes call stack → microtasks (Promises) → macrotasks (setTimeout) |
| 7 | What is a Promise? | Object representing future completion/failure of async operation |
| 8 | Promise vs async/await? | Same thing. async/await is syntactic sugar over Promises. Cleaner code |
| 9 | What is callback hell? | Deeply nested callbacks. Solution: Promises, async/await |
| 10 | map vs forEach? | map returns new array. forEach returns undefined |
| 11 | map vs filter vs reduce? | map: transform. filter: keep matching. reduce: combine into one |
| 12 | What is spread operator? | `...arr` expands array/object. Used for copies, merging, function args |
| 13 | What is destructuring? | Extract values: `const { name } = obj`, `const [a, b] = arr` |
| 14 | What is optional chaining? | `obj?.prop?.nested` — returns undefined instead of throwing on null |
| 15 | null vs undefined? | null: intentional empty. undefined: not assigned. typeof null = "object" |
| 16 | What is NaN? | "Not a Number". `NaN !== NaN`. Use `Number.isNaN()` to check |
| 17 | Truthy vs Falsy? | Falsy: false, 0, "", null, undefined, NaN. Everything else is truthy |
| 18 | What is prototype chain? | Objects inherit from prototype. `obj.__proto__` → `Object.prototype` → null |
| 19 | What is event delegation? | Attach listener to parent, handle child events via event.target |
| 20 | What is debounce vs throttle? | Debounce: delay until idle. Throttle: max once per interval |
| 21 | Deep copy vs shallow copy? | Shallow: `{...obj}` (nested refs shared). Deep: `structuredClone()` or recursive |
| 22 | What are template literals? | `` `Hello ${name}` `` — string interpolation with backticks |
| 23 | What is Symbol? | Unique, immutable primitive. Used for object property keys |
| 24 | What is a generator? | `function*` with `yield`. Lazy iteration. Returns iterator |
| 25 | What is a WeakMap/WeakSet? | Keys are weakly held — garbage collected when no other reference |
| 26 | What is CORS? | Cross-Origin Resource Sharing. Browser security restricting cross-domain requests |
| 27 | What is localStorage vs sessionStorage? | localStorage: persists. sessionStorage: cleared on tab close. Both: 5MB |
| 28 | What is JSON? | JavaScript Object Notation. `JSON.parse()` string→object, `JSON.stringify()` object→string |
| 29 | What is fetch API? | Modern HTTP request API. Returns Promise. `fetch(url).then(r => r.json())` |
| 30 | What is a module? | `export/import` (ES6) or `module.exports/require` (CommonJS) |
| 31 | What is TypeScript? | Typed superset of JS. Adds static types, interfaces, generics |
| 32 | Playwright vs Cypress? | Playwright: multi-browser, multi-lang, faster. Cypress: JS-only, easy setup |
| 33 | How to handle async in tests? | async/await in test functions. Playwright auto-waits |
| 34 | What is Page Object in Playwright? | Class per page: locators as properties, actions as methods |
| 35 | How to mock APIs in tests? | `page.route()` in Playwright, `cy.intercept()` in Cypress |

---

*JavaScript + Playwright is the hottest combo in test automation. Master async/await + array methods + Promises!*

