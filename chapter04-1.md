# 【JVM 规范】第四章-类文件结构-1

> 本文是 JVM19 规范的个人中文译本，原文为 https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html

本章描述了 Java 虚拟机的类文件格式。 每个类文件都包含单个类、接口或模块的定义。 尽管类、接口或模块不需要在文件中逐字包含外部表示（例如，因为类是由类加载器生成的），我们将通俗地引用类、接口或模块的任何有效表示 作为类文件格式。

类文件由 8 位字节流组成。 16 位和 32 位量分别通过读取两个和四个连续的 8 位字节来构造。 多字节数据项始终以**大端**顺序存储，其中高字节在前。 本章定义了数据类型 `u1`、`u2` 和 `u4`，分别表示*无符号*的 1、2 或 4 字节数量。

在 Java SE Platform API 中，类文件格式由接口 java.io.DataInput 和 java.io.DataOutput 以及 java.io.DataInputStream 和 java.io.DataOutputStream 等类支持。 例如，u1、u2、u4 类型的值可以通过接口 java.io.DataInput 的 readUnsignedByte、readUnsignedShort、readInt 等方法读取。

本章介绍使用类 C 结构符号编写的伪结构的类文件格式。 为了避免与类的字段和类实例等混淆，将描述类文件格式的结构内容称为项。 连续的元素按顺序存储在类文件中，没有填充或对齐。

表由零个或多个可变大小的元素组成，用于多个类文件结构。 尽管我们使用类似 C 的数组语法来引用表项，但表是不同大小结构的流这一事实意味着不可能将表索引直接转换为表中的字节偏移量。

我们将数据结构称为数组的地方，它由零个或多个连续的固定大小的元素组成，并且可以像数组一样进行索引。

本章中对 ASCII 字符的引用应解释为与 ASCII 字符对应的 Unicode 代码点。

## 4.1. 类文件结构

类文件由单个 ClassFile 结构组成：

