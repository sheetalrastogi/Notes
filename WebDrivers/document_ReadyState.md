## Code Explanation

```java
	private void injectAjaxTracker() {

		JavascriptExecutor js = (JavascriptExecutor) driver;

		js.executeScript("if (!window.ajaxMonitorInstalled) {" + "window.ajaxMonitorInstalled = true;"
				+ "window.pendingAjaxRequests = 0;" + "" + "let oldOpen = XMLHttpRequest.prototype.open;"
				+ "let oldSend = XMLHttpRequest.prototype.send;" + "" + "XMLHttpRequest.prototype.open = function() {"
				+ "    this.addEventListener('loadstart', function(){" + "        window.pendingAjaxRequests++;"
				+ "    });" + "    this.addEventListener('loadend', function(){"
				+ "        window.pendingAjaxRequests--;" + "    });" + "    oldOpen.apply(this, arguments);" + "};"
				+ "" + "let oldFetch = window.fetch;" + "window.fetch = function() {"
				+ "    window.pendingAjaxRequests++;" + "    return oldFetch.apply(this, arguments)"
				+ "        .finally(function(){" + "            window.pendingAjaxRequests--;" + "        });" + "};"
				+ "}");
	}

```

This code injects a JavaScript monitoring mechanism into the browser that tracks all AJAX/XHR and Fetch API requests. Selenium can then wait until all asynchronous network calls have completed before proceeding with the next test step.

# High-Level Flow

1. Inject tracker into browser
2. Intercept every AJAX/XHR request
3. Intercept every Fetch request
4. Maintain a counter:
      pendingAjaxRequests
5. Increment when request starts
6. Decrement when request finishes
7. Selenium waits until counter becomes 0

# Step 1: Get JavaScript Executor
```java
JavascriptExecutor js = (JavascriptExecutor) driver;
```
Purpose
- Allows Selenium to execute JavaScript inside the currently loaded page.
- Equivalent to opening browser DevTools Console and running JavaScript manually.

# Step 2: Prevent Multiple Installation
```java
if (!window.ajaxMonitorInstalled) {
```

Purpose
- Checks whether the tracker has already been injected.

**Without this check**:
```text
First call  -> installs tracker
Second call -> installs tracker again
Third call  -> installs tracker again
```

**Multiple installations would cause**:
```text
Counter incrementing multiple times
Incorrect request counts
Memory leaks
```

# Step 3: Mark Tracker Installed
```java
window.ajaxMonitorInstalled = true;
```

**Creates a global flag**:
```java
window.ajaxMonitorInstalled
```

**After first execution**:
```java
window.ajaxMonitorInstalled == true
```

Future calls skip reinjection.

# Step 4: Create Request Counter
```java
window.pendingAjaxRequests = 0;
```

**Global counter**.
```text
Example:

No requests running  = 0
1 request running    = 1
3 requests running   = 3
```

Selenium later waits for:
```java
window.pendingAjaxRequests == 0
```

# Step 5: Save Original XHR Methods
```java
let oldOpen = XMLHttpRequest.prototype.open;
let oldSend = XMLHttpRequest.prototype.send;
```

**What is XMLHttpRequest?**

Traditional AJAX requests:
```java
var xhr = new XMLHttpRequest();
xhr.open("GET", "/users");
xhr.send();
```

**Browser internally uses**:
```java
XMLHttpRequest.prototype.open()
XMLHttpRequest.prototype.send()
```

We save originals so they can still execute normally.

# Step 6: Override XMLHttpRequest.open()
```java
XMLHttpRequest.prototype.open = function() {
```

This is called every time an AJAX request is created.

Original flow:
```java
xhr.open(...)
```

Now becomes:

our custom function, which adds tracking logic.


# Step 7: Monitor Request Start
```java
this.addEventListener('loadstart', function() {
    window.pendingAjaxRequests++;
});
```

loadstart occurs when request begins.

Example:
```text
Request A starts
Counter = 1

Request B starts
Counter = 2

Request C starts
Counter = 3
```

# Step 8: Monitor Request End
```java
this.addEventListener('loadend', function() {
    window.pendingAjaxRequests--;
});
```


loadend fires when request completes regardless of:
```text
Success
Failure
404
500
Timeout
```

Example:
```text
3 active requests

Request A completes
Counter = 2

Request B completes
Counter = 1

Request C completes
Counter = 0
```

# Step 9: Call Original open()
```java
oldOpen.apply(this, arguments);
```

Without this line:
```text
Tracking works
Actual AJAX request never happens
Application breaks
```

This executes the browser's original implementation.

Example:
```java
xhr.open("GET", "/employees");
```

still works normally.

# Step 10: Save Original Fetch Method
```java
let oldFetch = window.fetch;
```

Modern frameworks like:
```text
React
Angular
Vue
NextJS
Micro-frontends
```

usually use:

  - fetch(...)

  - instead of XMLHttpRequest.

Example:

fetch("/employees")

# Step 11: Override Fetch
```java
window.fetch = function() {
```

Intercepts every fetch request.

Original:

fetch(...)

Now becomes:

custom tracking code

# Step 12: Increment Counter
```java
window.pendingAjaxRequests++;
```

When fetch starts:

Counter = Counter + 1


Example:
```text
Counter = 0

fetch starts

Counter = 1
```

# Step 13: Execute Real Fetch
```text
return oldFetch.apply(this, arguments)


Calls actual browser fetch implementation.

Equivalent to:

return fetch(...)
```

but using the saved original version.

# Step 14: Decrement Counter
```java
.finally(function() {
    window.pendingAjaxRequests--;
});
```

When the request completes:
```text
Success
Failure
Exception
```

counter decreases.

Example:
```text
Request started
Counter = 1

Request finished successfully
Counter = 0


or

Request failed
Counter = 0
```

**Example Execution**

```text
Application loads.

API Call #1
GET /users

Counter:  0 -> 1

API Call #2
GET /roles

Counter: 1 -> 2

API Call #3
GET /permissions

Counter: 2 -> 3

APIs Complete
/users completed
3 -> 2

/roles completed
2 -> 1

/permissions completed
1 -> 0
```

Now Selenium can continue:
```java
wait.until(driver ->
    Long.valueOf(0).equals(
        ((JavascriptExecutor) driver)
            .executeScript(
                "return window.pendingAjaxRequests")));
```

Why This Is Useful

Consider a React application:

Page loads
document.readyState = complete


But immediately after:

GET /users
GET /accounts
GET /roles
GET /permissions


start running in the background.

If Selenium clicks too early:

NoSuchElementException
ElementNotInteractableException
StaleElementReferenceException


may occur.

This tracker ensures:

```text
Page loaded
+
All API calls completed
+
UI fully rendered
```

before test execution continues.

**One Limitation**

If the tracker is injected after the page has already started making requests, those earlier requests will not be counted.

Therefore the best practice is:
```text
injectAjaxTracker();

driver.get(url);

waitForAjaxAndPageLoad();
```

or inject immediately after navigation and before interacting with the page.
