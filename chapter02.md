# 【JVM规范】第二章-JVM结构
> 本文是JVM19规范的个人中文译本，原文为 https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-2.html

本文档描述了一个抽象机器。 它没有描述 Java 虚拟机的任何特定实现。

要正确实现 Java 虚拟机，您只需要能够读取类文件格式并正确执行其中指定的操作即可。 

不属于Java虚拟机规范的实现细节会额外地限制实现者的创造力。 例如，运行时数据区的内存布局、使用的垃圾收集算法以及 Java 虚拟机指令的任何内部优化（例如，将它们转换为机器代码）都由实现者自行决定。

本规范中对 Unicode 的所有引用均根据 Unicode 标准版本 13.0 提供，可从 https://www.unicode.org/ 获得。

## 2.1. 类文件格式
一种由 Java 虚拟机执行的编译代码使用的、独立于硬件和操作系统的二进制格式表示，通常（但非必须）存储在文件中，称为类文件格式。 类文件格式精确地定义了类或接口的表示，包括在特定于平台的目标文件格式中可能被认为是理所当然的字节顺序等细节。

第 4 章，“类文件格式”，详细介绍了类文件格式。

## 2.2. 数据类型
与 Java 编程语言一样，Java 虚拟机对两种类型进行操作：原始类型和引用类型。 相应地，有两种值可以存储在变量中、作为参数传递、由方法返回并对其进行操作：原始值和引用值。

Java 虚拟机希望所有类型检查都应该在运行时之前完成，通常由编译器完成，而不必由 Java 虚拟机本身完成。 原始类型的值不需要标记或以其他方式检查，以确定它们在运行时的类型或与引用类型的值区分开来。 相反，Java 虚拟机的指令集使用针对对特定类型的值进行操作的指令来区分其操作数类型。 例如，iadd、ladd、fadd 和 dadd 都是 Java 虚拟机指令，它们将两个数值相加并产生数值结果，但每个指令都专门用于其操作数类型：分别为 int、long、float 和 double。 

Java 虚拟机包含对对象的显式支持。 对象是一个动态分配的类实例或者数组。 对对象的引用被认为具有 Java 虚拟机类型引用。 引用可以被认为是指向对象的指针。 一个对象可能存在多个引用。 对象总是通过引用进行操作、传递和测试。

## 2.3. 原始类型和值
Java 虚拟机支持的原始数据类型是数字类型、布尔类型 和 returnAddress 类型。

数字类型由整数类型和浮点类型  组成。

整数类型是：
- byte  , 8位有符号二进制补码制整数，默认值为0
- short , 16位有符号二进制补码制整数，默认值为0
- int   , 32位有符号二进制补码制整数，默认值为0
- long  , 64位有符号二进制补码制整数，默认值为0
- char  , 16位无符号整数，表示基本多语言平面中的Unicode码点，使用UTF-16编码，默认值为空码点（'\u0000'）

浮点数类型是：

- float  , 32位 IEEE-754 binary32 格式中可表示的值，其默认值为正零
- double , 64位 IEEE-754 binary64 格式中可表示的值，其默认值为正零

布尔类型的值编码真值 true 和 false，默认值为 false。

虚拟机规范第一版并未将 boolean 视为 Java 虚拟机类型。 但是，布尔值在 Java 虚拟机中的支持有限。 虚拟机规范第二版通过将布尔值视为一种类型来澄清这个问题。

returnAddress 类型的值是指向 Java 虚拟机指令操作码的指针。 在原始类型中，只有 returnAddress 类型不直接与 Java 编程语言类型相关联。

原始类型小结如下
1. 数字类型
    1. 整数类型
        1. byte 
        2. short
        3. int  
        4. long 
        5. char 
    2. 浮点类型
        1. float 
        2. double
2. 布尔类型
3. returnAddress类型


### 2.3.1. 整数类型和数值

|  类型   | 数值范围  |参考  |
|  ----  | ----  |----  |
| byte  | [-128,127] | [-2^7,2^7-1]|
| short  | [-32768,32767] |  [-2^16,2^16-1] |
| int  | [-2147483648,2147483647] |  [-2^31,2^31-1] |
| long  | [-9223372036854775808,9223372036854775807] | [-2^63,2^63-1] |
| char  | [0,65536] | [0,2^16-1] |

### 2.3.2. 浮点数类型和数值 
浮点类型是 float 和 double，它们在概念上与 IEEE 754 值和运算的 32 位 binary32 和 64 位 binary64 浮点格式相关联，如 IEEE 754 标准 中所指定。

在 Java SE 15 及更高版本中，Java 虚拟机使用 2019 版 IEEE 754 标准。

在 Java SE 15 之前，Java 虚拟机使用 1985 版的 IEEE 754 标准，其中 binary32 格式称为单精度格式，binary64 格式称为双精度格式。

IEEE 754 不仅包括由符号和大小组成的正数和负数，还包括`正零`和`负零`、`正无穷大`和`负无穷大`以及特殊的`非数字值`（以下简称 `NaN`）。 `NaN` 值用于表示某些无效操作的结果，例如零除以零。 float 和 double 类型的 NaN 常量都预定义为 `Float.NaN` 和 `Double.NaN`。

有限的非零浮点数使用公式  $s⋅m⋅2^{(e-N+1)}$ 表示
- s 的值为 +1 或 -1
- m 是一个小于$2^{N}$的正值
- e 是一个位于[Emin,Emax]=[$-(2^{(k-1)}-2)$,$2^{(k-1)}-1$]
- N 和 K是根据类型不同取不同的值

某些值可以以不止一种方式以这种形式表示。 例如，假设一个浮点类型的值 v 可以使用 s、m 和 e 的特定值以这种形式表示，那么如果 m 是偶数且 e 小于 $2^{K-1}$，那么可以 将 m 减半并将 e 增加 1 以生成相同值 v 的第二个表示。

如果 $m ≥ 2^{N-1}$，则这种形式的表示称为归一化； 否则表示表示是次正规的。 如果浮点类型的值不能以$m ≥ 2^{N-1}$ 的方式表示，则该值被称为次正规值，因为它的大小低于最小归一化值的大小。

下表总结了 float 和 double 的参数 N 和 K（以及派生参数 Emin 和 Emax）的约束。

|参数 |float |	double|
|---|---|---|
|N | 24 | 53 |
|K | 8  | 11|
|Emax |+127	|+1023|
|Emin |-126	|-1022|


除 NaN 外，浮点值都是有序的。 从小到大依次为负无穷大、负有限非零值、正负零值、正有限非零值、正无穷大。

IEEE 754 允许其每个 binary32 和 binary64 浮点格式有多个不同的 NaN 值。 但是，Java SE 平台通常将给定浮点类型的 NaN 值视为折叠成单个规范值，因此该规范通常将任意 NaN 视为规范值。

根据 IEEE 754，使用非 NaN 参数的浮点运算可能会生成 NaN 结果。 IEEE 754 指定了一组 NaN 位模式，但没有强制要求使用哪个特定的 NaN 位模式来表示 NaN 结果； 这留给硬件架构。 程序员可以创建具有不同位模式的 NaN 来编码，例如，追溯诊断信息。 这些 NaN 值可以分别使用 Float.intBitsToFloat 和 Double.longBitsToDouble 方法为 float 和 double 创建。 相反，要检查 NaN 值的位模式，Float.floatToRawIntBits 和 Double.doubleToRawLongBits 方法可分别用于 float 和 double。

正零和负零比较相等，但有其他操作可以区分它们； 例如，1.0 除以 0.0 产生正无穷大，但 1.0 除以 -0.0 产生负无穷大。

NaN 是无序的，因此如果其中一个或两个操作数是 NaN，则数值比较和数值相等性测试的值为 false。 特别是，当且仅当值为 NaN 时，一个值与其自身的数值相等性测试的值为 false。 如果任一操作数为 NaN，则数值不等式测试的值为真。

### 2.3.3. returnAddress 类型和值
returnAddress 类型由 Java 虚拟机的 jsr、ret 和 jsr_w 指令使用。

