---
title: windows 下的命令行工具
tags: javascript
---

写代码的时候，总免不了和命令行打交道。但是在 Windows 下，默认的命令行 cmd.exe 实在难用。 Windows 7 之后，微软推出了 powershell 作为 cmd 的替代品，但是仍然有很多缺点（PS：Windows 10 又推出了 Bash shell，不过手头还没有 Win 10 设备，暂时还没体验到），似乎对于目前在 Windows 下的开发，命令行增强是个刚需。现在这样的工具随便搜搜也能找出一堆，质量却参差不齐，本文将对几个人气比较高的工具做一些对比评测。

## shell，终端(terminal)，控制台(console)与 tty/pty
在进入正题之前，我们需要先弄清楚 shell，terminal，console 三个概念。这三个概念都是从早期的大型机保留下来的，我们这里只讨论在类 unix 环境下这三个概念的区别。同时，unix 上还有 tty 与 pty 之间的区别，这里也需要做一个简单的区分。

**太长懒看版本：**
1. Shell : 命令行解释器
2. Terminal : 命令行的 I/O 环境
3. Console : 物理 Terminal
4. tty: Terminal 的简称
5. pty: 和 Terminal 的接口一致，但内部用不同方式处理的模拟 Terminal

Shell 是实际处理我们输入的指令，并返回输出的程序。在 \*nix 环境下，shell 一般特指命令行 shell（一般以应用程序 + 参数 + Enter 的方式调用），在其他环境（如 Windows）下一般不使用 shell 这个词。通俗的说，虽然 shell 的意思是“壳”，但它却是实际完成工作的组件。我们常说的 bash, zsh, fish 等等，都属于 shell。

Terminal 一般指代文本的I/O 环境（运行 Shell 的环境）。简单意义上，它的工作就是接受输入，将其转换成 shell 识别的控制序列传递给 shell，然后接受 shell 的 output 并显示。当然，一个良好设计的 Terminal 一定会有一些附加功能，比如显示配置、多任务多窗口管理等等。它还有一个常见的指代名称是 tty。比如 Mac 上的 iTerm，Gnome 下的 Gnome Terminal 和 Guake，都属于 Terminal。

Console 是一种特殊的 Terminal，一般指代物理终端，例如键盘、显示器。

pty（虚拟终端 pseudo-tty）是指有着和终端一致的 I/O 接口，但内部会使用不同的处理方式对输入的指令进行处理的设备。比如我们常用的 ssh/xterm 就用到了 pty ：比如当我们 ssh 到一台机器上，执行`ls`，这时`ls`的结果将会输出到与 ssh 进程连接的 pty 中。

然而在 Windows 上，并不能太严格的区分出这几个概念。比如**Windows 上没有 pty 的概念**，而我们平时说的 cmd / powershell ，实际对应的是封装好 Shell + Terminal 的一个完整的应用。实际与“终端”对应的概念是 Windows 的内部进程`conhost.exe`（在 Vista 以及之前是`csrss.exe`）。

## Cygwin 与 MinGW
想要在 Windows 上使用命令行，我们可能马上会想到 Cygwin/MinGW。Cygwin 和 MinGW 都提供了在 windows 上使用 GNU 工具的能力，但是他们两个之间有一些差别需要注意。

