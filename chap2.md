2 LAMMPS Syntax and Source Code Hierarchy

## 2 LAMMPS 语法和源代码层级结构

本章中，我们会概述 LAMMPS 输入脚本结构，并把脚本命令与背后的源代码运行联系起来。我们将介绍源码库，跟着介绍一些类层级结构，这些类都是负责模拟框架的设置和命令解析的，最后简要概述几个顶层类。

我们将讨论以下话题：
- 介绍典型的 LAMMPS 输入文件结构
- 介绍源码库文件
- 概述源代码层级结构
- 一些顶层类的函数

本章的最终目标是，解释源代码如何安排模拟的基本设置，这将帮助您更好地理解 LAMMPS 的修改和扩展的流程。

### 技术要求
要执行本章中的说明，我们仅需一文本编译器（如 **Notepad++, Gedit**）

本章中所有源代码都可在此找到：[https://github.com/PacktPublishing/Extending-and-Modifying-LAMMPS-Writing-YourOwn-Source-Code](https://github.com/PacktPublishing/Extending-and-Modifying-LAMMPS-Writing-YourOwn-Source-Code)。

### LAMMPS 输入脚本结构介绍
LAMMPS 提供内置功能，可使用它自己的脚本语言构建 MD 模拟。LAMMPS 网站上提供了 LAMMPS 脚本语法清单（[www.lammps.sandia.gov](www.lammps.sandia.gov)）。输入脚本从开头到结尾按行执行。一个典型的输入脚本由以下部分组成：
- 初始化设置
- 定义体系
- 设置模拟
- 执行模拟

示例输入脚本可能如下所示：
![fig2.1]()

输入脚本通过指定模拟盒子和边界、生成原子、定义原子间 pair 势、应用热浴和最后的执行模拟部分来设置 MD 模拟。以 `pair` 开头的命令行定义 pair 势，以 `fix` 开头的命令行执行很多对体系的操作，包括热浴和时间积分。许多 MD 功能都通过 `pair` 和 `fix` 命令实现，二者将在 *第5章 理解 pair 样式*  和 *第7章 理解 fix* 中详细讨论。

命令脚本的每一行都会被 LAMMPS 可执行程序察看并执行对应的源代码。控制所有操作的源码将在本书的下个部分介绍。

### 源码库介绍
在下载解压 LAMMPS 后（说明请访问 [https://lammps.sandia.gov/doc/Install_tarball.html]( https://lammps.sandia.gov/doc/Install_tarball.html)），源码可在 `src` 文件夹中找到，如下图：
![fig2.2]()
源码文件大多可分成成对的 C++ 文件和头文件，分别带有 `.cpp` 和 `.h` 后缀，在模拟中共同扮演一个给定的角色。

源代码文件会在编译时和其它可选包一同被加入 LAMMPS 可执行程序。图 2.2 中全大写字母命名的文件夹都是可选包，包如果被指定的话就会加入 LAMMPS 可执行程序。一但编译成功，可执行程序就有能力读取 LAMMPS 输入脚本命令并调用恰当的已安装源码文件。

执行 LAMMPS 脚本时，源代码中的顶层类首先实例化 LAMMPS ，并通过分配内存、解析输入脚本行、处理器分区、实例化集成类和构造 neighbor list 来设置模拟。顶层类使得输入脚本的剩余部分能够执行，包括 pair 和 fix 命令，完成时打印屏幕输出。顶层类和源代码层次结构将在下部分讨论。

### 概述源代码层级结构
源代码层结构的最高层类在 `lammps.cpp` （与 `lammps.h`）中，它通过提供 LAMMPS 的实例来启动 LAMMPS 。通过这种方式，`lammps.cpp` 分配的基本类可在全部代码中访问，如下截图所示：
![fig2.3]()
`mian.cpp` 文件调用实例化，为了接下来的程序运行，该文件也将输入脚本传递给 LAMMPS 实例，如下截图所示：
![fig2.4]()
余下的模拟设置由其它几个顶层类进行，其中一些简述如下：
- `memory.cpp`：创造和销毁多维数组
- `error.cpp`：打印错误和警告信息，中止模拟
- `universe.cpp`：创建和初始化 partition ，以划分和分配模拟 domain 到不同的核
- `input.cpp`：解析和执行输入文件命令
- `finish.cpp`：完成模拟时打印输入到屏幕
- `atom.cpp` `atom_vec.cpp`：储存和分配包含原子信息如位置、力、分子索引等的数组
- `update.cpp`：实例化 integrator ；包括多种单位的物理常量，提供对时间步长的访问
- `neighbor.cpp` `nigh_list.cpp` `neigh_request.cpp`：构造 neighbor list ，为所有原子储存列表；当需要时，让特别种类的 neighbor list 可被调用
- `group.cpp`：将原子分组，控制和计算各组原子的性质
- `force.cpp`：通过创造和验证原子对、键长、键角、二面角与其它类，设置平台计算键/非键力
- `modify.cpp`：通过创造和验证 `fix` 与 `compute` 类列表，设置平台应用 fix 与计算
- `output.cpp`：设置将输出写入到文件或屏幕的类与内存

此外，还有一些称为 `style` 的父类，其中包含大量子类（例如，*fix 样式* 、*pair 样式* 和 *计算样式* ）。这些父类和它们的子类在实现 MD 功能时是最相关的，尤其是在 LAMMPS 脚本里这些类被调用的设置模拟和执行模拟阶段。

> **重要提示**
> 更多源代码库和层级结构的信息见 [https://lammps.sandia.gov/doc/](https://lammps.sandia.gov/doc/)。

通常而言顶层类无需因为最终用户的需求而变动，可以简单地修改子样式类来实现定制的 MD 功能。这些类的细节将在接下来的章节讨论

### 总结
本章介绍了 LAMMPS 源代码层次结构，以说明各种顶级类在建立模拟基础时所扮演的角色。此外，我们提到了三种样式，这三种样式能更恰当地将自定义功能加入到 LAMMPS 中，这些样式将被深入探讨。

下一章将引导您了解源代码文件的结构，和在时间步的每次迭代中源代码经历的执行阶段。

### 扩展阅读
- LAMMPS 开发指南- 源文件与类结构：[https://lammps.sandia.gov/doc/Developer_org.html](https://lammps.sandia.gov/doc/Developer_org.html)

### 习题
1. 在 `src` 文件夹中，以大写字母命名的文件夹保存了什么文件？
2. 在源代码层结构中，哪个是最高层的类？
