HDML 语法说明
========================

**注**：`HDML` 只是一种语言规范，并不做实际的文档展示约束，
但会针对普遍的情形提供一些建议和参考。

HDML 将一个文件视为一个**文档**（`Document`）。

文档由不同的**块**（`Block`）组成，在一个**块**内还可以嵌套其他**块**。

一个**文档**在本质上即为一个块——**超级块**（`Super Block`），
在该块内嵌入了文档**内容块**，在数据结构上构成一颗倒置的树。

## 文档

```
@doc
@.title 三天速成？不存在的！
@.author
@..name 张三
@..email zhangsan@example.com

文档正文。。。

===
@title 第 1 节

这是第 1 节。。。

===
@title 第 2 节

这是第 2 节。。。

下面是一段 Java 代码：

{{Source|
@lang java

Long number = 10L;

}}
```

文档（`Document`）由零个或多个**属性**和**块**组成，
属性用于指示文档和块的基础信息和布局控制等，块则是文档的内容，
除了属性声明和空行，剩下的皆为文档的块。

### 文档数据结构

以前面的示例为例，其将被解析为如下结构的数据：
```json
{
  attr: {
    doc: {
      value: None
      , title: {
        value: "三天速成？不存在的！"
      }
      , author: {
        value: None
        , name: {
          value: "张三"
        }
        , email: {
          value: "zhangsan@example.com"
        }
      }
    }
  }
  , blocks: [{
    attr: {
      name: { value: "Paragraph" }
    }
    , text: [{
      attr: {  } // ==> { name: "Text" }
      , value: "文档正文。。。"
    }]
  }, {
    attr: {
      name: { value: "Section" }
      , title: {
        value: "第 1 节"
      }
    }
    , blocks: [{
      attr: {
        name: { value: "Paragraph" }
      }
      , text: [{
        attr: {  } // ==> { name: "Text" }
        , value: "这是第 1 节。。。"
      }]
    }]
  }, {
    attr: {
      name: { value: "Section" }
      , title: {
        value: "第 2 节"
      }
    }
    , blocks: [{
      attr: {
        name: { value: "Paragraph" }
      }
      , text: [{
        attr: {  } // ==> { name: "Text" }
        , value: "这是第 2 节。。。"
      }]
    }, {
      attr: {
        name: { value: "Paragraph" }
      }
      , text: [{
        attr: {  } // ==> { name: "Text" }
        , value: "下面是一段 Java 代码："
      }]
    }, {
      attr: {
        name: { value: "Source" }
        , lang: { value: "java" }
      }
      , blocks: [{
        attr: {
          name: { value: "Paragraph" }
        }
        , text: [{
          attr: {  } // ==> { name: "Text" }
          , value: "Long number = 10L;"
        }]
      }]
    }]
  }]
}
```

