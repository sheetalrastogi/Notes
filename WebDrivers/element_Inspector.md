
**Visual Element Inspector**, similar to Chrome DevTools, where hovering over any element displays a floating AJAX-style diagnostic panel containing all WebElement health indicators.

This is extremely useful for:

- Selenium framework debugging
- Self-healing locator analysis
- Root-cause failure classification
- UI health validation
- Framework development and maintenance
- AI-assisted element interaction decisions


# Utility workflow

```text
User Hover
    │
    ▼
Selenium Actions.moveToElement()
    │
    ▼
Inject Element Inspector Overlay
    │
    ▼
Collect Element Diagnostics
    │
    ├── Visibility
    ├── Clickability
    ├── Viewport
    ├── DOM State
    ├── CSS State
    ├── Frame State
    ├── Shadow Root State
    ├── Ajax Status
    ├── Network Status
    ├── Rendering Health
    └── CDP Information
    │
    ▼
Display Floating Inspector Popup
```

# Sample Inspector Popup

```text
──────────────────────────────
ELEMENT INSPECTOR
──────────────────────────────

Tag          : button
Id           : submitBtn
Class        : btn primary

State        : READY

Display      : block
Visibility   : visible
Opacity      : 1

Width        : 120
Height       : 40

Enabled      : true
ReadOnly     : false

Viewport     : true
Overlapped   : false

Frame        : Main Document
Shadow DOM   : No

Ajax Active  : false

Page State   : complete

Network      : OK

Renderer     : Ready

Suggested Action:
CLICK

──────────────────────────────
```


# Selenium Utility


```java
	public void showElementInspector(WebElement element) {

		JavascriptExecutor js = (JavascriptExecutor) driver;

		js.executeScript("""
				const el = arguments[0];
				const old = document.getElementById('seleniumInspector');

				if(old){
				    old.remove();
				}

				const style = window.getComputedStyle(el);

				const rect = el.getBoundingClientRect();

				const over = document.elementFromPoint(
				                rect.left + rect.width / 2,
				                rect.top + rect.height / 2);

				const popup = document.createElement('div');

				popup.id = 'seleniumInspector';

				popup.style.position='fixed';
				popup.style.zIndex='999999';
				popup.style.background='#1e1e1e';
				popup.style.color='#00ff00';
				popup.style.padding='10px';
				popup.style.border='1px solid #00ff00';
				popup.style.borderRadius='5px';
				popup.style.fontSize='12px';
				popup.style.fontFamily='Consolas';
				popup.style.maxWidth='400px';

				popup.style.left= (rect.right + 10) + 'px';

				popup.style.top= rect.top + 'px';

				popup.innerHTML = `
				        <b>ELEMENT INSPECTOR</b><hr>

				        Tag :
				        ${el.tagName}<br>

				        Id :
				        ${el.id}<br>

				        Class :
				        ${el.className}<br><br>

				        Display :
				        ${style.display}<br>

				        Visibility :
				        ${style.visibility}<br>

				        Opacity :
				        ${style.opacity}<br>

				        Width :
				        ${rect.width}<br>

				        Height :
				        ${rect.height}<br>

				        Enabled :
				        ${!el.disabled}<br>

				        ReadOnly :
				        ${el.readOnly || false}<br>

				        Pointer Events :
				        ${style.pointerEvents}<br>

				        In Viewport :
				        ${
				            rect.top >= 0 &&
				            rect.left >= 0 &&
				            rect.bottom <= window.innerHeight
				        }<br>

				        Overlapped :
				        ${
				            over !== el &&
				            !el.contains(over)
				        }
				        `;

				document.body.appendChild(popup);

				el.style.outline = '3px solid red';
				
				""", element);
	}

```

Usage:

Hover over an element and Display Inspector

```java
WebElement element = driver.findElement(By.id("submitBtn"));

new Actions(driver)
        .moveToElement(element)
        .perform();

showElementInspector(element);

```

Auto Display Inspector On Hover

Inject once per page:

```java
public void enableInspectorMode() {

    JavascriptExecutor js =
            (JavascriptExecutor) driver;

    js.executeScript(
            """
            if(window.inspectorEnabled){
                return;
            }

            window.inspectorEnabled = true;

            document.addEventListener(
                'mouseover',
                function(event){

                    const e = event.target;

                    const old =
                        document.getElementById(
                            'seleniumInspector');

                    if(old){
                        old.remove();
                    }

                    const s =
                        window.getComputedStyle(e);

                    const r =
                        e.getBoundingClientRect();

                    const panel =
                        document.createElement('div');

                    panel.id =
                        'seleniumInspector';

                    panel.style.position='fixed';
                    panel.style.top='20px';
                    panel.style.right='20px';
                    panel.style.zIndex='999999';

                    panel.style.background='#000';
                    panel.style.color='#00ff00';

                    panel.style.padding='15px';

                    panel.innerHTML =
                        `
                        <b>ELEMENT INSPECTOR</b><hr>

                        Tag :
                        ${e.tagName}<br>

                        Display :
                        ${s.display}<br>

                        Visibility :
                        ${s.visibility}<br>

                        Opacity :
                        ${s.opacity}<br>

                        Width :
                        ${r.width}<br>

                        Height :
                        ${r.height}
                        `;

                    document.body.appendChild(
                        panel);

                });
            """);
}

```

# 1. Manual Debugging During Script Development

When developing a locator or troubleshooting a flaky element:

```java
WebElement loginBtn = driver.findElement(By.id("loginBtn"));
new Actions(driver)
        .moveToElement(loginBtn)
        .pause(Duration.ofSeconds(2))
        .perform();
showElementInspector(loginBtn);
```

Use Cases:

✓ Verify CSS visibility
✓ Verify element dimensions
✓ Check overlay issues
✓ Check viewport issues
✓ Validate locator accuracy