Cygwin 的用途，它的[官方网站](http://www.cygwin.com/)已经描述的非常清楚了：
> Cygwin 是：
> - 一个 GNU 和开源工具的合集，为 Windows 提供类似 Linux 发行版的功能
> - 一个提供大量 POSIX API 的功能的 DLL(cygwin1.dll)
>
> Cygwin **不是**：
> - 一个在 Windows 上运行原生 Linux 程序的方法。如果想要在 Windows 上运行，你必须从源代码重新构建。
> - 一个神奇的让 Windows 程序感知 Unix<sup>®</sup>的功能（如信号、pty 等）。同样，如果要利用 Cygwin 的功能，还需要从源代码重新构建。

再来看 [MinGW](http://www.mingw.org/)：
> MinGW提供了一个完整的开源编程工具集，适用于开发本地MS-Windows应用程序，而不依赖于任何第三方C-Runtime DLL。
>
> MinGW 编译器提供对对 Microsoft C （和某些特定语言）的运行时访问的功能。简单来讲，** MinGW 不会试图为一个 POSIX 应用提供在 Windows 上的 POSIX 运行时的环境。 **如果你有类似的需求，可以尝试 Cygwin。

MinGW 使用了 MSYS 作为他的命令行解释环境。MSYS（Minimal SYStem）是一个 Cygwin-1.3 的轻量分支，包括一小部分 Unix 的工具链。但是，MSYS 现在已经基本没人维护了，MinGW 的主要维护者牵头建立了 MSYS2 ，这是一个基于 Cygwin 更新版本的分支，基本可以替代 MSYS。

回到我们的需求来看，如果我们只是需要一个**类 Unix 命令行工具**，Cygwin 是我们的首选。当然，你也可以选择轻量一些的 MSYS2 作为替代。需要注意的是，由于 Cygwin 并不能支持 pty 等特性，因此很多为 Windows 命令行设计的程序会出现运行问题（比如 Node.js，可以参考[这个 issue](https://github.com/nodejs/node/issues/3006)中的讨论内容），在使用的时候需要特别注意这些场景。


## 终端的局限性
Windows 下的命令行，按照实现方式，大概分为两类，一类是使用 stdin/stdout 重定向实现，一类使用 Windows 的 console API 实现。

### stdin/stdout 重定向方式
多数这类终端模拟器需要依赖一个“POSIX 层”来运行，充当这个角色的一般是 Cygwin 或 MSYS。它的优点是运行非常快，同时不需要考虑支持控制字符（因为 Cygwin 都实现了），并可以做到任意回滚。不过这种方式的最大问题在于：**无法使用为 Windows 控制台设计的程序**（例如 Powershell 或者 官方版的 vim 等等）。使用这种方式实现的常见终端有 mintty，puttycyg 等。

### Windows console API 方式
这种方式运行的终端相当于运行在一个隐藏的标准 Windows 控制台窗口中，可以完美兼容 Windows 控制台程序。但是相应的，有些在 Cygwin 上能够正常运行的程序，运行会有些异常。使用这种方式实现的终端有 ConEmu、ConsoleZ 等。


## 几个命令行工具对比
下文提到的命令行工具，主要是指终端(terminal)。这些终端多数都支持我们选择自己的 shell ，也有些工具中直接集成了 shell 或者一些工具集。

### minitty
官方网站：[https://mintty.github.io/](https://mintty.github.io/)

minitty 是 Cygwin 和 MinGW 的默认终端，它在保持轻量的同时，解决了不少 Windows 原生终端的痛点，包括但不限于：
1. 字符集支持问题
2. 窗口宽度不可调整
3. 复制粘帖困难
4. 没有主题定制

但是 minitty 有很多地方支持的比较不完善，比如[不支持控制台程序](https://github.com/mintty/mintty/issues/56)，[不支持多 Tab](https://github.com/mintty/mintty/issues/645)，[在输入特殊字符时，光标移动会出现问题](https://github.com/mintty/mintty/issues/612)。综合来看，minitty 做到了可用，但是并不是很好用，如果你是一个轻度命令行使用者，可以考虑用 minitty 做一个 backup 方案（因为是 Cygwin 默认，省去安装的步骤）。

### ConEmu
官方网站：[https://conemu.github.io/](https://conemu.github.io/)

作为一个终端，ConEmu 做的非常出色，它甚至可以在 tab 中整合一些 GUI 的应用（比如整合一个其它的终端进来）。优点很多不再举例，这里只说使用中遇到的一些问题：
1. 在分屏时，只有第一个 Console 能使用鼠标滚轮。详见[这个 issue](https://github.com/Maximus5/ConEmu/issues/216)。
2. 使用 WriteConsoleW 输出中文时，会重复输出。详见[这个 issue](https://github.com/Maximus5/ConEmu/issues/945)。

### cmder
官方网站：[http://cmder.net](http://cmder.net)

简单的讲，cmder ≈ ConEmu + [Clink](https://mridgers.github.io/clink/)。它同时具备具备了 ConEmu 终端的功能，又通过 Clink 引入了 Bash-style 的行编辑功能。但同时，它也集合了二者的缺点。比如我在使用中，遇到最大的问题是[使用方向键进行历史补全的显示问题](https://github.com/mridgers/clink/issues/409)。另外，cmder 对中文的支持有一点小问题，比如`ls`默认不能输出中文，要加上`--show-control-chars`才能解决（这个问题已经修复，见[这个PR](https://github.com/cmderdev/cmder/pull/1070)）。

### babun
### ConsoleZ
### XShell 和 PowerCmd
网上也有很多人推荐 XShell 和 PowerCmd，但这两个是收费软件，而且闭源也不方便我们做一些定制化的修(zhe)改(teng)，因此这里一票否决。另外，单纯从使用体验上讲，它们并没有比其它免费的终端更加出彩的地方，这也是个人没有选择这两款软件的原因之一。

## 参考资料
1. [Stackoverflow: What is the difference between shell, console, and terminal?](https://superuser.com/questions/144666/what-is-the-difference-between-shell-console-and-terminal#answer-144668)
2. [StackExchange: What is the exact difference between a 'terminal', a 'shell', a 'tty' and a 'console'? ](http://unix.stackexchange.com/questions/4126/what-is-the-exact-difference-between-a-terminal-a-shell-a-tty-and-a-con#answer-4132)
3. [Stackoverflow: What do pty and tty mean?](http://stackoverflow.com/questions/4426280/what-do-pty-and-tty-mean)
4. [Stackoverflow: What is the difference between Cygwin and MinGW?](http://stackoverflow.com/questions/771756/what-is-the-difference-between-cygwin-and-mingw)
5. [ConEmu Document: RealConsole](http://conemu.github.io/en/RealConsole.html)
6. [ConEmu Document: Troubleshooting Cygwin and Msys problems](https://conemu.github.io/en/CygwinMsys.html#Some_techinfo_first)
