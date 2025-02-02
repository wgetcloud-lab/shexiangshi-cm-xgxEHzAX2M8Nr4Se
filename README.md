[合集 \- DVWA(8\)](https://github.com)[1\.DVWA靶场搭建及错误解决教程2024\-12\-29](https://github.com/GuijiH6/p/18638537)[2\.DVWA靶场Brute Force (暴力破解) 漏洞low(低)，medium(中等)，high(高)，impossible(不可能的)所有级别通关教程及代码审计2024\-12\-30](https://github.com/GuijiH6/p/18640192)[3\.DVWA靶场Command Injection(命令注入) 漏洞所有级别通关教程及源码审计2024\-12\-31](https://github.com/GuijiH6/p/18643521)[4\.DVWA靶场File Inclusion (文件包含) 漏洞所有级别通关教程及源码解析01\-01](https://github.com/GuijiH6/p/18645826)[5\.DVWA靶场File Upload(文件上传) 漏洞所有级别通关教程及源码审计01\-02](https://github.com/GuijiH6/p/18647155)[6\.DVWA靶场Weak Session IDs(弱会话) 漏洞所有级别通关教程及源码审计01\-03](https://github.com/GuijiH6/p/18647154)[7\.DVWA靶场Insecure CAPTCHA(不安全验证)漏洞所有级别通关教程及源码审计01\-04](https://github.com/GuijiH6/p/18651665)8\.DVWA靶场Open HTTP Redirect (重定向) 漏洞所有级别通关教程及源码审计01\-05收起
# Open HTTP Redirect


HTTP 重定向（HTTP Redirect Attack）是一种网络，利用 HTTP 协议中的重定向机制，将用户引导至恶意网站或非法页面，进而进行钓鱼、恶意软件传播等恶意行为。攻击者通常通过操控重定向响应头或 URL 参数实现这种


**HTTP 重定向基本原理**


HTTP 重定向是一种用于通知客户端（如浏览器）请求的资源已被移动到另一个位置的机制，通常由服务器发送 3xx 系列状态码响应。常见的重定向状态码包括：


* **301 Moved Permanently**：永久重定向，表示请求的资源已被永久移动到新的 URL。
* **302 Found**：临时重定向，表示请求的资源临时在另一个 URL 上。
* **303 See Other**：建议客户端使用 GET 方法获取资源。
* **307 Temporary Redirect**：临时重定向，保持请求方法不变。
* **308 Permanent Redirect**：永久重定向，保持请求方法不变。


**HTTP 重定向方式**


HTTP 重定向主要利用了合法的重定向机制，通过各种方式将用户重定向到恶意网站。常见的方式包括：


1. **开放重定向（Open Redirect）**：


* 通过操控网站的 URL 参数，实现对重定向目标的控制。例如，合法网站的 URL 参数 `redirect=http://example.com` 被替换为 `redirect=http://malicious.com`，导致用户被重定向到恶意网站。


1. **钓鱼（Phishing）**：


* 利用重定向将用户引导到伪装成合法网站的恶意网站，诱骗用户输入敏感信息（如登录凭证、银行账号）。


1. **恶意软件传播（Malware Distribution）**：


* 通过重定向将用户引导到托管恶意软件的网站，诱骗用户下载和安装恶意软件。


## low


随便点击一个链接，发现url栏有传参点


[![](https://track123.oss-cn-beijing.aliyuncs.com/20250101144208112.png)](https://track123.oss-cn-beijing.aliyuncs.com/20250101144208112.png):[蓝猫加速器配置下载](https://yunbeijia.com)


定位源码查看，发现重定向点


[![](https://track123.oss-cn-beijing.aliyuncs.com/20250101144349298.png)](https://track123.oss-cn-beijing.aliyuncs.com/20250101144349298.png)


修改为**source/low.php?redirect\=http://www.baidu.com**


[![](https://track123.oss-cn-beijing.aliyuncs.com/20250101144537481.png)](https://track123.oss-cn-beijing.aliyuncs.com/20250101144537481.png)


成功跳转


### 源码审计


没有存在过滤，不安全



```
php</span
// 检查URL中是否存在'redirect'参数，并且该参数不为空。
if (array_key_exists("redirect", $_GET) && $_GET['redirect'] != "") {
    // 如果存在'redirect'参数且不为空，则进行重定向到指定的路径。
    header("location: " . $_GET['redirect']);
    exit; // 终止脚本执行
}
// 如果'redirect'参数不存在或为空，则返回HTTP 500状态码并显示缺少重定向目标的错误信息。
http_response_code(500);
?>
Missing redirect target.


php</span
exit; // 终止脚本执行
?>

```

## medium


与**low**级别的方法没什么区别，查看源码可以发现不同的地方在于禁用了**http://，https://** 字段


构造url绕过 **source/low.php?redirect\=www.baidu.com**，如果没有明确指定协议，直接以 `//` 开头，则表示使用和当前页面相同的协议，便可以绕过了


[![](https://track123.oss-cn-beijing.aliyuncs.com/20250101144949520.png)](https://track123.oss-cn-beijing.aliyuncs.com/20250101144949520.png)


### 源码审计


利用正则表达式检查是否含有**http:// https://** 字段。如果有则过滤



```
php</span
// 检查URL中是否存在'redirect'参数，并且该参数不为空。
if (array_key_exists("redirect", $_GET) && $_GET['redirect'] != "") {
    // 使用正则表达式检查'redirect'参数是否包含不安全的绝对URL。
    if (preg_match("/http:\/\/|https:\/\//i", $_GET['redirect'])) {
        // 如果是绝对URL，则返回HTTP 500状态码，并显示错误信息。
        http_response_code(500);
        ?>
        Absolute URLs not allowed.


        php</span
        exit; // 终止脚本执行
    } else {
        // 如果是相对路径，则进行重定向到指定的路径。
        header("location: " . $_GET['redirect']);
        exit; // 终止脚本执行
    }
}
// 如果'redirect'参数不存在，则返回HTTP 500状态码并显示缺少重定向目标的错误信息。
http_response_code(500);
?>
Missing redirect target.


php</span
exit; // 终止脚本执行
?>

```

## high


查看源码可以发现与上面两个级别不同的是检查是否有**info.php**字段，如果没有，则不能进行重定向


构造代码绕过：source/low.php?redirect\=http://www.baidu.com?id\=info.php


[![](https://track123.oss-cn-beijing.aliyuncs.com/20250101145317979.png)](https://track123.oss-cn-beijing.aliyuncs.com/20250101145317979.png)


成功绕过


### 源码审计


检查了url种是否含有**info.php**字段，如果没有则会过滤



```
php</span
// 检查URL中是否存在'redirect'参数，并且该参数不为空。
if (array_key_exists("redirect", $_GET) && $_GET['redirect'] != "") {
    // 检查'redirect'参数中是否包含"info.php"。
    if (strpos($_GET['redirect'], "info.php") !== false) {
        // 如果包含"info.php"，则进行重定向。
        header("location: " . $_GET['redirect']);
        exit; // 终止脚本执行
    } else {
        // 如果不包含"info.php"，返回HTTP 500状态码和错误信息。
        http_response_code(500);
        ?>
        You can only redirect to the info page.


        php</span
        exit; // 终止脚本执行
    }
}
// 如果'redirect'参数不存在或为空，则返回HTTP 500状态码并显示缺少重定向目标的错误信息。
http_response_code(500);
?>
Missing redirect target.


php</span
exit; // 终止脚本执行
?>

```

## impossible


### 源码审计


采用了更加符合现实情况的方法，较为安全



```
php</span
// 初始化目标URL为空字符串
$target = "";
// 检查URL中是否存在'redirect'参数，并且该参数是一个数字。
if (array_key_exists("redirect", $_GET) && is_numeric($_GET['redirect'])) {
    // 根据'redirect'参数的整数值选择不同的重定向目标。
    switch (intval($_GET['redirect'])) {
        case 1:
            // 如果参数值为1，设置目标为"info.php?id=1"
            $target = "info.php?id=1";
            break;
        case 2:
            // 如果参数值为2，设置目标为"info.php?id=2"
            $target = "info.php?id=2";
            break;
        case 99:
            // 如果参数值为99，设置目标为"https://digi.ninja"
            $target = "https://digi.ninja";
            break;
    }
    // 如果目标URL已被设置，执行重定向。
    if ($target != "") {
        header("location: " . $target);
        exit; // 结束脚本执行
    } else {
        ?>
        Unknown redirect target. 
        php</span
        exit; // 结束脚本执行
    }
}
?>
Missing redirect target. 

```

 \_\_EOF\_\_

   ![](https://github.com/GuijiH6)track  - **本文链接：** [https://github.com/GuijiH6/p/18653215](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
