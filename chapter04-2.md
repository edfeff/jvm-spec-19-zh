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


### 4.7.2. ConstantValue
ConstantValue 属性是 field_info 结构（§4.5）的属性表中的固定长度属性。 ConstantValue 属性表示常量表达式 (JLS §15.28) 的值，并按如下方式使用：

- 如果 field_info 结构的 access_flags 项中的 ACC_STATIC 标志被设置，则 field_info 结构表示的字段将被分配由其 ConstantValue 属性表示的值，作为声明该字段的类或接口的初始化的一部分（§5.5）。 这发生在调用该类或接口的类或接口初始化方法之前（第 2.9.2 节）。

- 否则，Java 虚拟机必须默默地忽略该属性。

在 field_info 结构的属性表中最多可能有一个 ConstantValue 属性。

ConstantValue 属性具有以下格式：
```
ConstantValue_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 constantvalue_index;
}
```

- attribute_name_index;
  - attribute_name_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是表示字符串“ConstantValue”的 CONSTANT_Utf8_info 结构（§4.4.7）。

- attribute_length;
  - attribute_length 项的值必须为二。

- constantvalue_index;
  - constantvalue_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目给出了该属性表示的值。 constant_pool 条目必须是适合该字段的类型，如表 4.7.2-A 中指定的那样。

  Table 4.7.2-A. Constant value attribute types
  <table border="1">
      <thead>
          <tr>
              <th>字段类型</th>
              <th>条目类型</th>
          </tr>
      </thead>
      <tbody>
          <tr>
              <td><code>int</code>, <code>short</code>, <code>char</code>,
                  <code>byte</code>, <code>boolean</code>
              </td>
              <td><code>CONSTANT_Integer</code></td>
          </tr>
          <tr>
              <td><code>float</code></td>
              <td><code>CONSTANT_Float</code></td>
          </tr>
          <tr>
              <td><code>long</code></td>
              <td><code>CONSTANT_Long</code></td>
          </tr>
          <tr>
              <td><code>double</code></td>
              <td><code>CONSTANT_Double</code></td>
          </tr>
          <tr>
              <td><code>String</code></td>
              <td><code>CONSTANT_String</code></td>
          </tr>
      </tbody>
  </table>


### 4.7.3. Code
Code 属性是 method_info 结构（§4.6）的属性表中的可变长度属性。 代码属性包含 Java 虚拟机指令和方法的辅助信息，包括实例初始化方法和类或接口初始化方法（§2.9.1，§2.9.2）。

如果该方法是本地方法或抽象方法，并且不是类或接口初始化方法，则其 method_info 结构在其属性表中不得具有 Code 属性。 否则，其 method_info 结构在其属性表中必须只有一个代码属性。

代码属性具有以下格式：
```
Code_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 max_stack;
    u2 max_locals;
    u4 code_length;
    u1 code[code_length];
    u2 exception_table_length;
    {   u2 start_pc;
        u2 end_pc;
        u2 handler_pc;
        u2 catch_type;
    } exception_table[exception_table_length];
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```

Code_attribute结构体的各项如下：

- `attribute_name_index;`
  - attribute_name_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是表示字符串“Code”的 CONSTANT_Utf8_info 结构（§4.4.7）。
- `attribute_length;`
  - attribute_length 项的值表示属性的长度，不包括最初的六个字节。
- `max_stack;`
  - max_stack 项的值给出了该方法（§2.6.2）在执行该方法期间的任何时间点的操作数栈的最大深度。
- `max_locals;`
  - max_locals 项的值给出调用此方法时分配的局部变量数组中局部变量的数量（第 2.6.1 节），包括用于在方法调用时将参数传递给方法的局部变量。
  - long 或 double 类型值的最大局部变量索引是 max_locals - 2。任何其他类型值的最大局部变量索引是 max_locals - 1。

- `code_length;`
  - code_length 项的值给出了此方法的代码数组中的字节数。
  - code_length 的值必须大于零（因为代码数组不能为空）且小于 65536。

- `code[];`
  - 代码数组给出了实现该方法的 Java 虚拟机代码的实际字节数。

  - 在字节寻址机器上将代码数组读入内存时，如果数组的第一个字节在 4 字节边界上对齐，则 tableswitch 和 lookupswitch 32 位偏移量将是 4 字节对齐的。 （有关代码数组对齐结果的更多信息，请参阅这些指令的描述。）

  - 对代码数组内容的详细约束非常广泛，在单独的部分 (§4.9) 中给出。

