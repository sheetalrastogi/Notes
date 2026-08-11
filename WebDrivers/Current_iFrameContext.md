Goal is to **verify whether Selenium is currently on the main page or inside an iframe**, and if they differ then print the complete frame hierarchy, you can use the following utility.

Utility Method
```java
public void validateFrameContext() {
    JavascriptExecutor js = (JavascriptExecutor) driver;
    try {
        String currentTitle = String.valueOf(js.executeScript("return document.title"));
        String topWindowTitle = String.valueOf(js.executeScript("return window.top.document.title"));

        if (currentTitle.equals(topWindowTitle)) {
            System.out.println("Selenium is currently on Main Page.");
        } else {
            System.out.println("Selenium is inside an IFrame.");
            printFrameHierarchy();
        }
    } catch (Exception e) {

        System.out.println("Unable to compare page titles. Possible cross-origin iframe.");
        printFrameHierarchy();
    }
}
```
**Print Current Frame Hierarchy**

```java
public void printFrameHierarchy() {

    JavascriptExecutor js = (JavascriptExecutor) driver;
    try {
        String hierarchy = String.valueOf(js.executeScript(
                        "var path=[];" +
                        "var win=window;" +
                        "while(win !== win.top) {" +
                        "  var parent=win.parent;" +
                        "  var frames=parent.frames;" +
                        "  for(var i=0;i<frames.length;i++){" +
                        "     if(frames[i]===win){" +
                        "        path.unshift('iframe['+i+']');" +
                        "        break;" +
                        "     }" +
                        "  }" +
                        "  win=parent;" +
                        "}" +
                        "return path.join(' -> ');"

                ));

        System.out.println("Frame Hierarchy : " + (hierarchy.isEmpty() ? "Main Page" : hierarchy));
    } catch (Exception e) {
        System.out.println("Unable to determine frame hierarchy.");
    }
}
```

**Example Outputs**

```text
Case 1: Main Page
Current Title : Customer Portal
Top Page Title : Customer Portal

Selenium is currently on Main Page.

Case 2: Single iframe
Current Title : Payment Gateway
Top Page Title : Customer Portal

Selenium is inside an IFrame.

Frame Hierarchy : iframe[0]

Case 3: Nested iframe
Current Title : Card Entry
Top Page Title : Customer Portal

Selenium is inside an IFrame.

Frame Hierarchy : iframe[0] -> iframe[2]
```
