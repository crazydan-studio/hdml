HDML 语法说明
========================

**注**：`HDML` 只是一种语言规范，并不做实际的文档展示约束，
但会针对普遍的情形提供一些建议和参考。

HDML 将一个文件视为一个**文档**（`Document`）。

文档由不同的**块**（`Block`）组成，在一个**块**内还可以嵌套其他**块**。

一个**文档**在本质上即为一个块——**超级块**（`Super Block`），
在该块内嵌入了文档**内容块**，因此，在数据结构上，文档实际上是一颗倒置的树。

注意，`HDML` 描述的仅仅是文档的数据结构，并不包含数据的处理逻辑，
因此，在文档中不应该出现逻辑代码，也不会有动态的数据。
**文档处理器**应该根据文档提供的数据做相应的处理，其处理过程不是由文档定义的。

逻辑的处理核心是数据，所以，定义好数据，便能够满足任意的需求。

## 文档

```
@title 三天速成？不存在的！
@author
@.name 张三
@.email zhangsan@example.com

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
```js
{
  attr: {
    title: { value: "三天速成？不存在的！" }
    , author: {
      value: None
      , name: { value: "张三" }
      , email: { value: "zhangsan@example.com" }
    }
  }
  , blocks: [
    {
      name: "Paragraph"
      , attr: {  }
      , blocks: [
        {
          name: "Text"
          , attr: {  }
          , content: "文档正文。。。"
        }
      ]
    }
    , {
      name: "Section"
      , attr: {
        title: { value: "第 1 节" }
      }
      , blocks: [
        {
          name: "Paragraph"
          , attr: {  }
          , blocks: [
            {
              name: "Text"
              , attr: {  }
              , content: "这是第 1 节。。。"
            }
          ]
        }
      ]
    }
    , {
      name: "Section"
      , attr: {
        title: { value: "第 2 节" }
      }
      , blocks: [
        {
          name: "Paragraph"
          , attr: {  }
          , blocks: [
            {
              name: "Text"
              , attr: {  }
              , content: "这是第 2 节。。。"
            }
          ]
        }
        , {
          name: "Paragraph"
          , attr: {  }
          , blocks: [
            {
              name: "Text"
              , attr: {  }
              , content: "下面是一段 Java 代码："
            }
          ]
        }
        , {
          name: "Source"
          , attr: {
            lang: { value: "java" }
          }
          , blocks: [
            {
              name: "Paragraph"
              , attr: {  }
              , blocks: [
                {
                  name: "Text"
                  , attr: {  }
                  , content: "Long number = 10L;"
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

**注**：
- 这里仅引出结构，具体的解释将在[属性声明](#属性声明)和[块](#块)章节中说明；
- 从数据结构上来看，除了`Text`块以外，其他类型的块均能够嵌套其他块；

## 属性声明

文档和块的属性（`Attribute`）通过**属性标记符**（`@`）进行声明，其语法结构为：
```
@<attrName> <attrValue>
```

属性命名风格没有强制规范，既可以为**驼峰式**，也可以为**下划线式**。
除了**文档处理器**和**块处理器**所内置的属性，其他自定义属性均可自由命名。

对于多级属性，则其语法结构为：
```
@<topAttrName>
@.<subAttrName> <subAttrValue>
```

也即，在子级属性名称开头添加`.`并且紧挨者父级属性进行定义。

属性名称前面`.`的数量代表着属性的层级级数，在定义三级及以上属性时，
则会出现两个以上的`.`：
```
@title 三天速成？不存在的！
@author
@.name 张三
@.email
@..user zhangsan
@..host example.com
```

**注**：`email.user`和`email.host`只是用于举例，
实际应该写为`@.email zhangsan@example.com`。

父子级属性需紧邻在一起，之间不能有空白行，也不能有属于其他属性的子级属性。

文档属性放置在文件的开始位置，也即，放在所有块的最前面，
而块的属性则紧挨着**块起始符**，置于块起始符的后面的行。

属性值默认按照普通文本处理，不做语法解析，若需要按`HDML`语法解析，
则需要为属性指定子属性`.markupped?`（语法说明见[布尔类型属性](#布尔类型属性)），
也即声明其值为标记语言，需要按标记语法进行解析：
```
@title
**三天**速成？不存在的！

    -- 评[《Xxx》](https://three-days-done.example.com)
@$
@.markupped?
```

### 多行属性值

属性值可以包含多行，但需以`@$`标识值的结束：
```
@title
三天速成？不存在的！

    -- 评《Xxx》
@$
@author
@.name 张三
```

### 属性值引用

属性可以引用在其之间定义的属性或其上级属性的值，其引用方式为：
```
@doc_author 张三 <zhangsan@example.com>

@author @@doc_author
```

也即，以`@@<prevAttrName>`形式引用属性值，且可引用的范围是所处的块内可见的属性。

以这种方式可以在文档开始处定义全局常量，在块内便可以复用这些常量，从而减少重复定义。

在文本内，包括属性值为文本的情况，则以`{@@<attrName>}`方式进行值引用：
```
@author 张三 <zhangsan@example.com>

本文作者为 {@@author}。
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
// 定义文档属性
/** 这是文档作者 */
@author 张三
/** 这是文档标题 */
@title 世界需要多样的色彩

开始正文。。。
```

**注**：注释只能出现在属性上方的行，出现在其他位置的，均视为文本。

### 属性数据结构

以下声明的属性：
```
@title 三天速成？不存在的！
@author
@.name 张三
@.email zhangsan@example.com
```

其数据结构将被解析为：
```js
{
  title: { value: "三天速成？不存在的！" }
  , author: {
    value: None
    , name: {  value: "张三" }
    , email: { value: "zhangsan@example.com" }
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
```js
{
  author: {
    value: "张三 <zhangsan@example.com>"
    , name: { value: "张三" }
    , email: { value: "zhangsan@example.com" }
  }
}
```

也就是，在任意属性下，均可添加子级属性，并且可以为父级属性设置独立的值。

## 块

块由**块起止符**标识其内容的边界，其**起始符**为 `{{|`，而**结束符**为 `}}`。
一般要求在块起止符之间保留两个空行，块的内容从两个空行之间开始书写。

**注**：为了便于书写和阅读，某些块（比如，段落）会省略**块起止符**，
或使用更加简洁分隔符标识块的范围和名称。

块分为以下两种：
- 匿名块
```
{{|

这是匿名块。

}}
```
- 命名块
```
{{BlockName|

这是命名块。

}}
```

每种类型的块都对应特定的**块处理器**，因此，相同名称的块具有相同的处理方式，
并由**块属性**控制具体的处理流程。

**块属性**紧挨着**块起始符**，并与**块内容**间隔至少一个空行：
```
{{Toc|
@title
@.numbered?
@.format 第N章

@link ./a.hdoc
@.title 缘由

@link ./b.hdoc
@.title 发展

}}
```

块内容可以为文本，可以为属性集合，也可以为其他块。
实际可处理的内容由**块处理器**决定，
**文档解析器**只是将解析得到的数据结构交给**块处理**去处理。

以上示例的数据结构为：
```js
{
  name: "Toc"
  , attr: {
    title: {
      value: None
      , numbered: { value: true }
      , format: { value: "第N章" }
    }
  }
  , blocks: [
    {
      name: "Paragraph"
      , attr: {
        link: {
          value: "./a.hdoc"
          , title: { value: "缘由" }
        }
      }
      , blocks: [  ]
    }
    , {
      name: "Paragraph"
      , attr: {
        link: {
          value: "./b.hdoc"
          , title: { value: "发展" }
        }
      }
      , blocks: [  ]
    }
  ]
}
```

### 块标识

每个块均有一个唯一标识，在未指定时，由**文档处理器**根据确定的规则自动生成。
块的标识将用于为块生成链接锚点，以便于快速定位块的位置。

通过属性`@id`可以手动指定块的标识：
```
@id para-1
这是第一段。。。

===
@id section-1
@title 章节一

。。。
```

### 文本

文本是最小的块单元，不可再被细分。文本才是文档的「骨与肉」。

文本为**段落**的组成单元，其除了文字以外，
仅具备与布局相关的[样式属性](#内联块)：
```
这段文字有**加粗**`红`{@style.font.color red}字。
```

以上示例的数据结构为：
```js
[
  {
    name: "Text"
    , attr: {  }
    , content: "这段文字有"
  }
  , {
    name: "Text"
    , attr: {
      style: {
        value: None
        , font: {
          value: None
          , bold: { value: true }
        }
      }
    }
    , enclosure: "**"
    , content: "加粗"
  }
  , {
    name: "Text"
    , attr: {
      style: {
        value: None
        , font: {
          value: None
          , color: { value: "red" }
        }
      }
    }
    , content: "红"
  }
  , {
    name: "Text"
    , attr: {  }
    , content: "字。"
  }
]
```

**注**：`enclosure`表示文字块被什么符号包围（默认为反引号），
用于反向生成`HDML`。

### 段落

```
这段文本实际上也是一个块。
```

每一个**段落**（`Paragraph`）均为一个块，但其不需要被**块起止符**包围，
仅需在段落之间或段落与其他块之间间隔一个或多个空白行即可。以上示例实际为：
```
{{Paragraph|

这段文本实际上也是一个块。

}}
```

**段落块**的作用就是将该块范围内的一个或多个**内嵌块**（主要为**文本块**）
组织在一行显示，形成一个自然段落。

由于段落也是块，所以，可以为段落设置属性信息，如：
```
@indent 4
这段文本将缩进4个字符。
```

也即，将段落的属性声明放在段落的上方，且二者之间没有空行。

以上也可以书写为如下形式：
```
{{Paragraph|
@indent 4

这段文本将缩进4个字符。

}}
```

以上示例的数据结构为：
```js
{
  name: "Paragraph"
  , attr: {
    indent: { value: 4 }
  }
  , blocks: [
    {
      name: "Text"
      , attr: {  }
      , content: "这段文本将缩进4个字符。"
    }
  ]
}
```

**注**：段落可以换行书写，只要行间没有空白行（不包含块内的空白，块均视为一个整体）即可，
相邻的行均会被视为同一段落。

### 章节

**章节**（`Section`）以至少三个`=`连写作为分隔符，
在遇到下一个**章节分隔符**之前的内容均属于当前章节：
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

章节的属性声明紧挨着放在分隔符下面。

章节之间的层级关系通过**章节分隔符**的相对数量确定，
相对多出来的数量就是该章节所处的相对层级级数，
且相邻的层级级数无需连续，但渲染时不会自动弥补该断层。

以上示例的数据结构为：
```js
[
  {
    name: "Section"
    , attr: {
      title: { value: "第 1 节" }
    }
    , blocks: [
      {
        name: "Paragraph"
        , attr: {  }
        , blocks: [
          {
            name: "Text"
            , attr: {  }
            , content: "这是第 1 节。。。"
          }
        ]
      }
      , {
        name: "Section"
        , attr: {
          title: { value: "第 1.1 节" }
        }
        , blocks: [
          {
            name: "Paragraph"
            , attr: {  }
            , blocks: [
              {
                name: "Text"
                , attr: {  }
                , content: "这是第 1.1 节。。。"
              }
            ]
          }
          , {
            name: "Section"
            , attr: {
              title: { value: "第 1.1.1 节" }
            }
            , blocks: [
              {
                name: "Paragraph"
                , attr: {  }
                , blocks: [
                  {
                    name: "Text"
                    , attr: {  }
                    , content: "这是第 1.1.1 节。。。"
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
  , {
    name: "Section"
    , attr: {
      title: { value: "第 2 节" }
    }
    , blocks: [
      {
        name: "Paragraph"
        , attr: {  }
        , blocks: [
          {
            name: "Text"
            , attr: {  }
            , content: "这是第 2 节。。。"
          }
        ]
      }
    ]
  }
]
```

### 块嵌套

在块中可以任意其他类型或相同类型的块，形成嵌套的块：
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

以上示例的数据结构为：
```js
{
  name: "Translation"
  , attr: {
    layout: { value: "row" }
  }
  , blocks: [
    {
      name: "Section"
      , attr: {
        lang: { value: "en" }
        , title: { value: "English" }
        , author: { value: "张三" }
      }
      , blocks: [
        {
          name: "Paragraph"
          , attr: {  }
          , blocks: [
            {
              name: "Text"
              , attr: {  }
              , content: "This is a English paragraph."
            }
          ]
        }
      ]
    }
    , {
      name: "Section"
      , attr: {
        lang: { value: "zh" }
        , title: { value: "中文" }
        , author: { value: "李四" }
      }
      , blocks: [
        {
          name: "Paragraph"
          , attr: {  }
          , blocks: [
            {
              name: "Text"
              , attr: {  }
              , content: "这是一段英文。"
            }
          ]
        }
      ]
    }
  ]
}
```

### 代码块

代码块采用如下形式书写：
<pre>
```
@title This is a Java example
@lang java

int a = 0;
```
</pre>

其等价于：
```
{{Source|
@title This is a Java example
@lang java

int a = 0;

}}
```

即将代码块内容放在两行反引号内，且每行均有且只有三个反引号。

以上示例的数据结构为：
```js
{
  name: "Source"
  , attr: {
    lang: { value: "java" }
    , title: { value: "This is a Java example" }
  }
  , blocks: [
    {
      name: "Paragraph"
      , attr: {  }
      , blocks: [
        {
          name: "Text"
          , attr: {  }
          , content: "int a = 0;"
        }
      ]
    }
  ]
}
```

### 图片

```
@title
@.position top
@.align center
@align center
![Little Cat](https://example.com/image/cat.jpg)
```

等价于

```
{{Image|
@title
@.position top
@.align center
@align center
@url https://example.com/image/cat.jpg

Little Cat

}}
```

### 列表

```
{{List|
@collapsable?

@collapsed?
@numbered?
. 列表项1
.. 列表项1.1
... 列表项1.1.1
... 列表项1.1.2
.. 列表项1.2
... 列表项1.2.1

. 列表项2
.. 列表项2.1
.. 列表项2.2

}}
```

注，每一个列表项同样也为一个块。

### 表格

```
{{Table|

===
@head?
====
Head 1
====
Head 2
====
Head 3

===
@align center
====
@align left

Column 1
====
Column 2
====
Column 3

===
====
Column 1
====
Column 2
====
Column 3

}}
```

**注**：如何统一块内层级与章节层级的标记符号？

### 块引入

// TODO 链接跳转、块内容引入

## 内联块

注：在行内的块均为**内联块**（`Inline Block`）并且在同一行展示。

内联块属性定义的一般形式为：
```
{@<attr1> val1, @<attr2> val2, ...}
```

但可以引用其他属性：
```
{@<attr1> @@<prevAttr1>, @<attr2> val2, ...}
```

还可以做属性引用合并：
```
{@@<prevAttr1>, @<attr2> val2, ...}
```

以上为将`@@<prevAttr1>`的子级属性与后序的属性列表合并成为一个内联块属性。

- 加粗
```
**粗体**
```

等价于

```
`粗体`{@style.font.bold?}
```

- 指定样式属性
```
**粗体**{@style.font.color red, @style.font.size 24px}
```

等价于

```
`粗体`{@style.font.bold?, @style.font.color red, @style.font.size 24px}
```

- 无样式
<pre>
`只是为了限定该小段文本的边界`
</pre>

- 引用样式
```
@bigRedFont
@.font
@..color red
@..size 24px

**大红字体**{@style @@bigRedFont}
```

以上也可以按如下方式定义样式：

```
@bigRedFontStyle
@.style
@..font
@...color red
@...size 24px

**大红字体**{@@bigRedFontStyle, @style.font.background.color yellow}
```

即，`@style.font.background.color`将与`@bigRedFontStyle.style`做合并。

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

## 参考

- [AsciiDoc](https://docs.asciidoctor.org/asciidoc/latest/)
- [Markdown](https://www.markdownguide.org/getting-started/)
