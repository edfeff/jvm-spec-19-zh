# 【JVM 规范】第四章-类文件结构-2

> 本文是 JVM19 规范的个人中文译本，原文为 https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html


## 4.7. 属性

属性用于类文件格式的 ClassFile、field_info、method_info、Code_attribute 和 record_component_info 结构（§4.1、§4.5、§4.6、§4.7.3、§4.7.30）。

所有属性都具有以下通用格式：

```
attribute_info {
    u2 attribute_name_index;
    u4 attribute_length;
    u1 info[attribute_length];
}
```

对于所有属性，attribute_name_index 项必须是该类常量池中的有效无符号 16 位索引。 attribute_name_index 处的 constant_pool 条目必须是表示属性名称的 CONSTANT_Utf8_info 结构（§4.4.7）。 attribute_length 项的值指示后续信息的长度（以字节为单位）。 该长度不包括包含 attribute_name_index 和 attribute_length 项的前六个字节。

本规范预定义了 30 个属性。 为了便于导航，它们被列出了三次：
- 表 4.7-A 按本章中属性的节号排序。 每个属性都与定义它的类文件格式的第一个版本一起显示。 还显示了引入类文件格式版本（§4.1）的 Java SE 平台版本。
- 表 4.7-B 按定义每个属性的类文件格式的第一个版本排序。
- 表 4.7-C 按类文件中每个属性定义出现的位置排序。


在本规范中使用它们的上下文中，即在它们出现的类文件结构的属性表中，这些预定义属性的名称是保留的。

属性表中预定义属性存在的任何条件都在描述该属性的部分中明确指定。 如果未指定条件，则该属性可以在属性表中出现任意次数。

预定义属性根据其用途分为三组：
1. 七个属性对于 Java 虚拟机正确解释类文件至关重要：

   - ConstantValue
   - Code
   - StackMapTable
   - BootstrapMethods
   - NestHost
   - NestMembers
   - PermittedSubclasses

   在版本号为 v 的类文件中，如果实现支持类文件格式的版本 v，并且属性首先在版本 v 中定义，则这些属性中的每一个都必须被 Java 虚拟机的实现识别并正确读取 较早的类文件格式，并且该属性出现在它被定义出现的位置。

2. 十个属性对于 Java 虚拟机正确解释类文件并不重要，但对于 Java SE 平台的类库正确解释类文件至关重要，或者对工具有用（在这种情况下，部分 指定一个属性将其描述为“可选”）：
   - Exceptions
   - InnerClasses
   - EnclosingMethod
   - Synthetic
   - Signature
   - Record
   - SourceFile
   - LineNumberTable
   - LocalVariableTable
   - LocalVariableTypeTable

   在版本号为 v 的类文件中，如果实现支持类文件格式的版本 v，并且属性首先在版本 v 中定义，则这些属性中的每一个都必须被 Java 虚拟机的实现识别并正确读取 较早的类文件格式，并且该属性出现在它被定义出现的位置。


3. 十三个属性对于 Java 虚拟机正确解释类文件并不重要，但包含有关类文件的元数据，这些元数据要么由 Java SE 平台的类库公开，要么由工具提供（在这种情况下，部分 指定一个属性将其描述为“可选”）：
   - SourceDebugExtension
   - Deprecated
   - RuntimeVisibleAnnotations
   - RuntimeInvisibleAnnotations
   - RuntimeVisibleParameterAnnotations
   - RuntimeInvisibleParameterAnnotations
   - RuntimeVisibleTypeAnnotations
   - RuntimeInvisibleTypeAnnotations
   - AnnotationDefault
   - MethodParameters
   - Module
   - ModulePackages
   - ModuleMainClass
   
   Java 虚拟机的实现可能会使用这些属性包含的信息，否则必须默默地忽略这些属性。

Table 4.7-A. Predefined class file attributes (by section)

