# Projects(项目结构描述)

文档项目包含文档所需的所有配置和内容文件。

每个文档项目至少包含一个帮助模块。这是文档项目的所有配置和内容文件所在的位置。

它包含以下文件和目录：

<procedure title="writerside.cfg" id="writerside_cfg">
<p>主配置文件定义了基本的Writerside项目设置和实例。这个文件是Writerside知道帮助模块的根目录在哪里的方式。</p>
</procedure>

<procedure title="topics/" id="topics_">
<p>
主题文件的目录包含<b>.md</b>和<b>.topic</b>文件。
可以在<b><a href="#writerside_cfg">writerside.cfg</a></b>中为主题注册一个不同的目录。
</p>
</procedure>


<procedure title="images/" id="images_">
<p>
媒体目录包含项目中使用的图像、动画GIF和视频。您可以在<b><a href="#writerside_cfg">writerside.cfg</a></b>中为图像注册一个不同的目录。
</p>
</procedure>

<procedure title="hi.tree" id="hi_tree">
<p>
树文件定义了目录：实例中主题的顺序和层次结构。
在树文件中，您还指定此实例的名称和唯一ID。
名称用作输出中的主标题，ID必须与树文件的名称相同，
但不带扩展名。例如，aa. tree可能会定义一个<code>id ="aa"</code>和<code>name ="Awesome App"</code>的实例。
每个实例都必须通过在<code>&lt;instance&gt;</code>元素中指定其树文件来注册到<b><a href="#writerside_cfg"></a></b>中。
</p>
</procedure>
<procedure title="v.list" id="v_list">
<p>全局项目变量及其默认值的列表。 将该文件注册到Writerside.cfg的<code>&lt;vars&gt;</code>元素中。</p>
</procedure>
<procedure title="c.list" id="c_list">
<p>类别列表定义主题的“另请参阅”部分的相关链接组。在writerside.cfg的<code>&lt;categories&gt;</code>元素中注册这个文件。</p>
</procedure>
以下文件和目录是可选的：

<procedure title="redirection-rules.xml" id="redirection-rules.xml">
<p>重定向规则的配置文件。</p>
</procedure>

<procedure title="code-snippets/" id="code-snippets_">
<p>代码段的目录包含了一些代码段的文件，您可以将这些代码段作为示例插入到代码块中。在writerside.cfg的<code>&lt;snippets&gt;元素中注册这个目录。</code></p>
</procedure>
<procedure title="cfg/" id="cfg_">
<step>buildprofiles.xml 用于配置最终输出。</step>
<step>build-script.xml 用于配置构建过程。</step>
<step>glossary.xml 用于定义可在工具提示中使用的术语及其描述。</step>
</procedure>


  

        

        