- `exception_table_length;`
  - exception_table_length 项的值给出了 exception_table 数组中的条目数。

- `exception_table[]`
  - exception_table 数组中的每一项都描述了代码数组中的一个异常处理程序。 exception_table 数组中处理程序的顺序很重要（§2.10）。

  - 每个 exception_table 条目包含以下四项：

    - start_pc, end_pc

      - start_pc 和 end_pc 两项的值表示代码数组中异常处理程序处于活动状态的范围。 start_pc 的值必须是指令操作码代码数组中的有效索引。 end_pc 的值必须是指令操作码代码数组的有效索引，或者必须等于代码数组的长度 code_length。 start_pc 的值必须小于 end_pc 的值。

      - start_pc 是包含的，end_pc 是独占的； 也就是说，当程序计数器在区间 [start_pc, end_pc) 内时，异常处理程序必须处于活动状态。

      - end_pc 是独占的这一事实是 Java 虚拟机设计中的一个历史性错误：如果 Java 虚拟机代码的一个方法恰好是 65535 字节长并且以 1 字节长的指令结束，那么该指令就不能被保护 通过异常处理程序。 编译器编写者可以通过将为任何方法、实例初始化方法或静态初始化程序（任何代码数组的大小）生成的 Java 虚拟机代码的最大大小限制为 65534 字节来解决此错误。

    - handler_pc
      - handler_pc 项的值表示异常处理程序的开始。 该项的值必须是代码数组中的有效索引，并且必须是指令操作码的索引。

    - catch_type
      - 如果 catch_type 项的值非零，则它必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Class_info 结构（第 4.4.1 节），表示此异常处理程序指定要捕获的一类异常。 仅当抛出的异常是给定类或其子类之一的实例时，才会调用异常处理程序。

      - 验证器检查该类是 Throwable 还是 Throwable 的子类（§4.9.2）。

      - 如果 catch_type 项的值为零，则为所有异常调用此异常处理程序。

      - 这用于实现 finally (§3.13)。
- attributes_count
  - attributes_count项的值表示Code属性的属性个数。

- attributes[]
  - 属性表的每个值都必须是一个 attribute_info 结构（§4.7）。
  - 代码属性可以有任意数量的可选属性与之关联。
  - 本规范定义的出现在代码属性的属性表中的属性在表 4.7-C 中列出。
  - §4.7 中给出了有关定义为出现在代码属性的属性表中的属性的规则。
  - Code 属性的属性表中有关非预定义属性的规则在§4.7.1 中给出。

### 4.7.4. StackMapTable

StackMapTable 属性是代码属性（§4.7.3）的属性表中的可变长度属性。 StackMapTable 属性在通过类型检查（§4.10.1）进行验证的过程中使用。

Code 属性的属性表中最多可能有一个 StackMapTable 属性。

在版本号为 50.0 或更高版本的类文件中，如果方法的代码属性没有 StackMapTable 属性，则它具有隐式栈映射属性（§4.10.1）。 此隐式栈映射属性等效于 number_of_entries 等于零的 StackMapTable 属性。

StackMapTable 属性具有以下格式：
```
StackMapTable_attribute {
    u2              attribute_name_index;
    u4              attribute_length;
    u2              number_of_entries;
    stack_map_frame entries[number_of_entries];
}
```

- `attribute_name_index;`
  - attribute_name_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是表示字符串“StackMapTable”的 CONSTANT_Utf8_info 结构（§4.4.7）。
- `attribute_length;`
  - attribute_length 项的值表示属性的长度，不包括最初的六个字节。
- `number_of_entries;`
  - number_of_entries 项的值给出了条目表中 stack_map_frame 条目的数量。
- `entries[];`
  - 条目表中的每个条目描述了该方法的一个栈映射帧。 条目表中栈映射帧的顺序很重要。


栈映射帧指定（显式或隐式）它应用的字节码偏移量，以及该偏移量的局部变量和操作数栈条目的验证类型。

条目表中描述的每个栈映射帧都依赖于前一帧的某些语义。 方法的第一个栈映射帧是隐式的，由类型检查器（§4.10.1.6）根据方法描述符计算得出。 因此，entries[0] 处的 stack_map_frame 结构描述了该方法的第二个栈映射帧。

