# 选项卡

要创建可切换的内容部分，例如，使用不同语言编写代码示例或为不同平台编写相同的内容，请使用`<tabs>`标记。

## 示例

<tabs group="version" id="version">
    <tab title="在Windows上安装" group-key="windows">
        在Windows上如何安装...
    </tab>
    <tab title="在Macos上安装" group-key="macos">
        在Macos上如何安装...
    </tab>
</tabs>

<tabs group="version">
    <tab title="在Windows上进行包管理" group-key="windows">
        在Windows上进行包管理...
    </tab>
    <tab title="在Macos上进行包管理" group-key="macos">
        在Macos上如何进行包管理...
    </tab>
</tabs>

## `<tabs>`的属性

它具有以下属性：

<procedure title="group" id="group">
<p>指定几个选项卡组以同时切换它们。</p>
</procedure>

<procedure title="id" id="id">
<p>指定要用作链接锚点的唯一标识符，并为此选项卡块生成有意义的URL。您可以引用和重用任何具有ID的元素。</p>
</procedure>

<procedure title="filter" id="filter">
<p>定义一组自定义筛选器，以便有条件地包含此选项卡块。</p>
</procedure>

<procedure title="instance" id="instance">
<p>定义一组实例，用于有条件地过滤此选项卡块。</p>
</procedure>

<procedure title="switcher-key" id="switcher-key">
<p>根据所选键使此选项卡可切换, 详细请阅读:<a href="https://www.jetbrains.com/help/writerside/switchable-content.html">selected key</a></p>
</procedure>

## `<tab>`的属性

当`<tab>`标签在`<tabs>`内的时候, 具有以下属性:

<procedure title="title" id="title">
<p>必备, 指定标签标题</p>
</procedure>


<procedure title="group-key" id="group-key">
<p>将此键分配给不同组中必须同步的选项卡</p>
</procedure>

<procedure title="id" id="tab-id">
<p>指定要用作链接锚点的唯一标识符，并为此选项卡生成有意义的URL。您可以引用和重用任何具有ID的元素。。</p>
</procedure>

<procedure title="filter" id="tab-filter">
<p>定义一组自定义筛选器，以便有条件地包含此选项卡块。</p>
</procedure>

<procedure title="instance" id="tab-instance">
<p>定义一组实例，用于有条件地过滤此选项卡块。</p>
</procedure>

<procedure title="switcher-key" id="tab-switcher-key">
<p>根据所选键使此选项卡可切换, 详细请阅读:<a href="https://www.jetbrains.com/help/writerside/switchable-content.html">selected key</a></p>
</procedure>




