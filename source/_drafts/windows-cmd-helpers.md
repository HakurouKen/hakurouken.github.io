---
title: windows 下的命令行工具
tags: javascript
---

写代码的时候，总免不了和命令行打交道。但是在 Windows 下，默认的命令行 cmd.exe 实在难用。 Windows 7 之后，微软推出了 powershell 作为 cmd 的替代品，但是仍然有很多缺点（PS：Windows 10 又推出了 Bash shell，不过手头还没有 Win 10 设备，暂时还没体验到），似乎对于目前在 Windows 下的开发，命令行增强是个刚需。现在这样的工具随便搜搜也能找出一堆，质量却参差不齐，本文将对几个人气比较高的工具做一些对比评测。

## shell，终端(terminal)与控制台(console)
在进入正题之前，我们需要先弄清楚 shell，terminal，console 三个概念。这三个概念都是从早期的大型机保留下来的，我们这里只讨论在类 unix 环境下这三个概念的区别。

Shell 是实际处理我们输入的指令，并返回输出的程序。在 \*nix 环境下，shell 一般特指命令行 shell（一般以应用程序 + 参数 + Enter 的方式调用），在其他环境（如 Windows）下一般不使用 shell 这个词。通俗的说，虽然 shell 的意思是“壳”，但它却是实际完成工作的组件。我们常说的 bash, zsh, fish 等等，都属于 shell。

Terminal 一般指代文本的I/O 环境（运行 Shell 的环境）。简单意义上，它的工作就是接受输入，将其转换成 shell 识别的控制序列传递给 shell，然后接受 shell 的 output 并显示。当然，一个良好设计的 Terminal 一定会有一些附加功能，比如显示配置、多任务多窗口管理等等。它还有一个常见的指代名称是 tty。比如 Mac 上的 iTerm，Gnome 下的 Gnome Terminal 和 Guake，都属于 Terminal。

Console 是一种特殊的 Terminal，一般指代物理终端，例如键盘、显示器。

## Cygwin 与 MinGW

## 几个命令行工具对比
### msysgit(Git Bash)
### cmder
### conemu
### xshell
### babun
### ConsoleZ
### PowerCmd


## 参考资料
1. [Stackoverflow: What is the difference between shell, console, and terminal?](https://superuser.com/questions/144666/what-is-the-difference-between-shell-console-and-terminal#answer-144668)
2. [StackExchange: What is the exact difference between a 'terminal', a 'shell', a 'tty' and a 'console'? ](http://unix.stackexchange.com/questions/4126/what-is-the-exact-difference-between-a-terminal-a-shell-a-tty-and-a-con#answer-4132)
