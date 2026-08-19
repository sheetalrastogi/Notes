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
