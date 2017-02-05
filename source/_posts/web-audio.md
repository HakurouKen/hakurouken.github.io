---
title: 使用 Web Audio API
tags: javascript
date: 2017-02-06 00:09:40
---

Web Audio API 是浏览器提供的音频处理的高级接口，我们可以通过它开发复杂的混音，音效，平移以及获取音频信息等等。本文将通过一个简单的可视化例子，介绍 Web Audio API 的基本应用。
<!-- more -->

## 基础概念
一个典型的 Web Audio 的应用流程如下：

1. 构建音频上下文 AudioContext 对象
2. 在 AudioContext 对象内，创建音源
3. 构建效果节点或分析节点，例如混响、双二阶滤波器、平移、压缩或分析器等等
4. 如果需要输出，选择 **一个** 音频输出目的地
5. 连接 源、效果器、分析节点、输出

![Audio Context](http://ok880r6rs.bkt.clouddn.com/blog/web-audio/audio-context.png)

在一个 AudioContext 中，允许有多个输入以及多个效果器，但是只有一个输出。

## 示例：音乐可视化
有了上述概念之后，下文以一个简单的音乐可视化的 DEMO，详细介绍下 Web Audio API 的使用方法。

### 创建 AudioContext 上下文
使用 canvas 绘图时，我们可以直接通过 canvas DOM 元素的 `getContext('2d')` 方法获取 canvas 对应的绘制上下文，而使用 Web Audio API 时，我们需要自己 new 一个上下文。如果必要的话，使用 webkit 前缀修正。
```javascript
let audioCtx = new AudioContext();
```

### 创建音源
在有了上下文环境之后，我们接下来需要创建一个音频输入源。一般情况下输入源可以来自：
1. 用 javascript 创建正弦波。相关函数：[AudioContext.createOscillator()](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext/createOscillator)。
2. 用原始 PCM 数据创建。相关函数： [AudioContext.createBuffer()](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext/createBuffer), [AudioContext.createBufferSource()](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext/createBufferSource), [AudioContext.decodeAudioData()](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext/decodeAudioData)。
3. 从 HTML 标签(`<video>`和`<audio>`)获取。相关函数：[AudioContext.createMediaElementSource()](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext/createMediaElementSource)。
4. 从 WebRTC 的输入流中获取。相关函数：[AudioContext.createMediaStreamSource()](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext/createMediaStreamSource)。

音源一共有 OscillatorNode, AudioBuffer, AudioBufferSourceNode, MediaElementAudioSourceNode 和 MediaStreamAudioSourceNode 五种类型，分别对应着 AudioContext 的五个音源的 create 方法，从函数名中可以简单的看出对应关系。

我们这里选用`<audio>`作为输入源，代码如下：

HTML
```html
<audio src="audio.mp3" controls id="source"></audio>
```
Javascript
```javascript
let audio = document.getElementById('source');
let source = audioCtx.createMediaElementSource(audio);
```

### 构建分析节点/效果器节点
有了音源之后，接下来需要对音源的数据进行处理/分析，就需要各种分析/效果器节点。例如我们可以用一个 GainNode 的节点来调整音量（用`AudioContext.createGainNode()`创建）、由 BiquadFilterNode 来应用一个双二阶滤波器（用`AudioContext.createBiquadFilter()`创建），全部效果器的列表可以参考 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API#Defining_audio_effects_filters)。我们这里需要获取音乐的数据，需要一个分析节点 AnalyserNode。
```javascript
let analyser = audioCtx.createAnalyser();
```

### 选定输出目标
多数情况下，我们的输出直接选定默认输出`AudioContext.destination`即可。在 WebRTC 应用中，如果需要输入到 WebRTC 媒体流，需要使用`AudioContext.createMediaStreamDestination()`手动新建一个输出目标。我们这里直接取 `audioCtx.destination` 即可（如果只分析不需要输出声音，也可以不选择输出源）。

### 连接
用 connect 方法，按 **输入 -> 效果器/分析器 -> 输出** 顺序连接所需节点，如果需要的话，我们可以将多个输入源连接到同一个输出上。
```javascript
source.connect(analyser);
analyser.connect(audioCtx.destination);
```
在连接之后，我们就可以通过修改效果器的参数，来改变输出的效果，或者通过分析器获取音频数据。例如，这里我们可以通过`AnalyserNode.getByteFrequencyData()`方法获取音频的频率数据：
```javascript
let bufferLength = analyser.frequencyBinCount;
let dataArray = new Uint8Array(bufferLength);
analyser.getByteFrequencyData(dataArray);
// console.log(dataArray);
```

## 示例演示

<a class="jsbin-embed" href="http://jsbin.com/qoyemuh/embed?js,output&height=500px">Audio Visualizer Demo</a>
<script src="http://static.jsbin.com/js/embed.min.js?3.41.0"></script>

## 参考资料
- [Using the Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Using_Web_Audio_API)
- [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [Visualizations with Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Visualizations_with_Web_Audio_API)
- [Basic concepts behind Web Audio API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Audio_API/Basic_concepts_behind_Web_Audio_API)
