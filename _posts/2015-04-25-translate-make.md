---
layout: post
title: Make 详解（译文）
---

## [Make](http://www.ploxiln.net/make.html)
### 简介

*make* 是一款很有价值而且已经被广泛应用的开发工具。它是一款“构建”工具：它依据一份“谱子”来构建程序和文档。如果你在编辑一些文件，然后经过一系列的处理步骤从编辑过的文件产生出其他形式的文件；那么你就可以使用make来做这些。然而，大部分情况下make用来构建和安装软件。make起源于Unix中，如今每一个BSD项目和GNU项目都有他们自己的版本。

我经常感觉到许多其他经验丰富的开发者对于make最基本的了解都没有，而且不能从对其更深入的了解中获益。我并不是在责备他们：make当然很古老并且有一些奇怪的语法和古怪的习惯。所以许多开发者都只是做最少的必要工作来将他们新的资源加入到构建系统中，然后就回到实际的代码编写工作中去了。有一个号的构建系统并且明白它是怎么工作的能让软件的开发和部署变的更舒服，所以我建议花一点时间去真正的学一个构建工具。

这篇博客主要是关于利用*[GNU Make](https://www.gnu.org/software/make/)*的部分特性来写一个好的构建系统的。目前有好多种构建系统，其中一些用make作为零件，比如*[CMake](http://www.cmake.org/)*和*[GNU Autotools](https://www.gnu.org/software/autoconf/)*，还有一些不使用make的，例如*[scons](http://www.scons.org/)*、*[jam](http://www.freetype.org/jam/index.html)*、*[ninja](http://martine.github.com/ninja/)*。我不会挑任何用其他构建工具的人的错，如果它很好用，但是我会通过判断它是否符合以下目标来鉴定某个工具的好坏：

开发人员应该可以通过执行一条命令（例如 make [args]），然后就可以很快的得到准确的结果。

更详细的说就是：

* The build system should produce the exact same final outputs as if it was doing a completely clean from-scratch build, even when it's not.
* 这个构建系统应该很灵活。
* 这个构建系统应该很快。

灵活性方面：

* 它应该能工作在不同的文件系统层次结构上，各种依赖和输出文件的位置可以很灵活的指定。
* 它应该能够安装在任意的“根”目录下，使得构建结果能够被很容易的打包，或者简单的安装到其他的系统的文件系统中。
* 它应该是可配置的，以便能够适当的对于不同的体系结构进行交叉编译。它应该能够很方便的指定编译器和连接器的路径和选项。

速度方面：

* 一个快速的构建系统会避免不必要的工作，如果已有的结果文件和重新生成的无差，那么它应该能够重用这些文件。
* 如果可能，一个快速的构建系统会并行处理任务。

软件开发者们不想等太久才看到和测试某一更改的结果。无论是工作在嵌入式系统还是大型应用系统上的系统集成者都是这样，对于他们来说，缩短构建时长可以对系统做一个大的改动了。这些开发者还需要更大的灵活性以便他们能够只打很小的补丁就能将自己的软件集成到系统中。

如果一个开发者正在开发一款软件，他做了一些更改并运行构建程序之后发现他的更改没有生效，他就会有很多疑虑并且会经常在构建之前删除资源或者构建树。这严重破坏了对更改的反馈机制。


### 为什么选择GNU Make

本文中，我使用GNU Make是因为：

* 几乎每一个流行的操作系统都会安装一份GNU Make的拷贝，或者在发行商那里可以获取到，包括BSD、Mac OS X、Linux 发行版和其他的Unix系统。诚然Windows，故事很复杂，我这里不会涉及到。
* make && sudo make install 的习惯很普遍。如果你构建过一些从web上发现的开源软件，那么你很可能已经使用到了make。
* 如果你需要debug一个用make作为组建的构建系统，也许和尝试使用上面提到的灵活性有关并且发现有些不对劲，那么结实的make的基础知识会对你有很大帮助。
* 可以用make构建出一个准确、灵活、快速的构建系统，并且保持它相对简单。
* GNU Make只依赖于C标准库（and optionally "guile", but don't use that）。

对于make的多种实现，我只使用GNU Make，还有当我批评GNU Make的某些特性时，我会用到一些GNU的扩展，这些扩展和BSD的版本不一样，例如 $(patsubst ...)。然而基础的概念对于BSD的make实现都是一样的，并且这还会让你对于他们有一个比较好的了解，使得你能够在任何还在使用unix/make的系统上debug和创建合适的Makefile。

### 回顾make
你也许会觉得Makefiles会是下面这样（这是我故意写的一个烂例子）：

```
# subroutine def subroutine call another subroutine call 
#     v            v            vinstall: install-doc build-binary	install binary $(bindir)/	mkdir -p $(datadir)/myprog/	cp -r images $(datadir)/myprog/#       ^#       body of subroutine
```
这是make的一项规则，"install"即是构建的目标，它的依赖是"install-doc"和"build-binary"，其他部分就是构建需要的命令步骤，这些命令以tab作为缩紧。tab在make里边比较特别。一个tab只能用来缩紧命令步骤，没有其他的用途。
This model obscures the property that "install-doc" and "build-binary" could heappen in parallel with each other if make is invoked with "-j2". It also obscures some ways in which unnecessary work could be avoided, and that this rule might not run if a local file named "install" was accidentally created.
Makefiles 应该更像是一张依赖关系的图谱，而不是一个脚本。这张图谱中的大部分节点都是文件。脚本片段将这些节点联系到一起。我将整个的图谱看作一种架构在人为编辑的源文件之上的结构，这个通过横梁和街头建立起来的结构支持在特定的位置输出特定的文件。通过改变某个变量整个机构立即重新配置以支持不同位置的不同的输出，尽可能的重用结构中现有的部分。当我意识到这种机制的健壮性和高效性时，我发现make是一个很不错的工具。

一开始我将强调指出图谱这种机制，之后我马上就会介绍一个make特性的精选集，这些特性很简洁而优雅。同时，我会告诉你“你不能这样做”



