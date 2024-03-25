# 语法

Writerside支持Markdown和语义标记。使用Markdown，您可以直接定义内容的格式
除此之外还支持一些"语义标记", 类xml写法。

## Markdown 对比 Semantic Markup(语义化标记) {id="markdown-vs-semantic-markup"}

当你需要快速编写内容并保持源文件易于阅读时，Markdown是一个不错的选择，尤其适用于小型文档项目。
语义化标记则更适合大型项目，尤其是当你有许多作者在相关文档上协作、并且没有很多外部贡献者时。
使用Writerside，你可以在Markdown主题中结合使用这两种方法。

例如，你可以使用Markdown的通用标记来突出显示UI控件的名称：

```Text
**OK**
```

或者，你可以用UI控件的语义元素包裹它：

```xml
<control>OK</control>
```

渲染的输出结果将是一样的：**OK**。
但是，您也可以将通用粗体用于其他目的，例如突出显示关键字。
要更改使用Markdown呈现UI控件的方式，您必须手动检查所有出现的粗体并相应地进行更改。
使用语义化标记，只需更改<code>&lt;control&gt;</code>元素的呈现方式。

在Markdown中，你可以将一个系列的步骤以数字列表的形式记录下来，但在另一个语境中，数字列表可能代表着不同的东西, 比如: 优先事项的列表或插图中的编号项目.

然而，在语义化标记中有一个<code>&lt;procedure&gt;</code>元素，具有<code>&lt;step&gt;</code>子元素，专门用于记录一系列的步骤。

## Semantic Markup(语义化标记) {id="semantic-markup"}

语义化标记(Semantic Markup)使用XML语法进行描述, Writerside为其提供了帮助，例如：
* 智能补全，建议在当前上下文中有效的元素
* 用于验证标记、突出语法错误并提出改进建议的检查
* 用于生成和转换各种元素的操作
* 用于插入复杂语义元素的live template

您可以在其专用的 **.topic** 文件类型中使用语义化标记，也可以在必要时将其混合到 **.md** 文件中。

有块元素，如章节、过程、表格、图像和段落。还有一些内联元素应用于段落块的特定部分。例如，一个章节可以包含几个段落和一个过程，它由步骤组成，每个步骤可以有一个或多个段落和代码块。在每个段落中，您可以指定哪些单词是UI控件、文件名或路径，添加链接和快捷方式。还有一些特殊的语义元素用于内容重用，比如用于条件输出的元素: `<if>`

 
您可以键入 `<` 查看当前上下文中可用的整个标签列表。

![all-tags-list-peek.png](all-tags-list-peek.png)


## Markdown {id="markdown"}

Writerside实现Markdown的CommonMarkSpec，并使用flexmark作为解析器。有关更多信息，请参阅CommonMark快速参考。

除了标准的CommonMark语法外，Writerside还使用自定义属性来扩展带有语义属性的标记（例如，定义章节的ID），并支持将语义标记元素直接注入Markdown。

例如，如果要在Markdown主题中添加选项卡，可以执行以下操作:

```
## Java vs Kotlin

Here is a simple "Hello, World!" program in Java and Kotlin:

<tabs>
    <tab title="Java">
        <code-block lang="java">
            class MyClass {
                public static void main(String[] args) {
                    System.out.println("Hello, World!");
                }
            }
        </code-block>
    </tab>
    <tab title="Kotlin">
        <code-block lang="kotlin">
            fun main() {
                println("Hello, World!")
            }
        </code-block>
    </tab>
</tabs>

More content in Markdown.
```

Markdown中注入的XML标记必须是连续的块，因为Markdown会在段落换行时解析空行。
这也意味着您可以使用空行在XML元素中插入Markdown，但要确保没有Markdown解析为块引号的制表符缩进。

<code-block><![CDATA[## Java vs Kotlin

Here is a simple "Hello, World!" program in Java and Kotlin:

<tabs>
<tab title="Java">

```java
    class MyClass {
        public static void main(String[] args) {
            System.out.println("Hello, World!");
        }
    }
```

</tab>
<tab title="Kotlin">

```kotlin
    fun main() {
        println("Hello, World!")
    }
```

</tab>
</tabs>

More content in Markdown.]]>
</code-block>