**注**：这里仅引出结构，具体的解释将在[属性声明](#属性声明)和[块](#块)章节中说明。

## 属性声明

文档和块的属性（`Attribute`）通过**属性标记符**（`@`）进行声明，其语法结构为：
```
@<attrName> <attrValue>
```

属性命名风格没有强制规范，既可以为**驼峰式**，也可以为**下划线式**。
除了*文档处理器*所内置的属性，其他自定义属性均可自由命名。

对于多级属性，则其语法结构为：
```
@<topAttrName>
@.<subAttrName> <subAttrValue>
```

也即，在子级属性名称开头添加`.`并且紧挨者父级属性进行定义。

属性名称前面`.`的数量代表着属性的层级级数，在定义三级及以上属性时，
则会出现两个以上的`.`：
```
@doc
@.title 三天速成？不存在的！
@.author
@..name 张三
@..email zhangsan@example.com
```

父子级属性需紧邻在一起，之间不能有空白行，也不能有属于其他属性的子级属性。

文档属性放置在文件的开始位置，也即，放在所有块的最前面，
而块的属性则紧挨着**块起始符**，置于块起始符的后面的行。

### 多行属性值

属性值可以包含多行，但需以`@$`标识值的结束：
```
@doc
@.title
三天速成？不存在的！

    -- 评《Xxx》
@$
@.author
@..name 张三
```

**注**：属性值无论是单行还是多行，均按照块的方式解析，所以，在其中可以使用块。

### 属性值引用

在属性内可以引用其前序属性或上级属性的值，其引用方式为：
```
@author 张三 <zhangsan@example.com>
@doc.author @@author
```

也即，以`@@<prevAttrName>`形式引用属性值，且可引用的范围是所处的块内可见的属性。

以这种方式可以在文档开始处定义全局常量，在块内便可以复用这些常量，从而减少重复定义。

在文本内（包括属性值）则以`{{@@<attrName>}}`方式引用：
```
@author 张三 <zhangsan@example.com>

本文作者为 {{@@author}}。
```

**注**：同等级之间的块的属性不能相互引用。

### 布尔类型属性

布尔（`Boolean`）类型的属性定义语法为：
```
@collapsed? true
@collapsed? false

@collapsed?
```

即在属性名称后加上`?`，且其值只能为`true`或`false`，若值为`true`，
则可以省略值，仅保留属性即可，如，`@collapsed?`等价于`@collapsed? true`。

### 属性注释

在**属性标记符**上方可以添加单行或多行注释，单行注释以`// `开头，
多行注释则包含在`/** ... */`内：
```
// Note: doc 是文档属性
@doc
/** 这是文档标题 */
@.title 世界需要多样的色彩
```

**注**：注释只能出现在属性上方的行，出现在其他位置的，均视为文本。

### 属性数据结构

以下声明的属性：
```
@doc
@.title 三天速成？不存在的！
@.author
@..name 张三
@..email zhangsan@example.com
```

其数据结构将被解析为：
```json
{
  doc: {
    value: None
    , title: {
      value: "三天速成？不存在的！"
    }
    , author: {
      value: None
      , name: {
        value: "张三"
      }
      , email: {
        value: "zhangsan@example.com"
      }
    }
  }
}
```

也即，每一级属性都是一个至少包含`value`属性的对象，无论该属性是否有值，
其都默认存在。因此，可以声明如下形式的属性：
```
@author 张三 <zhangsan@example.com>
@.name 张三
@.email zhangsan@example.com
```

其将被解析为：
```json
{
  author: {
    value: "张三 <zhangsan@example.com>"
    , name: {
      value: "张三"
    }
    , email: {
      value: "zhangsan@example.com"
    }
  }
}
```

其等价于：
```
@author {{@@author.name}} <{{@@author.email}}>
```

## 块

块由**块起止符**标识其内容的边界，其**起始符**为 `{{|`，而**结束符**为 `}}`。
一般要求在块起止符之间保留两个空行，块的内容从两个空行之间开始书写。

块分为以下两种：
- 匿名块
```
{{|

这是匿名块。

}}
```
- 命名块
```
{{SomeName|

这是命名块。

}}
```

命名块是有特定的*渲染处理模块*的，通过名称可以统一限定对块内容的渲染方式，
同时也能更加明确块在文档中的职能，比如，索引目录块、引用文献块、代码块等。

*块的属性信息*紧挨着**块起始符**，并与块内容间隔一个空行：
```
{{Toc|
@numbered?
@format 第N章

@link ./a.hdoc
@.title Xxxx
@link ./b.hdoc
@.title Xxxx

}}
```

块的属性名称必须以`.`开头，也即，指示该属性为块的子级属性。

### 段落

```
这段文本实际上也是一个块。
```

每一个文本**段落**（`Paragraph`）均为一个块，但其不需要包裹在**块起止符**内，
仅需段落之间，段落与其他块之间间隔一个或多个空白行即可。以上实例等价于：
```
{{Paragraph|

这段文本实际上也是一个块。

}}
```

由于段落也是块，所以，可以为段落设置属性信息：
```
@indent 4
这段文本将缩进4个字符。
```

**注**：段落的属性声明需放在段落的上方，且二者之间不能有空行。

### 章节

**章节**（`Section`）以至少三个`=`作为分隔符，
在遇到下一个**章节分隔符**之前的内容均属于当前章节。

```
===
@title 第 1 节

这是第 1 节。。。

====
@title 第 1.1 节

这是第 1.1 节。。。

=====
@title 第 1.1.1 节

这是第 1.1.1 节。。。

===
@title 第 2 节

这是第 2 节。。。
```

章节之间的层级关系通过**章节分隔符**的相对数量确定，
相对多出来的数量就是该章节所处的相对层级级数。

### 块嵌套

```
{{Translation|
@layout row

===
@lang en
@title English
@author 张三

This is a English paragraph.

===
@lang zh
@title 中文
@author 李四

这是一段英文。

}}
```

### 代码块

<pre>
```
@lang java

int a = 0;
```
</pre>

等价于

```
{{Source|
@lang java

int a = 0;

}}
```

## 文本样式

- 加粗
```
**粗体**
```

- 指定样式属性
```
**粗体**{@style.color red, @style.font.size 24px}
```

- 无样式
<pre>
`只是为了限定该小段文本的边界`
</pre>

- 引用样式
```
@bigRedFont
@.color red
@.font.size 24px

**大红字体**{@style @@bigRedFont}
```

## 常用属性

### @link

链接其他文档：
```
@link a/b/c.hdoc
@.title 第一章. 什么是什么
```

### @import

引入其他文档内容：
```
@import a/b/c.hdoc
```
