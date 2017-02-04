---
title: 使用 Web Audio API
tags: javascript
---

Web Audio API 是浏览器提供的音频处理的高级接口，我们可以通过它开发复杂的混音，音效，平移以及获取音频信息等等。本文介绍 Web Audio API 的基本应用。

## 基础概念
一个典型的 Web Audio 的应用流程如下：

1. 构建音频上下文 AudioContext 对象
2. 在 AudioContext 对象内，创建音源
3. 构建效果节点或分析节点，例如混响、双二阶滤波器、平移、压缩或分析器等等
4. 如果需要输出，选择 **一个** 音频输出目的地
5. 连接 源、效果器、分析节点、输出

![Audio Context](http://ok880r6rs.bkt.clouddn.com/blog/web-audio/audio-context.png)

在一个 AudioContext 中，允许有多个输入以及多个效果器，但是只有一个输出。

## 参考资料
- [Using the Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Using_Web_Audio_API)
- [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [Visualizations with Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Visualizations_with_Web_Audio_API)
- [Basic concepts behind Web Audio API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Audio_API/Basic_concepts_behind_Web_Audio_API)