returnAddress 类型的值是指向 Java 虚拟机指令操作码的指针。 与数字原始类型不同，returnAddress 类型不对应于任何 Java 编程语言类型，并且不能被正在运行的程序修改。

### 2.3.4. 布尔类型
尽管 Java 虚拟机定义了一个 boolean 类型，但它只提供了非常有限的支持。 没有专门用于布尔值操作的 Java 虚拟机指令。 相反，Java 编程语言中对布尔值进行运算的表达式被编译为使用 Java 虚拟机 int 数据类型的值。

Java 虚拟机确实直接支持布尔数组。 它的 newarray 指令支持创建布尔数组。 使用字节数组指令 baload 和 bastore 访问和修改布尔类型的数组。

在 Oracle 的 Java 虚拟机实现中，Java 编程语言中的布尔数组被编码为 Java 虚拟机字节数组，每个布尔元素使用 8 位。

Java 虚拟机使用 1 表示 true 和 0 表示 false 对布尔数组组件进行编码。 编译器将 Java 编程语言布尔值映射到 Java 虚拟机类型 int 的值时，编译器必须使用相同的编码。

## 2.4. 引用类型和值
引用类型分为三种：类类型、数组类型和接口类型。 它们的值分别是对动态创建的类实例、数组或实现接口的类实例或数组的引用。

数组类型由具有单一维度的组件类型组成（其长度未由类型指定）。 数组类型的组件类型本身可能是数组类型。 如果从任何数组类型开始，考虑其组件类型，然后（如果它也是数组类型）该类型的组件类型，依此类推，最终必须到达不是数组类型的组件类型； 这称为数组类型的元素类型。 数组类型的元素类型必须是基本类型、类类型或接口类型。

引用值也可以是特殊的空引用，即没有对象的引用，这里用null表示。 空引用最初没有运行时类型，但可以转换为任何类型。 引用类型的默认值为 null。

本规范不要求具体值编码为 null。


## 2.5. 运行时数据区
Java 虚拟机定义了在程序执行期间使用的各种运行时数据区域。 其中一些数据区域是在 Java 虚拟机启动时创建的，只有在 Java 虚拟机退出时才被销毁。 其他数据区域是按线程分配的。 每线程数据区在创建线程时创建，并在线程退出时销毁。

### 2.5.1. 程序计数器
Java 虚拟机可以同时支持多个执行线程。 每个 Java 虚拟机线程都有自己的 pc（程序计数器）寄存器。 在任何时候，每个 Java 虚拟机线程都在执行单个方法的代码，即该线程的当前方法。 如果该方法不是本机方法，则 pc 寄存器包含当前正在执行的 Java 虚拟机指令的地址。 如果线程当前正在执行的方法是native，那么Java虚拟机的pc寄存器的值是未定义的。 Java 虚拟机的 pc 寄存器足够宽，可以容纳 returnAddress 或特定平台上的本机指针。

### 2.5.2. Java 虚拟机栈
每个 Java 虚拟机线程都有一个私有的 Java 虚拟机栈，与线程同时创建。 Java 虚拟机栈存储栈帧。 Java 虚拟机栈类似于 C 等常规语言的栈：它保存局部变量和部分结果，并在方法调用和返回中发挥作用。 因为 Java 虚拟机从不直接操作栈，除了压入和弹出帧外，所以帧可能是在堆上分配的。 Java 虚拟机栈的内存不需要是连续的。

在 Java® 虚拟机规范的第一版中，Java 虚拟机栈被称为 Java 栈。

此规范允许 Java 虚拟机栈具有固定大小或根据计算需要动态扩展和收缩。 如果 Java 虚拟机栈的大小是固定的，则每个 Java 虚拟机栈的大小可以在创建该栈时独立选择。

Java 虚拟机实现可以为程序员或用户提供对 Java 虚拟机栈初始大小的控制，以及在动态扩展或收缩 Java 虚拟机栈的情况下，对最大和最小尺寸的控制。

以下异常情况与 Java 虚拟机栈相关：

- 如果线程中的计算需要比允许的更大的 Java 虚拟机栈，Java 虚拟机将抛出 StackOverflowError。

- 如果 Java 虚拟机栈可以动态扩展，并且尝试扩展但没有足够的内存可用于实现扩展，或者如果没有足够的内存可用于为新线程创建初始 Java 虚拟机栈，则 Java 虚拟机 机器抛出 OutOfMemoryError。


### 2.5.3. 堆
Java 虚拟机有一个在所有 Java 虚拟机线程之间共享的堆。 堆是运行时数据区域，从中分配所有类实例和数组的内存。

堆是在虚拟机启动时创建的。 对象的堆存储由自动存储管理系统（称为垃圾收集器 GC）回收； 对象永远不会显式释放。 Java 虚拟机没有假定特定类型的自动存储管理系统，可以根据实现者的系统要求选择存储管理技术。 堆可以是固定大小的，也可以根据计算的需要进行扩展，如果不需要更大的堆，则可以收缩。 堆的内存不需要是连续的。

Java 虚拟机实现可以让程序员或用户控制堆的初始大小，如果堆可以动态扩展或收缩，还可以控制最大和最小堆大小。

以下异常情况与堆相关联：

- 如果计算需要的堆多于自动存储管理系统所能提供的堆，则 Java 虚拟机将抛出 OutOfMemoryError。

### 2.5.4. 方法区
Java 虚拟机有一个在所有 Java 虚拟机线程之间共享的方法区。 方法区类似于常规语言的编译代码的存储区，或者类似于操作系统进程中的“文本”段。 它存储每个类的结构，例如运行时常量池、字段和方法数据，以及方法和构造函数的代码，包括类和接口初始化以及实例初始化中使用的特殊方法。

方法区是在虚拟机启动时创建的。 尽管方法区在逻辑上是堆的一部分，但简单的实现可能会选择不对其进行垃圾收集或压缩。 本规范不强制要求方法区的位置或用于管理已编译代码的策略。 方法区的大小可以是固定的，也可以根据计算的需要进行扩展，如果不需要更大的方法区，则可以缩小。 方法区的内存不需要是连续的。

Java 虚拟机实现可以为程序员或用户提供对方法区初始大小的控制，以及在可变大小方法区的情况下，对最大和最小方法区大小的控制。

以下异常情况与方法区相关联：

- 如果方法区中的内存无法满足分配请求，Java 虚拟机将抛出 OutOfMemoryError。

### 2.5.5. 运行时常量池
运行时常量池是类文件中 constant_pool 表的按类或按接口的运行时表示。 它包含多种常量，从编译时已知的数字文字到必须在运行时解析的方法和字段引用。 运行时常量池的功能类似于传统编程语言的符号表，尽管它包含的数据范围比典型的符号表更广泛。

每个运行时常量池都是从 Java 虚拟机的方法区分配的。 类或接口的运行时常量池是在 Java 虚拟机创建类或接口时构建的。

以下异常情况与类或接口的运行时常量池的构造有关：

- 在创建类或接口时，如果构建运行时常量池需要的内存多于 Java 虚拟机方法区可用的内存，则 Java 虚拟机将抛出 OutOfMemoryError。

有关构建运行时常量池的信息，请参阅第 5 节（加载、链接和初始化）。

### 2.5.6. 本地方法栈
Java 虚拟机的实现可以使用传统的栈，通俗地称为“C 栈”，以支持本地方法（用 Java 编程语言以外的语言编写的方法）。 Java 虚拟机指令集的解释器的实现也可以使用本地方法栈，这些语言使用诸如 C 的语言。无法加载本地方法并且本身不依赖于传统栈的 Java 虚拟机实现不需要提供本地方法栈。 如果提供，本地方法栈通常在创建每个线程时按线程分配。

此规范允许本机方法栈具有固定大小或根据计算需要动态扩展和收缩。 如果本机方法栈的大小是固定的，则每个本机方法栈的大小可以在创建该栈时独立选择。

