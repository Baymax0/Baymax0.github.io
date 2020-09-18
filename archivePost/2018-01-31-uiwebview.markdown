---
layout:     post
title:      "UIWebView播放视频问题"
subtitle:   " \"试试写个与众不同的跳转动画\""
date:       2018-02-01 09:30:00
author:     "Leezw"
header-img: "img/post-bg-2015.jpg"
tags:
    - iOS
---

最近的项目需求在cell中嵌套视频播放功能，视频类型主要分为用户上传到我们自己服务器的视频，和优酷，酷6等视频网站的视频。对于自己服务器上的视频链接我采用了AVFoundation框架进行播放，而一些视频网站的链接则采取UIWebView进行播放。在UIWebView的使用中遇到不少问题。
##视频无法播放问题
通常我们得到服务器传来的视频url时，直接扔到webview中时，就像下面的代码：

    NSString *videoUrl = @"http://v.qq.com/iframe/player.html?vid=z1411odvy7m&tiny=0&auto=1&allowfullscreen=\"\"";
    NSURL *url = [NSURL URLWithString:model.videoUrl];
    NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:str]];
    [webView loadRequest:request];
我发现只有个别视频可以直接播放，大多数的webview都显示白屏。总结的解决方法如下

###### 1. 确认地址是否含有中文，或者特殊字符

方法1：使用stringByAddingPercentEscapesUsingEncoding:

    NSString * urlStr = [videoUrl stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    NSURL *url = [NSURL URLWithString: urlStr];
该方法已在iOS9.0中被遗弃，并且在使用中有给依然会导致url为nil。于是使用了苹果推荐的新方法，该方法最早出现在iOS7.0：

    NSString * urlStr = [videoUrl stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
    NSURL *url = [NSURL URLWithString: urlStr];

###### 2. 确认Webview是否支持该视频格式

通过上面的格式转换后解决了大部分视频无法播放的问题。但是还使用有个别无法播放，webview显示“Not Found”，于是我怀疑是视频格式的问题，发现比较简便的方法便是：把原视频地址拷贝到手机浏览器里打开,浏览器的地址搜索栏通常具备转码功能，如果打开依然显示“Not Found”，并且视频地址在电脑上可以打开播放，则基本确定是视频格式问题。如下面这个视频地址：

    NSString *videoUrl = @"http://player.ku6.com/refer/YV54qgeOOWsvKwwR2O_ADA../v.swf";

## 视频点击播放后自动全屏

像上面的方法那样获得url后通过Webview的loadRequest方法加载视频，默认情况下会自动全屏，缩小后又自动暂停。无法在webview的小窗口内播放视频，解决方法如下：

     webView.allowsInlineMediaPlayback = YES;
     
## Webview清除缓存和cookie

    //清除cookies
    NSHTTPCookie *cookie;
    NSHTTPCookieStorage *storage = [NSHTTPCookieStorage   sharedHTTPCookieStorage];
    for (cookie in [storage cookies])  {
        [storage deleteCookie:cookie];
    }

    //清除UIWebView的缓存
    NSURLCache * cache = [NSURLCache sharedURLCache];  
    [cache removeAllCachedResponses];  
    [cache setDiskCapacity:0];  
    [cache setMemoryCapacity:0];  

    //webview暂停加载
    [webView stopLoading]

---


