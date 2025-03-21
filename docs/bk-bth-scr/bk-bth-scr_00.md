

# 前言



![](img/chapter.jpg)

一本关于 bat 文件的书？为什么？难道阿兹特克人也用过 Batch 吗？它不是脚本语言中的 Betamax 吗？你应该写一本关于更新、更有吸引力的编程语言的书，而不是一本 Model T 的维修手册。

我希望我能直接忽略这种抗议，将其视为啰嗦小人的消极言论，但这是一个我感到必须回应的观点。Batch 语言并不新颖，按照今天的标准来看，它缺乏一些功能让人费解，但它依然是一个极为有用的语言，短期内不会消失，尤其是它已经与每台 Windows 电脑上的操作系统捆绑在一起。虽然 Batch 只是众多脚本语言中的一种，但仍然有许多大型和小型公司在使用 Batch 代码，有些任务确实比任何其他语言更适合用 bat 文件来完成。至于那些缺失的功能——布尔值、数组、哈希表、栈，甚至面向对象设计等等——在本书的结尾，我会教你如何自己构建这些功能。

但我个人写一本关于 Batch 脚本的书的最直接原因是，经过二十年的个人和职业用途的 bat 文件编写，我相信自己已经在这个领域学到了足够的知识，能够与更广泛的社区分享我的经验和见解。在很多年里，我在一家公司为 Windows 服务器编写大规模处理程序，所有的程序都由 bat 文件驱动。其他人可能会选择更现代的脚本语言，但在我之前的那位程序员已经精通了 bat 文件的艺术，以至于没有人认真考虑过使用 Batch 之外的替代方案。直到他退休，我才在非正式的情况下接替了他的工作，扮演起了罗宾的角色，直到后来我被正式晋升为蝙蝠侠。

编写 Batch 代码仍然是任何程序员甚至非程序员的重要技能，但现有的文档往往稀缺、分散，甚至有时不准确。与其他语言相比，掌握 Batch 语言需要大量的经验和实验，而我有一个独特的视角可以分享。这就是我写这本书的原因。

## 本书的读者群体

这本书既不是为初学者写的，也不是为专家写的；它是为*两者*而写的。实际上，我希望能够接触到三个群体。第一个是那些几乎每天都在编写、维护或与 bat 文件打交道的程序员。第二个是所有在 Windows 机器上工作的其他程序员，第三个是那些也在 Windows 电脑上工作的非程序员。

第一类人群，那些与 Batch 密切合作的人，显然也在本书的读者范围内。这本书是我在 Batch 脚本领域二十年深入工作的结晶。到本书的最后，你将会探索到一些复杂的概念，比如创建命令、数据结构、运算符，甚至是语言的创造者未曾预见的编码范式。我将逐步讲解这些复杂的内容，但我希望在这些页面中，你能找到掌握这门语言以及进一步探索其未涉及部分所需的所有工具。

如果你属于第二类人群，你可能不会维护成千上万行的 Batch 代码，但在 Windows 电脑上，你会用其他语言编写代码，并且你至少应该对 Batch 有一定的了解。这项技能让你可以通过运行一个简单的（或者可能不太简单的）bat 文件，完成一些常见且重复的任务。用其他语言编写的代码在动画化时会面临一些挑战，其中之一就是你的机器环境与程序最终运行的生产环境不同。为此，我将向你展示如何用几行 Batch 代码模拟或模拟另一个计算机的环境。到本书的最后，我相信你会发现，bat 文件是许多问题的解决方案。

即使是非程序员，最后这一类人，也能通过一些 Batch 代码来简化重复性任务，比如移动文件、合并报告或连接到网络驱动器，从而使 Windows 资源管理器更易使用。由于编程不在你的工作职责范围内，你的雇主不太可能为你的计算机安装其他编程语言的基础设施，来让你完成相对简单的编码任务，但你所需要的编写和执行 bat 文件的工具已经在你的工作站上了。编写 bat 文件所需的技能就是能够创建文本文件、重命名文件，并向文件中输入几行内容。如果你能双击文件，你就能运行 bat 文件。这就是你所需要的一切（除了这本书）。

