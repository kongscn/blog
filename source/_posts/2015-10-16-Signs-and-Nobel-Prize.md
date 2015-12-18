title: "Signs and Nobel Prize  诺贝尔奖与星座"
date: 2015-10-16
tags:
  - nobel prize
  - python
category: toy
---

First I'd like to state that I'm not really a fan of signs(astrology). I just hear people talking about it. This post is done just for fun, do not take it as serious.

This post is originally posted at Wechat in Chinese, and I'm not planning to translate it here. But you can read the images. Still I've translated the only import part, that is, do not take seriously, relax, read and forget.

<!-- more -->

昨天看公众号的时候看到个标题曰《全世界都在黑处女座》（具体什么座其实根本没留意），诺奖结束还没太久就想，要不来统计一下诺奖与星座吧。

无图无真相，首先是十二星座的获奖分布：
![][signs12]

然后是分学科的，首先是我大经济学：
![][signs12-vs-economics]

神的世界物理学：
![][signs12-vs-physics]

既然全世界都在黑处女座，看看处女座的表现：
![][signs12-vs-virgo]

大金牛:
![][signs12-vs-taurus]

etc.


# Data Source

首先需要一个**可靠**的数据源。记得之前看诺奖网站有一个列表，今天去看发现竟然提供了一个Developer API世界果然进步了。今年(2015)年的获奖者的生日数据仍缺失，所以近使用14年及以前的信息。

# 星座的计算方法

确实被玩晕了好久。参见[Zodiac on wiki](https://en.wikipedia.org/wiki/Zodiac)，不懂天文学就不解释了，主要是这一段：
> Historically, these twelve divisions are called signs. Essentially, the zodiac is a celestial coordinate system, or more specifically an ecliptic coordinate system, which takes the ecliptic as the origin of latitude, and the position of the Sun at vernal equinox as the origin of longitude.

计算某日的星体位置使用`Python + ephem`，具体实现见[这里](http://canonicalmomentum.tumblr.com/post/131082972702/can-you-work-out-a-date-of-birth-from-astrology)。（这篇文章也是上一个小迷的解释，某人公布了出生日几个行星所在星座，就被博主翻出来了🎂😄）

也就是说没有简单采用某月某日的区间算法，而是根据天文坐标计算的。另因为需要日期，所以忽略了所有不包含生日的获奖记录。

瞎折腾，**不适用于任何严肃场合**😄

[signs12]: http://i12.tietuku.com/5cd4a64184e37288.png
[signs12-vs-economics]: http://i12.tietuku.com/570823bfad05f8ff.png
[signs12-vs-physics]: http://i12.tietuku.com/48872ff8f3ae9408.png
[signs12-vs-virgo]: http://i12.tietuku.com/bc4a7d5c47068a04.png
[signs12-vs-taurus]: http://i12.tietuku.com/aec681c1c9069043.png