Java 虚拟机实现可以为程序员或用户提供对本地方法栈的初始大小的控制，以及在可变大小的本地方法栈的情况下，对最大和最小方法栈大小的控制。

以下异常情况与本机方法栈相关联：

- 如果线程中的计算需要比允许的更大的本机方法栈，Java 虚拟机将抛出 StackOverflowError。

- 如果本机方法栈可以动态扩展并且尝试本机方法栈扩展但可用内存不足，或者如果可用内存不足以为新线程创建初始本机方法栈，Java 虚拟机将抛出 OutOfMemoryError .


## 2.6. 栈帧
栈帧用于存储数据和部分结果，以及执行动态链接、方法返回值和分派异常。

每次调用方法时都会创建一个新栈帧。 栈帧在其方法调用完成时被销毁，无论该完成是正常的还是突然的（它抛出未捕获的异常）。 帧是从创建帧的线程的 Java 虚拟机堆栈分配的。 每个帧都有自己的局部变量数组、自己的操作数栈 和对当前方法类的运行时常量池的引用 .

可以使用附加的特定于实现的信息（例如调试信息(指令/代码行，变量名称等)）来扩展帧。

局部变量数组和操作数栈的大小在编译时确定，并随与帧关联的方法的代码一起提供。 因此，帧数据结构的大小仅取决于 Java 虚拟机的实现，并且这些结构的内存可以在方法调用时同时分配。

在给定的控制线程中的任何时候，只有一个栈帧（执行方法的栈帧）处于活动状态。 此帧称为当前帧，其方法称为当前方法。 定义当前方法的类是当前类。 对局部变量和操作数栈的操作通常参考当前帧。

如果一个栈帧的方法调用了另一个方法或者它的方法完成了，那么这个栈帧就不再是当前的。 调用方法时，将创建一个新栈帧，并在控制权转移到新方法时成为当前栈帧。 在方法返回时，当前帧将其方法调用的结果（如果有）传回给前一帧。 当前一帧成为当前帧时，当前帧将被丢弃。

请注意，一个线程创建的帧是该线程的本地帧，不能被任何其他线程引用。

### 2.6.1. 局部变量表
每个帧都包含一个称为局部变量表的变量数组。 帧的局部变量数组的长度在编译时确定，并以类或接口的二进制表示形式连同与帧关联的方法的代码一起提供。

单个局部变量可以保存类型为 boolean、byte、char、short、int、float、reference 或 returnAddress 的值。 一对局部变量可以保存 long 或 double 类型的值。

局部变量通过索引寻址。 第一个局部变量的索引为零。 当且仅当该整数介于零和比局部变量数组的大小小一之间时，该整数才被认为是局部变量数组的索引。

long 或 double 类型的值占用两个连续的局部变量。 这样的值只能使用较小的索引来寻址。 例如，在索引为 n 的局部变量数组中存储的一个 double 类型的值实际上占用了索引为 n 和 n+1 的局部变量； 但是，无法从索引 n+1 处加载局部变量。 可以存入。 但是，这样做会使局部变量 n 的内容无效。

Java 虚拟机不要求 n 是偶数。 直观地说，long 和 double 类型的值不需要在局部变量数组中进行 64 位对齐。 实现者可以自由决定使用为值保留的两个局部变量来表示此类值的适当方式。

Java 虚拟机使用局部变量在方法调用时传递参数。 在类方法调用中，任何参数都从局部变量 0 开始传递到连续的局部变量中。在实例方法调用中，局部变量 0 始终用于传递对调用实例方法的对象的引用（Java 中的 `this` 编程语言）。 从局部变量 1 开始，随后将任何参数传递到连续的局部变量中。

### 2.6.2. 操作数栈
每个帧都包含一个后进先出 (LIFO) 堆栈，称为操作数栈。 帧的操作数栈的最大深度在编译时确定，并随与帧关联的方法的代码一起提供。

在上下文清楚的情况下，我们有时会将`当前帧的操作数栈`简称为`操作数栈`。

创建包含它的帧时，操作数栈为空。 Java 虚拟机提供了将常量或值从局部变量或字段加载到操作数栈的指令。 其他 Java 虚拟机指令从操作数栈中获取操作数，对其进行运算，然后将结果推回操作数栈。 操作数栈还用于准备传递给方法的参数和接收方法结果。

例如，iadd 指令 将两个 int 值相加。 它要求要添加的 int 值是操作数栈的顶部两个值，由先前的指令推送到那里。 两个 int 值都从操作数栈中弹出。 它们被相加，它们的和被推回操作数栈。 子计算可以嵌套在操作数栈上，从而产生可由包含计算使用的值。

操作数栈上的每个条目都可以保存任何 Java 虚拟机类型的值，包括 long 类型或 double 类型的值。

必须以适合其类型的方式对操作数栈中的值进行操作。 例如，不可能压入两个 int 值并随后将它们视为 long 或压入两个 float 值并随后使用 iadd 指令将它们相加。 少量 Java 虚拟机指令（dup 指令 和 swap ）作为原始值在运行时数据区域上运行，而不考虑它们的特定类型； 这些指令的定义方式使其不能用于修改或分解单个值。 这些对操作数栈操作的限制是通过类文件验证强制执行的。

在任何时间点，操作数栈都有关联的深度，其中 long 或 double 类型的值贡献两个单位的深度，任何其他类型的值贡献一个单位。


### 2.6.3. 动态链接
每个帧都包含对当前方法类型的运行时常量池的引用，以支持方法代码的动态链接。 方法的类文件代码是指通过符号引用调用的方法和访问的变量。 动态链接将这些符号方法引用转换为具体方法引用，加载类以解析尚未定义的符号，并将变量访问转换为与这些变量的运行时位置关联的存储结构中的适当偏移量。

这种方法和变量的后期绑定使得方法使用的其他类中的更改不太可能破坏此代码。

### 2.6.4. 正常方法调用完成
无论是直接从 Java 虚拟机还是作为执行显式 throw 语句的结果,如果方法调用没有导致异常被抛出，则方法调用正常完成。如果当前方法的调用正常完成，则可以向调用方法返回一个值。 当调用的方法执行其中一个返回指令 时会发生这种情况，该指令的选择必须适合返回值的类型（如果有）。

当前帧在这种情况下用于恢复调用者的状态，包括其局部变量和操作数栈，调用者的程序计数器适当递增以跳过方法调用指令。 然后在调用方法的帧中正常继续执行，并将返回值（如果有）压入该帧的操作数栈。

### 2.6.5. 打断方法调用完成
如果在方法内执行 Java 虚拟机指令导致 Java 虚拟机抛出异常，并且该异常未在方法内处理，则打断方法调用完成。 执行 athrow 指令也会导致显式抛出异常，如果当前方法未捕获异常，则会导致打断方法调用完成。 打断完成的方法调用永远不会向其调用者返回值。


## 2.7. 对象的表示
Java 虚拟机不要求对象有任何特定的内部结构。

在 Oracle 的一些 Java 虚拟机实现中，对类实例的引用是指向句柄的指针，该句柄本身是一对指针：一个指针指向包含方法的表和表示对象类类型的指针，另一个指针指向堆中为对象数据分配的内存。

## 2.8. 浮点运算
Java 虚拟机包含 IEEE 754 标准中指定的浮点算法的子集。

在 Java SE 15 及更高版本中，Java 虚拟机使用 2019 版 IEEE 754 标准。 
在 Java SE 15 之前，Java 虚拟机使用 1985 版的 IEEE 754 标准，其中 binary32 格式称为单格式，binary64 格式称为双格式。

许多用于算术 和类型转换  的 Java 虚拟机指令都使用浮点数。 这些指令通常对应于 IEEE 754 操作：


