# Topic(主题)

在Writerside中，内容组织在主题文件中。每个主题对应于一个网页。它通常是一篇涉及特定主题的文章。

主题文件存储在<a href="Writerside-projects.md#topics_"><b>topics/</b>目录</a>中。

同一主题可以在多个情况下使用。例如，如果创建适用于多个文档输出的内容，则只需创作一次即可在不同的树文件中重复使用。

## 主题格式

Writerside支持两种主题类型：Markdown和XML。
Markdown优势是更易于源码阅读
XML使用. topic文件在语义XML标记中创作文档。牺牲源码可阅读性, 提供更加强大格式支持

<tabs>
<tab title="Markdown 主题示例">
<code-block >
<![CDATA[# Getting started with My Awesome App

A paragraph]]>
</code-block>
</tab>
<tab title="XML 主题示例">
<code-block  lang="xml">
<![CDATA[<topic xsi:noNamespaceSchemaLocation="https://resources.jetbrains.com/writerside/1.0/topic.v2.xsd"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    title="Getting started with My Awesome App">
    <p>A paragraph.</p>
</topic>]]>

</code-block>
</tab>
</tabs>

## 主题文件名和ID
使用文件的名称来引用该主题。

在`.tree`文件中:

```xml
<toc-element topic="getting-started.topic"/>
```

作为链接：

```xml
<a href="getting-started.topic"/>
```
```
[](getting-started.md)
```

包括 Include: 
```xml
<include from="getting-started.topic" element-id="intro"/>
```

ID是没有扩展名的主题文件的名称，例如getting—started。在帮助模块中不能有多个具有相同ID的主题

<note>
对于XML主题，您必须在根<code>&lt;topic&gt;</code>标记中将ID显式指定为id="Growing-Started"。
</note>