## 如何阅读本书

每个作者，无论哪种类型的作品，都会设想读者坐在火旁，品着雪利酒（或者对我来说是一杯好的麦芽酒，不要太甜），全神贯注地读着每一个字，阅读、理解，再继续阅读，直到书本结束。嗯……这是一本技术书籍，所以我的读者中有相当一部分是程序员，他们坐在电脑前，试图弄明白为什么他们的 bat 文件没有按预期运行。我曾经也经历过这种困境，深刻理解其中的难处，为了帮助你，我将这本书组织成了有标题、子标题、详细的目录和索引。你可以找到能解答问题的章节和页码，并直接跳到那一部分，但这并不是阅读这本书或任何一本书的理想方式。

我将本书结构设计为简短而简洁的章节。即使你只是想解决某个特定问题，我也建议你通读相关章节，因为每一章都像是一份课程计划。（我的日常工作是编码，但我受过数学训练，已经在康涅狄格州曼彻斯特社区学院教授了二十多年的数学课程。）

一堂典型的课程从基本概念开始，接着是一些简单的示例。然后我会深入探讨该主题的复杂性，展示该概念的应用，甚至解释常见的陷阱和避免的错误。并不是每一课（或每一章）都会遵循这个结构，但很多都会。如果你有关于如何复制文件的疑问，我建议你从头到尾阅读第七章。跳到章节中间就像是上课迟到 20 分钟。

我还建议你亲自执行一些我提供的编码示例。大部分代码片段都很短且容易输入，你可以从本书的在线版本中获取较长的代码。更好的做法是修改代码，探索其结果，并将其变成你自己的代码。

## 本书结构

Batch 的独特之处在于一个单一的命令——for 命令，远远超过其他命令，支配了整个 Batch 脚本语言。因此，我将本书分为三部分，围绕这个至关重要的命令进行组织。第一部分标题为“基础知识”，涵盖了你在学习 for 命令之前需要掌握的相关内容。第一部分包括以下章节：

**第一章：Batch**    本章将向你介绍 Batch 脚本语言，同时帮助你编写可能是你第一次编写的 bat 文件。我还会提供一些编辑技巧，由于 Batch 是一种解释型语言，我还会讨论解释器的角色和重要性。

**第二章：变量与值**    本章讲解如何定义变量，并查询其值，无论是显示到控制台还是用于其他目的。

**第三章：作用域与延迟扩展**    在学习如何定义变量在 bat 文件中的访问范围后，我将介绍 Batch 中最有趣的特性之一——延迟扩展，它影响如何解析变量。

**第四章：条件执行**    if...else 语句是大多数编程语言中的基本结构，Batch 也不例外。在本章中，你将学习如何根据不同的条件语句执行或跳过代码片段。

**第五章：字符串与布尔数据类型**    本章讲解构建和连接字符串，提取更大字符串中的子字符串，以及在字符串中替换特定文本的任务。我还将介绍我们将要构建的许多非 Batch 内建工具中的第一个——布尔值或评估为真或假的变量。

**第六章: 整数与浮点数据类型**    你将学习加法、减法、乘法和除法等整数操作的所有细节。本章还详细介绍了取余运算，以及八进制和十六进制的算术运算。然后我将深入探讨另一种在批处理语言中并非固有的数据类型：浮点数。

**第七章: 与文件的操作**    本章处理与文件相关的许多任务，如复制、移动、删除、重命名文件，甚至创建一个空文件。

**第八章: 执行编译后的程序**    本章探讨了如何在有路径和没有路径的情况下调用程序，特别是当你没有提供路径时，解释器是如何找到你的程序的。

**第九章: 标签与非顺序执行**    本章介绍了标签以及它们在允许你将代码执行转向批处理文件中的前后命令中所起的作用，有时甚至会启动一个循环。

**第十章: 调用例程和批处理文件**    在上一章的基础上，你将学习在批处理文件中创建可调用例程的所有内容，以及如何从另一个批处理文件中调用一个批处理文件。

