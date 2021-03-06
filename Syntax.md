HDML 语法说明
========================

**注**：`HDML` 只是一种语言规范，并不做实际的文档展示约束，
但会针对普遍的情形提供一些建议和参考。

**文档**（`Document`）由不同的**块**（`Block`）组成，
并且在一个**块**内还可以嵌套其他**块**。

所以，一个**文档**在本质上也就是一个块——**超级块**（`Super Block`），
在该块内嵌入了**段落块**、**表格块**、**列表块**、**图片块**等类型的**块**，
因此，在数据结构上，文档实际上是一颗倒置的树。

注意，`HDML` 描述的仅仅是文档的数据结构，并不包含数据的处理逻辑，
因此，在文档中不应该出现逻辑代码，也不会有动态的数据。
**文档处理器**应该根据文档提供的数据做相应的处理，其处理过程不是由文档定义的。

逻辑的处理核心是数据，所以，定义好数据，便能够满足任意的需求。

在解析数据结构时，`HDML`默认将一个文件视为一个文档，但在实际应用中，
也可将一个文件按照其他类型的块进行解析（调用解析器时指定），
以识别出块所独有的内部块标记符号等。

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
  name: "Document"
  , attr: {
    title: { value: "三天速成？不存在的！", child: {} }
    , author: {
      value: None
      , child: {
        name: { value: "张三", child: {} }
        , email: { value: "zhangsan@example.com", child: {} }
      }
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
        title: { value: "第 1 节", child: {} }
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
        title: { value: "第 2 节", child: {} }
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
            lang: { value: "java", child: {} }
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
- 解析时，以单独的数据结构记录源文本与块的对应关系，且不需要完整记录输入内容，可根据块的结构去掉空格等无用字符；

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
*三天*速成？不存在的！

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

在文本内，包括属性值为文本的情况，则以`{{@@<attrName>}}`方式进行值引用：
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
  title: { value: "三天速成？不存在的！", child: {} }
  , author: {
    value: None
    , child: {
      name: {  value: "张三", child: {} }
      , email: { value: "zhangsan@example.com", child: {} }
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
```js
{
  author: {
    value: "张三 <zhangsan@example.com>"
    , child: {
      name: { value: "张三", child: {} }
      , email: { value: "zhangsan@example.com", child: {} }
    }
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

块内容可以为文本，也可以为其他块。实际可处理的内容由**块处理器**决定，
**文档解析器**只是将解析得到的数据结构交给**块处理**去处理。

以上示例中的两个`@link`的块视为内容为空的`段落块`，
且`@link`为该段落块的属性。

以上示例的数据结构为：
```js
{
  name: "Toc"
  , attr: {
    title: {
      value: None
      , child: {
        numbered: { value: true, child: {} }
        , format: { value: "第N章", child: {} }
      }
    }
  }
  , blocks: [
    {
      name: "Paragraph"
      , attr: {
        link: {
          value: "./a.hdoc"
          , child: {
            title: { value: "缘由", child: {} }
          }
        }
      }
      , blocks: [  ]
    }
    , {
      name: "Paragraph"
      , attr: {
        link: {
          value: "./b.hdoc"
          , child: {
            title: { value: "发展", child: {} }
          }
        }
      }
      , blocks: [  ]
    }
  ]
}
```

**注**：匿名块的`name`为`Block`。

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

文本是最小的块单元，不可再被细分。文本才是文档的「血肉」。

文本是**段落**的组成单元，其除了文字以外，
其仅具备与布局相关的[样式属性](#内联块)：
```
这段文字有*加粗*`红`{@font.color red}字。
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
        , child: {
          font: {
            value: None
            , child: {
              bold: { value: true, child: {} }
            }
          }
        }
      }
      , enclosure: { value: "*", child: {} }
    }
    , content: "加粗"
  }
  , {
    name: "Text"
    , attr: {
      style: {
        value: None
        , child: {
          font: {
            value: None
            , child: {
              color: { value: "red", child: {} }
            }
          }
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

**注**：属性`@enclosure`表示文字块被什么符号包围（默认为反引号），
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
    indent: { value: 4, child: {} }
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

可通过属性`@hardBreaks?`设置为多行模式：
```
@hardBreaks?
这是第一行，
这里换行了。
```

通过`@align`设置对其方式：
```
@align center
该段文字居中显示。
```

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

**注**：**章节**是具有层级关系的块。

以上示例的数据结构为：
```js
[
  {
    name: "Section"
    , attr: {
      level: { value: 1, child: {} }
      , marker: { value: "===", child: {} }
      , title: { value: "第 1 节", child: {} }
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
          level: { value: 2, child: {} }
          , marker: { value: "====", child: {} }
          , title: { value: "第 1.1 节", child: {} }
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
              level: { value: 3, child: {} }
              , marker: { value: "=====", child: {} }
              , title: { value: "第 1.1.1 节", child: {} }
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
      level: { value: 1, child: {} }
      , marker: { value: "===", child: {} }
      , title: { value: "第 2 节", child: {} }
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

在块中可以与其他类型或相同类型的块组合成为嵌套的块：
```
{{Translation|
@layout row

{{|
@lang en
@title English
@author 张三

This is a English paragraph.

}}

{{|
@lang zh
@title 中文
@author 李四

这是一段英文。

}}

}}
```

以上示例的数据结构为：
```js
{
  name: "Translation"
  , attr: {
    layout: { value: "row", child: {} }
  }
  , blocks: [
    {
      name: "Block"
      , attr: {
        lang: { value: "en", child: {} }
        , title: { value: "English", child: {} }
        , author: { value: "张三", child: {} }
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
      name: "Block"
      , attr: {
        lang: { value: "zh", child: {} }
        , title: { value: "中文", child: {} }
        , author: { value: "李四", child: {} }
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

### 块分隔

当某个**块内容**需要拆分成多个无层级关系的块时，
也可以使用**块分隔符**对内容进行分隔（在块内使用匿名块也具有相同的效果）。

**块分隔符**需使用三个以上的短横线`-`做为标记符，并独立成行。

如此，前例中的`Translation`块便可以书写为如下形式：
```
{{Translation|
@layout row

----------------
@lang en
@title English
@author 张三

This is a English paragraph.

----------------
@lang zh
@title 中文
@author 李四

这是一段英文。

}}
```

**注**：为了保持视觉上的一致性，分隔出来的每一个块都需要放置**块分隔符**，
且**分隔标记符**的长度应当保持一致。

**块分隔符**没有结束标记符，
分隔出的最后一个块的结束位置只能通过其所处的父块的边界决定，
所以，必要时需在匿名块内对**块内容**做分隔。

使用了块分隔符的块，其数据结构略有不同：
```js
{
  name: "Translation"
  , attr: {
    layout: { value: "row", child: {} }
  }
  , blocks: [
    {
      name: "Block"
      , attr: {
        marker: { value: "----------------", child: {} }
        , lang: { value: "en", child: {} }
        , title: { value: "English", child: {} }
        , author: { value: "张三", child: {} }
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
      name: "Block"
      , attr: {
        marker: { value: "----------------", child: {} }
        , lang: { value: "zh", child: {} }
        , title: { value: "中文", child: {} }
        , author: { value: "李四", child: {} }
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

也即，需要增加`@marker`属性，用于记录所使用的**分隔标记符**，
主要用于双向解析`HDML`。

### 块引用

// TODO 对块内容的引用、对块位置的引用

### 代码块

代码块采用如下形式书写：
```
{{Source|
@title This is a Java example
// @lang 值为 diff 时，按照文本差异方式展示内容
@lang java

int a = 0;

}}
```

以上示例的数据结构为：
```js
{
  name: "Source"
  , attr: {
    lang: { value: "java", child: {} }
    , title: { value: "This is a Java example", child: {} }
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

// TODO 图片块的本质为对目标链接内容的嵌入展示

### 列表

```
{{List|
@collapsable?
// 前缀符: https://docs.asciidoctor.org/asciidoc/latest/lists/ordered/
// - 无序列表: square|circle|disc|none
// - 有序列表: arabic|decimal|loweralpha|upperalpha|lowerroman|upperroman|lowergreek
@marker square

@numbered?
. 列表项1

@collapsed?
.. 列表项1.1

... 列表项1.1.1

... 列表项1.1.2

.. 列表项1.2

... 列表项1.2.1

. 列表项2

.. 列表项2.1

![Little Cat](https://example.com/image/cat.jpg)

.. 列表项2.2

}}
```

每一个列表项同样也为一个块，其前缀符`.`的数量代表着列表项的层级。

在列表项下的块均为列表内容，在展示上会与列表项的缩进对齐。

列表项的属性作用于自身及其子列表。

// TODO 代办列表: https://docs.asciidoctor.org/asciidoc/latest/lists/checklist/

//      - 使用 [ ] 和 [x]

// TODO 描述列表: https://docs.asciidoctor.org/asciidoc/latest/lists/description/

//      - 使用 @title 设置描述信息

### 表格

```
{{Table|

{{Head|

--------------
Head 1

--------------
Head 2

--------------
Head 3

}}


{{Row|
@align center

--------------
@align left

Column 1

--------------
Column 2

--------------
Column 3

}}


{{|

--------------
@span 2

Column 1

--------------
Column 3

}}

}}
```

在`Table`块内，使用`Head`和`Row`块表示表格的**表格头**和**表格行**，
且表格行可以直接使用匿名块，任何第一层的匿名块均默认视为行，
除非该块标注了其*具有非表格行职能*的属性。

在行内（包括表头）均使用[块分隔符](#块分隔)分隔列，且分隔标记符长度保持一致。
在分隔符下面可以声明列的相关属性，且列块中仍可嵌套任意层次的其他*非行/列*的块，
也可嵌套新的表格。

### 引述

{{Quote|



}}

### 备注

{{Comment|

// TODO 列表？

}}

联动效果，鼠标移入时，高亮备注到达路径以及备注内容。

### 参考

{{Reference|

// TODO 列表？https://docs.asciidoctor.org/asciidoc/latest/sections/bibliography/

}}

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
*粗体*
```

等价于

```
`粗体`{@font.bold?}
```

等价于

```
@fontBold
@.font
@..bold?

`粗体`{@@fontBold}
```

- 斜体
```
_斜体_
```

等价于

```
`斜体`{@font.italic?}
```

等价于

```
@fontItalic
@.font
@..italic?

`斜体`{@@fontItalic}
```

- 下标
```
H~2~O
```

等价于

```
H`2`{@downgrade?}O
```

- 上标
```
E=mc^2^
```

等价于

```
E=mc`2`{@upgrade?}
```

- 指定样式属性
```
*粗体*{@font.color red, @font.size 24px}
```

等价于

```
`粗体`{@font.bold?, @font.color red, @font.size 24px}
```

- 无样式
<pre>
`只是为了限定该小段文本的边界`
</pre>

- 引用样式
```
@bigRedFont
@.color red
@.size 24px

*大红粗字体*{@font @@bigRedFont}
```

以上也可以按如下方式定义样式：

```
@bigRedFont
@.font
@..color red
@..size 24px

*大红粗字体*{@@bigRedFont, @font.background.color yellow}
```

即，`@font.background.color`将与`@bigRedFont.font`做合并。

## 常用属性

### @link

链接其他文档：
```
@link a/b/c.hdoc
@.title 第一章. 什么是什么
```

### @import

引入其他文档的数据结构：
```
@import a/b/c.hdoc
```

## 编辑器设计

- 以块为单元进行展示和编辑，鼠标在块内时，浮动高亮突出块（嵌套块，多层浮动），并显示相关的可操作按钮（可移动块）
- 块属性置灰显示
- 块类型在鼠标移入块时，浮动显示，并可直接修改块类型

## 参考

- [AsciiDoc](https://docs.asciidoctor.org/asciidoc/latest/)
- [Markdown](https://www.markdownguide.org/getting-started/)