<table border="1">
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Section</th>
      <th><code>class</code> file</th>
      <th>Java SE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>ConstantValue</code></td>
      <td>
        <a class="xref" title="4.7.2.&nbsp;The ConstantValue Attribute" marked="1">§4.7.2</a>
      </td>
      <td>45.3</td>
      <td>1.0.2</td>
    </tr>
    <tr>
      <td><code>Code</code></td>
      <td>
        <a class="xref">§4.7.3</a>
      </td>
      <td>45.3</td>
      <td>1.0.2</td>
    </tr>
    <tr>
      <td><code>StackMapTable</code></td>
      <td>
        <a class="xref" title="4.7.4.&nbsp;The StackMapTable Attribute" marked="1">§4.7.4</a>
      </td>
      <td>50.0</td>
      <td>6</td>
    </tr>
    <tr>
      <td><code>Exceptions</code></td>
      <td>
        <a class="xref">§4.7.5</a>
      </td>
      <td>45.3</td>
      <td>1.0.2</td>
    </tr>
    <tr>
      <td><code>InnerClasses</code></td>
      <td>
        <a class="xref" title="4.7.6.&nbsp;The InnerClasses Attribute" marked="1">§4.7.6</a>
      </td>
      <td>45.3</td>
      <td>1.1</td>
    </tr>
    <tr>
      <td><code>EnclosingMethod</code></td>
      <td>
        <a class="xref" title="4.7.7.&nbsp;The EnclosingMethod Attribute" marked="1">§4.7.7</a>
      </td>
      <td>49.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td><code>Synthetic</code></td>
      <td>
        <a class="xref">§4.7.8</a>
      </td>
      <td>45.3</td>
      <td>1.1</td>
    </tr>
    <tr>
      <td><code>Signature</code></td>
      <td>
        <a class="xref">§4.7.9</a>
      </td>
      <td>49.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td><code>SourceFile</code></td>
      <td>
        <a class="xref" title="4.7.10.&nbsp;The SourceFile Attribute" marked="1">§4.7.10</a>
      </td>
      <td>45.3</td>
      <td>1.0.2</td>
    </tr>
    <tr>
      <td><code>SourceDebugExtension</code></td>
      <td>
        <a class="xref" title="4.7.11.&nbsp;The SourceDebugExtension Attribute" marked="1">§4.7.11</a>
      </td>
      <td>49.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td><code>LineNumberTable</code></td>
      <td>
        <a class="xref" title="4.7.12.&nbsp;The LineNumberTable Attribute" marked="1">§4.7.12</a>
      </td>
      <td>45.3</td>
      <td>1.0.2</td>
    </tr>
    <tr>
      <td><code>LocalVariableTable</code></td>
      <td>
        <a class="xref" title="4.7.13.&nbsp;The LocalVariableTable Attribute" marked="1">§4.7.13</a>
      </td>
      <td>45.3</td>
      <td>1.0.2</td>
    </tr>
    <tr>
      <td><code>LocalVariableTypeTable</code></td>
      <td>
        <a class="xref" title="4.7.14.&nbsp;The LocalVariableTypeTable Attribute" marked="1">§4.7.14</a>
      </td>
      <td>49.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td><code>Deprecated</code></td>
      <td>
        <a class="xref" title="4.7.15.&nbsp;The Deprecated Attribute" marked="1">§4.7.15</a>
      </td>
      <td>45.3</td>
      <td>1.1</td>
    </tr>
    <tr>
      <td><code>RuntimeVisibleAnnotations</code></td>
      <td>
        <a class="xref" title="4.7.16.&nbsp;The RuntimeVisibleAnnotations Attribute" marked="1">§4.7.16</a>
      </td>
      <td>49.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td><code>RuntimeInvisibleAnnotations</code></td>
      <td>
        <a class="xref" title="4.7.17.&nbsp;The RuntimeInvisibleAnnotations Attribute" marked="1">§4.7.17</a>
      </td>
      <td>49.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td><code>RuntimeVisibleParameterAnnotations</code></td>
      <td>
        <a class="xref" title="4.7.18.&nbsp;The RuntimeVisibleParameterAnnotations Attribute" marked="1"
          >§4.7.18</a
        >
      </td>
      <td>49.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td><code>RuntimeInvisibleParameterAnnotations</code></td>
      <td>
        <a class="xref" title="4.7.19.&nbsp;The RuntimeInvisibleParameterAnnotations Attribute" marked="1"
          >§4.7.19</a
        >
      </td>
      <td>49.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td><code>RuntimeVisibleTypeAnnotations</code></td>
      <td>
        <a class="xref" title="4.7.20.&nbsp;The RuntimeVisibleTypeAnnotations Attribute" marked="1"
          >§4.7.20</a
        >
      </td>
      <td>52.0</td>
      <td>8</td>
    </tr>
    <tr>
      <td><code>RuntimeInvisibleTypeAnnotations</code></td>
      <td>
        <a class="xref" title="4.7.21.&nbsp;The RuntimeInvisibleTypeAnnotations Attribute" marked="1"
          >§4.7.21</a
        >
      </td>
      <td>52.0</td>
      <td>8</td>
    </tr>
    <tr>
      <td><code>AnnotationDefault</code></td>
      <td>
        <a class="xref" title="4.7.22.&nbsp;The AnnotationDefault Attribute" marked="1">§4.7.22</a>
      </td>
      <td>49.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td><code>BootstrapMethods</code></td>
      <td>
        <a class="xref" title="4.7.23.&nbsp;The BootstrapMethods Attribute" marked="1">§4.7.23</a>
      </td>
      <td>51.0</td>
      <td>7</td>
    </tr>
    <tr>
      <td><code>MethodParameters</code></td>
      <td>
        <a class="xref" title="4.7.24.&nbsp;The MethodParameters Attribute" marked="1">§4.7.24</a>
      </td>
      <td>52.0</td>
      <td>8</td>
    </tr>
    <tr>
      <td>
        <p><code>Module</code></p>
      </td>
      <td>
        <a class="xref">§4.7.25</a>
      </td>
      <td>53.0</td>
      <td>9</td>
    </tr>
    <tr>
      <td>
        <p><code>ModulePackages</code></p>
      </td>
      <td>
        <a class="xref" title="4.7.26.&nbsp;The ModulePackages Attribute" marked="1">§4.7.26</a>
      </td>
      <td>53.0</td>
      <td>9</td>
    </tr>
    <tr>
      <td>
        <p><code>ModuleMainClass</code></p>
      </td>
      <td>
        <a class="xref" title="4.7.27.&nbsp;The ModuleMainClass Attribute" marked="1">§4.7.27</a>
      </td>
      <td>53.0</td>
      <td>9</td>
    </tr>
    <tr>
      <td>
        <p><code>NestHost</code></p>
      </td>
      <td>
        <a class="xref">§4.7.28</a>
      </td>
      <td>55.0</td>
      <td>11</td>
    </tr>
    <tr>
      <td>
        <p><code>NestMembers</code></p>
      </td>
      <td>
        <a class="xref" title="4.7.29.&nbsp;The NestMembers Attribute" marked="1">§4.7.29</a>
      </td>
      <td>55.0</td>
      <td>11</td>
    </tr>
    <tr>
      <td>
        <p><code>Record</code></p>
      </td>
      <td>
        <a class="xref">§4.7.30</a>
      </td>
      <td>60.0</td>
      <td>16</td>
    </tr>
    <tr>
      <td>
        <p><code>PermittedSubclasses</code></p>
      </td>
      <td>
        <a class="xref" title="4.7.31.&nbsp;The PermittedSubclasses Attribute" marked="1">§4.7.31</a>
      </td>
      <td>61.0</td>
      <td>17</td>
    </tr>
  </tbody>