**第十一章: 参数与参数值**    如果你不能将参数传递给被调用的代码，或者被调用的代码不能将参数值返回给你，那么调用其他代码往往毫无意义。本章深入探讨了这一过程的所有细节，甚至揭示了隐藏的参数。

**第十二章: 输出、重定向与管道**    在区分了程序员和解释器所产生的输出后，我讨论了如何将这两者重定向到控制台或文件，这自然引出了将一个命令的输出传递给另一个命令的管道技术及其应用。

**第十三章: 与目录的操作**    本章详细介绍了如何创建和删除目录，以及如何检索有关目录及其内容的大量信息。我还演示了将本地和网络目录映射到驱动器字母的技术。

**第十四章: 转义**    如果你想在字符串中使用某个字符，而这个字符在批处理（Batch）中是一个具有特定功能的特殊字符，你会遇到问题。本章详细介绍了针对这一问题的解决方案，有时这些解决方案可能会出乎意料地复杂。

**第十五章: 交互式批处理**    在本章中，你将构建一个功能完整的批处理用户界面，允许从控制台接受自由格式文本，并让用户从列表中选择一个项目，除此之外还有其他功能。

**第十六章: 代码块**    代码块不仅仅是代码的块。本章探讨了代码块中的变量为何以及如何能够拥有两个不同的值。我还将介绍裸代码块，并解释它的意义。

第二部分的标题为“`for`命令”，正如其名称所暗示的，这部分内容深入探讨了前面提到的`for`命令，它为你提供了一大批（双关含义）功能。你将在这里找到以下内容：

**第十七章：`for`命令的基础**    本章详细介绍了`for`命令的功能，未引入任何选项，但仍然非常强大。它可以创建处理任意数量输入文件或文本字符串的循环，并通过使用修饰符，你将能够确定关于文件的几乎所有信息，除了文件内容。

**第十八章：目录、递归和迭代循环**    本章探讨了`for`命令的一些选项，这些选项提供了更多的功能。使用其中一个选项，命令可以遍历一个目录列表，而不是文件名列表。使用另一个选项，你可以递归地处理目录和子目录，例如，在一个文件夹及其所有子文件夹中搜索符合特定模式的文件。还有一个选项将命令转换为一个迭代循环，每次执行时递增或递减索引。

**第十九章：读取文件和其他输入**    最后一个选项为`for`命令提供了强大功能，允许你读取文件。本章详细介绍了如何在读取文件时解析或重新格式化每一条记录。除了传统的文件，命令还可以读取和处理普通文本，无论是硬编码的还是来自变量的，甚至可以将另一个命令的输出作为文件读取。

**第二十章：`for`命令的高级技巧**    本章深入探讨了`for`命令的一些令人印象深刻的应用，例如将其他语言（例如 PowerShell 和 Python）的命令嵌入到你的批处理脚本中。我还讨论了绕过命令限制的一些技巧。

“高级主题”是第三部分的标题，讨论了各种各样的主题，特别是那些在我拥有`for`命令工具之前无法涉及的内容。以下是详细内容：

**第二十一章：伪环境变量**    本章详细介绍了伪环境变量，或称特殊变量，这些变量并不总是由你控制。例如，批处理有一些特定的变量，用来保存日期、时间以及批处理命令和被调用程序的返回代码。我还解释了如何安全地设置这些变量，并分享了批处理文件（.bat）和命令文件（.cmd）之间的区别。

**第二十二章：编写报告**    本章解释了如何使用批处理格式化基础的文本文件报告，包括标题、详细记录和结尾记录。

**第二十三章：递归**    一些问题非常适合使用递归技术，即代码调用自身的方法。本章通过详细且有趣的例子演示了如何在批处理语言中实现递归。

**第二十四章：文本字符串搜索**    本章探讨了多种文本字符串搜索的排列方式。搜索文件、变量或硬编码文本中的一个或多个单词或字面字符串。你还会找到一些使用正则表达式的例子。

