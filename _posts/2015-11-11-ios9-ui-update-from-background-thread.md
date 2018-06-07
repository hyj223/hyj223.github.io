---
layout: post
title:  "iOS9~UI update from a background thread"
date:   2015-11-11
tags:
    - iOS
---


### iOS 9 控制台出现提示信息
> “This application is modifying the autolayout engine from a background thread, which can lead to engine corruption and weird crashes. This will cause an exception in a future release.”
  
为什么呢？代理回调的方法中有UI的更新造成的，
> Apple have started (with iOS 9) detecting when you're doing this and warning you.


处理代码

```
dispatch_async(dispatch_get_main_queue(), ^{
    // code here
});
```