</table>



Table 4.7-B. Predefined class file attributes (by class file format)
<table border="1">
  <thead>
    <tr>
      <th>Attribute</th>
      <th><code>class</code> file</th>
      <th>Java SE</th>
      <th>Section</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>ConstantValue</code></td>
      <td>45.3</td>
      <td>1.0.2</td>
      <td>
        <a class="xref" title="4.7.2.&nbsp;The ConstantValue Attribute" marked="1">§4.7.2</a>
      </td>
    </tr>
    <tr>
      <td><code>Code</code></td>
      <td>45.3</td>
      <td>1.0.2</td>
      <td>
        <a class="xref">§4.7.3</a>
      </td>
    </tr>
    <tr>
      <td><code>Exceptions</code></td>
      <td>45.3</td>
      <td>1.0.2</td>
      <td>
        <a class="xref">§4.7.5</a>
      </td>
    </tr>
    <tr>
      <td><code>SourceFile</code></td>
      <td>45.3</td>
      <td>1.0.2</td>
      <td>
        <a class="xref" title="4.7.10.&nbsp;The SourceFile Attribute" marked="1">§4.7.10</a>
      </td>
    </tr>
    <tr>
      <td><code>LineNumberTable</code></td>
      <td>45.3</td>
      <td>1.0.2</td>
      <td>
        <a class="xref" title="4.7.12.&nbsp;The LineNumberTable Attribute" marked="1">§4.7.12</a>
      </td>
    </tr>
    <tr>
      <td><code>LocalVariableTable</code></td>
      <td>45.3</td>
      <td>1.0.2</td>
      <td>
        <a class="xref" title="4.7.13.&nbsp;The LocalVariableTable Attribute" marked="1">§4.7.13</a>
      </td>
    </tr>
    <tr>
      <td><code>InnerClasses</code></td>
      <td>45.3</td>
      <td>1.1</td>
      <td>
        <a class="xref" title="4.7.6.&nbsp;The InnerClasses Attribute" marked="1">§4.7.6</a>
      </td>
    </tr>
    <tr>
      <td><code>Synthetic</code></td>
      <td>45.3</td>
      <td>1.1</td>
      <td>
        <a class="xref">§4.7.8</a>
      </td>
    </tr>
    <tr>
      <td><code>Deprecated</code></td>
      <td>45.3</td>
      <td>1.1</td>
      <td>
        <a class="xref" title="4.7.15.&nbsp;The Deprecated Attribute" marked="1">§4.7.15</a>
      </td>
    </tr>
    <tr>
      <td><code>EnclosingMethod</code></td>
      <td>49.0</td>
      <td>5.0</td>
      <td>
        <a class="xref" title="4.7.7.&nbsp;The EnclosingMethod Attribute" marked="1">§4.7.7</a>
      </td>
    </tr>
    <tr>
      <td><code>Signature</code></td>
      <td>49.0</td>
      <td>5.0</td>
      <td>
        <a class="xref">§4.7.9</a>
      </td>
    </tr>
    <tr>
      <td><code>SourceDebugExtension</code></td>
      <td>49.0</td>
      <td>5.0</td>
      <td>
        <a class="xref" title="4.7.11.&nbsp;The SourceDebugExtension Attribute" marked="1">§4.7.11</a>
      </td>
    </tr>
    <tr>
      <td><code>LocalVariableTypeTable</code></td>
      <td>49.0</td>
      <td>5.0</td>
      <td>
        <a class="xref" title="4.7.14.&nbsp;The LocalVariableTypeTable Attribute" marked="1">§4.7.14</a>
      </td>
    </tr>
    <tr>
      <td><code>RuntimeVisibleAnnotations</code></td>
      <td>49.0</td>
      <td>5.0</td>
      <td>
        <a class="xref" title="4.7.16.&nbsp;The RuntimeVisibleAnnotations Attribute" marked="1">§4.7.16</a>
      </td>
    </tr>
    <tr>
      <td><code>RuntimeInvisibleAnnotations</code></td>
      <td>49.0</td>
      <td>5.0</td>
      <td>
        <a class="xref" title="4.7.17.&nbsp;The RuntimeInvisibleAnnotations Attribute" marked="1">§4.7.17</a>
      </td>
    </tr>
    <tr>
      <td><code>RuntimeVisibleParameterAnnotations</code></td>
      <td>49.0</td>
      <td>5.0</td>
      <td>
        <a class="xref" title="4.7.18.&nbsp;The RuntimeVisibleParameterAnnotations Attribute" marked="1"
          >§4.7.18</a
        >
      </td>
    </tr>
    <tr>
      <td><code>RuntimeInvisibleParameterAnnotations</code></td>
      <td>49.0</td>
      <td>5.0</td>
      <td>
        <a class="xref" title="4.7.19.&nbsp;The RuntimeInvisibleParameterAnnotations Attribute" marked="1"
          >§4.7.19</a
        >
      </td>
    </tr>
    <tr>
      <td><code>AnnotationDefault</code></td>
      <td>49.0</td>
      <td>5.0</td>
      <td>
        <a class="xref" title="4.7.22.&nbsp;The AnnotationDefault Attribute" marked="1">§4.7.22</a>
      </td>
    </tr>
    <tr>
      <td><code>StackMapTable</code></td>
      <td>50.0</td>
      <td>6</td>
      <td>
        <a class="xref" title="4.7.4.&nbsp;The StackMapTable Attribute" marked="1">§4.7.4</a>
      </td>
    </tr>
    <tr>
      <td><code>BootstrapMethods</code></td>
      <td>51.0</td>
      <td>7</td>
      <td>
        <a class="xref" title="4.7.23.&nbsp;The BootstrapMethods Attribute" marked="1">§4.7.23</a>
      </td>
    </tr>
    <tr>
      <td><code>RuntimeVisibleTypeAnnotations</code></td>
      <td>52.0</td>
      <td>8</td>
      <td>
        <a class="xref" title="4.7.20.&nbsp;The RuntimeVisibleTypeAnnotations Attribute" marked="1"
          >§4.7.20</a
        >
      </td>
    </tr>
    <tr>
      <td><code>RuntimeInvisibleTypeAnnotations</code></td>
      <td>52.0</td>
      <td>8</td>
      <td>
        <a class="xref" title="4.7.21.&nbsp;The RuntimeInvisibleTypeAnnotations Attribute" marked="1"
          >§4.7.21</a
        >
      </td>
    </tr>
    <tr>
      <td><code>MethodParameters</code></td>
      <td>52.0</td>
      <td>8</td>
      <td>
        <a class="xref" title="4.7.24.&nbsp;The MethodParameters Attribute" marked="1">§4.7.24</a>
      </td>
    </tr>
    <tr>
      <td>
        <p>
          <code>Module</code>
        </p>
      </td>
      <td>53.0</td>
      <td>9</td>
      <td>
        <a class="xref">§4.7.25</a>
      </td>
    </tr>
    <tr>
      <td>
        <p>
          <code>ModulePackages</code>
        </p>
      </td>
      <td>53.0</td>
      <td>9</td>
      <td>
        <a class="xref" title="4.7.26.&nbsp;The ModulePackages Attribute" marked="1">§4.7.26</a>
      </td>
    </tr>
    <tr>
      <td>
        <p>
          <code>ModuleMainClass</code>
        </p>
      </td>
      <td>53.0</td>
      <td>9</td>
      <td>
        <a class="xref" title="4.7.27.&nbsp;The ModuleMainClass Attribute" marked="1">§4.7.27</a>
      </td>
    </tr>
    <tr>
      <td>
        <p>
          <code>NestHost</code>
        </p>
      </td>
      <td>55.0</td>
      <td>11</td>
      <td>
        <a class="xref">§4.7.28</a>
      </td>
    </tr>
    <tr>
      <td>
        <p>
          <code>NestMembers</code>
        </p>
      </td>
      <td>55.0</td>
      <td>11</td>
      <td>
        <a class="xref" title="4.7.29.&nbsp;The NestMembers Attribute" marked="1">§4.7.29</a>
      </td>
    </tr>
    <tr>
      <td>
        <p>
          <code>Record</code>
        </p>
      </td>
      <td>60.0</td>
      <td>16</td>
      <td>
        <a class="xref">§4.7.30</a>
      </td>
    </tr>
    <tr>
      <td>
        <p>
          <code>PermittedSubclasses</code>
        </p>
      </td>
      <td>61.0</td>
      <td>17</td>
      <td>
        <a class="xref" title="4.7.31.&nbsp;The PermittedSubclasses Attribute" marked="1">§4.7.31</a>
      </td>
    </tr>
  </tbody>
