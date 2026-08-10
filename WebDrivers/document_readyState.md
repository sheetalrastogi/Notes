
## Wait for Page load to complete 
---------

Generally following checks are mandatory to validate "Ajax Page load to complete" before interacting with WebElements:

 - JavaScript execution is enabled and working
 - document.readyState = complete
 - AJAX requests are completed
 - jQuery requests are completed (if jQuery exists)
 - Angular requests are completed (if Angular exists)
 - Fetch/XHR requests are completed (modern SPA applications)
 - Page does not have pending network activity


# Step 1:  Inject JavaScript to monitor all XHR and Fetch requests 

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

# Step 2: 
```java
	private void waitForAjaxAndPageLoad() {

		injectAjaxTracker();

		WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));

		JavascriptExecutor js = (JavascriptExecutor) driver;

		// Verify JavaScript execution is working
		try {
			Object jsEnabled = js.executeScript("return true;");
			if (!(jsEnabled instanceof Boolean) || !((Boolean) jsEnabled)) {
				throw new RuntimeException("JavaScript execution is not available in browser.");
			}
		} catch (Exception e) {
			throw new RuntimeException("JavaScript is disabled or execution failed.", e);
		}

		// Wait for complete page load
		wait.until(webDriver -> "complete"
				.equals(((JavascriptExecutor) webDriver).executeScript("return document.readyState")));

		// Wait for jQuery AJAX requests
		try {
			wait.until(webDriver -> (Boolean) ((JavascriptExecutor) webDriver)
					.executeScript("return window.jQuery == undefined || jQuery.active === 0"));
		} catch (Exception ignored) {
			// Non-jQuery application
		}

		// Wait for Angular requests
		try {
			wait.until(webDriver -> (Boolean) ((JavascriptExecutor) webDriver)
					.executeScript("return (window.getAllAngularTestabilities)"
							+ " ? window.getAllAngularTestabilities()" + ".every(x => x.isStable()) : true"));
		} catch (Exception ignored) {
			// Non-Angular application
		}

		// Wait for Fetch/XHR requests (if application exposes tracker)
		try {
			wait.until(webDriver -> (Boolean) ((JavascriptExecutor) webDriver).executeScript(
					"return window.pendingAjaxRequests === undefined " + "|| window.pendingAjaxRequests === 0"));
		} catch (Exception ignored) {
		}
	}

```

This combination is significantly more reliable than relying solely on document.readyState == complete, because modern applications often continue making asynchronous API calls long after the page load event has finished.


