# Selenium Notes (Java) — Basic to Advanced

A complete reference guide with Java code examples. Selenium is an open-source tool for automating web browsers, primarily used for testing web applications.

---

## Table of Contents
1. [Setup & Installation](#1-setup--installation)
2. [WebDriver Basics](#2-webdriver-basics)
3. [Locators](#3-locators)
4. [Basic Element Interactions](#4-basic-element-interactions)
5. [Waits](#5-waits)
6. [Dropdowns](#6-dropdowns)
7. [Alerts](#7-alerts)
8. [Frames & iFrames](#8-frames--iframes)
9. [Windows & Tabs](#9-windows--tabs)
10. [Mouse & Keyboard Actions](#10-mouse--keyboard-actions)
11. [Cookies & Sessions](#11-cookies--sessions)
12. [Screenshots](#12-screenshots)
13. [File Upload & Download](#13-file-upload--download)
14. [JavaScript Executor](#14-javascript-executor)
15. [Browser Options & Headless Mode](#15-browser-options--headless-mode)
16. [Page Object Model (POM)](#16-page-object-model-pom)
17. [Data-Driven Testing with TestNG](#17-data-driven-testing-with-testng)
18. [TestNG Integration](#18-testng-integration)
19. [Selenium Grid](#19-selenium-grid)
20. [Shadow DOM](#20-shadow-dom)
21. [Best Practices](#21-best-practices)

---

## 1. Setup & Installation

### Maven `pom.xml`

```xml
<dependencies>
    <!-- Selenium Java -->
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.21.0</version>
    </dependency>

    <!-- TestNG -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.10.2</version>
        <scope>test</scope>
    </dependency>

    <!-- WebDriverManager (optional - Selenium 4.6+ has built-in driver mgmt) -->
    <dependency>
        <groupId>io.github.bonigarcia</groupId>
        <artifactId>webdrivermanager</artifactId>
        <version>5.8.0</version>
    </dependency>
</dependencies>
```

### Gradle `build.gradle`

```groovy
dependencies {
    implementation 'org.seleniumhq.selenium:selenium-java:4.21.0'
    testImplementation 'org.testng:testng:7.10.2'
    implementation 'io.github.bonigarcia:webdrivermanager:5.8.0'
}
```

### First Test

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class FirstTest {
    public static void main(String[] args) {
        // Selenium 4.6+ auto-downloads the driver
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com");
        System.out.println(driver.getTitle());
        driver.quit();
    }
}
```

---

## 2. WebDriver Basics

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.Dimension;

WebDriver driver = new ChromeDriver();

// Navigation
driver.get("https://google.com");
driver.navigate().to("https://github.com");
driver.navigate().back();
driver.navigate().forward();
driver.navigate().refresh();

// Window management
driver.manage().window().maximize();
driver.manage().window().minimize();
driver.manage().window().fullscreen();
driver.manage().window().setSize(new Dimension(1280, 800));

// Page info
System.out.println(driver.getTitle());
System.out.println(driver.getCurrentUrl());
System.out.println(driver.getPageSource());

// Closing
driver.close();   // Closes current tab/window
driver.quit();    // Closes all windows + ends session
```

---

## 3. Locators

Selenium provides 8 built-in locator strategies through the `By` class.

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import java.util.List;

driver.findElement(By.id("username"));
driver.findElement(By.name("email"));
driver.findElement(By.className("btn-primary"));
driver.findElement(By.tagName("input"));
driver.findElement(By.linkText("Click Here"));
driver.findElement(By.partialLinkText("Click"));
driver.findElement(By.cssSelector("input#username"));
driver.findElement(By.xpath("//input[@id='username']"));

// Find multiple elements
List<WebElement> links = driver.findElements(By.tagName("a"));
for (WebElement link : links) {
    System.out.println(link.getAttribute("href"));
}
```

### CSS Selectors vs XPath cheatsheet

| Goal | CSS Selector | XPath |
|---|---|---|
| By ID | `#username` | `//*[@id='username']` |
| By class | `.btn` | `//*[@class='btn']` |
| By attribute | `input[name='email']` | `//input[@name='email']` |
| Contains text | — | `//button[contains(text(),'Submit')]` |
| Child | `div > p` | `//div/p` |
| Descendant | `div p` | `//div//p` |
| Nth child | `li:nth-child(2)` | `//li[2]` |
| Parent | — | `//span/..` |

CSS selectors are generally faster than XPath; prefer them when possible.

---

## 4. Basic Element Interactions

```java
WebElement element = driver.findElement(By.id("search"));

// Input
element.sendKeys("Selenium");
element.clear();

// Click
element.click();

// Read properties
System.out.println(element.getText());                  // Visible text
System.out.println(element.getAttribute("value"));      // Attribute
System.out.println(element.getDomProperty("checked"));  // JS property
System.out.println(element.getTagName());
System.out.println(element.getSize());                  // Dimension
System.out.println(element.getLocation());              // Point
System.out.println(element.getRect());                  // Rectangle

// State checks
System.out.println(element.isDisplayed());
System.out.println(element.isEnabled());
System.out.println(element.isSelected());

// Form submission
element.submit();
```

---

## 5. Waits

**Never use `Thread.sleep()` in production tests.** Use Selenium's wait mechanisms instead.

### Implicit Wait

```java
import java.time.Duration;

driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
```

### Explicit Wait (recommended)

```java
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;

WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

WebElement el = wait.until(
    ExpectedConditions.presenceOfElementLocated(By.id("username")));
el = wait.until(
    ExpectedConditions.visibilityOfElementLocated(By.id("username")));
el = wait.until(
    ExpectedConditions.elementToBeClickable(By.id("submit")));

wait.until(ExpectedConditions.titleContains("Dashboard"));
wait.until(ExpectedConditions.urlContains("/home"));
wait.until(ExpectedConditions.textToBePresentInElementLocated(
    By.id("msg"), "Success"));
wait.until(ExpectedConditions.invisibilityOfElementLocated(By.id("loader")));
wait.until(ExpectedConditions.alertIsPresent());
```

### Fluent Wait

```java
import org.openqa.selenium.support.ui.FluentWait;
import org.openqa.selenium.NoSuchElementException;
import java.util.function.Function;

Wait<WebDriver> fluentWait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(15))
    .pollingEvery(Duration.ofSeconds(1))
    .ignoring(NoSuchElementException.class);

WebElement element = fluentWait.until(d -> d.findElement(By.id("result")));
```

> **Tip:** Don't mix implicit and explicit waits — it can cause unpredictable timeouts.

---

## 6. Dropdowns

### Standard `<select>` dropdowns

```java
import org.openqa.selenium.support.ui.Select;

Select dropdown = new Select(driver.findElement(By.id("country")));

dropdown.selectByVisibleText("India");
dropdown.selectByValue("IN");
dropdown.selectByIndex(2);

// Read options
for (WebElement option : dropdown.getOptions()) {
    System.out.println(option.getText());
}

System.out.println(dropdown.getFirstSelectedOption().getText());

// Multi-select
if (dropdown.isMultiple()) {
    dropdown.deselectAll();
    dropdown.deselectByValue("US");
}
```

### Custom dropdowns (div-based)

```java
driver.findElement(By.id("dropdown-toggle")).click();
wait.until(ExpectedConditions.elementToBeClickable(
    By.xpath("//li[text()='Option 1']"))).click();
```

---

## 7. Alerts

```java
import org.openqa.selenium.Alert;

Alert alert = driver.switchTo().alert();

System.out.println(alert.getText());
alert.accept();                  // Click OK
alert.dismiss();                 // Click Cancel
alert.sendKeys("My input");      // For prompt alerts
```

---

## 8. Frames & iFrames

```java
// Switch by index, name/id, or WebElement
driver.switchTo().frame(0);
driver.switchTo().frame("frameName");
WebElement frameEl = driver.findElement(By.tagName("iframe"));
driver.switchTo().frame(frameEl);

// Do work inside the frame
driver.findElement(By.id("inner-button")).click();

// Switch back
driver.switchTo().parentFrame();   // Up one level
driver.switchTo().defaultContent(); // Back to main page
```

---

## 9. Windows & Tabs

```java
import java.util.Set;

// Save current window handle
String mainWindow = driver.getWindowHandle();

// Open new tab/window (Selenium 4)
driver.switchTo().newWindow(WindowType.TAB);
driver.switchTo().newWindow(WindowType.WINDOW);
driver.get("https://example.com");

// Get all handles
Set<String> allWindows = driver.getWindowHandles();

// Switch between windows
for (String handle : allWindows) {
    if (!handle.equals(mainWindow)) {
        driver.switchTo().window(handle);
        break;
    }
}

// Close current and return to main
driver.close();
driver.switchTo().window(mainWindow);
```

---

## 10. Mouse & Keyboard Actions

### Actions class

```java
import org.openqa.selenium.interactions.Actions;
import org.openqa.selenium.Keys;

Actions actions = new Actions(driver);

// Hover
WebElement menu = driver.findElement(By.id("menu"));
actions.moveToElement(menu).perform();

// Right-click
actions.contextClick(element).perform();

// Double-click
actions.doubleClick(element).perform();

// Drag and drop
WebElement source = driver.findElement(By.id("source"));
WebElement target = driver.findElement(By.id("target"));
actions.dragAndDrop(source, target).perform();

// Or with offset
actions.clickAndHold(source)
       .moveByOffset(100, 50)
       .release()
       .perform();

// Chained actions
actions.click(field)
       .keyDown(Keys.SHIFT)
       .sendKeys("hello")
       .keyUp(Keys.SHIFT)
       .perform();
```

### Keyboard shortcuts

```java
element.sendKeys(Keys.ENTER);
element.sendKeys(Keys.TAB);
element.sendKeys(Keys.chord(Keys.CONTROL, "a"));   // Select all
element.sendKeys(Keys.chord(Keys.CONTROL, "c"));   // Copy
element.sendKeys(Keys.ARROW_DOWN);
element.sendKeys(Keys.ESCAPE);
```

---

## 11. Cookies & Sessions

```java
import org.openqa.selenium.Cookie;
import java.util.Set;

// Add a cookie
driver.manage().addCookie(new Cookie("session_id", "abc123"));

// Get cookies
Cookie cookie = driver.manage().getCookieNamed("session_id");
Set<Cookie> allCookies = driver.manage().getCookies();

// Delete
driver.manage().deleteCookieNamed("session_id");
driver.manage().deleteAllCookies();

// Reuse session — save cookies to file
import java.io.*;

try (FileOutputStream fos = new FileOutputStream("cookies.ser");
     ObjectOutputStream oos = new ObjectOutputStream(fos)) {
    oos.writeObject(driver.manage().getCookies());
}

// Later, load and add them back
try (FileInputStream fis = new FileInputStream("cookies.ser");
     ObjectInputStream ois = new ObjectInputStream(fis)) {
    Set<Cookie> cookies = (Set<Cookie>) ois.readObject();
    for (Cookie c : cookies) driver.manage().addCookie(c);
}
driver.navigate().refresh();
```

---

## 12. Screenshots

```java
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.apache.commons.io.FileUtils;
import java.io.File;

// Full page
File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(src, new File("page.png"));

// As bytes / base64
byte[] bytes = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
String b64 = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BASE64);

// Specific element (Selenium 4)
WebElement element = driver.findElement(By.id("logo"));
File elFile = element.getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(elFile, new File("logo.png"));
```

---

## 13. File Upload & Download

```java
// Upload — send the absolute path to a file input
WebElement upload = driver.findElement(By.cssSelector("input[type='file']"));
upload.sendKeys("/absolute/path/to/file.pdf");
```

### Download — set Chrome preferences

```java
import org.openqa.selenium.chrome.ChromeOptions;
import java.util.HashMap;
import java.util.Map;

ChromeOptions options = new ChromeOptions();
Map<String, Object> prefs = new HashMap<>();
prefs.put("download.default_directory", "/path/to/downloads");
prefs.put("download.prompt_for_download", false);
prefs.put("safebrowsing.enabled", true);
options.setExperimentalOption("prefs", prefs);

WebDriver driver = new ChromeDriver(options);
```

---

## 14. JavaScript Executor

Useful when Selenium's native APIs aren't enough.

```java
import org.openqa.selenium.JavascriptExecutor;

JavascriptExecutor js = (JavascriptExecutor) driver;

// Scroll
js.executeScript("window.scrollTo(0, document.body.scrollHeight);");
js.executeScript("arguments[0].scrollIntoView(true);", element);

// Click via JS (bypasses overlay issues)
js.executeScript("arguments[0].click();", element);

// Set value
js.executeScript("arguments[0].value='hello';", inputBox);

// Get value back
String title = (String) js.executeScript("return document.title;");

// Highlight an element (debugging)
js.executeScript("arguments[0].style.border='3px solid red';", element);

// Open new tab
js.executeScript("window.open('https://example.com', '_blank');");
```

---

## 15. Browser Options & Headless Mode

### Chrome

```java
import org.openqa.selenium.chrome.ChromeOptions;

ChromeOptions options = new ChromeOptions();
options.addArguments("--headless=new");           // Headless (Chrome 109+)
options.addArguments("--window-size=1920,1080");
options.addArguments("--disable-gpu");
options.addArguments("--no-sandbox");
options.addArguments("--disable-dev-shm-usage");
options.addArguments("--incognito");
options.addArguments("--user-agent=Mozilla/5.0 ...");
options.addArguments("--proxy-server=http://proxy:8080");

// Disable images for speed
Map<String, Object> prefs = new HashMap<>();
prefs.put("profile.managed_default_content_settings.images", 2);
options.setExperimentalOption("prefs", prefs);

// Hide automation banner
options.setExperimentalOption("excludeSwitches",
        new String[]{"enable-automation"});
options.setExperimentalOption("useAutomationExtension", false);

WebDriver driver = new ChromeDriver(options);
```

### Firefox

```java
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxOptions;

FirefoxOptions ffOptions = new FirefoxOptions();
ffOptions.addArguments("--headless");
WebDriver driver = new FirefoxDriver(ffOptions);
```

### Edge

```java
import org.openqa.selenium.edge.EdgeDriver;
import org.openqa.selenium.edge.EdgeOptions;

EdgeOptions edgeOptions = new EdgeOptions();
edgeOptions.addArguments("--headless=new");
WebDriver driver = new EdgeDriver(edgeOptions);
```

---

## 16. Page Object Model (POM)

POM separates page structure from test logic — improves reusability and maintenance.

### Base Page

```java
// pages/BasePage.java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    protected WebElement find(By locator) {
        return wait.until(ExpectedConditions.presenceOfElementLocated(locator));
    }

    protected void click(By locator) {
        wait.until(ExpectedConditions.elementToBeClickable(locator)).click();
    }

    protected void type(By locator, String text) {
        WebElement el = find(locator);
        el.clear();
        el.sendKeys(text);
    }

    protected String getText(By locator) {
        return find(locator).getText();
    }
}
```

### Login Page

```java
// pages/LoginPage.java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class LoginPage extends BasePage {
    public static final String URL = "https://example.com/login";

    private final By username = By.id("username");
    private final By password = By.id("password");
    private final By loginBtn = By.id("loginBtn");
    private final By errorMsg = By.className("error");

    public LoginPage(WebDriver driver) {
        super(driver);
    }

    public LoginPage load() {
        driver.get(URL);
        return this;
    }

    public void login(String user, String pass) {
        type(username, user);
        type(password, pass);
        click(loginBtn);
    }

    public String errorMessage() {
        return getText(errorMsg);
    }
}
```

### Page Factory variant

`@FindBy` annotations let you initialize elements declaratively.

```java
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

public class LoginPageFactory {
    @FindBy(id = "username") private WebElement username;
    @FindBy(id = "password") private WebElement password;
    @FindBy(id = "loginBtn") private WebElement loginBtn;

    public LoginPageFactory(WebDriver driver) {
        PageFactory.initElements(driver, this);
    }

    public void login(String u, String p) {
        username.sendKeys(u);
        password.sendKeys(p);
        loginBtn.click();
    }
}
```

### Test Using POM

```java
// tests/LoginTest.java
import org.testng.annotations.*;
import org.testng.Assert;
import pages.LoginPage;

public class LoginTest extends BaseTest {

    @Test
    public void invalidLoginShowsError() {
        LoginPage page = new LoginPage(driver).load();
        page.login("wrong", "creds");
        Assert.assertTrue(page.errorMessage().contains("Invalid"));
    }
}
```

---

## 17. Data-Driven Testing with TestNG

TestNG's `@DataProvider` lets you run the same test with multiple inputs.

```java
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

public class DataDrivenLoginTest extends BaseTest {

    @DataProvider(name = "loginData")
    public Object[][] loginData() {
        return new Object[][] {
            {"user1@test.com", "Pass123", true},
            {"invalid@test.com", "wrong", false},
            {"", "", false},
        };
    }

    @Test(dataProvider = "loginData")
    public void testLogin(String email, String pwd, boolean expected) {
        LoginPage page = new LoginPage(driver).load();
        page.login(email, pwd);
        Assert.assertEquals(page.isLoggedIn(), expected);
    }
}
```

### Reading from CSV

```java
import java.nio.file.*;
import java.util.*;

@DataProvider(name = "csvData")
public Object[][] csvData() throws Exception {
    List<String> lines = Files.readAllLines(Paths.get("data/users.csv"));
    Object[][] data = new Object[lines.size() - 1][];
    for (int i = 1; i < lines.size(); i++) {
        data[i - 1] = lines.get(i).split(",");
    }
    return data;
}
```

### Reading from Excel (Apache POI)

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.3.0</version>
</dependency>
```

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileInputStream;

@DataProvider(name = "excelData")
public Object[][] excelData() throws Exception {
    FileInputStream fis = new FileInputStream("data/users.xlsx");
    Workbook wb = new XSSFWorkbook(fis);
    Sheet sheet = wb.getSheetAt(0);

    int rows = sheet.getLastRowNum();
    int cols = sheet.getRow(0).getLastCellNum();
    Object[][] data = new Object[rows][cols];

    for (int i = 1; i <= rows; i++) {
        for (int j = 0; j < cols; j++) {
            data[i - 1][j] = sheet.getRow(i).getCell(j).toString();
        }
    }
    wb.close();
    return data;
}
```

---

## 18. TestNG Integration

### Base Test class with setup/teardown

```java
// tests/BaseTest.java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.*;
import java.time.Duration;

public class BaseTest {
    protected WebDriver driver;

    @BeforeMethod
    public void setUp() {
        driver = new ChromeDriver();
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(5));
    }

    @AfterMethod
    public void tearDown() {
        if (driver != null) driver.quit();
    }
}
```

### TestNG annotations reference

| Annotation | When it runs |
|---|---|
| `@BeforeSuite` / `@AfterSuite` | Once before/after the entire suite |
| `@BeforeTest` / `@AfterTest` | Once before/after each `<test>` in testng.xml |
| `@BeforeClass` / `@AfterClass` | Once before/after the test class |
| `@BeforeMethod` / `@AfterMethod` | Before/after each `@Test` method |
| `@Test` | Marks a test method |
| `@DataProvider` | Supplies test data |

### Screenshot on failure (TestNG listener)

```java
import org.testng.ITestListener;
import org.testng.ITestResult;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.apache.commons.io.FileUtils;
import java.io.File;
import java.lang.reflect.Field;

public class TestListener implements ITestListener {
    @Override
    public void onTestFailure(ITestResult result) {
        try {
            Object instance = result.getInstance();
            Field field = instance.getClass().getSuperclass()
                                 .getDeclaredField("driver");
            field.setAccessible(true);
            WebDriver driver = (WebDriver) field.get(instance);

            File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
            FileUtils.copyFile(src,
                new File("screenshots/" + result.getName() + ".png"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

Register it in your test class:
```java
@Listeners(TestListener.class)
public class LoginTest extends BaseTest { ... }
```

### testng.xml — running suites

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Smoke Suite" parallel="methods" thread-count="3">
    <test name="Login Tests">
        <classes>
            <class name="tests.LoginTest"/>
            <class name="tests.DataDrivenLoginTest"/>
        </classes>
    </test>
</suite>
```

Run via Maven:
```bash
mvn test -DsuiteXmlFile=testng.xml
```

---

## 19. Selenium Grid

Run tests on remote machines or in parallel across browsers.

### Start the Grid

```bash
# Start hub
java -jar selenium-server.jar hub

# Start node
java -jar selenium-server.jar node --hub http://localhost:4444
```

### Connect with RemoteWebDriver

```java
import org.openqa.selenium.remote.RemoteWebDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import java.net.URL;

ChromeOptions options = new ChromeOptions();
WebDriver driver = new RemoteWebDriver(
    new URL("http://localhost:4444/wd/hub"),
    options
);
```

### Docker Selenium Grid

```bash
docker run -d -p 4444:4444 --name selenium-hub selenium/hub
docker run -d --link selenium-hub:hub selenium/node-chrome
docker run -d --link selenium-hub:hub selenium/node-firefox
```

---

## 20. Shadow DOM

Modern web components hide their DOM. Selenium 4 has built-in support.

```java
import org.openqa.selenium.SearchContext;

WebElement host = driver.findElement(By.cssSelector("my-component"));
SearchContext shadowRoot = host.getShadowRoot();

// Find inside shadow root
WebElement inner = shadowRoot.findElement(By.cssSelector(".inner-button"));
inner.click();
```

For nested shadow roots, use JavaScript:

```java
JavascriptExecutor js = (JavascriptExecutor) driver;
WebElement inner = (WebElement) js.executeScript(
    "return arguments[0].shadowRoot.querySelector('.inner')", host);
```

---

## 21. Best Practices

1. **Use explicit waits** instead of `Thread.sleep()`. Tests run faster and more reliably.
2. **Prefer ID and CSS selectors** over XPath when possible — they're faster and more readable.
3. **Use Page Object Model** for maintainable test suites.
4. **Always quit the driver** in `@AfterMethod` to avoid memory leaks.
5. **Keep locators in one place** (page classes) — never hardcode them in tests.
6. **Use unique, stable locators** — avoid auto-generated IDs and brittle XPaths like `/html/body/div[3]/...`.
7. **Run headless in CI** for speed, but test in headed mode locally for debugging.
8. **Take screenshots on failure** via TestNG listeners.
9. **Parallelize tests** with TestNG (`parallel="methods"`) or Selenium Grid.
10. **Don't mix implicit and explicit waits** — pick one strategy.
11. **Isolate tests** — each test should be independent and clean up after itself.
12. **Use config files** (`config.properties`) for URLs, credentials — never hardcode.
13. **Add retry logic** for flaky tests using `IRetryAnalyzer`.
14. **Log meaningful info** — use Log4j or SLF4J instead of `System.out.println`.
15. **Keep tests small and focused** — one logical assertion per test where practical.

### Retry analyzer example

```java
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class Retry implements IRetryAnalyzer {
    private int count = 0;
    private static final int MAX_RETRY = 2;

    @Override
    public boolean retry(ITestResult result) {
        if (count < MAX_RETRY) {
            count++;
            return true;
        }
        return false;
    }
}
```
Use it: `@Test(retryAnalyzer = Retry.class)`

---

## Common Exceptions Reference

| Exception | When |
|---|---|
| `NoSuchElementException` | Element not found in DOM |
| `TimeoutException` | Wait condition not met in time |
| `StaleElementReferenceException` | Element reference is no longer valid (DOM changed) |
| `ElementClickInterceptedException` | Another element is covering the target |
| `ElementNotInteractableException` | Element exists but can't be interacted with |
| `NoSuchFrameException` | Frame doesn't exist |
| `NoAlertPresentException` | No alert to switch to |
| `WebDriverException` | Generic driver-level error |
| `SessionNotCreatedException` | Driver/browser version mismatch |

```java
import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.TimeoutException;

try {
    driver.findElement(By.id("missing"));
} catch (NoSuchElementException e) {
    System.out.println("Element not found");
}
```

---

## Quick Reference: Complete Test Example

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class SauceDemoLoginTest {
    public static void main(String[] args) {
        WebDriver driver = new ChromeDriver();
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

        try {
            driver.get("https://www.saucedemo.com/");
            driver.findElement(By.id("user-name")).sendKeys("standard_user");
            driver.findElement(By.id("password")).sendKeys("secret_sauce");
            driver.findElement(By.id("login-button")).click();

            wait.until(ExpectedConditions.urlContains("/inventory"));
            String title = driver.findElement(By.className("title")).getText();
            assert title.equals("Products") : "Expected 'Products' but got " + title;
            System.out.println("✅ Login test passed");
        } finally {
            driver.quit();
        }
    }
}
```

---

## Project Structure (Recommended)

```
selenium-framework/
├── pom.xml
├── testng.xml
├── src/
│   ├── main/java/
│   │   ├── pages/         (Page Object classes)
│   │   ├── utils/         (Helpers, config readers, listeners)
│   │   └── base/          (BasePage, BaseTest)
│   └── test/java/
│       └── tests/         (Test classes)
├── src/test/resources/
│   ├── config.properties
│   └── testdata/
│       └── users.xlsx
└── screenshots/
```

---

*Happy automating! Selenium docs: https://www.selenium.dev/documentation/*