| 字节码指令	| IEEE 754 操作 |解释|
| ---	|--- |---|
|dcmp<op> , fcmp<op> | compareQuietLess, compareQuietLessEqual, compareQuietGreater,  compareQuietGreaterEqual, compareQuietEqual, compareQuietNotEqual| 比较|
|dadd , fadd |	addition| 相加|
|dsub , fsub |	subtraction|相减|
|dmul , fmul |	multiplication|相乘|
|ddiv , fdiv |	division|相除|
|dneg , fneg |	negate|取反|
|i2d , i2f , l2d , l2f| convertFromInt|从证书转换|
|d2i , d2l , f2i , f2l |convertToIntegerTowardZero|转换成整数|
|d2f , f2d 	| convertFormat|互相转换|


Java 虚拟机指令和 IEEE 754 标准支持的浮点运算之间的主要区别是：

- 浮点余数指令 drem  和 frem 不对应于 IEEE 754 余数运算。 这些指令基于使用向零舍入策略的隐含除法； IEEE 754 余数是基于使用舍入到最近舍入策略的隐含除法。 （舍入策略在下面讨论。）

- 浮点取反指令 dneg   和 fneg  并不精确对应于 IEEE 754 取反操作。 特别是，这些指令不需要反转 NaN 操作数的符号位。

- Java 虚拟机的浮点指令不会抛出异常、陷阱或以其他方式发出 IEEE 754 无效操作、被零除、上溢、下溢或不精确等异常情况的信号。

- Java 虚拟机不支持 IEEE 754 信令浮点比较，并且没有信令 NaN 值。

- IEEE 754 包含与 Java 虚拟机中的舍入策略不对应的舍入方向属性。 Java 虚拟机不提供任何方法来更改给定浮点指令使用的舍入策略。

- Java 虚拟机不支持 IEEE 754 定义的 binary32 扩展和 binary64 扩展浮点格式。在操作或存储浮点值时，不能使用超出为 float 和 double 类型指定的扩展范围和扩展精度。


Java 虚拟机中一些没有相应指令的 IEEE 754 运算是通过 Math 和 StrictMath 类中的方法提供的，包括用于 IEEE 754 平方根运算的 sqrt 方法，用于 IEEE 754 fusedMultiplyAdd 运算的 fma 方法，以及用于 IEEE 754 余数运算。

Java 虚拟机需要支持 IEEE 754 次正规浮点数和渐进下溢，这使得证明特定数值算法的理想属性变得更加容易。

浮点运算是对实数运算的近似。 虽然实数的数量是无限的，但特定的浮点格式只有有限数量的值。 在 Java 虚拟机中，舍入策略是一种函数，用于将实数映射为给定格式的浮点值。 对于浮点格式可表示范围内的实数，实数行的连续段被映射到单个浮点值。 其值在数值上等于浮点值的实数被映射到该浮点值； 例如，实数 1.5 以给定格式映射到浮点值 1.5。 Java虚拟机定义了两种舍入策略，如下：

一、舍入到最接近的舍入策略适用于所有浮点指令，但 (i) 转换为整数值和 (ii) 余数除外。 在舍入到最接近的舍入策略下，不精确的结果必须舍入到最接近无限精确结果的可表示值； 如果两个最接近的可表示值同样接近，则选择最低有效位为零的值。

舍入到最近舍入策略对应于 IEEE 754 中二进制算法的默认舍入方向属性 roundTiesToEven。

roundTiesToEven 舍入方向属性在 IEEE 754 标准的 1985 版中称为“舍入到最近”舍入模式。 Java 虚拟机中的舍入策略就是以这种舍入方式命名的。

二、向零舍入策略适用于 (i) 通过 d2i、d2l、f2i 和 f2l 指令将浮点值转换为整数值，以及 (ii ) 浮点余数指令 drem 和 frem。 在向零舍入策略的舍入下，不精确的结果被舍入到最接近的可表示值，该值不大于无限精确的结果。 对于转换为整数，向零舍入策略的舍入相当于舍弃小数有效位的截断。

向零舍入策略的舍入对应于 IEEE 754 中二进制算术的 roundTowardZero 舍入方向属性。

在 IEEE 754 标准的 1985 版中，roundTowardZero 舍入方向属性被称为“向零舍入”舍入模式。 Java 虚拟机中的舍入策略就是以这种舍入方式命名的。

Java 1.0 和 1.1 要求对浮点表达式进行严格的计算。 严格评估意味着每个浮点操作数对应一个 IEEE 754 binary32 格式可表示的值，每个双精度操作数对应一个 IEEE 754 binary64 格式可表示的值，每个具有相应 IEEE 754 运算的浮点运算符与 IEEE 754 匹配 相同操作数的结果。

严格的评估提供了可预测的结果，但在 Java 1.0/1.1 时代常见的某些处理器系列的 Java 虚拟机实现中导致了性能问题。 因此，在 Java 1.2 到 Java SE 16 中，Java SE 平台允许 Java 虚拟机实现具有一个或两个与每个浮点类型相关联的值集。 float 类型与 float 值集和 float-extended-exponent 值集相关联，而 double 类型与 double 值集和 double-extended-exponent 值集相关联。 浮点值集对应于 IEEE 754 binary32 格式中可表示的值； float-extended-exponent 值集具有相同数量的精度位但更大的指数范围。 类似地，double 值集对应于 IEEE 754 binary64 格式中可表示的值； 双扩展指数值集具有相同数量的精度位数，但指数范围更大。 默认情况下允许使用扩展指数值集可以改善某些处理器系列的性能问题。

为了兼容性，Java 1.2 允许类文件禁止实现使用扩展指数值集。 类文件通过在方法声明中设置 ACC_STRICT 标志来表达这一点。 ACC_STRICT 限制了方法指令的浮点语义，以使用 float 操作数的 float 值集和 double 操作数的 double 值集，确保此类指令的结果完全可预测。 标记为 ACC_STRICT 的方法因此具有与 Java 1.0 和 1.1 中指定的相同的浮点语义。

在 Java SE 17 及更高版本中，Java SE 平台始终要求对浮点表达式进行严格的评估。 执行严格评估时遇到性能问题的处理器系列的新成员不再有这种困难。 本规范不再将 float 和 double 与上述四个值集相关联，并且 ACC_STRICT 标志不再影响浮点运算的评估。 为了兼容性，在主版本号为 46-60 的类文件中分配用于表示 ACC_STRICT 的位模式在主版本号大于 60 的类文件中未分配（即不表示任何标志）. Java 虚拟机的未来版本可能会在未来的类文件中为位模式分配不同的含义。


## 2.9. 特殊方法

### 2.9.1. 实例初始化方法
一个类有零个或多个实例初始化方法（构造方法），每个方法通常对应一个用 Java 编程语言编写的构造函数。

如果满足以下所有条件，则方法是实例初始化方法：
- 它是在类（而不是接口）中定义的
- 它有一个特殊的名字 `<init>`
- 它是void的

在类中，任何名为 `<init>` 的非 void 方法都不是实例初始化方法。 在接口中，任何名为 `<init>` 的方法都不是实例初始化方法。 此类方法不能由任何 Java 虚拟机指令调用，并且会被格式检查拒绝。

实例初始化方法的声明和使用受 Java 虚拟机的约束。 对于声明，方法的 access_flags 项和代码数组受到约束。 就用途而言，实例初始化方法只能由未初始化类实例上的 invokespecial 指令调用。

因为名称 `<init>` 在 Java 编程语言中不是有效的标识符，所以它不能直接用在用 Java 编程语言编写的程序中。

### 2.9.2. 类初始化方法
一个类或接口最多有一个类或接口初始化方法，并由调用该方法的 Java 虚拟机初始化。

如果满足以下所有条件，则方法是类或接口初始化方法：
- 它有一个特殊的名字`<clinit>`。
- 它是void的
- 在版本号为 51.0 或更高版本的类文件中，该方法设置了 ACC_STATIC 标志并且不带任何参数。

ACC_STATIC 的要求在 Java SE 7 中引入，在 Java SE 9 中要求不带参数。在版本号为 50.0 或以下的类文件中，名为 `<clinit>` 且为 void 的方法被视为类或接口初始化方法 不管其 ACC_STATIC 标志的设置或它是否接受参数。