**第二十五章：批处理文件构建批处理文件**    本章详细介绍了一个批处理文件如何构建第二个完全功能的批处理文件，包括动态和静态代码，同时也思考了如果是阿基米德，他会如何使用批处理。

**第二十六章：自动重启与多线程**    在讨论如何自动重启失败的进程后，本章通过构建一个批处理文件来自动终止并重启一个挂起的进程。我还讨论了如何在单一批处理文件的控制下同时执行多个线程或并发任务。

**第二十七章：与/或运算符**    这可能听起来像是一个基础话题，但批处理语言本身并没有与运算符或或运算符。本章构建了模拟这些运算符的技术，用于各种情况。

**第二十八章：紧凑的条件执行**    本章详细介绍了一种紧凑且有趣的结构，它看起来和行为非常类似于 if...else 结构。我会在检查两者之间微妙但重要的差异后，讨论何时最好使用每种方式。

**第二十九章：数组和哈希表**    这些数据结构并非批处理的内建功能，但你将学习如何填充和从数组及哈希表中检索数据。

**第三十章：杂项**    本章涵盖了一些不同的主题：文件属性、位操作、查询 Windows 注册表以及排序文件内容。

**第三十一章：故障排除技巧与测试技术**    我分享了多年来在开发和测试批处理文件中学到的许多技巧和方法。

**第三十二章：面向对象设计**    尽管听起来有点疯狂，本章将呈现用户自定义批处理文件功能的顶峰。我将在讲解面向对象设计的四个支柱后，带领你通过一个尽可能全面实现这些原则的模型。我希望经验丰富的编码者会觉得本章既有信息性又有娱乐性。

**第三十三章：堆栈、队列与现实世界中的对象**    本章应用刚学到的面向对象设计原则，构建实现堆栈和队列数据结构的对象。

在第一部分和第三部分的每一章中，我都设定了一个狭窄的主题或任务讨论；我不打算讨论某个特定命令，但通常会在章节中介绍一条或多条命令。对于每一条命令，我都会解释它的功能，展示它的语法，并详细介绍我认为最有用的特性。

我的目标是，如果你不是程序员，你至少能读懂前两部分并从中获得信息。读得更深入一点，你或许就能成为一名程序员。

## 其他资源

