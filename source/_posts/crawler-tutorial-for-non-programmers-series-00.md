---
title: 给非程序员的爬虫教程（零）：开发环境搭建
tags:
  - python
  - 非程序员的爬虫教程
date: 2017-12-31 21:15:55
---

## Python 的安装与初始配置
我们选用 Python 语言来进行开发爬虫，原因是 Python 语言容易上手，而且有相对丰富的学习资料。Python 现在有 Python 2 和 Python 3 两个大版本，3 是 2 的升级版，同时语言细节表现上有些细微的不同，也正是因为这个原因，Python 2 也作为历史版本一直在开发维护着。对于我们，可以直接选择 Python 3。建议从[官网](https://www.python.org/downloads/)直接下载最新版，当前(2017.12)最新版本为 3.6.4 ，我们将以这个版本为例，介绍一下安装时需要关注的选项。
<!-- more -->

![Python 安装截图](http://ww1.sinaimg.cn/large/9f9426adgy1fn08jrek25j20im0bg750.jpg)

"Add Python to PATH" 是否把 Python 添加到 PATH 的环境变量，关系到我们能不能在命令行中使用 Python，确保这个是勾选状态，然后选择自定义安装。

![Python 安装截图](http://ww1.sinaimg.cn/large/9f9426adgy1fn08kbiuxkj20im0bg74w.jpg)

1. Documentation：文档，当我们忘记了某些具体函数的用法时，可以快速查阅，必选
2. pip： python 的包管理工具，帮助我们快速安装第三方包，必选
3. tcl/tk and IDLE：图形界面编辑器和交互式环境，必选
4. py launcher：安装 launcher 后，我们可以直接双击 .py 文件来执行，就像 .exe 文件一样，建议勾选

![Python 安装截图](http://ww1.sinaimg.cn/large/9f9426adgy1fn08ktduavj20im0bgmxw.jpg)

1. Associate files with Python: 将 .py 文件和 py launcher 关联，和前文的 py launcher 配套，建议勾选
2. Add Python to environment variables: 将 Python 添加到**环境变量**，勾选这个选项，我们才能在命令行中方便的使用`python`命令，必选

安装成功后，点击完成，进入安装目录并打开 python.exe，打开一个窗口，在弹出的窗口中，输入`print("Hello World!")`，按下回车，可以得到下列输出：

![Hello World](http://ww1.sinaimg.cn/large/9f9426adgy1fn09daqlh2j20it0camxp.jpg)

可以看到，我们得到了“Hello World!”的输出，这就是一行最简单的 Python 代码。

## 命令行(command line)
在 windows 环境下，我们说的命令行一般指 cmd.exe 或 powershell.exe 。powershell 比 cmd 更加强大，但是要相对复杂一点，对于初学者执行简单的命令，并没有太大的区别。为了方便，下文的命令行，统一指代 cmd.exe 。输入 win + R 打开“运行窗口”，输入 cmd 即可打开命令行。

![CMD 窗口](http://ww1.sinaimg.cn/large/9f9426adgy1fn08myyw9aj20it0caglu.jpg)

光标前代表我们现在所在的文件夹，我们可以在这个窗口里输入对应的命令，然后按回车执行。对于初学者，我们首先需要记住几个基础命令：

```bash
X:
```
`X` 可以替换为任意盘符，比如`D:`可以将当前执行路径切到 D 盘。

```bash
dir
```
DIRectory ，用来列出（当前）文件夹下所有的文件，以及一些文件的基础信息。

```bash
cd FOLDER_PATH
```
Change Directory，用于切换到 FOLDER_PATH 的文件夹，在实际执行的时候，我们可以把 FOLDER_PATH 换成我们想要的任何一个路径。我们可以用`..`来代表上一级目录，用`.`来代表这一层目录自己。需要注意的是，这里不能切换盘符，文件夹所在的路径必须和切换前一致。想要切换盘符，请用`X:`的方式来切换。
这里的`FOLDER_PATH`一般称为“命令行参数”，是对于我们执行的主命令的一些补充说明。

类似的，我们安装的 Python 也可以像这些命令一样执行，输入`python`并回车，会进入一个和我们双击 python.exe 差不多的交互界面，我们称这个叫做 repl(Read-Eval-Print-Loop)，一般用来输入一些测试代码执行，并马上得到结果。
和`cd`命令一样，python 执行时也可以带参数，比如我们想要查看我们安装的 Python 版本，可以输入`python --version`，我们可以看到并没有打开 repl，而是输出了一个版本号。我们也可以将一个 python 文件(.py 后缀)作为参数（例如`python test.py`），来执行这个 python 文件。

实际操作一下，打开一个记事本文件，将下列代码拷贝到记事本中，并保存为`date.py`:
```python
from datetime import datetime
def get_date(d):
  return '{}-{}-{}'.format(d.year, d.month, d.day)

print(get_date(datetime.now()))
```
之后，使用上文的`cd`命令切换到`test.py`所在的目录，输入`python date.py`，可以发现控制台中输出了今天的日期。恭喜，我们已经成功完成了一个简单的 Python 脚本的编写！

## 编辑器
当我们实际开始写代码的时候，我们就会体会到，如果使用记事本编辑是件很不方便的事情，因此我们需要一些专用的编辑器(editor)，来辅助我们写代码。与编辑器类似的，还有一个 IDE（Integrated Development Environment，集成开发环境）的概念，从名字可以看出来，比起编辑器来，IDE 有着更加强大的功能，但同时也相对复杂笨重一些。编辑器也可以自行通过插件扩展，实现类似 IDE 的功能，二者也没有严格意义的界限。编辑器/IDE 的选择，基本上看个人喜好，对于初学者来讲没有太大区别。这里大概推荐几款编辑器（IDE），可以按照自己的喜好随意选择：

1. Sublime Text
官网：[https://www.sublimetext.com/](https://www.sublimetext.com/)

2. VSCode **（注意不是VisualStudio）**
官网：[https://code.visualstudio.com/](https://code.visualstudio.com/)

3. Atom
官网：[https://atom.io/](https://atom.io/)

4. PyCharm
官网：[https://www.jetbrains.com/pycharm/](https://www.jetbrains.com/pycharm/)

编辑器和 IDE 的安装过程中，没有太多注意事项，像普通软件一样安装即可。有一点需要注意的是，对于这种常用软件，如果你的硬盘是混合硬盘，尽量把编辑器装在固态硬盘的分区，这样会大大提高编辑器的启动速度。