类文件中名为 `<clinit>` 的其他方法不是类或接口初始化方法。 它们永远不会被 Java 虚拟机本身调用，不能被任何 Java 虚拟机指令调用，并且会被格式检查拒绝。

因为名称 `<clinit>` 在 Java 编程语言中不是有效的标识符，所以不能直接在用 Java 编程语言编写的程序中使用。


### 2.9.3. 签名多态方法
如果满足以下所有条件，则方法是签名多态的：

- 它在 java.lang.invoke.MethodHandle 类或 java.lang.invoke.VarHandle 类中声明。
- 它有一个 Object[] 类型的形式参数。
- 它设置了 ACC_VARARGS 和 ACC_NATIVE 标志。

Java 虚拟机在 invokevirtual 指令中对签名多态方法给予特殊处理，以便影响方法句柄的调用或影响对 java.lang.invoke.VarHandle 实例引用的变量的访问。

方法句柄是对底层方法、构造函数、字段或类似低级操作的动态强类型和直接可执行引用，具有参数或返回值的可选转换。 java.lang.invoke.VarHandle 的实例是对变量或变量族的动态强类型引用，包括静态字段、非静态字段、数组元素或堆外数据结构的组件。 有关详细信息，请参阅 Java SE 平台 API 中的 java.lang.invoke 包。


## 2.10. 异常
Java 虚拟机中的异常由类 Throwable 或其子类之一的实例表示。 抛出异常会导致从抛出异常的点立即进行非本地控制转移。

大多数异常是同步发生的，是它们发生的线程的操作的结果。 相比之下，异步异常可能发生在程序执行的任何时刻。 Java 虚拟机出于以下三个原因之一抛出异常：
- 执行了抛出指令 (athrow)。
- Java 虚拟机同步检测到异常执行情况。 这些异常不会在程序中的任意点抛出，而只会在执行以下指令后同步抛出：
    - 将异常指定为可能的结果，例如：
        - 当指令包含违反 Java 编程语言语义的操作时，例如在数组边界之外进行索引。
        - 当加载或链接部分程序时发生错误。
    - 导致超出资源的某些限制，例如当使用过多的内存时。
- 发生异步异常是因为：
    - 调用了类 Thread 或 ThreadGroup 的停止方法
    - Java 虚拟机实现中发生内部错误。

一个线程可以调用停止方法来影响另一个线程或指定线程组中的所有线程。 它们是异步的，因为它们可能发生在其他线程或多个线程执行的任何时刻。 内部错误被认为是异步的。

Java 虚拟机可能允许在抛出异步异常之前进行少量但有限制的执行。 允许这种延迟以允许优化的代码在遵守 Java 编程语言的语义的同时在处理它们的实际点处检测并抛出这些异常。

一个简单的实现可能会在每个控制传输指令处轮询异步异常。 由于程序的大小是有限的，这就限制了检测异步异常的总延迟。 由于在控制传输之间不会发生异步异常，因此代码生成器具有一定的灵活性，可以在控制传输之间重新排序计算以获得更高的性能。 

Java 虚拟机抛出的异常是精确的：当控制转移发生时，在抛出异常之前执行的指令的所有效果必须看起来已经发生。 在抛出异常的点之后出现的任何指令可能看起来都已被评估。 如果优化代码已经推测性地执行了异常发生点之后的一些指令，则此类代码必须准备好将这种推测性执行从程序的用户可见状态中隐藏起来。

Java 虚拟机中的每个方法都可能与零个或多个异常处理程序相关联。 异常处理程序指定实现异常处理程序处于活动状态的方法的 Java 虚拟机代码的偏移范围，描述异常处理程序能够处理的异常类型，并指定要处理的代码的位置 那个例外。 如果导致异常的指令的偏移量在异常处理程序的偏移量范围内，并且异常类型与异常处理程序处理的异常类是同一类或者是异常类的子类，则异常与异常处理程序相匹配。 抛出异常时，Java 虚拟机会在当前方法中搜索匹配的异常处理程序。 如果找到匹配的异常处理程序，系统将分支到匹配的处理程序指定的异常处理代码。

如果在当前方法中没有找到这样的异常处理程序，则打断当前方法调用会完成。 在打断时，当前方法调用的操作数栈和局部变量被丢弃，它的帧被弹出，恢复调用方法的帧。 然后在调用者栈的上下文中重新抛出异常，依此类推，继续方法调用链。 如果在到达方法调用链的顶部之前没有找到合适的异常处理程序，则终止抛出异常的线程的执行。

在方法的异常处理程序中搜索匹配项的顺序很重要。 在类文件中，每个方法的异常处理程序都存储在一个表中。 在运行时，当抛出异常时，Java 虚拟机按照它们在类文件中相应异常处理程序表中出现的顺序，从该表的开头开始搜索当前方法的异常处理程序。

请注意，Java 虚拟机不强制嵌套方法的异常表条目或对其进行任何排序。 Java 编程语言的异常处理语义只能通过与编译器的合作来实现。 当通过其他方式生成类文件时，定义的搜索过程可确保所有 Java 虚拟机实现的行为一致。

## 2.11. 指令集总结
Java 虚拟机指令由一个单字节操作码组成，该操作码指定要执行的操作，后跟零个或多个操作数，提供操作使用的参数或数据。 许多指令没有操作数，只包含一个操作码。

忽略异常，Java 虚拟机解释器的内部循环是有效的
```java
do {
    自动计算程序计数器并读取操作码
    如果需要操作数，就读取操作数
    执行操作码表示的动作
} while (there is more to do);
```

操作数的数量和大小由操作码决定。 如果一个操作数的大小超过一个字节，那么它以`大端`顺序存储——高位字节在前。 例如，局部变量的无符号 16 位索引存储为两个无符号字节，byte1 和 byte2，因此其值为 `(byte1 << 8) | byte2`。

字节码指令流只是单字节对齐的。 两个例外是 lookupswitch 和 tableswitch 指令，它们被填充以强制其某些操作数在 4 字节边界上进行内部对齐。

将 Java 虚拟机操作码限制为一个字节并放弃编译代码中的数据对齐的决定反映了一种有意识的偏向于紧凑性的倾向，这可能是以朴素实现中的一些性能为代价的。 一个字节的操作码也限制了指令集的大小。 不假设数据对齐意味着大于1字节的数据必须在运行时从单个字节构造。
（译者注，类文件是非常紧凑的，优点是很明显的，可以减小文件的尺寸，利于网络间传输，但是多字节的数据就必须在运行中组装出来，执行的效率收到影响）



### 2.11.1. 类型 和 Java 虚拟机
Java 虚拟机指令集中的大多数指令都对有关它们执行的操作的类型信息进行编码。 例如，iload 指令 将必须为 int 的局部变量的内容加载到操作数栈中。 fload 指令 对浮点值执行相同的操作。 这两条指令可能具有相同的实现，但具有不同的操作码。

对于大多数类型化指令，指令类型在操作码助记符中用一个字母明确表示：i 表示 int 操作，l 表示 long，s 表示短，b 表示 byte，c 表示 char，f 表示 float，d 表示 double , 供参考。 一些类型明确的指令在它们的助记符中没有类型字母。 例如，arraylength 始终对数组对象进行操作。 一些指令，例如 goto，无条件控制转移，不对类型化操作数进行操作。

鉴于 Java 虚拟机的一字节操作码大小，将类型编码为操作码对其指令集的设计施加了压力。 如果每条类型化指令都支持 Java 虚拟机的所有运行时数据类型，那么指令的数量将超过一个字节所能表示的数量。 相反，Java 虚拟机的指令集为某些操作提供了较低级别的类型支持。 换句话说，指令集是故意不正交的。 必要时，可以使用单独的指令在不受支持和受支持的数据类型之间进行转换。


下表总结了 Java 虚拟机指令集中的类型支持。 通过用类型列中的字母替换操作码列中指令模板中的 T 来构建具有类型信息的特定指令。 如果某些指令模板和类型的类型列为空白，则不存在支持该类型操作的指令。 比如int类型有加载指令iload，而byte类型没有加载指令。

