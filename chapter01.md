# 【JVM规范】第一章-前言

> 本文是JVM19规范的个人中文译本，原文为 https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-1.html

## 1.1. 一丁点历史

Java® 编程语言是一种通用的、并发的、面向对象的语言。它的语法类似于 C 和 C++，但它省略了许多使 C 和 C++ 变得复杂、混乱和不安全的特性。Java平台最初是为了解决为网络消费设备构建软件的问题而开发的。它可以支持多种主机架构并允许安全地交付软件组件。为了满足这些要求，编译后的代码必须能够跨网络传输，在任何客户端上运行，并向客户端保证它可以安全运行。

万维网的普及使这些特性变得更加有趣。Web浏览器使数百万人能够以简单的方式上网冲浪和访问富媒体内容。终于出现了一种媒体，无论您使用的是什么机器，也无论它连接到快速网络还是慢速调制解调器，您所看到和听到的内容基本上都是一样的。

Web爱好者很快发现Web的HTML文档格式支持的内容太有限了。HTML扩展，如表单，只强调了这些限制，同时明确表示没有浏览器可以包含用户想要的所有功能。可扩展性就是答案。

HotJava 浏览器首先展示了Java编程语言和平台的有趣特性，它使在HTML页面中嵌入程序成为可能。程序连同它们出现的HTML页面被透明地下载到浏览器中。在被浏览器接受之前，程序会被仔细检查以确保它们是安全的。与HTML页面一样，编译后的程序与网络和主机无关。这些程序的行为方式是相同的，无论它们来自哪里，也不管它们被加载到何种机器上运行。

包含Java平台的Web浏览器不再局限于一组预先确定的功能。访问包含动态内容的网页的人可以放心，他们的机器不会被这些动态内容破坏。程序员只需编写一次程序，就可以在任何提供了Java运行时环境的机器上运行。

## 1.2. java虚拟机

Java虚拟机是Java平台的基石。它是负责硬件和操作系统独立性、编译代码的小尺寸以及保护用户免受恶意程序侵害的软件。

Java虚拟机是一种抽象的计算机器。就像真正的计算机器一样，它有一个指令集并在运行时操作各种内存区域。使用虚拟机实现编程语言是相当普遍的； 最著名的虚拟机可能是 UCSD Pascal 的 P-Code 机器。

Java 虚拟机的第一个原型实现由 Sun Microsystems, Inc. 完成，它在类似于现代个人数字助理 (PDA) 的手持设备托管的软件中模拟了Java虚拟机指令集。Oracle 当前的实现在移动、桌面和服务器设备上模拟Java虚拟机，但Java虚拟机不采用任何特定的实现技术、主机硬件或主机操作系统。它本身并没有被解释，但也可以通过将其指令集编译为 CPU硬件 的指令集来实现。它也可以用微代码或直接用硬件来实现。

Java虚拟机对Java编程语言一无所知，只知道一种特定的二进制格式，即类文件（Class File）格式。类文件包含Java虚拟机指令（或字节码）和符号表，以及其他辅助信息（后面文章将详细介绍）。

为了安全起见，Java 虚拟机对类文件中的代码施加了强大的语法和结构约束。但是，任何具有可以用有效类文件表示的功能的语言都可以由Java虚拟机托管。受到普遍可用的、独立于机器的平台的吸引，其他语言的实现者可以转向Java虚拟机作为他们语言的交付工具。

这里指定的Java虚拟机兼容Java SE 19平台，支持Java语言规范，Java SE 19版中指定的Java编程语言。

## 1.3. 规范的结构
第2章概述了Java虚拟机体系结构。

第3章介绍了将Java编程语言编写的代码编译成Java虚拟机指令集的过程。

第4章指定了类文件格式，这是一种独立于硬件和操作系统的二进制格式，用于表示已编译的类和接口。

第5章详细说明了Java虚拟机的启动以及类和接口的加载、链接和初始化。

第6章详细介绍了Java虚拟机的指令集，按照操作码助记符的字母顺序呈现指令。

第7章给出了一个由操作码值索引的Java虚拟机操作码助记符表。

在 Java® 虚拟机规范第二版中，第2章概述了Java编程语言，该语言旨在支持Java虚拟机规范，但它本身并不是规范的一部分。
在 Java® 虚拟机规范第二版中，第8章详细介绍了解释Java虚拟机线程与共享主内存交互的低级操作。


## 1.4. 符号
在本规范中，我们指的是从 Java SE 平台 API 中提取的类和接口。 每当我们使用单个标识符 N 引用类或接口（示例中声明的那些除外）时，预期的引用是对包 java.lang 中名为 N 的类或接口。 我们对 java.lang 以外的包中的类或接口使用完全限定名称。

每当我们引用在包 java 或其任何子包中声明的类或接口时，预期的引用是指由引导类加载器加载的类或接口。

每当我们引用名为 java 的包的子包时，预期的引用就是由引导类加载器确定的那个子包。

## 1.5. 反馈
欢迎读者向 jls-jvms-spec-comments@openjdk.java.net 报告 Java® 虚拟机规范中的技术错误和歧义。

有关 javac（Java 编程语言的参考编译器）生成和操作类文件的问题可以发送至 compiler-dev@openjdk.java.net。