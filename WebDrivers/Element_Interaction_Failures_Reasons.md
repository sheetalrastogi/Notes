# WebElement Interaction Conditions That Can Trick Selenium 4 Automation Scripts

**Element Availability Issues**
-----------------------------
- Element not present in DOM
- Element dynamically added after page load
- Element removed and recreated
- Incorrect iframe context
- Incorrect Shadow DOM context
- Element located in another browser window/tab
- Lazy-loaded element not yet rendered
- Virtualized grid row not yet loaded

**Visibility Issues**
-------------------
- Element hidden via CSS (display:none)
- Element hidden via CSS (visibility:hidden)
- Element opacity set to zero
- Element outside viewport
- Element inside collapsed panel
- Element inside inactive tab section
- Element hidden behind animation
- Element hidden by responsive layout changes

**Clickability Issues**
------------------------
- Element covered by loading spinner
- Element covered by overlay
- Element covered by modal dialog
- Element covered by toast message
- Element covered by sticky header/footer
- Element disabled (disabled=true)
- Element becomes disabled after page update
- Element not yet clickable after rendering

**Scrolling Issues**
--------------------
- Element below fold
- Element horizontally off-screen
- Nested scroll container not scrolled
- Infinite scrolling content
- Incorrect scroll position
- Auto-scroll moves element unexpectedly

**DOM Change Issues**
----------------------
- StaleElementReferenceException
- DOM re-render after Ajax call
- React component re-render
- Angular change detection refresh
- Vue component refresh
- AJAX partial page refresh
- Single Page Application (SPA) route update

**Timing and Synchronization Issues**
---------------------------------------
- AJAX request still running
- JavaScript execution incomplete
- Page load incomplete
- API response pending
- Delayed rendering of component
- Background polling updates DOM
- Race condition between actions

**Locator Issues**
--------------------
- Dynamic IDs
- Dynamic CSS classes
- Duplicate locators
- Incorrect XPath
- Incorrect CSS selector
- Relative locator ambiguity
- Index-based locator instability

**Browser and Rendering Issues**
----------------------------------
- Browser zoom level changed
- Responsive layout breakpoint change
- Browser resize event refreshes DOM
- GPU rendering delay
- Browser-specific rendering differences
- Headless versus headed rendering differences

**JavaScript-Related Issues**
-------------------------------
- JavaScript exception on page
- Front-end rendering failure
- Angular digest cycle incomplete
- React hydration delay
- Unhandled promise rejection
- Client-side validation blocking interaction

**Frame and Shadow DOM Issues**
---------------------------------
- Element inside iframe
- Nested iframe handling failure
- Shadow DOM host not available
- Nested Shadow DOM traversal failure
- Dynamic Shadow DOM rendering

**Network Issues**
-------------------
- Slow API response
- API failure causing missing UI elements
- CDN resource loading delay
- Network timeout
- WebSocket updates altering DOM

**Advanced UI Issues**
------------------------
- Canvas-based elements
- SVG-based controls
- Custom dropdowns
- Drag-and-drop widgets
- Virtual scrolling grids
- Dynamic tables
- Interactive charts
- Floating action menus

**State Management Issues**
-----------------------------
- Session expired
- Authentication token expired
- User permissions changed
- Feature flag changed UI structure
- Application cache inconsistency

**CDP DOM Inspection Detection Cases**
-----------------------------------------
- Element exists in DOM but not visible
- Element exists outside viewport
- Element exists inside virtualized grid
- Element exists but covered by overlay
- Element exists but coordinate position changed
- Element exists but Selenium locator resolves stale reference
- Element exists but interaction blocked by frontend state
- Element found by CDP but not interactable by Selenium
- Element rendered after API response but before viewport update
- Element present in DOM tree but not rendered on screen

# Common Selenium Exceptions Triggered
-----------------------------------------
- NoSuchElementException
- StaleElementReferenceException
- ElementNotInteractableException
- ElementClickInterceptedException
- TimeoutException
- MoveTargetOutOfBoundsException
- NoSuchFrameException
- NoSuchWindowException
- JavascriptException
- InvalidElementStateException
- UnexpectedAlertPresentException
- WebDriverException
