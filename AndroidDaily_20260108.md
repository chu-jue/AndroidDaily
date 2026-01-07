---
title: "Android WebView 跨域漏洞成因与防护"
date: 2026-01-08 08:00:00
categories: ["Android安全攻防","常见漏洞与防护策略"]
tags: ["Android WebView","跨域漏洞","漏洞成因","防护措施","同源策略","JavaScript交互"]
---

# 🕵️‍♂️Android WebView 跨域漏洞成因与防护详细教程

想象一下，你家有很多房间，每个房间都有特定的权限，只有特定的人能进入。在网络世界里，不同的网站就像是不同的房间，它们也有自己的权限范围。而 Android WebView 就像是一个可以打开不同房间门的“钥匙”，但有时候这把“钥匙”可能会出现问题，导致不该进入的人进入了房间，这就是跨域漏洞。在这篇教程里，我们将从头到尾了解 Android WebView 跨域漏洞的成因以及如何防护。

## 1. 什么是 Android WebView 和跨域
### 1.1 Android WebView 是什么
Android WebView 是 Android 系统提供的一个组件，它就像是一个小型的浏览器窗口，可以在你的 Android 应用里显示网页内容。比如，你在一个新闻应用里看到的新闻详情页，可能就是通过 WebView 加载的网页。

### 1.2 跨域是什么
在网络中，每个网站都有自己的“域名”，就像每个家庭都有自己的地址一样。当一个网页试图访问另一个不同域名的资源时，就会涉及到“跨域”。正常情况下，为了保证网络安全，浏览器会遵循“同源策略”，就像每个家庭都有自己的门禁系统，不允许外人随意进入。同源策略要求访问的资源必须和当前网页的协议、域名、端口都相同，否则就会被阻止。

## 2. Android WebView 跨域漏洞成因
### 2.1 允许跨域访问设置不当
在 Android WebView 中，如果开发者为了方便，错误地设置了允许跨域访问的参数，就可能导致漏洞。比如，以下代码：
```java
WebSettings webSettings = webView.getSettings();
// 允许跨域访问
webSettings.setAllowUniversalAccessFromFileURLs(true); 
```
**解释**：`setAllowUniversalAccessFromFileURLs(true)` 这个方法允许从本地文件 URL 加载的页面访问任何来源的资源，这就像是把家里的门禁系统关掉了，任何人都可以随意进出，很容易被不法分子利用。

### 2.2 JavaScript 交互漏洞
当 WebView 允许 JavaScript 交互时，如果没有对传入的 JavaScript 代码进行严格的过滤和验证，就可能会被注入恶意代码。例如：
```java
webView.getSettings().setJavaScriptEnabled(true);
webView.addJavascriptInterface(new JavaScriptInterface(), "Android");
```
**解释**：`setJavaScriptEnabled(true)` 开启了 JavaScript 支持，`addJavascriptInterface` 方法允许 JavaScript 代码调用 Android 应用里的方法。如果恶意网站通过跨域访问这个 WebView，就可以注入恶意的 JavaScript 代码，调用 Android 应用里的敏感方法。

## 3. 常见错误及解决办法
### 3.1 允许跨域访问设置错误
**错误表现**：应用出现安全漏洞，可能会被恶意网站获取用户信息。
**解决办法**：避免使用 `setAllowUniversalAccessFromFileURLs(true)` 这样的危险设置。如果确实需要跨域访问，可以使用更安全的方式，比如设置白名单。以下是示例代码：
```java
WebViewClient webViewClient = new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        // 定义允许访问的域名白名单
        String[] allowedDomains = {"example.com", "another-example.com"}; 
        for (String domain : allowedDomains) {
            if (url.contains(domain)) {
                view.loadUrl(url);
                return true;
            }
        }
        return false;
    }
};
webView.setWebViewClient(webViewClient);
```
**解释**：这段代码通过自定义 `WebViewClient`，在加载 URL 之前检查 URL 是否在白名单里。如果在白名单里，就允许加载；否则，阻止加载。

### 3.2 JavaScript 交互漏洞
**错误表现**：恶意 JavaScript 代码可能会调用 Android 应用里的敏感方法，导致用户信息泄露。
**解决办法**：对传入的 JavaScript 代码进行严格的过滤和验证，避免直接暴露敏感方法。可以在 `JavaScriptInterface` 类里对方法进行权限控制。以下是示例代码：
```java
public class JavaScriptInterface {
    @JavascriptInterface
    public void safeMethod() {
        // 安全的方法
    }

    // 不暴露敏感方法
    private void sensitiveMethod() {
        // 敏感的方法
    }
}
```
**解释**：通过将敏感方法设置为 `private`，可以避免被 JavaScript 代码调用。只暴露安全的方法给 JavaScript 使用。

## 4. 防护措施总结
### 4.1 合理设置跨域访问
避免使用不安全的跨域访问设置，使用白名单机制来控制允许访问的域名。

### 4.2 严格过滤 JavaScript 代码
对传入的 JavaScript 代码进行严格的过滤和验证，避免注入恶意代码。

### 4.3 权限控制
在 `JavaScriptInterface` 类里对方法进行权限控制，只暴露安全的方法给 JavaScript 使用。

## 小结
在这篇教程里，我们了解了 Android WebView 和跨域的基本概念，分析了 Android WebView 跨域漏洞的成因，包括允许跨域访问设置不当和 JavaScript 交互漏洞。我们还学习了常见错误的解决办法和防护措施，如合理设置跨域访问、严格过滤 JavaScript 代码和权限控制。

### 补充资源链接
- [Android WebView 官方文档](https://developer.android.com/reference/android/webkit/WebView)
- [同源策略详细解释](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

## 下一步建议
- 深入学习 Android 安全开发相关知识，可以阅读《Android 安全开发指南》这本书。
- 实践更多的 Android WebView 安全防护案例，巩固所学知识。你可以在 GitHub 上搜索相关的开源项目进行学习。