</table>


Table 4.7-C. Predefined class file attributes (by location)

<table class="table" summary="Predefined class file attributes (by location)" border="1">
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Location</th>
      <th><code>class</code> file</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>SourceFile</code></td>
      <td><code>ClassFile</code></td>
      <td>45.3</td>
    </tr>
    <tr>
      <td><code>InnerClasses</code></td>
      <td><code>ClassFile</code></td>
      <td>45.3</td>
    </tr>
    <tr>
      <td><code>EnclosingMethod</code></td>
      <td><code>ClassFile</code></td>
      <td>49.0</td>
    </tr>
    <tr>
      <td><code>SourceDebugExtension</code></td>
      <td><code>ClassFile</code></td>
      <td>49.0</td>
    </tr>
    <tr>
      <td><code>BootstrapMethods</code></td>
      <td><code>ClassFile</code></td>
      <td>51.0</td>
    </tr>
    <tr>
      <td>
        <p>
          <code>Module</code>, <code>ModulePackages</code>,
          <code>ModuleMainClass</code>
        </p>
      </td>
      <td><code>ClassFile</code></td>
      <td>53.0</td>
    </tr>
    <tr>
      <td>
        <p><code>NestHost</code>, <code>NestMembers</code></p>
      </td>
      <td><code>ClassFile</code></td>
      <td>55.0</td>
    </tr>
    <tr>
      <td>
        <p>
          <code>Record</code>
        </p>
      </td>
      <td><code>ClassFile</code></td>
      <td>60.0</td>
    </tr>
    <tr>
      <td>
        <p>
          <code>PermittedSubclasses</code>
        </p>
      </td>
      <td><code>ClassFile</code></td>
      <td>61.0</td>
    </tr>
    <tr>
      <td><code>ConstantValue</code></td>
      <td><code>field_info</code></td>
      <td>45.3</td>
    </tr>
    <tr>
      <td><code>Code</code></td>
      <td><code>method_info</code></td>
      <td>45.3</td>
    </tr>
    <tr>
      <td><code>Exceptions</code></td>
      <td><code>method_info</code></td>
      <td>45.3</td>
    </tr>
    <tr>
      <td>
        <code>RuntimeVisibleParameterAnnotations</code>,
        <code>RuntimeInvisibleParameterAnnotations</code>
      </td>
      <td><code>method_info</code></td>
      <td>49.0</td>
    </tr>
    <tr>
      <td><code>AnnotationDefault</code></td>
      <td><code>method_info</code></td>
      <td>49.0</td>
    </tr>
    <tr>
      <td><code>MethodParameters</code></td>
      <td><code>method_info</code></td>
      <td>52.0</td>
    </tr>
  </tbody>