请注意，下表 中的大多数指令没有整数类型 byte、char 和 short 的形式。 没有一个具有布尔类型的形式。 编译器使用 Java 虚拟机指令对 byte 和 short 类型的文字值负载进行编码，这些指令在编译时或运行时将这些值符号扩展为 int 类型的值。 使用指令对 boolean 和 char 类型的文字值负载进行编码，这些指令在编译时或运行时将文字零扩展为 int 类型的值。 同样，使用 Java 虚拟机指令对 boolean、byte、short 和 char 类型值数组的加载进行编码，这些指令将值符号扩展或零扩展为 int 类型的值。 因此，对实际类型 boolean、byte、char 和 short 的值的大多数操作都由对计算类型 int 的值进行操作的指令正确执行。

<table border="1">
   <thead>
      <tr>
         <th>opcode</th>
         <th><code>byte</code></th>
         <th><code>short</code></th>
         <th><code>int</code></th>
         <th><code>long</code></th>
         <th><code>float</code></th>
         <th><code>double</code></th>
         <th><code>char</code></th>
         <th><code>reference</code></th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td><span><em>Tipush</em></span></td>
         <td><span><em>bipush</em></span></td>
         <td><span><em>sipush</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tconst</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>iconst</em></span></td>
         <td><span><em>lconst</em></span></td>
         <td><span><em>fconst</em></span></td>
         <td><span><em>dconst</em></span></td>
         <td>&nbsp;</td>
         <td><span><em>aconst</em></span></td>
      </tr>
      <tr>
         <td><span><em>Tload</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>iload</em></span></td>
         <td><span><em>lload</em></span></td>
         <td><span><em>fload</em></span></td>
         <td><span><em>dload</em></span></td>
         <td>&nbsp;</td>
         <td><span><em>aload</em></span></td>
      </tr>
      <tr>
         <td><span><em>Tstore</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>istore</em></span></td>
         <td><span><em>lstore</em></span></td>
         <td><span><em>fstore</em></span></td>
         <td><span><em>dstore</em></span></td>
         <td>&nbsp;</td>
         <td><span><em>astore</em></span></td>
      </tr>
      <tr>
         <td><span><em>Tinc</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>iinc</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Taload</em></span></td>
         <td><span><em>baload</em></span></td>
         <td><span><em>saload</em></span></td>
         <td><span><em>iaload</em></span></td>
         <td><span><em>laload</em></span></td>
         <td><span><em>faload</em></span></td>
         <td><span><em>daload</em></span></td>
         <td><span><em>caload</em></span></td>
         <td><span><em>aaload</em></span></td>
      </tr>
      <tr>
         <td><span><em>Tastore</em></span></td>
         <td><span><em>bastore</em></span></td>
         <td><span><em>sastore</em></span></td>
         <td><span><em>iastore</em></span></td>
         <td><span><em>lastore</em></span></td>
         <td><span><em>fastore</em></span></td>
         <td><span><em>dastore</em></span></td>
         <td><span><em>castore</em></span></td>
         <td><span><em>aastore</em></span></td>
      </tr>
      <tr>
         <td><span><em>Tadd</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>iadd</em></span></td>
         <td><span><em>ladd</em></span></td>
         <td><span><em>fadd</em></span></td>
         <td><span><em>dadd</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tsub</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>isub</em></span></td>
         <td><span><em>lsub</em></span></td>
         <td><span><em>fsub</em></span></td>
         <td><span><em>dsub</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tmul</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>imul</em></span></td>
         <td><span><em>lmul</em></span></td>
         <td><span><em>fmul</em></span></td>
         <td><span><em>dmul</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tdiv</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>idiv</em></span></td>
         <td><span><em>ldiv</em></span></td>
         <td><span><em>fdiv</em></span></td>
         <td><span><em>ddiv</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Trem</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>irem</em></span></td>
         <td><span><em>lrem</em></span></td>
         <td><span><em>frem</em></span></td>
         <td><span><em>drem</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tneg</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>ineg</em></span></td>
         <td><span><em>lneg</em></span></td>
         <td><span><em>fneg</em></span></td>
         <td><span><em>dneg</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tshl</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>ishl</em></span></td>
         <td><span><em>lshl</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tshr</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>ishr</em></span></td>
         <td><span><em>lshr</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tushr</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>iushr</em></span></td>
         <td><span><em>lushr</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tand</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>iand</em></span></td>
         <td><span><em>land</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tor</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>ior</em></span></td>
         <td><span><em>lor</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Txor</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>ixor</em></span></td>
         <td><span><em>lxor</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>i2T</em></span></td>
         <td><span><em>i2b</em></span></td>
         <td><span><em>i2s</em></span></td>
         <td>&nbsp;</td>
         <td><span><em>i2l</em></span></td>
         <td><span><em>i2f</em></span></td>
         <td><span><em>i2d</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>l2T</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>l2i</em></span></td>
         <td>&nbsp;</td>
         <td><span><em>l2f</em></span></td>
         <td><span><em>l2d</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>f2T</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>f2i</em></span></td>
         <td><span><em>f2l</em></span></td>
         <td>&nbsp;</td>
         <td><span><em>f2d</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>d2T</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>d2i</em></span></td>
         <td><span><em>d2l</em></span></td>
         <td><span><em>d2f</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tcmp</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>lcmp</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tcmpl</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>fcmpl</em></span></td>
         <td><span><em>dcmpl</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>Tcmpg</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>fcmpg</em></span></td>
         <td><span><em>dcmpg</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
      </tr>
      <tr>
         <td><span><em>if_TcmpOP</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>if_icmpOP</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>if_acmpOP</em></span></td>
      </tr>
      <tr>
         <td><span><em>Treturn</em></span></td>
         <td>&nbsp;</td>
         <td>&nbsp;</td>
         <td><span><em>ireturn</em></span></td>
         <td><span><em>lreturn</em></span></td>
         <td><span><em>freturn</em></span></td>
         <td><span><em>dreturn</em></span></td>
         <td>&nbsp;</td>
         <td><span><em>areturn</em></span></td>
      </tr>
   </tbody>
</table>


下表 总结了 Java 虚拟机实际类型和 Java 虚拟机计算类型之间的映射。

某些 Java 虚拟机指令（例如 pop 和 swap）在操作数栈上进行操作，而不考虑类型； 然而，此类指令仅限于用于某些计算类型类别的值，也在下表 中给出。

<table border="1">
   <thead>
      <tr>
         <th>Actual type</th>
         <th>Computational type</th>
         <th>Category</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td><code>boolean</code></td>
         <td><code>int</code></td>
         <td>1</td>
      </tr>
      <tr>
         <td><code>byte</code></td>
         <td><code>int</code></td>
         <td>1</td>
      </tr>
      <tr>
         <td><code>char</code></td>
         <td><code>int</code></td>
         <td>1</td>
      </tr>
      <tr>
         <td><code>short</code></td>
         <td><code>int</code></td>
         <td>1</td>
      </tr>
      <tr>
         <td><code>int</code></td>
         <td><code>int</code></td>
         <td>1</td>
      </tr>
      <tr>
         <td><code>float</code></td>
         <td><code>float</code></td>
         <td>1</td>
      </tr>
      <tr>
         <td><code>reference</code></td>
         <td><code>reference</code></td>
         <td>1</td>
      </tr>
      <tr>
         <td><code>returnAddress</code></td>
         <td><code>returnAddress</code></td>
         <td>1</td>
      </tr>
      <tr>
         <td><code>long</code></td>
         <td><code>long</code></td>
         <td>2</td>
      </tr>
      <tr>
         <td><code>double</code></td>
         <td><code>double</code></td>
         <td>2</td>
      </tr>
   </tbody>
</table>



### 2.11.2. 加载和存储指令
加载和存储指令 在局部变量表 和 Java 虚拟机栈的操作数栈之间传输值：

- 将局部变量加载到操作数栈：`iload`、`iload_<n>`、`lload`、`lload_<n>`、`fload`、`fload_<n>`、`dload`、`dload_<n>`、`aload`、`aload_<n>`。