栈映射帧应用的字节码偏移量是通过取帧中指定的值 offset_delta（显式或隐式）并将 offset_delta + 1 添加到前一帧的字节码偏移量来计算的，除非前一帧是初始帧 的方法。 在这种情况下，栈映射帧应用的字节码偏移量是帧中指定的值 offset_delta。

通过使用偏移量增量而不是存储实际的字节码偏移量，我们根据定义确保栈映射帧的排序顺序正确。 此外，通过对所有显式帧（而不是隐式第一帧）始终使用公式 offset_delta + 1，我们保证没有重复。

如果指令从 Code 属性的代码数组中的偏移量 i 开始，并且 Code 属性具有 StackMapTable 属性，其条目数组包含适用于字节码的栈映射帧，则我们说字节码中的指令具有相应的栈映射帧 抵消我。


验证类型指定一个或两个位置的类型，其中一个位置是单个局部变量或单个操作数栈条目。 验证类型由一个可区分的联合 verification_type_info 表示，它由一个单字节标记组成，指示联合中的哪一项正在使用，后跟零个或多个字节，提供有关标记的更多信息。

```
union verification_type_info {
    Top_variable_info;
    Integer_variable_info;
    Float_variable_info;
    Long_variable_info;
    Double_variable_info;
    Null_variable_info;
    UninitializedThis_variable_info;
    Object_variable_info;
    Uninitialized_variable_info;
}
```
指定局部变量数组或操作数堆栈中一个位置的验证类型由 verification_type_info 联合的以下项表示：

- Top_variable_info 项表示局部变量的验证类型为top。
  ```
  Top_variable_info {
    u1 tag = ITEM_Top; /* 0 */
  }
  ```
- Integer_variable_info 项表示该位置具有验证类型 int。
  ```
  Integer_variable_info {
      u1 tag = ITEM_Integer; /* 1 */
  }
  ```

- Float_variable_info 项指示该位置具有浮动验证类型。
  ```
  Float_variable_info {
    u1 tag = ITEM_Float; /* 2 */
  }
  ```

- Null_variable_info 类型表示该位置的验证类型为空。
  ```
  Null_variable_info {
    u1 tag = ITEM_Null; /* 5 */
  }
  ```

- UninitializedThis_variable_info 项指示该位置具有验证类型 uninitializedThis。
  ```
  UninitializedThis_variable_info {
    u1 tag = ITEM_UninitializedThis; /* 6 */
  }
  ```


- Object_variable_info 项指示该位置具有验证类型，该验证类型是由在 constant_pool 表中 cpool_index 给定索引处找到的 CONSTANT_Class_info 结构（§4.4.1）表示的类。
  ```
  Object_variable_info {
      u1 tag = ITEM_Object; /* 7 */
      u2 cpool_index;
  }
  ```

    
- Uninitialized_variable_info 项指示该位置具有未初始化的验证类型（偏移量）。 Offset 项表示在包含此 StackMapTable 属性的 Code 属性的代码数组中，创建存储在该位置的对象的新指令 (§new) 的偏移量。
  ```
  Uninitialized_variable_info {
    u1 tag = ITEM_Uninitialized; /* 8 */
    u2 offset;
  }
  ```
    
