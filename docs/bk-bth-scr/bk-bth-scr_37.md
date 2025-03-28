

# 第三十四章：后记



![](img/chapter.jpg)

我希望你读这本书的乐趣，能够与我写这本书的乐趣半分相当。对我来说，这个项目是一次全新、激动人心且富有启发性的尝试。这是我的第一本书，也可能是唯一的一本。如果你在我高中毕业时告诉我，我有一天会成为一名作者，我绝对不相信你。

我是一个数学迷，而且在我高年级时的一门 BASIC 编程课程中也表现优秀。我在英语课上成绩还不错，但当时我认为写作是一项苦差事。在大学时，作为我的专业课程，我上过两门技术写作课，但即便如此，写一本书的想法依然显得荒谬。经过多年阅读不同类型的书籍和文章后，我渐渐开始欣赏写作这一创造性输出。虽然偶尔尝试过写作，但从未发表过任何作品，甚至没有写过博客。大约在我 50 岁生日时，经过多年的 BAT 文件编程，我脑海中开始有了写这本书的一个微弱的想法，虽然那时我并没有准备好去面对这场看似荒唐的中年危机（如果它真的是一种危机的话）。

两年多后的 2020 年 5 月，两个事件碰撞在一起，使我有了一些空闲时间，可以把手指放到键盘上。 （在计算机时代，我们应该有一个更好的数字化委婉说法来代替“提笔写作”）。首先，显而易见的是，新冠疫情将持续超过几个月。我有充足的假期时间，却没有旅行计划。其次，在经历了十五年的紧张育儿生活，包括教导多个青少年运动队之后，我的儿子到了一个年龄，开始不再要求我陪伴，反而要求我完全放手（像大多数青少年一样）。

我和妻子（虽然有时是强行带着儿子）那个夏天做了相当多的远足。我也跑了不少步，但我还是回到了这本书的构思，尤其是意识到正常生活在东北地区的气温降下之前是不会恢复的。我时不时地、没有特定顺序地写了一章又一章，不知道是要将它们发布在博客上，还是找其他媒介，但很快我知道，我想写一本书，一本真正的实体书。随着气温的下降，我每个周末都写作，这种状态一直持续到春天、夏天……再到下一个秋天，甚至更久。（我写代码很快，但写作很慢。）2021 年 6 月初，我妻子对我说：“那么，父亲节你想要什么，除了一个人待着，好让你写作？”

经过几年的努力，我将我认为已经完成的作品呈现给了 No Starch Press。令我惊讶的是，我的首选出版社签约了我。作为新手，我的手稿还很粗糙，但他们提出了一些很好的建议，帮助我在不失去个人风格的前提下改进和组织内容。（至于我的风格，我首先是为自己写作，然后才考虑可能的评论。谁会写“一个若有若无的想法的迹象”？这算是矫情还是空洞呢？我不知道，但我喜欢这样，只能希望你也喜欢。）

本书中的 90%以上内容代表了我个人和职业生涯中经常使用的编程和技术。至于那些我必须研究的小部分内容，我希望我能够将它们与我擅长的内容无缝地融合在一起，做到不留痕迹。

对于大多数编程语言来说，一本书涵盖该语言大部分功能的想法是荒谬的。例如，有介绍 Python 的入门书籍，还有一些专注于 Python 和 Web 开发的书籍，甚至还有一些专注于 Python 和数据库的书籍，仅举几例。但是批处理脚本是一个相对更易于管理的主题。我希望我已经包括了对初学者和专家都很重要的大部分语言特性，但没有任何一本书可以做到真正的全面性。

许多命令没有被纳入到本书中。例如，curl 命令用于从服务器传输数据，cipher 命令用于加密和解密文件，runas 命令用于在不同的用户配置文件下运行批处理文件。还有一些特定于系统管理的命令我没有涉及，而有些命令我之所以没有包含，是因为我认为它们不太实用。我之前没有提到 replace 命令（直到现在）是因为批处理有更好的工具来用一个文件替换另一个文件。

对于那些我没有涉及的批处理语言的部分，我希望你现在已经有了自己进行研究的工具。我曾多次提到，你可以在 *[`<wbr>ss64<wbr>.com<wbr>/nt<wbr>/`](https://ss64.com/nt/)* 上找到一个完整的命令列表，包括描述、语法、选项和示例。你可以浏览命令列表，寻找可能对某个特定问题有用的命令，或者如果你听说过某个用于配置互联网协议的命令并想了解更多，只需点击 **ipconfig** 进行探索。没有任何一本书能够包含所有内容，但我希望你能在这本书中找到必备的知识，甚至更多。

对于完成本书的编程人员，下次当你在处理一个需要一些不同寻常的内容——数组、布尔值、浮点运算、栈，甚至可能是对象——的简单批处理文件时，我希望你不要立刻回过头去编写一些编译代码。根据具体情况，几行批处理代码可能是最有效的解决方案。

对于非程序员来说，如果你已经读完了第三部分，可以认为自己是一个荣誉程序员。但更重要的是，下次如果你遇到必须在特定时间间隔内（可能是每日）完成的重复任务，考虑使用 Batch。每天结束时运行一个简单的 bat 文件，将你当天的工作复制到备份服务器或共享目录。如果需要将几个报告合并为一个文件，不要进行剪切粘贴。给你的老板和同事留下深刻印象，或许你自己也会觉得惊讶。

按照这个思路，我有一个最后的 bat 文件想要分享。当我写这本书的时候，显然我会将文件备份到云端，但我也希望有一个带有日期戳的文件夹，里面存放着所有 Word 文档、bat 文件和与这本书相关的其他文件，并且我希望在每个写作日的结束时进行备份。每次登出前，我都会将 U 盘插入我的笔记本电脑，并从桌面运行一个仅包含以下代码的 bat 文件：

```
set @=%date%
xcopy C:\Batch\*.* D:\Batch\%@:~10,4%%@:~4,2%%@:~7,2%\ /F /S /Y
pause 
```

这个简单粗暴的 bat 文件为目标文件夹的名称生成日期戳，文件夹位于*D:\Batch\*。xcopy 命令会从*C:\Batch\*创建备份，而 pause 命令会保持窗口打开，这样我就能知道是否磁盘空间已满。这是一个非常简单的解决方案，满足了我的需求，但你可以在此基础上扩展，构建一个更强大的 bat 文件。

我通常认为自己是一个脚本推广者，尤其是 Batch 脚本推广者。程序员们常常在需要执行一些可以更轻松、更高效地通过脚本完成的任务时，依赖编译过的代码。在第一章中，我将*batphile*一词定义为“喜欢夜行飞行哺乳动物的人”。现在，我想给它一个次要的定义，“喜欢 bat 文件的人”。（Batfilephile 听起来实在太笨拙了。）我希望如果你还不是，现在至少在某种意义上你已经是一个骄傲的 batphile 了。有些人害怕这些飞行哺乳动物，但它们是迷人的动物，而且相当可爱。