- 将操作数栈中的值存储到局部变量中：`istore`、`istore_<n>`、`lstore`、`lstore_<n>`、`fstore`、`fstore_<n>`、`dstore`、`dstore_<n>`、`astore`、`astore_<n>`。

- 将常量加载到操作数栈：`bipush`、`sipush`、`ldc`、`ldc_w`、`ldc2_w`、`aconst_null`、`iconst_m1`、`iconst_<i>`、`lconst_<l>`、`fconst_<f>`、`dconst_<d>`。

- 使用更宽的索引或更大的直接操作数访问更多局部变量：`wide`。

访问对象字段和数组元素的指令也将数据传入和传出操作数栈。

上面显示的指令助记符在尖括号之间带有尾随字母（例如，`iload_<n>`）表示指令族（在 `iload_<n>` 的情况下具有成员 iload_0、iload_1、iload_2 和 iload_3）。 此类指令族是采用一个操作数的附加通用指令 (iload) 的特化。 对于专用指令，操作数是隐式的，不需要存储或取出。 语义在其他方面是相同的（iload_0 与操作数为 0 的 iload 意思相同）。 尖括号之间的字母指定该系列指令的隐式操作数的类型：对于 `<n>`，一个非负整数； 对于`<i>`，一个整数； 对于 `<l>`，一个长整数； 对于 `<f>`，一个浮点数； 对于 `<d>`，一个双精度值。 int 类型的形式在许多情况下用于对 byte、char 和 short 类型的值执行操作。

整个规范中都使用了这种指令族符号。

### 2.11.3. 算术指令
算术指令计算的结果通常是操作数堆栈上两个值的函数，将结果推回操作数堆栈。 有两种主要的算术指令：对整数值进行运算的指令和对浮点值进行运算的指令。 在每一种类型中，算术指令专门用于 Java 虚拟机数字类型。 不直接支持对 byte、short 和 char 类型的值（或布尔类型的值进行整数运算； 这些操作由对 int 类型进行操作的指令处理。 整数和浮点指令在溢出和被零除时的行为也不同。 算术指令如下：

- 加:`iadd`,`ladd`,`fadd`,`dadd`.
- 减:`isub`,`lsub`,`fsub`,`dsub`.
- 乘:`imul`,`lmul`,`fmul`,`dmul`.
- 除:`idiv`,`ldiv`,`fdiv`,`ddiv`.
- 余:`irem`,`lrem`,`frem`,`drem`.
- 取反:`ineg`,`lneg`,`fneg`,`dneg`.
- 位移:`ishl`,`ishr`,`iushr`,`lshl`,`lshr`,`lushr`.
- 位或:`ior`,`lor`.
- 位与:`iand`,`land`.
- 位异或:`ixor`,`lxor`.
- 自增:`iinc`.
- 比较:`dcmpg`,`dcmpl`,`fcmpg`,`fcmpl`,`lcmp`.

Java 编程语言运算符对整数和浮点值的语义直接由 Java 虚拟机指令集的语义支持。

Java 虚拟机在对整数数据类型进行操作时不会指示溢出。 唯一可以抛出异常的整数运算是整数除法指令（`idiv` 和 `ldiv`）和整数余数指令（`irem` 和 `lrem`），如果除数为零，它们将抛出 ArithmeticException。

Java 虚拟机在对浮点数据类型进行操作时不会指示上溢或下溢。 也就是说，浮点指令永远不会导致 Java 虚拟机抛出运行时异常（不要与 IEEE 754 浮点异常混淆）。 溢出的操作产生带符号的无穷大； 下溢的操作产生低于正常值或带符号的零； 没有唯一的数学定义结果的运算会产生 NaN。 所有以 NaN 作为操作数的数值运算都会产生 NaN 作为结果。

对 long (`lcmp`) 类型值的比较执行带符号的比较。

使用 IEEE 754 非信号比较执行浮点类型（`dcmpg`、`dcmpl`、`fcmpg`、`fcmpl`）值的比较。


### 2.11.4. 类型转换指令
类型转换指令允许在 Java 虚拟机数字类型之间进行转换。 这些可用于在用户代码中实现显式转换或缓解 Java 虚拟机指令集中缺乏正交性的问题。

Java 虚拟机直接支持以下扩展数字转换：
- int to long, float, or double
- long to float or double
- float to double

扩大的数值转换指令是 `i2l`、`i2f`、`i2d`、`l2f`、`l2d` 和 `f2d`。 鉴于类型指令的命名约定和双关语使用 2 表示“to”，这些操作码的助记符很简单。 例如，`i2d` 指令将 int 值转换为 double。

大多数扩大数值转换不会丢失有关数值总体大小的信息。 实际上，从 int 到 long 以及从 int 到 double 的转换根本不会丢失任何信息； 数值被准确保留。 从 float 扩大到 double 的转换也准确地保留了数值。

从 int 到 float，或从 long 到 float，或从 long 到 double 的转换可能会丢失精度，也就是说，可能会丢失值的一些最低有效位； 生成的浮点值是整数值的正确舍入版本，使用舍入到最接近的舍入策略。

尽管可能会丢失精度，但扩大数字转换永远不会导致 Java 虚拟机抛出运行时异常（不要与 IEEE 754 浮点异常混淆）。

从 int 到 long 的扩展数字转换只是符号扩展 int 值的二进制补码表示以填充更宽的格式。 一个 char 到整数类型的扩展数字转换零扩展 char 值的表示以填充更宽的格式。

请注意，不存在从整数类型 byte、char 和 short 到 int 类型的扩展数字转换。 如上中所述，byte、char 和 short 类型的值在内部扩展为 int 类型，从而使这些转换成为隐式的。

Java 虚拟机还直接支持以下窄化数字转换：
- int to byte, short, or char
- long to int
- float to int or long
- double to int, long, or float

缩小数值转换指令是 `i2b`、`i2c`、`i2s`、`l2i`、`f2i`、`f2l`、`d2i`、`d2l` 和 `d2f`。 缩小数字转换可能会导致值的符号不同、数量级不同或两者兼而有之； 它可能因此失去精度。

从 int 或 long 到整数类型 T 的缩小数字转换简单地丢弃除 n 个最低位以外的所有位，其中 n 是用于表示类型 T 的位数。这可能导致结果值不具有相同的符号 作为输入值。

在将浮点值缩小为整数类型 T 的数值转换中，其中 T 为 int 或 long，浮点值转换如下：

- 如果浮点值是 NaN，转换的结果是 int 或 long 0。
- 否则，如果浮点值不是无穷大，则使用向零舍入策略将浮点值舍入为整数值 V。 有两种情况：
    - 如果 T 是 long 并且这个整数值可以表示为 long，那么结果就是 long 值 V。
    - 如果 T 是 int 类型，并且这个整数值可以表示为 int，那么结果就是 int 值 V。
- 除此以外：
    - 该值必须太小（大负值或负无穷大），结果是 int 或 long 类型的最小可表示值。
    - 或者该值必须太大（幅度很大的正值或正无穷大），结果是 int 或 long 类型的最大可表示值。


从 double 到 float 的缩小数字转换的行为符合 IEEE 754。使用舍入到最接近的舍入策略正确舍入结果。 太小而无法表示为浮点数的值将转换为浮点类型的正或负零； 太大而无法表示为浮点数的值将转换为正无穷大或负无穷大。 double NaN 总是转换为 float NaN。

尽管可能会发生上溢、下溢或精度损失，但缩小数字类型之间的转换永远不会导致 Java 虚拟机抛出运行时异常（不要与 IEEE 754 浮点异常混淆）。

### 2.11.5. 对象创建和操作
尽管类实例和数组都是对象，但 Java 虚拟机使用不同的指令集创建和操作类实例和数组：

