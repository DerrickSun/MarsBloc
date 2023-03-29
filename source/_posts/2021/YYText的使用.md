---
title: YYText使用
categories: [iOS]
date: 2021-03-29 16:15:24
---
# 背景

在iOS开发中，经常遇到富文本内容的展示，虽然系统的NSAttributedString功能已经比较完善，但是比较缺乏定制化的内容，如果想自由度更高的显示富文本，可以尝试YYText这个框架。下面会对框架的使用进行简单的介绍。

# 使用

## 常用的富文本属性

### 1.背景色块

系统的AttributedString虽然也有背景色的属性设置，但是有时候需要色块加上圆角。就需要用到YYTextBorder

![image](yybg.webp)

```
NSMutableAttributedString* str = [[NSMutableAttributedString alloc] initWithString:XX];    //创建内容富文本

[str yy_insertString:@" " atIndex:0];

[str yy_appendString:@" "];            //前后的间距

YYTextBorder * border = [YYTextBorder borderWithFillColor:XXColor cornerRadius:XX];    //创建背景色（颜色与圆角）

border.insets=UIEdgeInsetsMake(0,0,0,0);        //背景色的偏移量

[str setYy_textBackgroundBorder:border];
```

### 2.图文混排

也就是字符串中添加图片，就需要用到YYTextAttachment这个类。

先引用一下作者的简单介绍

> YYTextAttachment是NSAttributedString的类簇，它的实例作为富文本key属性YYTextAttachmentAttributeName的value。当富文本包含YYTextAttachment时，它会采用文本规则展示。比如，attachment的内容是UIImage时，它会被绘制为CGContext；attachment的内容是UIView或CALayer时，它会被加入到text container的view层或layer层上。

简单用法

```
UIImage *image = [UIImage imageNamed:@"XX"]；    //要显示的图片

NSMutableAttributedString *attachText = [NSMutableAttributedString yy_attachmentStringWithContent:image contentMode:UIViewContentModeCenter attachmentSize:CGSizeMake(10, 10) alignToFont:[UIFont boldSystemFontOfSize:16] alignment:YYTextVerticalAlignmentCenter];    //content：内容  size：图片尺寸 alignToFont:字符串内字体的格式  

[result appendAttributedString:attachText];     //把图片加入字符串中
```

字符串的属性还有很多，比如高亮YYTextHightlight，边框YYTextBorder，下划线YYTextBorder 等等等等。可以说是非常丰富了。

## YYLabel的使用

有了富文本的内容，现在就需要一个显示内容的容器，YYKit提供了文本框YYLabel，使用也很方便，下面会举例几个常用的设置。

### 1.YYTextContainer

显示text的容器，初始化方法分为

```
矩形： 
+ (instancetype)containerWithSize:(CGSize)size;

和非矩形：
+ (instancetype)containerWithPath:(nullableUIBezierPath*)path;
```

也有一些常见的属性可以设置，如行数 maximumNumberOfRows  大小 size

### 2.YYTextLayout

用来控制text的布局，大神的分析如下

```
               这个真的是核心内容了，这个文件一共3300多行的代码，从代码量上就能看出它的地位。这个类中存储着text的layout结果，所有的property都是readonly的。实现的接口有：

               1、通过一些类方法初始化的方法（YYTextContainer、CGSize和text）

               2、layout之后的attributes，都是只读的

               3、从layout中读取信息（位置、range等等）

               4、绘制text layout

               这个类主要是使用上面讲过的所有的数据来绘制text，这部分的代码还是需要一点一点的去读的，如果是新手每一行都会有收获（比如说我），如果是老司机就没有必要一行行的读了，了解他的解题思路和解决这个问题的办法就可以。下面说一下生成layout的那个500行代码的情况，就按照代码的顺序从上往下大概的说明一下干了什么。

                         1）、初始化一系列使用到的变量

                         2）、安全判断，text和container

                         3）、判断是否需要修复emoji的bug（iOS8.3中）

                         4）、判断是否设置了path属性和exclusionPaths数组，做相应的计算拿到cgpath，如果cgpath为空则goto fail 返回nil（如果设置了path，size和insets就没有用了）

                         5）、判断是不是奇偶填充，判断pathWidth是否为0，判断是否是垂直展示

                         6）、使用text创建CTFramesetterRef，创建失败goto fail

                         7）、使用上一步中创建的frameSetter创建CTFrameRef

                         8）、从CTFrameRef的对象中获得每一行、ctRun数组，计算每一行的frame，判断是否实现了linePositionModifier这个协议，有的话调用协议方法。

                         9）、计算bounding size

                         10）、判断是否需要truncation，和按那种方式处理

                         11）、判断是否垂直布局text，需要的话，旋转布局

                         12）、判断得到的visibleRange长度，有效的话遍历text中的attributes，配置对应的layout属性

                         13）、配置layout中的attachments

                         14）、配置结束，返回layout

                    绘制时就是根据layout中的配置情况绘制相应的特征，这段代码在此就不做分析了。

```

我这边只是用来确定YYLabel的布局，用法如下
```
YYTextLayout *layout = [YYTextLayout layoutWithContainer:container text:str];

self.label.size= layout.textBoundingSize;

self.titleLabel.textLayout= layout;
```

# 总结
以上就是YYText的一些简单用法，后续如果有新的需求，会更仔细的研究下这个框架更高级的用法，很多功能其实还没有用到，但是已经体会到了代码开源的好处。

其实我们在开发中遇到的大部分复杂功能，在github中多多少少都能找到类似的框架去参考，很多情况下我们是不需要造轮子的。但是当你找到一个好轮子时，一定要多多理解实现原理，第一能让你后续对框架的改造更容易，第二也是让我们去学习，感受大神的思想，开阔自己的思维，总之是好处多多。而自己写代码时，对一些常用功能，也要做好解耦和封装，也要照着开源的规范要求自己，这样后续的开发中是非常提高效率的。
