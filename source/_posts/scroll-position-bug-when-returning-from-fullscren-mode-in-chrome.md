---
title: 视频退出全屏时在 chrome 下的滚动问题
tags:
  - javascript
date: 2017-08-01 21:32:24
---

## 问题

最近在开发一个信息流的相关功能时，产品提了一个体验问题：**视频全屏后再恢复后，滚动条会回到页面顶部**。

首先交代下背景，我们的产品是一个 PC 上的 hybrid 应用，客户端内嵌了 chromium 引擎用来加载 web 页，同时通过 jsapi 为 web 提供一系列扩展能力（和其它的 hybrid 应用），稍微有些不同的是，我们的主体页面是放在一个 iframe 内。出了问题之后，第一反应是看看是否能在 chrome 内复现，结果直接在浏览器打开主体的 web 页，发现并没有发现类似的现象。然后把整个页面放在 iframe 内，出现了类似的现象：虽然不是直接回到页面顶部，但是可以发现视频全屏前后，页面的位置明显发生了变化。同样的问题，在 firefox/IE 下没有出现，基本可以断定是 chrome 的 bug。Google 了一下，发现了一些开源的播放器内，也有相关的 issue（例如[video.js](https://github.com/videojs/video.js/issues/3355)）。所有的这些 issues，最终都指向了一个 chromium 的 [Bug Report](https://bugs.chromium.org/p/chromium/issues/detail?id=142427)，虽然这个 bug 已经标注为已修复，但是在 iframe 内的场景仍然会出现问题。因此我们需要寻找一个解决方案。

补充：这个 bug 目前是 Windows Only, Mac 和 linux 上的 Chrome(当前版本 60) 已不存在这个问题。

<!-- more -->
## 解决方案
知道 bug 产生的原因后，处理起来思路就比较清晰了，无非就是在全屏前保存`scrollTop`的值，退出全屏后手动设置`scrollTop`即可。由于在`fullscreenchange`事件触发后，`scrollTop`已经发生了改变，而 web 没有提供类似`beforefullscreenchange`这种接口，我们无法在全屏前截获并获取滚动条高度，因此只能在滚动的时候就把`scrollTop`存下来。按这个思路，还原出来的代码大致如下（不考虑兼容性）：
```javascript
var scrollTop = 0;
document.addEventListener('scroll', function() {
  scrollTop = document.body.scrollTop || scrollTop;
});

video.on("fullscreenchange", function() {
  if(
    document.fullscreenElement &&
    document.fullscreenElement === video
  ) {
    return;
  } else {
    document.body.scrollTop = scrollTop || 0;
    setTimeout(function() {
      document.body.scrollTop = scrollTop || 0;
    }, 50);
  }
});
```
至于为什么要延时 50ms 再重试一次，是因为实测发现，在较低版本的 Chrome 中，采用这种方法还原页面滚动位置会有一定的失败的概率，只好加上万能的延时器，延时若干毫秒之后重试一次。

另外这里只考虑了 y 方向的滚动，如果要考虑 x 方向的，如法炮制即可。可以参照[plyr](https://github.com/sampotts/plyr)的[相关修复代码](https://github.com/sampotts/plyr/commit/3c2921b9940857e2276922ce6d3235706ed256d7)。


## 参考资料
1. [Chromium Bug Report](https://bugs.chromium.org/p/chromium/issues/detail?id=142427)
2. [Video.js: Exit of full screen scroll content to page top](https://github.com/videojs/video.js/issues/3355)
