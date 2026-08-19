## Sample HTML Page to test XPath Functions and Axeses

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>XPath Functions and Axes Practice Page</title>

<style>
    body {
        font-family: Arial, sans-serif;
        margin: 20px;
    }

    section {
        margin-bottom: 30px;
        padding: 15px;
        border: 1px solid #ccc;
        border-radius: 6px;
    }

    h2 {
        color: darkgreen;
    }

    .card {
        border: 1px solid #999;
        padding: 10px;
        margin: 10px 0;
    }

    .card featured {
        background-color: #f3f3f3;
    }

    table {
        border-collapse: collapse;
        width: 100%;
    }

    table, th, td {
        border: 1px solid black;
    }

    th, td {
        padding: 8px;
    }
</style>

</head>
<body>

<h1>XPath Functions & Axes Demo Page</h1>

<!-- ================================================= -->
<!-- XPath Functions -->
<!-- ================================================= -->

<section>

<h2>normalize-space()</h2>

<button id="btnNormalize">
      Save Changes
</button>

</section>

<section>

<h2>starts-with()</h2>

<button id="save-12345">Save Record</button>

</section>

<section>

<h2>contains()</h2>

<input id="userNameField" value="john"/>

</section>

<section>

<h2>substring()</h2>

<input id="userDetails"/>

</section>

<section>

<h2>substring-before()</h2>

<input id="user_123"/>

</section>

<section>

<h2>substring-after()</h2>

<input id="user_123"/>

</section>

<section>

<h2>string-length()</h2>

<input id="comments"
       value="This value is longer than five characters"/>

</section>

<section>

<h2>concat()</h2>

<div class="card featured">
    Customer Information
</div>

</section>

<section>

<h2>translate()</h2>

<button>LOGIN</button>

</section>

<section>

<h2>text()</h2>

<button>Login</button>

</section>

<section>

<h2>last()</h2>

<button>Button 1</button>
<button>Button 2</button>
<button>Button 3</button>

</section>

<section>

<h2>position()</h2>

<table id="positionTable">

<tr>
<th>ID</th>
<th>Name</th>
</tr>

<tr>
<td>1</td>
<td>Alice</td>
</tr>

<tr>
<td>2</td>
<td>Bob</td>
</tr>

<tr>
<td>3</td>
<td>Charlie</td>
</tr>

</table>

</section>

<section>

<h2>count()</h2>

<table id="countTable">

<tr><td>1</td></tr>
<tr><td>2</td></tr>
<tr><td>3</td></tr>
<tr><td>4</td></tr>
<tr><td>5</td></tr>
<tr><td>6</td></tr>

</table>

</section>

<section>

<h2>name()</h2>

<input id="nameFunction"/>

</section>

<section>

<h2>local-name()</h2>

<svg height="100" width="100">
    <circle cx="50"
            cy="50"
            r="40"
            stroke="black"
            fill="yellow"/>
</svg>

</section>

<section>

<h2>not()</h2>

<input id="enabledInput"/>

<input id="disabledInput"
       disabled/>

</section>

<section>

<h2>boolean()</h2>

<input type="checkbox"
       checked="checked"/>

</section>

<section>

<h2>true() and false()</h2>

<div id="trueFalseSection">
    Sample Element
</div>

</section>

<section>

<h2>number()</h2>

<table id="numberTable">

<tr>
<td>Product A</td>
<td>150</td>
</tr>

<tr>
<td>Product B</td>
<td>75</td>
</tr>

</table>

</section>

<section>

<h2>sum()</h2>

<table id="sumTable">

<tr><td>10</td></tr>
<tr><td>20</td></tr>
<tr><td>30</td></tr>

</table>

</section>

<section>

<h2>floor / ceiling / round</h2>

<div data-score="10.9">Floor</div>
<div data-score="10.1">Ceiling</div>
<div data-score="10.5">Round</div>

</section>

<!-- ================================================= -->
<!-- AXES PRACTICE -->
<!-- ================================================= -->

<section>

<h2>following-sibling::</h2>

<label>Email</label>
<input id="email"/>

</section>

<section>

<h2>preceding::</h2>

<label>Password</label>
<input id="password"/>

</section>

<section>

<h2>preceding-sibling::</h2>

<label>Username</label>
<input id="usernamePS"/>

</section>

<section>

<h2>following::</h2>

<label>Email</label>

<div>
    Some Content
</div>

<input id="emailFollowing"/>

</section>

<section>

<h2>parent::</h2>

<div id="parentContainer">

<input id="username"/>

</div>

</section>

<section>

<h2>child::</h2>

