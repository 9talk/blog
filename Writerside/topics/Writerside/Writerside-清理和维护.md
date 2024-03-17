# 清理和维护

除非每次从文档源中删除某些内容时都要仔细清理，否则很可能会得到大量未使用的主题、图像和内容片段。Writerside提供了各种功能来帮助您保持源代码的清洁和执行定期维护。

<procedure title="删除未使用的图片" id="删除未使用的图片">
<p>
在Project Tool窗口中，选择要在其中检查未使用图像的目录，例如图像目录。您还可以选择特定的图像。
在主菜单, 选择 <b>Tools | Writerside | Check for Unused Images</b>
Writerside将报告没有被引用的图像，并建议删除它们。您可以打开所有检测到的未使用图像的列表，以选择要删除的图像。
</p>
</procedure>


<procedure title="删除未使用的topic文件" id="删除未使用的topic文件">
<p>
当您打开一个不在任何地方使用的主题文件时，Writerside会在编辑器顶部显示一个横幅，并建议删除该主题文件。
<img src="Remove unused topic files.png" alt="控制台提示" style="block"/>
</p>

<p>如果要查找项目中所有未使用的主题：</p>

<step>从主菜单中，选择<b>Code | Analyze Code | Run Inspection by Name</b></step>
<step>输入<code>Unused topic</code></step>
<step>在"运行"未使用的主题"对话框中，如果需要，选择作用域和文件掩码，然后单击确定。</step>
<img src="unused topic.png" alt="未使用的主题" style="block"/>
<p></p>
</procedure>

<procedure title="删除未使用的代码片段" id="删除未使用的代码片段">
 
<p>如果要查找项目中所有未使用的代码片段：</p>

<step>从主菜单中，选择<b>Code | Analyze Code | Run Inspection by Name</b></step>
<step>输入<code>Unused snippet</code></step>
<step>在"运行"未使用的主题"对话框中，如果需要，选择作用域和文件掩码，然后单击确定。</step>
<img src="unused snippet.png" alt="未使用的代码片段" style="block"/>
<p></p>
</procedure>