如果你在寻找单个批处理命令的全面且简明的解释，不妨去看看 *[`<wbr>ss64<wbr>.com<wbr>/nt<wbr>/`](https://ss64.com/nt/)*。这是一个很棒且组织良好的资源，在写这本书时我参考了它很多次。本书不是命令的列表，而是关于如何用这些命令解决问题的讨论。我通常会展示我认为最有用的命令选项，但你可以在这个网站上找到完整的命令列表。

如果你（希望）遇到的情况很少能在这些页面上找到解决方案，那么下一个最好的选择是向在线的技术社区寻求帮助。用“bat file”加上你的问题在网络上搜索应该能得到一些结果。在众多的在线论坛中，我发现最好的创意和建议常常来自于 *[`<wbr>stackoverflow<wbr>.com`](https://stackoverflow.com)*。

## 风格说明

大多数技术书籍和手册读起来都很枯燥，而我已尽最大努力打破这种趋势。首先，我没有忘记我的首要任务是解释我试图传授的技术内容。但举个例子，当讨论 `sort` 命令时，我不想对像苹果和香蕉这样的东西进行排序；与其如此，不如把《星际迷航》中的 *Enterprise* 星舰的舰长们排序，或者至少我觉得这样更有趣。讨论参数时，我使用了一个 Mad Libs 游戏，通过不同的词性作为参数传递。关于交互式批处理的章节还与用户分享了些笑话。

并非每一章都能举出有趣的例子或幽默的轶事，但我已经尽力避免让文件的第一条记录是“Record 1”或一串用管道符号分隔的字段（如 field 1| field 2| field 3）。

理想情况下，我希望能引发一阵可听的笑声；如果能看到一抹微笑和点头，我也会很高兴；即便只是翻个白眼和叹气，我也能接受。无聊去死。我秉持着“与其平凡，不如独特地糟糕”这一座右铭。（我希望能为这句话归功于自己，但很多年前，在加利福尼亚州索诺玛县的本齐格酒庄旅游时，我们的导游用这句话来形容酒庄的哲学。）

## 注意事项

根据我的经验，Batch 相较于其他语言有许多显著的注意事项。在接下来的章节中，我常常会在一些看似明确的语法或用法陈述后加上*除了*这个词。（例如，“&符号用于终止命令，*除非*后面跟着第二个&符号或……”）英语独特之处在于，它的语法中有很多注意事项，这些在其他语言中根本不存在——想想看“*i* 在 *e* 前面，*除了*在 *c* 后面。”也许这使得 Batch 成为典型的爱国美国编程语言（或者也可能是英国式的）。

这些*Batch 注意事项*如此普遍，以至于我已开始称它们为 *batveat*（发音为 bat-vē-ät，商标待定）。这些注意事项对于没有指导的新用户来说可能非常令人沮丧，但随着章节的展开，我会指出那些曾经让我受困的 batveat，希望你能避免这些痛苦。

## 伍迪·格思里

我为这本书选择的引言来自传奇艺术家伍迪·格思里（Woody Guthrie）的一句相对著名的名言，但我曾犹豫是否使用它，因为担心它可能会被误解。引用的目的并非自负，而是富有理想的。伍迪曾穿越美国，提倡经济正义，同时也宣扬反对种族主义和性别歧视。他并非通过枯燥无味的演讲，而是通过吉他和敏锐的歌词来传播这些思想，这些歌词即使在他早逝之后，依然引起人们的共鸣。

伍迪·格思里试图将历史的轨迹引向社会正义，而我则在尝试通过富有信息性、可读性和娱乐性的语言，使一个深奥的编程语言变得更易接近。我希望能够为理解一个复杂话题做出贡献，并且我只能以伍迪的崇高榜样为目标，努力追随。

## Batch 的爱

永远，永远不要邀请超过一个 Batch 程序员参加派对。一个人就足够了。如果我们没有同行，我们会像其他人一样谈论体育、政治、书籍、电影和旅行。但一旦你把至少两个程序员聚在一起，你就会听到类似“我最近找到了一种在 if 命令的条件语句中编码 or 操作符的新方法。你想让我分享给你吗？”我们会毁掉你的派对。

乐观主义者会说 Batch 是*深奥的*，而悲观主义者则会说它是*晦涩的*。事实可能介于两者之间，你会在本书中经常看到这两个词。它的语法与大多数编程语言不同，某些功能的缺失促使人们以富有创意的方式解决问题，这些问题在其他语言中可能显得无趣。结果就是，一些人在你的超级碗派对上讨论构造哈希表的不同方式，几乎让空气都变得稀薄。

我觉得这些难题令人精神焕发，这也是我喜欢在 Batch 脚本中编程的主要原因，而其他人可能会觉得这是一项苦差事。有时候，我真的很享受用一种编程语言来编写栈实现，这本身就是一项了不起的成就。为了简单展示一个挑战，"@"符号本身可以作为变量名，而从其值中提取倒数第二个字符需要使用语法 %@:~-2,1%。这看起来更像是漫画中的脏话，而不是代码，诚然，它确实显得有些深奥，甚至可能显得有些神秘，但请不要因为害怕就把这本书放下；我保证，只需几章，你就会完全理解。

在一群对这门技术略懂一二的程序员中成为 Batch 专家，可能会让你感觉像是一个苏美尔祭司——你是那种能解读脚本并将其意义与智慧传授给他人的特权人群中的一员。但我之所以能占据这个位置，并不是因为某种偶然的天赋，也不是为了个人私利而守护解读这门楔形文字的能力。通过这本书，我希望能让所有想学习这门并不那么古老的脚本的人，都成为高阶祭司和祭司女。在接下来的章节中，我会诚实地谈论我在这门语言中的问题和挫折，但我真的很喜欢编写批处理文件，等你读完这本书后，我希望能把你变成一个信徒。