<div id="container">

<button id="saveBtn">
    Save
</button>

</div>

</section>

<section>

<h2>ancestor::</h2>

<form id="loginForm">

<div>

<input id="usernameAncestor"/>

</div>

</form>

</section>

<section>

<h2>ancestor-or-self::</h2>

<form id="selfForm">

<div>

<input id="usernameSelf"/>

</div>

</form>

</section>

<section>

<h2>descendant::</h2>

<div id="containerDesc">

<form>

<input id="descendantInput"/>

</form>

</div>

</section>

<section>

<h2>descendant-or-self::</h2>

<div id="containerDescSelf">

<form>

<input id="descendantSelfInput"/>

</form>

</div>

</section>

<section>

<h2>self::</h2>

<input id="selfInput"/>

</section>

<section>

<h2>attribute::</h2>

<input id="attributeInput"
       name="attributeDemo"
       value="sample"/>

</section>

</body>
</html>
```
# XPath Functions Cheat Sheet

| Function | Purpose | Example XPath |
|----------|----------|----------|
| `normalize-space()` | Removes leading/trailing spaces and replaces multiple spaces with a single space. | `//button[normalize-space()='Save Changes']` |
| `starts-with()` | Checks whether an attribute starts with a specific value. | `//button[starts-with(@id,'save-')]` |
| `contains()` | Checks whether an attribute or text contains a partial value. | `//input[contains(@id,'user')]` |
| `substring()` | Returns part of a string. | `//input[substring(@id,1,4)='user']` |
| `substring-before()` | Returns the portion before a delimiter. | `//input[substring-before(@id,'_')='user']` |
| `substring-after()` | Returns the portion after a delimiter. | `//input[substring-after(@id,'user_')='123']` |
| `string-length()` | Returns the length of a string. | `//input[string-length(@value) > 5]` |
| `concat()` | Concatenates strings. | `//div[contains(concat(' ',normalize-space(@class),' '), ' card ')]` |
| `translate()` | Converts characters. | `//*[translate(text(),'ABCDEFGHIJKLMNOPQRSTUVWXYZ','abcdefghijklmnopqrstuvwxyz')='login']` |
| `text()` | Returns text content. | `//button[text()='Login']` |
| `last()` | Returns the last node. | `(//button)[last()]` |
| `position()` | Returns node position. | `//tr[position()=1]`<br>`//tr[position()>last()-2]` |
| `count()` | Counts matching nodes. | `//table[count(.//tr)>5]` |
| `name()` | Returns tag name. | `//*[name()='input']` |
| `local-name()` | Returns tag name without namespace. | `//*[local-name()='svg']` |
| `not()` | Negates a condition. | `//input[not(@disabled)]` |
| `boolean()` | Converts value to boolean. | `//*[boolean(@checked)]` |
| `true()` | Returns true. | `//*[true()]` |
| `false()` | Returns false. | `//*[false()]` |
| `number()` | Converts string to number. | `//tr[number(td[3])>100]` |
| `sum()` | Calculates totals. | `sum(//table//td)` |
| `floor()` | Rounds down. | `floor(10.9)` |
| `ceiling()` | Rounds up. | `ceiling(10.1)` |
| `round()` | Standard rounding. | `round(10.5)` |

# XPath Axes Cheat Sheet

| Axis | Purpose | Example XPath |
|--------|----------|----------|
| `following-sibling::` | Selects siblings after the current node. | `//label[text()='Email']/following-sibling::input` |
| `preceding::` | Selects all nodes before the current node. | `//input[@id='password']/preceding::label[1]` |
| `preceding-sibling::` | Selects siblings before the current node. | `//input[@id='email']/preceding-sibling::label` |
| `following::` | Selects everything after the current node in the document. | `//label[text()='Email']/following::input[1]` |
| `parent::` | Selects the parent of the current node. | `//input[@id='username']/parent::div` |
| `child::` | Selects direct children. | `//div[@id='container']/child::button` |
| `ancestor::` | Selects all ancestors (parent, grandparent, etc.). | `//input[@id='username']/ancestor::form` |
| `ancestor-or-self::` | Selects current node and its ancestors. | `//input[@id='username']/ancestor-or-self::*` |
| `descendant::` | Selects all descendants. | `//div[@id='container']/descendant::input` |
| `descendant-or-self::` | Selects current node plus all descendants. | `//div[@id='container']/descendant-or-self::*` |
| `self::` | Selects the current node. | `//input[@id='username']/self::input` |
| `attribute::` | Selects attributes of an element. | `//input[@id='username']/attribute::id` |