- 创建一个新的类实例：`new`。
- 创建一个新数组：`newarray`、`anewarray`、`multiawarray`。
- 访问类的字段（静态字段，称为类变量）和类实例的字段（非静态字段，称为实例变量）：`getstatic`、`putstatic`、`getfield`、`putfield`。
- 将数组组件加载到操作数堆栈：`baload`、`caload`、`saload`、`iaload`、`laload`、`faload`、`daload`、`aaload`。
- 将操作数堆栈中的值存储为数组组件：`bastore`、`castore`、`sastore`、`iastore`、`lastore`、`fastore`、`dastore`、`aastore`。
- 获取数组的长度：`arraylength`。
- 检查类实例或数组的属性：`instanceof`、`checkcast`。

### 2.11.6. 操作数栈管理指令
为直接操作操作数堆栈提供了许多指令：`pop`、`pop2`、`dup`、`dup2`、`dup_x1`、`dup2_x1`、`dup_x2`、`dup2_x2`、`swap`。

### 2.11.7. 控制转移指令
控制转移指令有条件或无条件地使 Java 虚拟机继续执行控制转移指令之后的指令以外的指令。 他们是：

- 条件分支：`ifeq`、`ifne`、`iflt`、`ifle`、`ifgt`、`ifge`、`ifnull`、`ifnonnull`、`if_icmpeq`、`if_icmpne`、`if_icmplt`、`if_icmple`、`if_icmpgt` `if_icmpge`、`if_acmpeq`、`if_acmpne`。

- 复合条件分支：`tableswitch`、`lookupswitch`。

- 无条件分支：`goto`、`goto_w`、`jsr`、`jsr_w`、`ret`。

Java 虚拟机具有不同的指令集，这些指令集在与 int 和引用类型的数据进行比较时有条件地分支。 它还具有用于测试 null 引用的不同条件分支指令，因此不需要为 null 指定具体值。

使用 int 比较指令执行 boolean、byte、char 和 short 类型数据之间比较的条件分支。 比较数据类型 long、float 或 double 的条件分支是使用比较数据并生成 int 比较结果的指令启动的。 随后的 int 比较指令测试此结果并影响条件分支。 由于强调 int 比较，Java 虚拟机为类型 int 提供了丰富的条件分支指令补充。

所有 int 条件控制传输指令都执行带符号的比较。

### 2.11.8. 方法调用和返回指令
以下五个指令调用方法：

- `invokevirtual` 调用对象的实例方法，调度对象的（虚拟）类型。 这是 Java 编程语言中的正常方法分派。

- `invokeinterface` 调用接口方法，搜索由特定运行时对象实现的方法以找到合适的方法。

- `invokespecial` 调用需要特殊处理的实例方法，可以是实例初始化方法，也可以是当前类或其超类型的方法。

- `invokestatic` 调用命名类中的类（静态）方法。

- `invokedynamic` 调用作为绑定到 invokedynamic 指令的调用站点对象的目标的方法。 作为在第一次执行指令之前运行引导方法的结果，调用站点对象被 Java 虚拟机绑定到 invokedynamic 指令的特定词法出现。 因此，与调用方法的其他指令不同，invokedynamic 指令的每次出现都具有唯一的链接状态。

方法返回指令，按返回类型区分，有ireturn（用于返回boolean、byte、char、short、int类型的值）、lreturn、freturn、dreturn、areturn。 此外，return 指令用于从声明为 void 的方法、实例初始化方法以及类或接口初始化方法中返回。

###  2.11.9. 抛出异常
使用 `athrow` 指令以编程方式抛出异常。 如果检测到异常情况，各种 Java 虚拟机指令也可以抛出异常。

### 2.11.10. 同步
Java 虚拟机通过一个同步结构支持方法和方法内指令序列的同步：`monitor`。

方法级同步是隐式执行的，作为方法调用和返回的一部分。 同步方法在运行时常量池的 method_info 结构中通过 ACC_SYNCHRONIZED 标志进行区分，该标志由方法调用指令检查。 当调用设置了 ACC_SYNCHRONIZED 的方法时，执行线程进入监视器，调用方法本身，然后退出监视器，无论方法调用是正常完成还是突然完成。 在执行线程拥有监视器期间，没有其他线程可以进入它。 如果在调用synchronized方法时抛出异常，并且synchronized方法没有处理异常，则在synchronized方法重新抛出异常之前自动退出该方法的监视器。

指令序列的同步通常用于对 Java 编程语言的同步块进行编码。 Java 虚拟机提供了 `monitorenter` 和 `monitorexit` 指令来支持这种语言结构。 同步块的正确实现需要来自以 Java 虚拟机为目标的编译器的合作。

结构化锁定是指在方法调用期间，给定监视器上的每个出口都与该监视器上的前一个条目相匹配的情况。 由于无法保证提交给 Java 虚拟机的所有代码都将执行结构化锁定，因此允许但不要求 Java 虚拟机的实现强制执行以下两条保证结构化锁定的规则。 假设 T 是一个线程，M 是一个监视器。 然后：

1. 在方法调用期间，T 在 M 上执行的监视器条目数必须等于在方法调用期间 T 在 M 上执行的监视器退出数，无论方法调用是正常完成还是突然完成。

2. 在方法调用期间，自方法调用以来 T 在 M 上执行的监控器退出次数绝不能超过自方法调用以来 T 在 M 上执行的监控器条目数。

请注意，在调用同步方法时由 Java 虚拟机自动执行的监视器进入和退出被认为是在调用方法的调用期间发生的。

## 2.12. 类库
Java 虚拟机必须为 Java SE 平台类库的实现提供足够的支持。 这些库中的一些类离不开Java虚拟机的配合是无法实现的。

可能需要 Java 虚拟机特殊支持的类包括支持：

- 反射，例如包java.lang.reflect中的类和类Class。

- 加载和创建类或接口。 最明显的例子是类 ClassLoader。

- 类或接口的链接和初始化。 上面引用的示例类也属于这一类。

- 安全性，例如包java.security中的类和其他类，例如SecurityManager。

- 多线程，例如 Thread 类。

- 弱引用，例如包 java.lang.ref 中的类。

上面的列表是说明性的，而不是全面的。 这些类或它们提供的功能的详尽列表超出了本规范的范围。 有关详细信息，请参阅 Java SE 平台类库的规范。

## 2.13. public设计，private实现
到目前为止，该规范已经勾勒出 Java 虚拟机的公共视图：类文件格式和指令集。 这些组件对于 Java 虚拟机的硬件、操作系统和实现独立性至关重要。 实现者可能更愿意将它们视为一种在每个实现 Java SE 平台的主机之间安全地通信程序片段的方法，而不是将其视为要严格遵循的蓝图。

了解公共设计和私有实现之间的界限在哪里很重要。 Java 虚拟机实现必须能够读取类文件，并且必须准确地实现其中的 Java 虚拟机代码的语义。 这样做的一种方法是将此文档作为规范并逐字执行该规范。 但是，实现者在本规范的约束范围内修改或优化实现也是完全可行和可取的。 只要可以读取类文件格式并保持其代码的语义，实现者就可以以任何方式实现这些语义。 “幕后”是实现者的事，只要认真维护正确的外部接口即可。

有一些例外：调试器、分析器和即时代码生成器都可能需要访问通常被认为是“底层”的 Java 虚拟机元素。 在适当的情况下，Oracle 与其他 Java 虚拟机实现者和工具供应商合作，开发 Java 虚拟机的通用接口以供此类工具使用，并在整个行业推广这些接口。

实现者可以使用这种灵活性来定制 Java 虚拟机实现以实现高性能、低内存使用或可移植性。 在给定的实现中什么有意义取决于该实现的目标。 实现选项的范围包括以下内容：

在加载时或执行期间将 Java 虚拟机代码翻译成另一个虚拟机的指令集。

在加载时或执行期间将 Java 虚拟机代码转换为主机 CPU 的本机指令集（有时称为即时或 JIT 代码生成）。

精确定义的虚拟机和目标文件格式的存在不需要显着限制实现者的创造力。 Java 虚拟机旨在支持许多不同的实现，提供新的和有趣的解决方案，同时保持实现之间的兼容性。