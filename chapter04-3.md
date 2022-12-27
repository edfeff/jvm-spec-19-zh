# 【JVM 规范】第四章-类文件结构-3

> 本文是 JVM19 规范的个人中文译本，原文为 https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html

## 4.8. 格式检查

当 Java 虚拟机（§5.3）加载预期的类文件时，Java 虚拟机首先确保该文件具有类文件的基本格式（§4.1）。 此过程称为格式检查。 检查如下：

  - 前四个字节必须包含正确的魔数。

  - 除了 StackMapTable、RuntimeVisibleAnnotations、RuntimeInvisibleAnnotations、RuntimeVisibleParameterAnnotations、RuntimeInvisibleParameterAnnotations、RuntimeVisibleTypeAnnotations、RuntimeInvisibleTypeAnnotations 和 AnnotationDefault 之外，所有预定义属性（§4.7）都必须具有适当的长度。

  - 类文件不得被截断或末尾有额外的字节。

  - 常量池必须满足 §4.4 中记录的约束。

    例如，常量池中的每个 CONSTANT_Class_info 结构必须在其 name_index 项中包含 CONSTANT_Utf8_info 结构的有效常量池索引。

    常量池中的所有字段引用和方法引用都必须具有有效的名称、有效的类和有效的描述符（§4.3）。

    格式检查不能确保给定的字段或方法确实存在于给定的类中，也不能确保给定的描述符引用真实的类。 格式检查仅确保这些项目格式正确。 当字节码本身被验证时，以及在解析期间，会执行更详细的检查。

这些对基本类文件完整性的检查对于类文件内容的任何解释都是必需的。 格式检查不同于字节码验证，尽管在历史上它们一直被混淆，因为两者都是完整性检查的一种形式。

