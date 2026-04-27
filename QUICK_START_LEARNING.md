# 🚀 Quick Start Guide - Learning RestAssured, Playwright & Selenium

## 📁 Files Created for Learning

I've created the following files for you:

| File | Purpose | Location |
|------|---------|----------|
| **LEARNING_GUIDE.md** | Complete theory & reference | `docs/LEARNING_GUIDE.md` |
| **RestAssuredExercise.java** | 10+ API testing exercises | `src/test/java/com/learning/` |
| **PlaywrightExercise.java** | 10+ Playwright exercises | `src/test/java/com/learning/` |
| **SeleniumExercise.java** | 10+ Selenium exercises | `src/test/java/com/learning/` |

---

## 🏃 How to Run Exercises

### Option 1: Run from IDE (Recommended)

1. Open the exercise file in IntelliJ
2. Click the green play button next to any `@Test` method
3. Watch the output in the console

### Option 2: Run from Command Line

```powershell
# Navigate to project
cd C:\XceleratorTest_Automation\XceleratorTestAutomation

# Run RestAssured exercises
.\gradlew test --tests "com.learning.RestAssuredExercise.exercise1_1*"

# Run Playwright exercises  
.\gradlew test --tests "com.learning.PlaywrightExercise.exercise1_1*"

# Run Selenium exercises
.\gradlew test --tests "com.learning.SeleniumExercise.exercise1_1*"

# Run all exercises from a class
.\gradlew test --tests "com.learning.RestAssuredExercise"
```

---

## 📚 Recommended Learning Order

### Week 1: RestAssured Basics
```
Day 1: exercise1_1 → exercise1_3 (Basic requests)
Day 2: exercise2_1 → exercise2_2 (Response validation)
Day 3: exercise3_1 → exercise3_3 (RequestSpecBuilder)
Day 4: exercise4_1 → exercise4_2 (Advanced patterns)
Day 5: bonusExercise (Complete CRUD)
```

### Week 2: Playwright Basics
```
Day 1: exercise1_1 → exercise1_2 (Browser basics)
Day 2: exercise2_1 → exercise2_2 (Locators)
Day 3: exercise3_1 → exercise3_3 (Form interactions)
Day 4: exercise4_1 → exercise4_4 (Advanced features)
Day 5: bonusExercise (Complete flow)
```

### Week 3: Selenium Basics
```
Day 1: exercise1_1 → exercise1_2 (Browser basics)
Day 2: exercise2_1 → exercise2_2 (Locators)
Day 3: exercise3_1 → exercise3_3 (Form interactions)
Day 4: exercise4_1 → exercise4_3 (Waits)
Day 5: exercise5_1 → exercise5_5 (Advanced)
```

### Week 4: Study Framework Code
```
Day 1: Read RestClient.java and RequestBuilder.java
Day 2: Read BrowserManager.java (Playwright implementation)
Day 3: Read DriverFactory.java (Selenium implementation)
Day 4: Read BasePage.java (Playwright actions)
Day 5: Read BasePageSelenium.java (Selenium actions)
```

---

## 🔗 Key Framework Files to Study

### For API Testing (RestAssured):
```
utaf/src/main/java/com/siemens/cas/api/core/
├── RestClient.java           ← HTTP client wrapper
├── RequestBuilder.java       ← Builds requests from JSON
└── AccessTokenHandler.java   ← Authentication handling

utaf/src/main/java/com/siemens/cas/api/stepdefinitions/
├── RequestStepDefinitions.java   ← API request steps
└── ResponseStepDefinitions.java  ← Response validation steps
```

### For UI Testing (Playwright):
```
utaf/src/main/java/com/siemens/cas/ui/core/
└── BrowserManager.java       ← Browser lifecycle management

utaf/src/main/java/com/siemens/cas/ui/pages/
└── BasePage.java            ← 1500+ lines of UI actions!
```

### For UI Testing (Selenium):
```
utaf/src/main/java/com/siemens/cas/ui/core/
└── DriverFactory.java       ← WebDriver management

utaf/src/main/java/com/siemens/cas/ui/pages/
└── BasePageSelenium.java    ← Selenium UI actions
```

---

## 💡 Tips for Learning

1. **Run exercises one at a time** - Don't rush, understand each one
2. **Read the comments** - I've added explanations in the code
3. **Modify the exercises** - Change URLs, add assertions, experiment!
4. **Compare with framework** - After each exercise, find similar code in UTAF
5. **Check the console output** - I've added detailed logging

---

## 🎯 After Completing Exercises

Once you've mastered the basics, study how the framework uses these technologies:

1. **Look at feature files** in `src/test/java/com/siemens/xf/api/features/`
2. **Run actual tests** with `.\gradlew test -Dtags="@XT-xxx"`
3. **Create your own test** following the framework patterns

---

## ❓ Need Help?

- Check `docs/LEARNING_GUIDE.md` for detailed explanations
- Look at existing tests in the `src/test/java/com/` folders
- Read the JavaDoc comments in framework classes

**Happy Learning! 🎓**