</table>

Table 4.7-C (cont.). Predefined class file attributes (by location)

<table border="1">
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Location</th>
      <th><code>class</code> file</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>Synthetic</code></td>
      <td>
        <code>ClassFile</code>, <code>field_info</code>,
        <code>method_info</code>
      </td>
      <td>45.3</td>
    </tr>
    <tr>
      <td><code>Deprecated</code></td>
      <td>
        <code>ClassFile</code>, <code>field_info</code>,
        <code>method_info</code>
      </td>
      <td>45.3</td>
    </tr>
    <tr>
      <td><code>Signature</code></td>
      <td>
        <p>
          <code>ClassFile</code>, <code>field_info</code>, <code>method_info</code>,
          <code>record_component_info</code>
        </p>
      </td>
      <td>49.0</td>
    </tr>
    <tr>
      <td>
        <code>RuntimeVisibleAnnotations</code>,
        <code>RuntimeInvisibleAnnotations</code>
      </td>
      <td>
        <p>
          <code>ClassFile</code>, <code>field_info</code>, <code>method_info</code>,
          <code>record_component_info</code>
        </p>
      </td>
      <td>49.0</td>
    </tr>
    <tr>
      <td><code>LineNumberTable</code></td>
      <td><code>Code</code></td>
      <td>45.3</td>
    </tr>
    <tr>
      <td><code>LocalVariableTable</code></td>
      <td><code>Code</code></td>
      <td>45.3</td>
    </tr>
    <tr>
      <td><code>LocalVariableTypeTable</code></td>
      <td><code>Code</code></td>
      <td>49.0</td>
    </tr>
    <tr>
      <td><code>StackMapTable</code></td>
      <td><code>Code</code></td>
      <td>50.0</td>
    </tr>
    <tr>
      <td>
        <code>RuntimeVisibleTypeAnnotations</code>,
        <code>RuntimeInvisibleTypeAnnotations</code>
      </td>
      <td>
        <p>
          <code>ClassFile</code>, <code>field_info</code>, <code>method_info</code>, <code>Code</code>,
          <code>record_component_info</code>
        </p>
      </td>
      <td>52.0</td>
    </tr>
  </tbody>
</table>


### 4.7.1. 定义和命名新属性

允许编译器在类文件结构、field_info 结构、method_info 结构和代码属性（§4.7.3）的属性表中定义和发出包含新属性的类文件。 允许 Java 虚拟机实现识别和使用在这些属性表中找到的新属性。 但是，任何未定义为本规范一部分的属性不得影响类文件的语义。 Java 虚拟机实现需要默默地忽略它们不识别的属性。

例如，允许定义一个新属性以支持特定于供应商的调试。 因为 Java 虚拟机实现需要忽略它们无法识别的属性，所以用于该特定 Java 虚拟机实现的类文件将可供其他实现使用，即使这些实现无法使用类文件包含的附加调试信息。

Java 虚拟机实现被明确禁止仅仅因为存在一些新属性而抛出异常或以其他方式拒绝使用类文件。 当然，如果给定的类文件不包含它们需要的所有属性，则对类文件进行操作的工具可能无法正确运行。

两个旨在不同的属性，但碰巧使用相同的属性名称并且具有相同的长度，将在识别任一属性的实现上发生冲突。 除本规范外定义的属性应根据 Java 语言规范，Java SE 19 版 (JLS §6.1) 中描述的包命名约定选择名称。

本规范的未来版本可能会定义其他属性。