在局部变量数组或操作数堆栈中指定两个位置的验证类型由 verification_type_info 联合的以下项表示：

  - Long_variable_info 项表示两个位置中的第一个位置的验证类型为 long。
    ```
    Long_variable_info {
    u1 tag = ITEM_Long; /* 4 */
    }
    ```
      
  - Double_variable_info 项指示两个位置中的第一个具有双精度验证类型。
    ```
    Double_variable_info {
    u1 tag = ITEM_Double; /* 3 */
    }
    ````

- Long_variable_info 和 Double_variable_info 项表示两个位置中第二个的验证类型，如下所示：
  - 如果两个位置中的第一个是局部变量，则：
    - 它不能是具有最高索引的局部变量。
    - 下一个更高编号的局部变量的验证类型为 top。
  - 如果两个位置中的第一个是操作数堆栈条目，则：
    - 它不能是操作数堆栈的最顶端位置。
    - 靠近操作数堆栈顶部的下一个位置的验证类型为 top。

堆栈映射帧由一个可区分的联合 stack_map_frame 表示，它由一个单字节标记组成，指示联合中的哪一项正在使用，后跟零个或多个字节，提供有关标记的更多信息。
```
union stack_map_frame {
    same_frame;
    same_locals_1_stack_item_frame;
    same_locals_1_stack_item_frame_extended;
    chop_frame;
    same_frame_extended;
    append_frame;
    full_frame;
}
```

标签表示栈映射帧的帧类型：

- 帧类型 same_frame 由 [0-63] 范围内的标签表示。 这种帧类型表明该帧具有与前一帧完全相同的局部变量并且操作数栈为空。 帧的 offset_delta 值是标记项 frame_type 的值。
  ```
  same_frame {
    u1 frame_type = SAME; /* 0-63 */
  }
  ```
- 框架类型 same_locals_1_stack_item_frame 由 [64, 127] 范围内的标签表示。 此帧类型表示该帧具有与前一帧完全相同的局部变量，并且操作数堆栈具有一个条目。 帧的 offset_delta 值由公式 frame_type - 64 给出。一个堆栈条目的验证类型出现在帧类型之后。
  ```
  same_locals_1_stack_item_frame {
    u1 frame_type = SAME_LOCALS_1_STACK_ITEM; /* 64-127 */
    verification_type_info stack[1];
  }
  ```
- `[128-246]` 范围内的标签保留供将来使用。

- 帧类型 same_locals_1_stack_item_frame_extended 由标记 247 表示。此帧类型表示该帧具有与前一帧完全相同的局部变量，并且操作数堆栈具有一个条目。 帧的 offset_delta 值是明确给出的，这与帧类型 same_locals_1_stack_item_frame 不同。 一个堆栈条目的验证类型出现在 offset_delta 之后。
  ```
  same_locals_1_stack_item_frame_extended {
    u1 frame_type = SAME_LOCALS_1_STACK_ITEM_EXTENDED; /* 247 */
    u2 offset_delta;
    verification_type_info stack[1];
  }
  ```
- 帧类型 chop_frame 由 `[248-250]` 范围内的标签表示。 这种帧类型表明该帧与前一帧具有相同的局部变量，只是缺少最后 k 个局部变量，并且操作数堆栈为空。 k 的值由公式 251 - frame_type 给出。 帧的 offset_delta 值是明确给出的。
  ```
  chop_frame {
    u1 frame_type = CHOP; /* 248-250 */
    u2 offset_delta;
  }
  ```
  假设上一帧局部变量的校验类型由locals给出，一个数组结构与full_frame帧类型相同。 如果前一帧的`locals[M-1]`代表局部变量X，`locals[M]`代表局部变量Y，那么去掉一个局部变量的效果就是新帧的`locals[M-1]`代表局部变量X， `locals[M`] 未定义。

  如果 k 大于前一帧的 locals 中的局部变量数，即如果新帧中的局部变量数小于零，则为错误。
- 帧类型 same_frame_extended 由标记 251 表示。此帧类型指示帧具有与前一帧完全相同的局部变量并且操作数堆栈为空。 与帧类型 same_frame 不同，帧的 offset_delta 值是明确给出的。
  ```
  same_frame_extended {
    u1 frame_type = SAME_FRAME_EXTENDED; /* 251 */
    u2 offset_delta;
  }
  ```
- 帧类型 append_frame 由 `[252-254]` 范围内的标签表示。 此帧类型表示该帧具有与前一帧相同的局部变量，除了定义了 k 个额外的局部变量，并且操作数堆栈为空。 k 的值由公式 frame_type - 251 给出。帧的 offset_delta 值是明确给出的。
  ```
  append_frame {
    u1 frame_type = APPEND; /* 252-254 */
    u2 offset_delta;
    verification_type_info locals[frame_type - 251];
  }
  ```
  locals 中的第 0 个条目表示第一个附加局部变量的验证类型。 如果 locals[M] 表示局部变量 N，则：

  - `locals[M+1]` 表示局部变量 N+1，如果 `locals[M]` 是 Top_variable_info、Integer_variable_info、Float_variable_info、Null_variable_info、UninitializedThis_variable_info、Object_variable_info 或 Uninitialized_variable_info 之一； 和

  - 如果 `locals[M]` 是 Long_variable_info 或 Double_variable_info，则 `locals[M+1]` 表示局部变量 N+2。

  如果对于任何索引 i，locals[i] 表示其索引大于该方法的最大局部变量数的局部变量，则会出错。

- 帧类型 full_frame 由标记 255 表示。帧的 offset_delta 值是明确给出的。
  ```
  full_frame {
    u1 frame_type = FULL_FRAME; /* 255 */
    u2 offset_delta;
    u2 number_of_locals;
    verification_type_info locals[number_of_locals];
    u2 number_of_stack_items;
    verification_type_info stack[number_of_stack_items];
  }
  ```

  locals中的第0项表示局部变量0的校验类型。若locals[M]表示局部变量N，则：

    - locals[M+1] 表示局部变量 N+1，如果 locals[M] 是 Top_variable_info、Integer_variable_info、Float_variable_info、Null_variable_info、UninitializedThis_variable_info、Object_variable_info 或 Uninitialized_variable_info 之一； 和

    - 如果 locals[M] 是 Long_variable_info 或 Double_variable_info，则 locals[M+1] 表示局部变量 N+2。

  如果对于任何索引 i，locals[i] 表示其索引大于该方法的最大局部变量数的局部变量，则会出错。
  
  栈中的第0项表示操作数栈底的验证类型，栈中后面的项表示距离操作数栈顶较近的栈项的验证类型。 我们将操作数堆栈的底部称为堆栈条目 0，并将操作数堆栈的后续条目称为堆栈条目 1、2 等。如果 stack[M] 表示堆栈条目 N，则：

    - stack[M+1] 表示堆栈条目 N+1，如果 stack[M] 是 Top_variable_info、Integer_variable_info、Float_variable_info、Null_variable_info、UninitializedThis_variable_info、Object_variable_info 或 Uninitialized_variable_info 之一； 和

    - 如果 stack[M] 是 Long_variable_info 或 Double_variable_info，则 stack[M+1] 表示堆栈条目 N+2。

  如果对于任何索引 i，stack[i] 表示其索引大于该方法的最大操作数堆栈大小的堆栈条目，则会出错。


### 4.7.5. Exceptions
Exceptions 属性是 method_info 结构（§4.6）的属性表中的可变长度属性。 Exceptions 属性指示方法可能抛出哪些已检查的异常。

在 method_info 结构的属性表中最多可能有一个 Exceptions 属性。

Exceptions 属性具有以下格式：

```
Exceptions_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 number_of_exceptions;
    u2 exception_index_table[number_of_exceptions];
}
```
Exceptions_attribute结构体的项如下：

- attribute_name_index
- attribute_name_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是表示字符串“Exceptions”的 CONSTANT_Utf8_info 结构（§4.4.7）。

- attribute_length
  - attribute_length 项的值表示属性的长度，不包括最初的六个字节。

- number_of_exceptions
  - number_of_exceptions 项的值指示exception_index_table 中的条目数。

- exception_index_table[]
  - exception_index_table 数组中的每个值都必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是 CONSTANT_Class_info 结构（第 4.4.1 节），表示声明此方法要抛出的类类型。

仅当满足以下三个条件中的至少一个时，方法才应抛出异常：

  - 异常是 RuntimeException 或其子类之一的实例。
  - 异常是 Error 或其子类之一的实例。
  - 异常是刚才描述的 exception_index_table 中指定的异常类之一的实例，或者它们的子类之一。

这些要求在 Java 虚拟机中没有强制执行； 它们仅在编译时强制执行。


### 4.7.6. InnerClasses
InnerClasses 属性是 ClassFile 结构（§4.1）的属性表中的可变长度属性。

如果类或接口 C 的常量池包含至少一个 CONSTANT_Class_info 条目（第 4.4.1 节），它代表一个不是包成员的类或接口，那么在属性表中必须恰好有一个 InnerClasses 属性 C的ClassFile结构。

InnerClasses 属性具有以下格式：
```
InnerClasses_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 number_of_classes;
    {   u2 inner_class_info_index;
        u2 outer_class_info_index;
        u2 inner_name_index;
        u2 inner_class_access_flags;
    } classes[number_of_classes];
}
```

- attribute_name_index;
  - attribute_name_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是表示字符串“InnerClasses”的 CONSTANT_Utf8_info 结构（§4.4.7）。
- attribute_length;
  - attribute_length 项的值表示属性的长度，不包括最初的六个字节。
- number_of_classes;
  - number_of_classes 项的值指示类数组中的条目数。

- classes[]
  - 代表非包成员的类或接口 C 的 constant_pool 表中的每个 CONSTANT_Class_info 条目必须在 classes 数组中恰好有一个对应的条目。
  - 如果一个类或接口的成员是类或接口，它的 constant_pool 表（以及它的 InnerClasses 属性）必须引用每个这样的成员（JLS §13.1），即使该成员没有被类以其他方式提及。
  - 此外，每个嵌套类和嵌套接口的 constant_pool 表都必须引用它的封闭类，因此，每个嵌套类和嵌套接口都会有每个封闭类以及它自己的每个嵌套类和接口的 InnerClasses 信息。
  - classes 数组中的每个条目都包含以下四项：
    - inner_class_info_index
      - inner_class_info_index 项的值必须是 constant_pool 表中的有效索引。 该索引处的 constant_pool 条目必须是表示 C 的 CONSTANT_Class_info 结构。

    - outer_class_info_index
      - 如果 C 不是类或接口的成员——也就是说，如果 C 是顶级类或接口 (JLS §7.6) 或本地类 (JLS §14.3) 或匿名类 (JLS §15.9.5) ) - 那么 outer_class_info_index 项的值必须为零。

      - 否则，outer_class_info_index 项的值必须是 constant_pool 表中的有效索引，并且该索引处的条目必须是表示 C 所属的类或接口的 CONSTANT_Class_info 结构。 outer_class_info_index 项的值不能等于 inner_class_info_index 项的值。

    - inner_name_index
      - 如果 C 是匿名的（JLS §15.9.5），则 inner_name_index 项的值必须为零。
      - 否则，inner_name_index 项的值必须是 constant_pool 表中的有效索引，并且该索引处的条目必须是代表 C 的原始简单名称的 CONSTANT_Utf8_info 结构，如此类文件的源代码中给出的那样 编译。

    - inner_class_access_flags
      - inner_class_access_flags 项目的值是标志的掩码，用于表示在编译此类文件的源代码中声明的类或接口 C 的访问权限和属性。 当源代码不可用时，编译器使用它来恢复原始信息。 标志在表 4.7.6-A 中指定。

      Table 4.7.6-A. Nested class access and property flags
      <table   border="1">
        <thead>
          <tr>
              <th>Flag Name</th>
              <th>Value</th>
              <th>Interpretation</th>
          </tr>
        </thead>
        <tbody>
          <tr>
              <td><code >ACC_PUBLIC</code></td>
              <td>0x0001</td>
              <td>Marked or implicitly <code >public</code> in source.
              </td>
          </tr>
          <tr>
              <td><code >ACC_PRIVATE</code></td>
              <td>0x0002</td>
              <td>Marked <code >private</code> in source.
              </td>
          </tr>
          <tr>
              <td><code >ACC_PROTECTED</code></td>
              <td>0x0004</td>
              <td>Marked <code >protected</code> in source.
              </td>
          </tr>
          <tr>
              <td><code >ACC_STATIC</code></td>
              <td>0x0008</td>
              <td>Marked or implicitly <code >static</code> in source.
              </td>
          </tr>
          <tr>
              <td><code >ACC_FINAL</code></td>
              <td>0x0010</td>
              <td>Marked or implicitly <code >final</code> in source.
              </td>
          </tr>
          <tr>
              <td><code >ACC_INTERFACE</code></td>
              <td>0x0200</td>
              <td>Was an <code >interface</code> in source.
              </td>
          </tr>
          <tr>
              <td><code >ACC_ABSTRACT</code></td>
              <td>0x0400</td>
              <td>Marked or implicitly <code >abstract</code> in source.
              </td>
          </tr>
          <tr>
              <td><code >ACC_SYNTHETIC</code></td>
              <td>0x1000</td>
              <td>Declared synthetic; not present in the source code.</td>
          </tr>
          <tr>
              <td><code >ACC_ANNOTATION</code></td>
              <td>0x2000</td>
              <td>Declared as an annotation interface.</td>
          </tr>
          <tr>
              <td><code >ACC_ENUM</code></td>
              <td>0x4000</td>
              <td>Declared as an <code >enum</code> class.
              </td>
          </tr>
        </tbody>
    </table>

  表 4.7.6-A 中未分配的 inner_class_access_flags 项的所有位保留供将来使用。 它们应该在生成的类文件中设置为零，并且应该被 Java 虚拟机实现忽略。

  如果类文件的版本号为 51.0 或更高，并且在其属性表中具有 InnerClasses 属性，则对于 InnerClasses 属性的类数组中的所有条目，outer_class_info_index 项的值必须为零，如果 inner_name_index 项目为零。

Oracle 的 Java 虚拟机实现不检查 InnerClasses 属性与代表该属性引用的类或接口的类文件的一致性。


