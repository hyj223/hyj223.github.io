---
layout:     post
title:      "uiimageview add blur effect"
description:   "试试咯"
date:       2015-12-08
tags:
    - iOS

---  

### 这是一个探索
上午看到了 `UIVisualEffect & UIVisualEffectView `, 可以实现简单的blur效果~

代码测试（iOS8+）

```

    UIImage *image = [UIImage imageNamed:@"you_1206_1.jpg"];
    UIImageView *imageView = [[UIImageView alloc] initWithImage:image];
    imageView.frame = CGRectMake(0, 64, imageView.bounds.size.width, imageView.bounds.size.height);
    self.imageView = imageView;   
    [self.view addSubview:imageView];
    
    // create effect
    UIBlurEffect *blur = [UIBlurEffect effectWithStyle:UIBlurEffectStyleLight];
    // add effect to an effect view
    UIVisualEffectView *effectView = [[UIVisualEffectView alloc]initWithEffect:blur];
    effectView.frame = CGRectMake(0, 0, self.imageView.bounds.size.width, self.imageView.bounds.size.height);
    // add the effect view to the image view
    [self.imageView addSubview:effectView];

``` 

可选的blur样式

```

typedef NS_ENUM(NSInteger, UIBlurEffectStyle) {
    UIBlurEffectStyleExtraLight,
    UIBlurEffectStyleLight,
    UIBlurEffectStyleDark
} NS_ENUM_AVAILABLE_IOS(8_0);

```