```
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

ClassFile 结构中的项如下：

_magic_
魔数项提供识别类文件格式的魔法数字； 它的值为 0xCAFEBABE。

_minor_version major_version_
minor_version 和 major_version 项的值是此类文件的次要和主要版本号。 主要版本号和次要版本号共同决定类文件格式的版本。 如果一个类文件有主版本号 M 和次版本号 m，我们将其类文件格式的版本表示为 M.m。

符合 Java SE N 的 Java 虚拟机实现必须完全支持表 4.1-A 第四列“支持的主要版本”中指定的类文件格式的主要版本。 符号 A .. B 表示主要版本 A 到 B，包括 A 和 B。第三列“主要版本”显示每个 Java SE 版本引入的主要版本，即第一个可以接受的版本 包含该 major_version 项的类文件。 对于非常早的版本，显示的是 JDK 版本而不是 Java SE 版本。

Table 4.1-A. class file format major versions

<table border="1">
   <thead>
      <tr>
         <th>Java SE</th>
         <th>发布时间</th>
         <th>主要版本</th>
         <th>支持的主要版本</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>1.0.2</td>
         <td>May 1996</td>
         <td>45</td>
         <td>45</td>
      </tr>
      <tr>
         <td>1.1</td>
         <td>February 1997</td>
         <td>45</td>
         <td>45</td>
      </tr>
      <tr>
         <td>1.2</td>
         <td>December 1998</td>
         <td>46</td>
         <td>45 .. 46</td>
      </tr>
      <tr>
         <td>1.3</td>
         <td>May 2000</td>
         <td>47</td>
         <td>45 .. 47</td>
      </tr>
      <tr>
         <td>1.4</td>
         <td>February 2002</td>
         <td>48</td>
         <td>45 .. 48</td>
      </tr>
      <tr>
         <td>5.0</td>
         <td>September 2004</td>
         <td>49</td>
         <td>45 .. 49</td>
      </tr>
      <tr>
         <td>6</td>
         <td>December 2006</td>
         <td>50</td>
         <td>45 .. 50</td>
      </tr>
      <tr>
         <td>7</td>
         <td>July 2011</td>
         <td>51</td>
         <td>45 .. 51</td>
      </tr>
      <tr>
         <td>8</td>
         <td>March 2014</td>
         <td>52</td>
         <td>45 .. 52</td>
      </tr>
      <tr>
         <td>9</td>
         <td>September 2017</td>
         <td>53</td>
         <td>45 .. 53</td>
      </tr>
      <tr>
         <td>
            <p>10</p>
         </td>
         <td>March 2018</td>
         <td>54</td>
         <td>45 .. 54</td>
      </tr>
      <tr>
         <td>
            <p>11</p>
         </td>
         <td>September 2018</td>
         <td>55</td>
         <td>45 .. 55</td>
      </tr>
      <tr>
         <td>
            <p>12</p>
         </td>
         <td>March 2019</td>
         <td>56</td>
         <td>45 .. 56</td>
      </tr>
      <tr>
         <td>
            <p>13</p>
         </td>
         <td>September 2019</td>
         <td>57</td>
         <td>45 .. 57</td>
      </tr>
      <tr>
         <td>
            <p>14</p>
         </td>
         <td>March 2020</td>
         <td>58</td>
         <td>45 .. 58</td>
      </tr>
      <tr>
         <td>
            <p>15</p>
         </td>
         <td>September 2020</td>
         <td>59</td>
         <td>45 .. 59</td>
      </tr>
      <tr>
         <td>
            <p>16</p>
         </td>
         <td>March 2021</td>
         <td>60</td>
         <td>45 .. 60</td>
      </tr>
      <tr>
         <td>
            <p>
               17
            </p>
         </td>
         <td>September 2021</td>
         <td>61</td>
         <td>45 .. 61</td>
      </tr>
      <tr>
         <td>
            <p>
               18
            </p>
         </td>
         <td>March 2022</td>
         <td>62</td>
         <td>45 .. 62</td>
      </tr>
      <tr>
         <td>
            <p>
               19
            </p>
         </td>
         <td>September 2022</td>
         <td>63</td>
         <td>45 .. 63</td>
      </tr>
   </tbody>
</table>

对于 major_version 为 56 或以上的 class 文件，minor_version 必须为 0 或 65535。

对于 major_version 介于 45 和 55 之间的类文件，minor_version 可以是任何值。

从历史的角度来看 JDK 对类文件格式版本的支持是有必要的。 JDK 1.0.2 支持版本 45.0 到 45.3（含）。 JDK 1.1 支持版本 45.0 到 45.65535（含）。 当 JDK 1.2 引入对主要版本 46 的支持时，该主要版本下唯一支持的次要版本是 0。后来的 JDK 继续引入对新主要版本（47、48 等）的支持但仅支持次要版本 0 的做法 在新的主要版本下。 最后，Java SE 12 中预览功能的引入（见下文）激发了小版本类文件格式的标准作用，因此 JDK 12 在大版本 56 下支持小版本 0 和 65535。后续的 JDK 引入了对 N 的支持 .0 和 N.65535，其中 N 是已实施 Java SE 平台的相应主要版本。 例如，JDK 13 支持 57.0 和 57.65535。

Java SE 平台可以定义预览特性。 符合 Java SE N（N ≥ 12）的 Java 虚拟机实现必须支持 Java SE N 的所有预览功能，并且不支持任何其他 Java SE 版本的预览功能。 实现必须默认禁用支持的预览功能，并且必须提供一种方法来启用所有这些功能，而不能提供一种方法来仅启用其中的一些功能。

如果一个类文件具有对应于 Java SE N（根据表 4.1-A）的 major_version 和 65535 的 minor_version，则称该类文件依赖于 Java SE N（N ≥ 12）的预览功能。

符合 Java SE N（N ≥ 12）的 Java 虚拟机实现必须表现如下：

- 依赖 Java SE N 预览特性的类文件只有在启用 Java SE N 预览特性的情况下才能被加载。
- 绝不能加载依赖于另一个 Java SE 版本的预览功能的类文件。
- 无论是否启用 Java SE N 的预览功能，都可以加载不依赖于任何 Java SE 版本的预览功能的类文件。

_constant_pool_count_

constant_pool_count 项的值等于 constant_pool 表中的条目数加一（常量池的中常量的个数实际是 constant_pool_count-1，译者注）。 如果 constant_pool 索引大于零且小于 constant_pool_count，则认为它是有效的，但 §4.4.5 中提到的 long 和 double 类型的常量除外。

_constant_pool[]_

constant_pool 是一个结构表 (§4.4)，表示各种字符串常量、类和接口名称、字段名称以及在 ClassFile 结构及其子结构中引用的其他常量。 每个 constant_pool 表条目的格式由其第一个“tag”字节指示。

constant_pool 表的索引从 1 到 constant_pool_count - 1。

_access_flags_

access_flags 项的值是类访问标志的掩码，用于表示对此类或接口的访问权限和属性。 表 4.1-B 中规定了每个标志在设置时的解释。

Table 4.1-B. Class access and property modifiers

<table border="1">
   <thead>
      <tr>
         <th>标志名称</th>
         <th>值</th>
         <th>解释</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td><code>ACC_PUBLIC</code></td>
         <td>0x0001</td>
         <td>public声明
         </td>
      </tr>
      <tr>
         <td><code>ACC_FINAL</code></td>
         <td>0x0010</td>
         <td>final声明</td>
      </tr>
      <tr>
         <td><code>ACC_SUPER</code></td>
         <td>0x0020</td>
         <td>继承超类声明
         </td>
      </tr>
      <tr>
         <td><code>ACC_INTERFACE</code></td>
         <td>0x0200</td>
         <td>接口声明</td>
      </tr>
      <tr>
         <td><code>ACC_ABSTRACT</code></td>
         <td>0x0400</td>
         <td>抽象类声明
         </td>
      </tr>
      <tr>
         <td><code>ACC_SYNTHETIC</code></td>
         <td>0x1000</td>
         <td>生成类声明</td>
      </tr>
      <tr>
         <td><code>ACC_ANNOTATION</code></td>
         <td>0x2000</td>
         <td>注解类声明</td>
      </tr>
      <tr>
         <td><code>ACC_ENUM</code></td>
         <td>0x4000</td>
         <td>枚举类声明
         </td>
      </tr>
      <tr>
         <td>
            <p>
               <code>ACC_MODULE</code>
            </p>
         </td>
         <td>0x8000</td>
         <td>模块声明</td>
      </tr>
   </tbody>
</table>

ACC_MODULE 标志表示此类文件定义了一个模块，而不是类或接口。 如果设置了 ACC_MODULE 标志，则特殊规则适用于本节末尾给出的类文件。 如果未设置 ACC_MODULE 标志，则当前段落下方的规则适用于类文件。

接口通过设置 ACC_INTERFACE 标志来区分。 如果未设置 ACC_INTERFACE 标志，则此类文件定义一个类，而不是接口或模块。

如果设置了 ACC_INTERFACE 标志，则还必须设置 ACC_ABSTRACT 标志，并且不得设置 ACC_FINAL、ACC_SUPER、ACC_ENUM 和 ACC_MODULE 标志。

如果未设置 ACC_INTERFACE 标志，则可以设置表 4.1-B 中的任何其他标志，但 ACC_ANNOTATION 和 ACC_MODULE 除外。 但是，此类文件不能同时设置其 ACC_FINAL 和 ACC_ABSTRACT 标志（JLS §8.1.1.2）。

ACC_SUPER 标志指示如果 invokespecial 指令 (§invokespecial) 出现在此类或接口中，将表示两种可选语义中的哪一种。 Java 虚拟机指令集的编译器应该设置 ACC_SUPER 标志。 在 Java SE 8 及更高版本中，Java 虚拟机认为在每个类文件中都设置了 ACC_SUPER 标志，而不管类文件中标志的实际值和类文件的版本。

ACC_SUPER 标志的存在是为了向后兼容由旧编译器为 Java 编程语言编译的代码。 在 JDK 1.0.2 之前，编译器生成 access_flags，其中现在表示 ACC_SUPER 的标志没有指定的含义，如果设置了标志，Oracle 的 Java 虚拟机实现将忽略该标志。

ACC_SYNTHETIC 标志表示此类或接口是由编译器生成的，不会出现在源代码中。

注释接口 (JLS §9.6) 必须设置其 ACC_ANNOTATION 标志。 如果设置了 ACC_ANNOTATION 标志，则还必须设置 ACC_INTERFACE 标志。

ACC_ENUM 标志表示此类或其超类被声明为枚举类（JLS §8.9）。

未在表 4.1-B 中分配的 access_flags 项的所有位保留供将来使用。 它们应该在生成的类文件中设置为零，并且应该被 Java 虚拟机实现忽略。

_this_class_
this_class 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Class_info 结构（第 4.4.1 节），表示此类文件定义的类或接口。

_super_class_
对于类，super_class 项的值必须为零或必须是 constant_pool 表中的有效索引。 如果 super_class 项的值不为零，则该索引处的 constant_pool 条目必须是 CONSTANT_Class_info 结构，表示此类文件定义的类的直接超类。 直接超类及其任何超类都不能在其 ClassFile 结构的 access_flags 项中设置 ACC_FINAL 标志。

如果 super_class 项的值为零，则此类文件必须代表类 Object，这是唯一没有直接超类的类或接口。

对于接口，super_class 项的值必须始终是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是表示类 Object 的 CONSTANT_Class_info 结构。

_interfaces_count_
interfaces_count 项的值给出了此类或接口类型的直接超接口的数量。

_interfaces[]_
interfaces 数组中的每个值都必须是 constant_pool 表中的有效索引。 `interfaces[i]` 的每个值处的 constant_pool 条目，其中 0 ≤ i < interfaces_count，必须是一个 CONSTANT_Class_info 结构，表示一个接口，该接口是此类或接口类型的直接超接口，按照在中给出的从左到右的顺序 类型的来源。

_fields_count_
fields_count 项的值给出了 fields 表中 field_info 结构的数量。 field_info 结构表示由此类或接口类型声明的所有字段，包括类变量和实例变量。

_fields[]_
fields 表中的每个值都必须是一个 field_info 结构，给出此类或接口中字段的完整描述。 字段表仅包含此类或接口声明的那些字段。 它不包括表示从超类或超接口继承的字段的元素。

_methods_count_
methods_count 项的值给出了 methods 表中 method_info 结构的数量。

_methods[]_
methods 表中的每个值都必须是 method_info 结构，给出此类或接口中方法的完整描述。 如果在 method_info 结构的 access_flags 项中没有设置 ACC_NATIVE 和 ACC_ABSTRACT 标志，则还提供实现该方法的 Java 虚拟机指令。

method_info 结构表示此类或接口类型声明的所有方法，包括实例方法、类方法、实例初始化方法以及任何类或接口初始化方法。 方法表不包括表示从超类或超接口继承的方法的元素。

_attributes_count_
attributes_count 项的值给出了该类的属性表中的属性数。

_attributes[]_
属性表的每个值都必须是一个 attribute_info 结构。

本规范定义的出现在 ClassFile 结构的属性表中的属性在表 4.7-C 中列出。

有关定义为出现在 ClassFile 结构的属性表中的属性的规则在 §4.7 中给出。

关于类文件结构的属性表中非预定义属性的规则在§4.7.1 中给出。

如果在 access_flags 项中设置了 ACC_MODULE 标志，则不能在 access_flags 项中设置其他标志，并且以下规则适用于 ClassFile 结构的其余部分：

- major_version、minor_version：≥ 53.0（即 Java SE 9 及更高版本）
- this_class: 模块信息
- super_class、interfaces_count、fields_count、methods_count：零
- attributes：必须存在一个 Module 属性。 除了 Module、ModulePackages、ModuleMainClass、InnerClasses、SourceFile、SourceDebugExtension、RuntimeVisibleAnnotations 和 RuntimeInvisibleAnnotations 之外，预定义属性 (§4.7) 都不会出现。

## 4.2. 名称/名字规范

### 4.2.1. 二进制类/接口名

出现在类文件结构中的类和接口名称始终以称为二进制名称的完全限定形式表示 (JLS §13.1)。 此类名称始终表示为 CONSTANT_Utf8_info 结构（第 4.4.7 节），因此可以从整个 Unicode 代码空间中提取（不受进一步限制）。 类和接口名称是从那些 CONSTANT_NameAndType_info 结构中引用的，这些结构将这些名称作为其描述符的一部分，并且从所有 CONSTANT_Class_info 结构中引用。

由于历史原因，出现在类文件结构中的二进制名称的语法与 §13.1 中记录的二进制名称的语法不同。 在这种内部形式中，通常分隔构成二进制名称的标识符的 ASCII 句点 (.) 被 ASCII 正斜杠 (/) 替换。 标识符本身必须是非限定名称。

例如，类 Thread 的正常二进制名称是 `java.lang.Thread`。 在类文件格式的描述符中使用的内部形式中，对类 Thread 名称的引用是使用表示字符串 `java/lang/Thread` 的 CONSTANT_Utf8_info 结构实现的。

### 4.2.2. 不合格的名称

方法、字段、局部变量和形式参数的名称存储为非限定名称。 非限定名称必须至少包含一个 Unicode 代码点，并且不得包含任何 ASCII 字符` . ; [ /`（即 句号、分号、左方括号、正斜杠）。

方法名称受到进一步限制，因此，除了特殊方法名称 `<init>` 和 `<clinit>` (§2.9) 之外，它们不得包含 ASCII 字符 `<` 或`>`（即左尖括号或右尖括号） .

请注意，任何方法调用指令都不能引用 `<clinit>`，只有 invokespecial 指令 (§invokespecial) 可以引用 `<init>`。

### 4.2.3. 模块和包名称

从 Module 属性引用的模块名称存储在常量池中的 CONSTANT_Module_info 结构中（§4.4.11）。 CONSTANT_Module_info 结构包装了一个表示模块名称的 CONSTANT_Utf8_info 结构。 模块名称不像类和接口名称那样以“内部形式”编码，也就是说，模块名称中分隔标识符的 ASCII 句点 (.) 不会被 ASCII 正斜杠 (/) 替换。

模块名称可以从整个 Unicode 代码空间中提取，但要遵守以下限制：

- 模块名称不得包含“\u0000”到“\u001F”范围内的任何代码点。
- ASCII 反斜杠 (`\`) 保留用作模块名称中的转义字符。 它不能出现在模块名称中，除非它后跟 ASCII 反斜杠、ASCII 冒号 (`:`) 或 ASCII at 符号 (`@`)。 ASCII 字符序列 `\\` 可用于对模块名称中的反斜杠进行编码。
- ASCII 冒号 (`:`) 和 at 符号 (`@`) 保留供将来在模块名称中使用。 它们不能出现在模块名称中，除非它们被转义。 ASCII 字符序列 `\:` 和 `\@` 可用于编码模块名称中的冒号和 at 符号。

从 Module 属性引用的包名称存储在常量池中的 CONSTANT_Package_info 结构中（§4.4.12）。 CONSTANT_Package_info 结构包装了一个 CONSTANT_Utf8_info 结构，该结构表示以内部形式编码的包名称。

## 4.3. 描述符

描述符是表示字段或方法类型的字符串。 描述符使用修改后的 UTF-8 字符串（第 4.4.7 节）以类文件格式表示，因此可以从整个 Unicode 代码空间中提取（不受进一步限制）。

### 4.3.1. 语法符号

描述符是使用语法指定的。 语法是一组描述字符序列如何形成语法正确的各种描述符的产生式。 语法的终端符号以固定宽度字体显示。 非终结符号以斜体显示。 非终结符的定义由被定义的非终结符的名称引入，后跟一个冒号。 非终结符的一个或多个替代定义随后在后续行中。
(译者注，下面的翻译可能不会按照上述规范进行字体的格式化，可以参考原文)

产生式右侧的语法 `{x}` 表示 x 出现零次或多次。
产生式右侧的短语 (one of) 表示接下来一行或几行中的每个终结符都是一个替代定义。
(译者注，这里的定义类似 BNF 定义)

### 4.3.2. 字段描述符

字段描述符表示类、实例或局部变量的类型。

```
FieldDescriptor:
    FieldType

FieldType:
    BaseType
    ObjectType
    ArrayType

BaseType:
    (one of)
    B C D F I J S Z

ObjectType:
    L ClassName ;

ArrayType:
    [ ComponentType

ComponentType:
    FieldType
```

BaseType 的字符，L 和; ObjectType 的 [ 和 ArrayType 的 [ 都是 ASCII 字符。

ClassName 表示以内部形式编码的二进制类或接口名称（§4.2.1）。

字段描述符作为类型的解释如表 4.3-A 所示。

表示数组类型的字段描述符仅当它表示具有 255 个或更少维度的类型时才有效。

Table 4.3-A. 字段描述符的解释

<table border="1">
   <thead>
      <tr>
         <th><span><em>字段</em></span> term
         </th>
         <th>类型</th>
         <th>描述</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td><code>B</code></td>
         <td><code>byte</code></td>
         <td>有符号byte</td>
      </tr>
      <tr>
         <td><code>C</code></td>
         <td><code>char</code></td>
         <td> Unicode基本多平面的码点，使用UTF-16编码
         </td>
      </tr>
      <tr>
         <td><code>D</code></td>
         <td><code>double</code></td>
         <td>双精度浮点</td>
      </tr>
      <tr>
         <td><code>F</code></td>
         <td><code>float</code></td>
         <td>单精度浮点</td>
      </tr>
      <tr>
         <td><code>I</code></td>
         <td><code>int</code></td>
         <td>整数</td>
      </tr>
      <tr>
         <td><code>J</code></td>
         <td><code>long</code></td>
         <td>长整数</td>
      </tr>
      <tr>
         <td><code>L</code> <span><em>ClassName</em></span> <code>;</code></td>
         <td><code>reference</code></td>
         <td>实例对象的类名</td>
      </tr>
      <tr>
         <td><code>S</code></td>
         <td><code>short</code></td>
         <td>有符号短整形</td>
      </tr>
      <tr>
         <td><code>Z</code></td>
         <td><code>boolean</code></td>
         <td><code>true</code> or <code>false</code></td>
      </tr>
      <tr>
         <td><code>[</code></td>
         <td><code>reference</code></td>
         <td>数组</td>
      </tr>
   </tbody>
</table>

int 类型的实例变量的字段描述符就是简单的 I。

Object 类型的实例变量的字段描述符是 Ljava/lang/Object;。 请注意，使用了类 Object 的二进制名称的内部形式。

多维数组类型 double[][][] 的实例变量的字段描述符是 [[[D.

### 4.3.3. 方法描述符

方法描述符包含零个或多个参数描述符，表示方法采用的参数类型，以及返回描述符，表示方法返回值（如果有）的类型。

```
MethodDescriptor:
    ( {ParameterDescriptor} ) ReturnDescriptor

ParameterDescriptor:
    FieldType

ReturnDescriptor:
    FieldType
    VoidDescriptor

VoidDescriptor:
    V
```

字符 V 表示该方法不返回任何值（其结果为 void）。

方法的方法描述符：

```
Object m(int i, double d, Thread t) {...}
->
(IDLjava/lang/Thread;)Ljava/lang/Object;
```

请注意，使用了 Thread 和 Object 的二进制名称的内部形式。

方法描述符仅当它表示总长度为 255 或更短的方法参数时才有效，其中该长度包括在实例或接口方法调用的情况下对此的贡献。 总长度是通过对各个参数的贡献求和来计算的，其中 long 或 double 类型的参数对长度贡献两个单位，任何其他类型的参数贡献一个单位。

无论它描述的方法是类方法还是实例方法，方法描述符都是相同的。 尽管向实例方法传递了 this，即对调用该方法的对象的引用，以及其预期的参数，但该事实并未反映在方法描述符中。 对 this 的引用由调用实例方法的 Java 虚拟机指令隐式传递（§2.6.1，§4.11）。

## 4.4. 常量池

Java 虚拟机指令不依赖于类、接口、类实例或数组的运行时布局。 相反，指令引用 constant_pool 表中的符号信息。

所有 constant_pool 表条目都具有以下通用格式：

```
cp_info {
    u1 tag;
    u1 info[];
}
```

constant_pool 表中的每个条目都必须以一个 1 字节的标记开头，该标记指示该条目表示的常量类型。 常量共有 17 种，在表 4.4-A 中列出了它们对应的标签，并按它们在本章中的章节编号排序。 每个标记字节后面必须跟两个或更多字节，提供有关特定常量的信息。 附加信息的格式取决于 tag 字节，即 info 数组的内容随着 tag 的值而变化。

Table 4.4-A. 常量池字节标志

<table border="1">
   <thead>
      <tr>
         <th>常量类型</th>
         <th>标志</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td><code >CONSTANT_Class</code></td>
         <td>7</td>
      </tr>
      <tr>
         <td><code >CONSTANT_Fieldref</code></td>
         <td>9</td>
      </tr>
      <tr>
         <td><code >CONSTANT_Methodref</code></td>
         <td>10</td>
      </tr>
      <tr>
         <td><code >CONSTANT_InterfaceMethodref</code></td>
         <td>11</td>
      </tr>
      <tr>
         <td><code >CONSTANT_String</code></td>
         <td>8</td>
      </tr>
      <tr>
         <td><code >CONSTANT_Integer</code></td>
         <td>3</td>
      </tr>
      <tr>
         <td><code >CONSTANT_Float</code></td>
         <td>4</td>
      </tr>
      <tr>
         <td><code >CONSTANT_Long</code></td>
         <td>5</td>
      </tr>
      <tr>
         <td><code >CONSTANT_Double</code></td>
         <td>6</td>
      </tr>
      <tr>
         <td><code >CONSTANT_NameAndType</code></td>
         <td>12</td>
      </tr>
      <tr>
         <td><code >CONSTANT_Utf8</code></td>
         <td>1</td>
      </tr>
      <tr>
         <td><code >CONSTANT_MethodHandle</code></td>
         <td>15</td>
      </tr>
      <tr>
         <td><code >CONSTANT_MethodType</code></td>
         <td>16</td>
      </tr>
      <tr>
         <td><code >CONSTANT_Dynamic</code></td>
         <td>17</td>
         </td>
      </tr>
      <tr>
         <td><code >CONSTANT_InvokeDynamic</code></td>
         <td>18</td>
         </td>
      </tr>
      <tr>
         <td><code >CONSTANT_Module</code></td>
         <td>19</td>
      </tr>
      <tr>
         <td><code >CONSTANT_Package</code></td>
         <td>20</td>
      </tr>
   </tbody>
</table>

在版本号为 v 的类文件中，constant_pool 表中的每个条目都必须有一个标记，该标记首先在版本 v 或更早的类文件格式（§4.1）中定义。 也就是说，每个条目必须表示一种批准在类文件中使用的常量。 表 4.4-B 列出了每个标签以及定义它的类文件格式的第一个版本。 还显示了引入该类文件格式版本的 Java SE 平台版本。

Table 4.4-B. Constant pool tags (by tag)

<table border="1">
   <thead>
      <tr>
         <th>常量类型</th>
         <th>标志</th>
         <th><code>class</code> file format
         </th>
         <th>Java SE</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td><code>CONSTANT_Utf8</code></td>
         <td>1</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Integer</code></td>
         <td>3</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Float</code></td>
         <td>4</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Long</code></td>
         <td>5</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Double</code></td>
         <td>6</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Class</code></td>
         <td>7</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_String</code></td>
         <td>8</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Fieldref</code></td>
         <td>9</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Methodref</code></td>
         <td>10</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_InterfaceMethodref</code></td>
         <td>11</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_NameAndType</code></td>
         <td>12</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_MethodHandle</code></td>
         <td>15</td>
         <td>51.0</td>
         <td>7</td>
      </tr>
      <tr>
         <td><code>CONSTANT_MethodType</code></td>
         <td>16</td>
         <td>51.0</td>
         <td>7</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Dynamic</code></td>
         <td>17</td>
         <td>55.0</td>
         <td>11</td>
      </tr>
      <tr>
         <td><code>CONSTANT_InvokeDynamic</code></td>
         <td>18</td>
         <td>51.0</td>
         <td>7</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Module</code></td>
         <td>19</td>
         <td>53.0</td>
         <td>9</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Package</code></td>
         <td>20</td>
         <td>53.0</td>
         <td>9</td>
      </tr>
   </tbody>
</table>

constant_pool 表中的一些条目是可加载的，因为它们代表可以在运行时被压入堆栈以启用进一步计算的实体。 在版本号为 v 的类文件中，如果 constant_pool 表中的条目具有在类文件格式的版本 v 或更早版本中首先被认为可加载的标记，则该条目是可加载的。 表 4.4-C 列出了每个标签以及被认为可加载的类文件格式的第一个版本。 还显示了引入该类文件格式版本的 Java SE 平台版本。

在除 CONSTANT_Class 之外的所有情况下，标签首先被认为可以加载到与首先定义该标签的类文件格式相同的版本中。

Table 4.4-C. Loadable constant pool tags

<table border="1">
   <thead>
      <tr>
         <th>常量类型</th>
         <th>标志</th>
         <th><code>class</code> file format
         </th>
         <th>Java SE</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td><code>CONSTANT_Integer</code></td>
         <td>3</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Float</code></td>
         <td>4</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Long</code></td>
         <td>5</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Double</code></td>
         <td>6</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Class</code></td>
         <td>7</td>
         <td>49.0</td>
         <td>5.0</td>
      </tr>
      <tr>
         <td><code>CONSTANT_String</code></td>
         <td>8</td>
         <td>45.3</td>
         <td>1.0.2</td>
      </tr>
      <tr>
         <td><code>CONSTANT_MethodHandle</code></td>
         <td>15</td>
         <td>51.0</td>
         <td>7</td>
      </tr>
      <tr>
         <td><code>CONSTANT_MethodType</code></td>
         <td>16</td>
         <td>51.0</td>
         <td>7</td>
      </tr>
      <tr>
         <td><code>CONSTANT_Dynamic</code></td>
         <td>17</td>
         <td>55.0</td>
         <td>11</td>
      </tr>
   </tbody>
</table>

### 4.4.1. CONSTANT_Class_info 结构

CONSTANT_Class_info 结构用于表示类或接口：

```
CONSTANT_Class_info {
    u1 tag;
    u2 name_index;
}
```

CONSTANT_Class_info 结构体的各项如下：

- tag 标记项的值为 CONSTANT_Class (7)。
- name_index name_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Utf8_info 结构（§4.4.7），表示以内部形式（§4.2.1）编码的有效二进制类或接口名称。

因为数组是对象，所以操作码 `anewarray` 和 `multianewarray` - 但不是操作码 new - 可以通过 constant_pool 表中的 CONSTANT_Class_info 结构引用数组“类”。 对于此类数组类，类的名称是数组类型的描述符（§4.3.2）。

例如表示二维数组类型`int[][]`的类名是`[[I`，而表示`Thread[]`类型的类名是`[Ljava/lang/Thread;`。

数组类型描述符只有在表示 255 个或更少的维度时才有效。

### 4.4.2. CONSTANT_Fieldref_info、CONSTANT_Methodref_info 和 CONSTANT_InterfaceMethodref_info 结构

字段、方法和接口方法用类似的结构表示：

```
CONSTANT_Fieldref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}

CONSTANT_Methodref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}

CONSTANT_InterfaceMethodref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}
```

这些结构的元素如下：

- tag

  - CONSTANT_Fieldref_info 结构的标记项的值为 CONSTANT_Fieldref (9)
  - CONSTANT_Methodref_info 结构的标记项的值为 CONSTANT_Methodref (10)。
  - CONSTANT_InterfaceMethodref_info 结构的标记项的值为 CONSTANT_InterfaceMethodref (11)。

- class_index
  - class_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Class_info 结构（第 4.4.1 节），表示具有字段或方法作为成员的类或接口类型。
  - 在 CONSTANT_Fieldref_info 结构中，class_index 项可以是类类型或接口类型。
  - 在 CONSTANT_Methodref_info 结构中，class_index 项应该是类类型，而不是接口类型。
  - 在 CONSTANT_InterfaceMethodref_info 结构中，class_index 项应该是接口类型，而不是类类型。
- name_and_type_index
  - name_and_type_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_NameAndType_info 结构（§4.4.6）。 此 constant_pool 条目指示字段或方法的名称和描述符。
  - 在 CONSTANT_Fieldref_info 结构中，指示的描述符必须是字段描述符（§4.3.2）。 否则，指示的描述符必须是方法描述符（§4.3.3）。
  - 如果 CONSTANT_Methodref_info 结构中的方法名称以“<”（“\u003c”）开头，则该名称必须是特殊名称 `<init>`，表示实例初始化方法（§2.9.1）。 这种方法的返回类型必须为 void。

### 4.4.3. CONSTANT_String_info 结构

CONSTANT_String_info 结构用于表示 String 类型的常量对象：

```
CONSTANT_String_info {
    u1 tag;
    u2 string_index;
}
```

- tag
  - 标记项的值为 CONSTANT_String (8)。
- string_index
  - string_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Utf8_info 结构（第 4.4.7 节），表示 String 对象要初始化到的 Unicode 代码点序列。

### 4.4.4. CONSTANT_Integer_info 和 CONSTANT_Float_info 结构

CONSTANT_Integer_info 和 CONSTANT_Float_info 结构表示 4 字节数字（int 和 float）常量：

```
CONSTANT_Integer_info {
    u1 tag;
    u4 bytes;
}

CONSTANT_Float_info {
    u1 tag;
    u4 bytes;
}
```

- tag
  - CONSTANT_Integer_info 结构的标记项的值为 CONSTANT_Integer (3)。
  - CONSTANT_Float_info 结构的标记项的值为 CONSTANT_Float (4)。
- bytes
  - CONSTANT_Integer_info 结构的 bytes 项表示 int 常量的值。 值的字节以大端（高字节在前）顺序存储。
  - CONSTANT_Float_info 结构的字节项表示 IEEE 754 binary32 浮点格式 (§2.3.2) 中浮点常量的值。 元素的字节以大端（高字节在前）顺序存储。
  - CONSTANT_Float_info 结构表示的值确定如下。 值的字节首先被转换成一个 int 常量位。 然后：
    - 如果位是 0x7f800000，则浮点值将为正无穷大。
    - 如果位为 0xff800000，则浮点值将为负无穷大。
    - 如果位在 0x7f800001 到 0x7fffffff 范围内或在 0xff800001 到 0xffffffff 范围内，则浮点值将为 NaN。
    - 在所有其他情况下，设 s、e 和 m 是可以从位计算的三个值：
      ```
      int s = ((bits >> 31) == 0) ? 1 : -1;
      int e = ((bits >> 23) & 0xff);
      int m = (e == 0) ?
              (bits & 0x7fffff) << 1 :
              (bits & 0x7fffff) | 0x800000;
      ```
    - 那么浮点值等于数学表达式 $s · m · 2^{e-150}$ 的结果。

### 4.4.5. CONSTANT_Long_info 和 CONSTANT_Double_info 结构

CONSTANT_Long_info 和 CONSTANT_Double_info 表示 8 字节数字（long 和 double）常量：

```
CONSTANT_Long_info {
    u1 tag;
    u4 high_bytes;
    u4 low_bytes;
}

CONSTANT_Double_info {
    u1 tag;
    u4 high_bytes;
    u4 low_bytes;
}
```

所有 8 字节常量在类文件的 constant_pool 表中占用两个条目。 如果 CONSTANT_Long_info 或 CONSTANT_Double_info 结构是 constant_pool 表中索引 n 处的条目，则表中的下一个可用条目位于索引 n+2 处。 constant_pool 索引 n+1 必须有效但被认为不可用。

回想起来，让 8 字节常量占用两个常量池条目是一个糟糕的选择。

这些结构的元素如下：

- tag
  - CONSTANT_Long_info 结构的标记项的值为 CONSTANT_Long (5)。
  - CONSTANT_Double_info 结构的标记项的值为 CONSTANT_Double (6)。
- high_bytes, low_bytes
  - CONSTANT_Long_info 结构体的 unsigned high_bytes 和 low_bytes 项共同表示 long 常量的值
    ```
    ((long) high_bytes << 32) + low_bytes
    ```
  - 其中每个 high_bytes 和 low_bytes 的字节以大端（高字节在前）顺序存储。
  - CONSTANT_Double_info 结构的 high_bytes 和 low_bytes 项一起表示 IEEE 754 binary64 浮点格式 (§2.3.2) 中的双精度值。 每个项目的字节以大端（高字节在前）顺序存储。
  - CONSTANT_Double_info 结构表示的值确定如下。 high_bytes 和 low_bytes 项目被转换为长常量位，等于
    `    ((long) high_bytes << 32) + low_bytes
   `
    然后： - 如果位为 0x7ff0000000000000L，则双精度值将为正无穷大。 - 如果位为 0xfff0000000000000L，则双精度值将为负无穷大。 - 如果位在 0x7ff0000000000001L 到 0x7fffffffffffffffL 范围内或在 0xfff0000000000001L 到 0xfffffffffffffffL 范围内，则双精度值将为 NaN。 - 在所有其他情况下，设 s、e 和 m 是可以从位计算的三个值：
    `        int s = ((bits >> 63) == 0) ? 1 : -1;
        int e = (int)((bits >> 52) & 0x7ffL);
        long m = (e == 0) ?
                (bits & 0xfffffffffffffL) << 1 :
                (bits & 0xfffffffffffffL) | 0x10000000000000L;
       `
    那么浮点值等于数学表达式 $s·m·2^{e-1075}$ 的双精度值。

### 4.4.6. CONSTANT_NameAndType_info 结构

CONSTANT_NameAndType_info 结构用于表示一个字段或方法，不指明它属于哪个类或接口类型：

```
CONSTANT_NameAndType_info {
    u1 tag;
    u2 name_index;
    u2 descriptor_index;
}
```

- tag
  - 标记项的值为 CONSTANT_NameAndType (12)。
- name_index
  - name_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Utf8_info 结构（§4.4.7），表示表示字段或方法的有效非限定名称（§4.2.2）或特殊方法名称 <init>（§2.9.1）。
- descriptor_index
  - descriptor_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Utf8_info 结构（§4.4.7），表示有效的字段描述符或方法描述符（§4.3.2、§4.3.3）。

### 4.4.7. CONSTANT_Utf8_info 结构

CONSTANT_Utf8_info 结构用于表示常量字符串值：

```
CONSTANT_Utf8_info {
    u1 tag;
    u2 length;
    u1 bytes[length];
}
```

- tag
  - 标记项的值为 CONSTANT_Utf8 (1)。
- length
  - 长度项的值给出字节数组中的字节数（不是结果字符串的长度）。
- `bytes[length]`
  - 字节数组包含字符串的字节。
  - 任何字节都不能具有值 (byte)0。
  - 任何字节都不能位于 (byte)0xf0 到 (byte)0xff 的范围内。

字符串内容以修改后的 UTF-8 编码。 修改后的 UTF-8 字符串经过编码，因此仅包含非空 ASCII 字符的代码点序列可以每个代码点仅使用 1 个字节来表示，但可以表示 Unicode 代码空间中的所有代码点。 修改后的 UTF-8 字符串不以 null 结尾。 编码如下：

- '\u0001' 到 '\u007F' 范围内的代码点由单个字节表示：
  ```
  0	bits 6-0
  ```
  字节中的 7 位数据给出了所表示的代码点的值。
- 空代码点 ('\u0000') 和 '\u0080' 到 '\u07FF' 范围内的代码点由一对字节 x 和 y 表示：
  ```
  x:
  1	1	0	bits 10-6
  y:
  1	0	bits 5-0
  ```
  这两个字节表示具有值的代码点：
  ```
  ((x & 0x1f) << 6) + (y & 0x3f)
  ```
- '\u0800' 到 '\uFFFF' 范围内的代码点由 3 个字节 x、y 和 z 表示：
  ```
  x:
  1	1	1	0	bits 15-12
  y:
  1	0	bits 11-6
  z:
  1	0	bits 5-0
  ```
  三个字节表示具有值的代码点：
  ```
  ((x & 0xf) << 12) + ((y & 0x3f) << 6) + (z & 0x3f)
  ```
- 代码点高于 U+FFFF 的字符（所谓的补充字符）通过分别编码其 UTF-16 表示的两个代理代码单元来表示。 每个代理代码单元由三个字节表示。 这意味着增补字符由六个字节 u、v、w、x、y 和 z 表示：
  ```
  u:
  1	1	1	0	1	1	0	1
  v:
  1	0	1	0	(bits 20-16)-1
  w:
  1	0	bits 15-10
  x:
  1	1	1	0	1	1	0	1
  y:
  1	0	1	1	bits 9-6
  z:
  1	0	bits 5-0
  ```
  六个字节代表具有值的代码点：
  ```
  0x10000 + ((v & 0x0f) << 16) + ((w & 0x3f) << 10) + ((y & 0x0f) << 6) + (z & 0x3f)
  ```

多字节字符的字节以大端（高字节在前）顺序存储在类文件中。

这种格式与“标准”UTF-8 格式有两点不同。 首先，空字符 (char)0 使用 2 字节格式而不是 1 字节格式进行编码，因此修改后的 UTF-8 字符串永远不会嵌入空值。 其次，只使用标准 UTF-8 的 1 字节、2 字节和 3 字节格式。 Java 虚拟机不识别标准 UTF-8 的四字节格式； 它使用自己的两倍三字节格式来代替。

有关标准 UTF-8 格式的更多信息，请参阅 Unicode 标准版本 13.0 的第 3.9 节 Unicode 编码形式。

### 4.4.8. CONSTANT_MethodHandle_info 结构

CONSTANT_MethodHandle_info 结构用于表示方法句柄：

```
CONSTANT_MethodHandle_info {
    u1 tag;
    u1 reference_kind;
    u2 reference_index;
}
```

- tag
  - 标记项的值为 CONSTANT_MethodHandle (15)。
- reference_kind
  - reference_kind 项的值必须在 1 到 9 的范围内。该值表示此方法句柄的种类，它表征其字节码行为（§5.4.3.5）。
- reference_index
  - reference_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须如下所示：
    - 如果 reference_kind 项的值为 1 (REF_getField)、2 (REF_getStatic)、3 (REF_putField) 或 4 (REF_putStatic)，则该索引处的 constant_pool 条目必须是表示字段的 CONSTANT_Fieldref_info 结构（§4.4.2） 要为其创建方法句柄。
    - 如果 reference_kind 项的值为 5 (REF_invokeVirtual) 或 8 (REF_newInvokeSpecial)，则该索引处的 constant_pool 条目必须是 CONSTANT_Methodref_info 结构（§4.4.2），表示类的方法或构造函数（§2.9.1） 将创建一个方法句柄。
    - 如果 reference_kind 项的值为 6 (REF_invokeStatic) 或 7 (REF_invokeSpecial)，那么如果类文件版本号小于 52.0，则该索引处的 constant_pool 条目必须是一个 CONSTANT_Methodref_info 结构，表示类的方法，方法句柄针对该类的方法 将被创建； 如果类文件版本号为 52.0 或更高版本，则该索引处的 constant_pool 条目必须是 CONSTANT_Methodref_info 结构或 CONSTANT_InterfaceMethodref_info 结构（§4.4.2），表示要为其创建方法句柄的类或接口的方法。
    - 如果 reference_kind 项的值为 9 (REF_invokeInterface)，则该索引处的 constant_pool 条目必须是 CONSTANT_InterfaceMethodref_info 结构，表示要为其创建方法句柄的接口方法。
  - 如果 reference_kind 项的值为 5 (REF_invokeVirtual)、6 (REF_invokeStatic)、7 (REF_invokeSpecial) 或 9 (REF_invokeInterface)，则由 CONSTANT_Methodref_info 结构或 CONSTANT_InterfaceMethodref_info 结构表示的方法的名称不得为 `<init>` 或 `<clinit>`.
  - 如果值为 8 (REF_newInvokeSpecial)，则由 CONSTANT_Methodref_info 结构表示的方法的名称必须是 `<init>`。

### 4.4.9. CONSTANT_MethodType_info 结构

CONSTANT_MethodType_info 结构用于表示方法类型：

```
CONSTANT_MethodType_info {
    u1 tag;
    u2 descriptor_index;
}
```

- tag
  - 标记项的值为 CONSTANT_MethodType (16)。
- descriptor_index
  - descriptor_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是表示方法描述符（§4.3.3）的 CONSTANT_Utf8_info 结构（§4.4.7）。

### 4.4.10。 CONSTANT_Dynamic_info 和 CONSTANT_InvokeDynamic_info 结构

constant_pool 表中的大多数结构通过组合静态记录在表中的名称、描述符和值来直接表示实体。 相反，CONSTANT_Dynamic_info 和 CONSTANT_InvokeDynamic_info 结构通过指向动态计算实体的代码来间接表示实体。 该代码称为引导程序方法，在解析从这些结构派生的符号引用期间由 Java 虚拟机调用（§5.1，§5.4.3.6）。 每个结构指定一个引导方法以及一个辅助名称和类型来表征要计算的实体。 更详细：

- CONSTANT_Dynamic_info 结构用于表示一个动态计算的常量，一个在 ldc 指令 (§ldc) 过程中通过调用引导方法产生的任意值，等等。 结构指定的辅助类型约束动态计算常量的类型。

- CONSTANT_InvokeDynamic_info 结构用于表示动态计算的调用站点，它是 java.lang.invoke.CallSite 的一个实例，它是通过在 invokedynamic 指令 (§invokedynamic) 过程中调用引导方法而产生的。 结构指定的辅助类型约束动态计算调用点的方法类型。

```
CONSTANT_Dynamic_info {
    u1 tag;
    u2 bootstrap_method_attr_index;
    u2 name_and_type_index;
}

CONSTANT_InvokeDynamic_info {
    u1 tag;
    u2 bootstrap_method_attr_index;
    u2 name_and_type_index;
}
```

- tag
  - CONSTANT_Dynamic_info 结构的标记项的值为 CONSTANT_Dynamic (17)。
  - CONSTANT_InvokeDynamic_info 结构的标记项的值为 CONSTANT_InvokeDynamic (18)。
- bootstrap_method_attr_index
  - bootstrap_method_attr_index 项的值必须是此类文件（§4.7.23）的引导方法表的 bootstrap_methods 数组中的有效索引。
  - CONSTANT_Dynamic_info 结构的独特之处在于它们在语法上允许通过引导方法表来引用自己。 我们不是强制在加载类时检测此类循环（一项可能代价高昂的检查），而是最初允许循环但强制在解决时失败（§5.4.3.6）。
- name_and_type_index
  - name_and_type_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_NameAndType_info 结构（§4.4.6）。 此 constant_pool 条目指示名称和描述符。
  - 在 CONSTANT_Dynamic_info 结构中，指示的描述符必须是字段描述符（§4.3.2）。
  - 在 CONSTANT_InvokeDynamic_info 结构中，指示的描述符必须是方法描述符（§4.3.3）。

### 4.4.11。 CONSTANT_Module_info 结构

CONSTANT_Module_info 结构用于表示一个模块：

```
CONSTANT_Module_info {
    u1 tag;
    u2 name_index;
}
```

- tag
  - 标记项的值为 CONSTANT_Module (19)。
- name_index
  - name_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是表示有效模块名称（§4.2.3）的 CONSTANT_Utf8_info 结构（§4.4.7）。
  - CONSTANT_Module_info 结构仅允许在声明模块的类文件的常量池中使用，即，其中 access_flags 项设置了 ACC_MODULE 标志的 ClassFile 结构。 在所有其他类文件中，CONSTANT_Module_info 结构是非法的。

### 4.4.12。 CONSTANT_Package_info 结构

CONSTANT_Package_info 结构用于表示模块导出或打开的包：

```
CONSTANT_Package_info {
    u1 tag;
    u2 name_index;
}

```

- tag
  - 标记项的值为 CONSTANT_Package (20)。
- name_index
  - name_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Utf8_info 结构（§4.4.7），表示以内部形式编码的有效包名称（§4.2.3）。
  - CONSTANT_Package_info 结构仅允许在声明模块的类文件的常量池中使用，即，其中 access_flags 项设置了 ACC_MODULE 标志的 ClassFile 结构。 在所有其他类文件中，CONSTANT_Package_info 结构是非法的。

## 4.5. 字段

每个字段都由 field_info 结构描述。

一个类文件中的两个字段不能具有相同的名称和描述符（§4.3.2）。

该结构具有以下格式：

```
field_info {
    u2             access_flags;
    u2             name_index;
    u2             descriptor_index;
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

- access_flags

  - access_flags 项的值是标志的掩码，用于表示对该字段的访问权限和属性。 表 4.5-A 中指定了设置时每个标志的解释。
     <table border="1">
     <thead>
        <tr>
           <th>标志名</th>
           <th>值</th>
           <th>解释</th>
        </tr>
     </thead>
     <tbody>
        <tr>
           <td><code>ACC_PUBLIC</code></td>
           <td>0x0001</td>
           <td>public声明</td>
        </tr>
        <tr>
           <td><code>ACC_PRIVATE</code></td>
           <td>0x0002</td>
           <td>
           <p>
              private声明
           </p>
           </td>
        </tr>
        <tr>
           <td><code>ACC_PROTECTED</code></td>
           <td>0x0004</td>
           <td>protected声明</td>
        </tr>
        <tr>
           <td><code>ACC_STATIC</code></td>
           <td>0x0008</td>
           <td>static声明</td>
        </tr>
        <tr>
           <td><code>ACC_FINAL</code></td>
           <td>0x0010</td>
           <td>final声明（不能修改的）</td>
        </tr>
        <tr>
           <td><code>ACC_VOLATILE</code></td>
           <td>0x0040</td>
           <td>volatile声明（不能缓存的字段）</td>
        </tr>
        <tr>
           <td><code>ACC_TRANSIENT</code></td>
           <td>0x0080</td>
           <td>transient声明（不能被持久化读写的）</td>
        </tr>
        <tr>
           <td><code>ACC_SYNTHETIC</code></td>
           <td>0x1000</td>
           <td>synthetic声明(生成的字段) </td>
        </tr>
        <tr>
           <td><code>ACC_ENUM</code></td>
           <td>0x4000</td>
           <td>enum字段</td>
        </tr>
     </tbody>
     </table>
  类的字段可以设置表 4.5-A 中的任何标志。 但是，一个类的每个字段最多可以设置其 ACC_PUBLIC、ACC_PRIVATE 和 ACC_PROTECTED 标志之一（JLS §8.3.1），并且不得同时设置其 ACC_FINAL 和 ACC_VOLATILE 标志（JLS §8.3.1.4）。

  接口字段必须设置其 ACC_PUBLIC、ACC_STATIC 和 ACC_FINAL 标志； 他们可以设置 ACC_SYNTHETIC 标志，并且不得设置表 4.5-A 中的任何其他标志（JLS §9.3）。

  ACC_SYNTHETIC 标志表示该字段是由编译器生成的，不会出现在源代码中。

  ACC_ENUM 标志表示该字段用于保存枚举类的元素（JLS §8.9）。

  表 4.5-A 中未分配的 access_flags 项的所有位都保留供将来使用。 它们应该在生成的类文件中设置为零，并且应该被 Java 虚拟机实现忽略。

- name_index;
  - name_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Utf8_info 结构（第 4.4.7 节），它表示表示字段的有效非限定名称（第 4.2.2 节）。

- descriptor_index;
  - descriptor_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Utf8_info 结构（§4.4.7），它表示有效的字段描述符（§4.3.2）。

- attributes_count;
  - attributes_count 项的值表示该字段附加属性的个数。

- `attributes[attributes_count]`;
  - 属性表的每个值都必须是一个 attribute_info 结构（§4.7）。
  - 一个字段可以有任意数量的与之关联的可选属性。
  - 本规范定义的出现在 field_info 结构的属性表中的属性在表 4.7-C 中列出。
  - 有关定义为出现在 field_info 结构的属性表中的属性的规则在 §4.7 中给出。
  - field_info 结构的属性表中有关非预定义属性的规则在§4.7.1 中给出。

## 4.6. 方法

每个方法，包括每个实例初始化方法（§2.9.1）和类或接口初始化方法（§2.9.2），都由 method_info 结构描述。

一个类文件中的两个方法不能具有相同的名称和描述符（第 4.3.3 节）。

该结构具有以下格式：
```
method_info {
    u2             access_flags;
    u2             name_index;
    u2             descriptor_index;
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

- access_flags;
   - access_flags 项的值是标志的掩码，用于表示对此方法的访问权限和属性。 表 4.6-A 中指定了设置时每个标志的解释。
   - Table 4.6-A. Method access and property flags
      <table border="1">
      <thead>
         <tr>
            <th>Flag Name</th>
            <th>Value</th>
            <th>Interpretation</th>
         </tr>
      </thead>
      <tbody>
         <tr>
            <td><code>ACC_PUBLIC</code></td>
            <td>0x0001</td>
            <td>Declared <code>public</code>; may be accessed from outside its package.</td>
         </tr>
         <tr>
            <td><code>ACC_PRIVATE</code></td>
            <td>0x0002</td>
            <td>
            <p>
               Declared <code>private</code>; accessible only within the defining class and other classes belonging
               to the same nest (<a
                  class="xref"
                  href="jvms-5.html#jvms-5.4.4"
                  title="5.4.4.&nbsp;Access Control"
                  marked="1"
                  >§5.4.4</a
               >).
            </p>
            </td>
         </tr>
         <tr>
            <td><code>ACC_PROTECTED</code></td>
            <td>0x0004</td>
            <td>Declared <code>protected</code>; may be accessed within subclasses.</td>
         </tr>
         <tr>
            <td><code>ACC_STATIC</code></td>
            <td>0x0008</td>
            <td>Declared <code>static</code>.</td>
         </tr>
         <tr>
            <td><code>ACC_FINAL</code></td>
            <td>0x0010</td>
            <td>
            Declared <code>final</code>; must not be overridden (<a
               class="xref"
               href="jvms-5.html#jvms-5.4.5"
               title="5.4.5.&nbsp;Method Overriding"
               marked="1"
               >§5.4.5</a
            >).
            </td>
         </tr>
         <tr>
            <td><code>ACC_SYNCHRONIZED</code></td>
            <td>0x0020</td>
            <td>Declared <code>synchronized</code>; invocation is wrapped by a monitor use.</td>
         </tr>
         <tr>
            <td><code>ACC_BRIDGE</code></td>
            <td>0x0040</td>
            <td>A bridge method, generated by the compiler.</td>
         </tr>
         <tr>
            <td><code>ACC_VARARGS</code></td>
            <td>0x0080</td>
            <td>Declared with variable number of arguments.</td>
         </tr>
         <tr>
            <td><code>ACC_NATIVE</code></td>
            <td>0x0100</td>
            <td>
            Declared <code>native</code>; implemented in a language other than the Java programming language.
            </td>
         </tr>
         <tr>
            <td><code>ACC_ABSTRACT</code></td>
            <td>0x0400</td>
            <td>Declared <code>abstract</code>; no implementation is provided.</td>
         </tr>
         <tr>
            <td><code>ACC_STRICT</code></td>
            <td>0x0800</td>
            <td>
            <p>
               In a <code>class</code> file whose major version number is at least 46 and at most 60: Declared
               <code>strictfp</code>.
            </p>
            </td>
         </tr>
         <tr>
            <td><code>ACC_SYNTHETIC</code></td>
            <td>0x1000</td>
            <td>Declared synthetic; not present in the source code.</td>
         </tr>
      </tbody>
      </table>

      值 0x0800 仅在主版本号至少为 46 且最多为 60 的类文件中被解释为 ACC_STRICT 标志。对于此类文件中的方法，以下规则确定 ACC_STRICT 标志是否可以与其他组合设置 旗帜。 （设置 ACC_STRICT 标志限制了 Java SE 1.2 到 16 (§2.8) 中方法的浮点指令。）对于主版本号小于 46 或大于 60 的类文件中的方法，值 0x0800 不被解释为 ACC_STRICT 标志，而是未分配； 在这样的类文件中“设置 ACC_STRICT 标志”是没有意义的。

      类的方法可以设置表 4.6-A 中的任何标志。 但是，一个类的每个方法最多可以设置其 ACC_PUBLIC、ACC_PRIVATE 和 ACC_PROTECTED 标志之一（JLS §8.4.3）。

      接口方法可以设置表 4.6-A 中的任何标志，除了 ACC_PROTECTED、ACC_FINAL、ACC_SYNCHRONIZED 和 ACC_NATIVE (JLS §9.4)。 在版本号小于 52.0 的类文件中，接口的每个方法都必须设置其 ACC_PUBLIC 和 ACC_ABSTRACT 标志； 在版本号为 52.0 或更高版本的类文件中，接口的每个方法必须恰好设置其 ACC_PUBLIC 和 ACC_PRIVATE 标志之一。

      如果类或接口的方法设置了 ACC_ABSTRACT 标志，则它不能设置任何 ACC_PRIVATE、ACC_STATIC、ACC_FINAL、ACC_SYNCHRONIZED 或 ACC_NATIVE 标志，也不能（在主版本号至少为 46 且位于 大多数 60) 都设置了 ACC_STRICT 标志。

      实例初始化方法（第 2.9.1 节）最多可以设置 ACC_PUBLIC、ACC_PRIVATE 和 ACC_PROTECTED 标志之一，也可以设置 ACC_VARARGS 和 ACC_SYNTHETIC 标志，并且还可以（在主版本号为 至少 46 个，最多 60 个）设置了 ACC_STRICT 标志，但不得设置表 4.6-A 中的任何其他标志。

      在版本号为 51.0 或更高版本的类文件中，名称为 `<clinit>` 的方法必须设置其 ACC_STATIC 标志。

      类或接口初始化方法（§2.9.2）由 Java 虚拟机隐式调用。 其access_flags项的值除了设置ACC_STATIC标志和（在主版本号至少为46，最多为60的类文件中）ACC_STRICT标志外被忽略，该方法免于上述关于合法的规则 标志的组合。

      ACC_BRIDGE 标志用于指示编译器为 Java 编程语言生成的桥接方法。

      ACC_VARARGS 标志表示此方法在源代码级别采用可变数量的参数。 声明为采用可变数量参数的方法必须在 ACC_VARARGS 标志设置为 1 的情况下进行编译。所有其他方法必须在 ACC_VARARGS 标志设置为 0 的情况下进行编译。

      ACC_SYNTHETIC 标志表示此方法由编译器生成并且不会出现在源代码中，除非它是 §4.7.8 中指定的方法之一。

      表 4.6-A 中未分配的 access_flags 项的所有位都保留供将来使用。 （这包括对应于主版本号小于 46 或大于 60 的类文件中的 0x0800 的位。）它们应该在生成的类文件中设置为零，并且应该被 Java 虚拟机实现忽略。
- name_index;
   - name_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Utf8_info 结构（第 4.4.7 节），表示表示方法的有效非限定名称（第 4.2.2 节），或者（如果此方法在类而不是接口中）特殊方法 名称 `<init>`，或特殊方法名称 `<clinit>`。

- descriptor_index;
   - descriptor_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是表示有效方法描述符的 CONSTANT_Utf8_info 结构（第 4.3.3 节）。 此外：
      - 如果此方法在类而不是接口中，并且方法的名称是 `<init>`，则描述符必须表示一个 void 方法。
      - 如果方法的名称是`<clinit>`，则描述符必须表示一个void 方法，并且在版本号为51.0 或更高版本的类文件中，是一个不带参数的方法。
   - 如果在 access_flags 项中设置了 ACC_VARARGS 标志，则本规范的未来版本可能要求方法描述符的最后一个参数描述符是数组类型。

- attributes_count;
   - attributes_count 项的值表示该方法的附加属性的数量。

- `attributes[attributes_count];`
   - 属性表的每个值都必须是一个 attribute_info 结构（§4.7）。
   - 一个方法可以有任意数量的与之关联的可选属性。
   - 本规范定义的出现在 method_info 结构的属性表中的属性在表 4.7-C 中列出。
   - 有关定义为出现在 method_info 结构的属性表中的属性的规则在第 4.7 节中给出。
   - 有关 method_info 结构的属性表中非预定义属性的规则在第 4.7.1 节中给出。