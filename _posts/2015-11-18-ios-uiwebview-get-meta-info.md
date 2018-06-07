---
layout: post
title:  "iOS webview"
date:   2016-10-16 11:29:08 +0800
tags:
    - iOS

---


### capture html property meta in webview
测试url：[www.fang.com](http://www.fang.com)  
在webview的`webViewDidFinishLoad`中：  

```  

- (void)webViewDidFinishLoad:(UIWebView *)webView
{
    NSString *str1 = [webView stringByEvaluatingJavaScriptFromString:@"document.querySelector(\"meta[name=description]\").getAttribute('content')"];
    NSString *str2 = [webView stringByEvaluatingJavaScriptFromString:@"document.title"];
    NSLog(@"===str1:%@,str2:%@====",str1,str2);

}

```  

### capture click link in webview  

```   

- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {

    NSURL *url = [request URL];
    
    //capture click link in webview
    if (navigationType == UIWebViewNavigationTypeLinkClicked) {
        self.urlStr = url.absoluteString;
    }
    return YES;

}  

```

    
    







