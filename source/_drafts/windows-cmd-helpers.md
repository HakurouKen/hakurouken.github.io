---
title: windows 下的命令行工具
tags: javascript
---

写代码的时候，总免不了和命令行打交道。但是在 Windows 下，默认的命令行 cmd.exe 实在难用。 Windows 7 之后，微软推出了 powershell 作为 cmd 的替代品，但是仍然有很多缺点（PS：Windows 10 又推出了 Bash shell，不过手头还没有 Win 10 设备，暂时还没体验到），似乎对于目前在 Windows 下的开发，命令行增强是个刚需。现在这样的工具随便搜搜也能找出一堆，质量却参差不齐，本文将对几个人气比较高的工具做一些对比评测。

## shell，终端(terminal)与控制台(console)
在进入正题之前，我们需要先弄清楚 shell，terminal，console 三个概念。这三个概念都是从早期的大型机保留下来的，我们这里只讨论在类 unix 环境下这三个概念的区别。

**太长懒看版本：**
1. Shell : 命令行解释器
2. Terminal : 命令行的 I/O 环境
3. Console : 物理 Terminal

Shell 是实际处理我们输入的指令，并返回输出的程序。在 \*nix 环境下，shell 一般特指命令行 shell（一般以应用程序 + 参数 + Enter 的方式调用），在其他环境（如 Windows）下一般不使用 shell 这个词。通俗的说，虽然 shell 的意思是“壳”，但它却是实际完成工作的组件。我们常说的 bash, zsh, fish 等等，都属于 shell。

Terminal 一般指代文本的I/O 环境（运行 Shell 的环境）。简单意义上，它的工作就是接受输入，将其转换成 shell 识别的控制序列传递给 shell，然后接受 shell 的 output 并显示。当然，一个良好设计的 Terminal 一定会有一些附加功能，比如显示配置、多任务多窗口管理等等。它还有一个常见的指代名称是 tty。比如 Mac 上的 iTerm，Gnome 下的 Gnome Terminal 和 Guake，都属于 Terminal。

Console 是一种特殊的 Terminal，一般指代物理终端，例如键盘、显示器。

## Cygwin 与 MinGW
想要在 Windows 上使用命令行，我们可能马上会想到 Cygwin/MinGW。Cygwin 和 MinGW 都提供了在 windows 上使用 GNU 工具的能力，但是他们两个之间有一些差别需要注意。

Cygwin 的用途，它的[官方网站](http://www.cygwin.com/)已经描述的非常清楚了：
> Cygwin 是：
> - 一个 GNU 和开源工具的合集，为 Windows 提供类似 Linux 发行版的功能
> - 一个提供大量 POSIX API 的功能的 DLL(cygwin1.dll)
>
> Cygwin **不是**：
> - 一个在 Windows 上运行原生 Linux 程序的方法。如果想要在 Windows 上运行，你必须从源代码重新构建。
> - 一个神奇的让 Windows 程序感知 Unix<sup>®</sup>的功能（如信号、pyt 等）。同样，如果要利用 Cygwin 的功能，还需要从源代码重新构建。

再来看 [MinGW](http://www.mingw.org/)：
> MinGW提供了一个完整的开源编程工具集，适用于开发本地MS-Windows应用程序，而不依赖于任何第三方C-Runtime DLL。
>
> MinGW 编译器提供对对 Microsoft C （和某些特定语言）的运行时访问的功能。简单来讲，** MinGW 不会试图为一个 POSIX 应用提供在 Windows 上的 POSIX 运行时的环境。 **如果你有类似的需求，可以尝试 Cygwin。

MinGW 使用了 MSYS 作为他的命令行解释环境。MSYS（Minimal SYStem）是一个 Cygwin-1.3 的轻量分支，包括一小部分 Unix 的工具链。但是，MSYS 现在已经基本没人维护了，MinGW 的主要维护者牵头建立了 MSYS2 ，这是一个基于 Cygwin 更新版本的分支，基本可以替代 MSYS。

回到我们的需求来看，如果我们只是需要一个**类 Unix 命令行工具**，Cygwin 是我们的首选。当然，你也可以选择轻量一些的 MSYS2 作为替代。

## 几个命令行工具对比
下文提到的命令行工具，主要是指终端(terminal)。这些终端多数都支持我们选择自己的 shell ，也有些工具中直接集成了 shell 或者一些工具集。有关 shell，在这里强烈推荐 **cygwin + zsh + oh-my-zsh** 的默认配置，然后在此基础上针对性的修改主题，并配置一些实用工具/插件（例如提示 alias 的[alias-tips](https://github.com/djui/alias-tips)，快速跳转的 [autojump](https://github.com/wting/autojump)），可以参照 [awesome-zsh-plugins](https://github.com/unixorn/awesome-zsh-plugins)。

### minitty
官方网站：[https://mintty.github.io/](https://mintty.github.io/)

minitty 是 Cygwin 和 MinGW 的默认终端，它在保持轻量的同时，解决了不少 Windows 原生终端的痛点，包括但不限于：
1. 字符集支持问题
2. 窗口宽度不可调整
3. 复制粘帖困难
4. 没有主题定制

但是 minitty 有很多地方支持的比较不完善，比如[不支持控制台程序](https://github.com/mintty/mintty/issues/56)，[不支持多 Tab](https://github.com/mintty/mintty/issues/645)，[在输入特殊字符时，光标移动会出现问题](https://github.com/mintty/mintty/issues/612)。

综合来看，minitty 做到了可用，但是并不是很好用，如果你是一个轻度命令行使用者，可以考虑用 minitty 做一个 backup 方案（因为是 Cygwin 默认，省去安装的步骤）。

### cmder
### conemu
### xshell
### babun
### ConsoleZ
### PowerCmd


## 参考资料
1. [Stackoverflow: What is the difference between shell, console, and terminal?](https://superuser.com/questions/144666/what-is-the-difference-between-shell-console-and-terminal#answer-144668)
2. [StackExchange: What is the exact difference between a 'terminal', a 'shell', a 'tty' and a 'console'? ](http://unix.stackexchange.com/questions/4126/what-is-the-exact-difference-between-a-terminal-a-shell-a-tty-and-a-con#answer-4132)
3. [Stackoverflow: What is the difference between Cygwin and MinGW?](http://stackoverflow.com/questions/771756/what-is-the-difference-between-cygwin-and-mingw)
