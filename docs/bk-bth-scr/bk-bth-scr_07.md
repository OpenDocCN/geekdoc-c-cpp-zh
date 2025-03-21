

## 第六章：6 整数与浮点数据类型



![](img/chapter.jpg)

在第五章中，我详细介绍了字符串和布尔数据类型。在这一章中，我将转向数值数据类型，特别是整数和浮点数据类型，并对它们进行深入研究。批处理能够轻松处理整数，无论是十进制、十六进制还是八进制变体。

然而，浮点数和布尔值类似，因为批处理实际上并不显式支持它们作为数据类型。但再一次，这个限制为富有创意的批处理程序员提供了发挥想象力的机会，这正是我们在本章结束之前将要做的。

### 八进制案例研究

8 月 1 日，某个“零零年代”的年份：我记不清确切的年份，但对于月份和日期我是非常确定的，原因到本章结束时就会明了。

我当时还相对较新于批处理，但我知道的比许多人都多，所以一个同事找到了我，帮忙处理他一直在挣扎的任务。在这段批处理代码中，他需要根据当前日期来确定前一天的日期。对于大部分日期来说，这个任务相当简单，但当今天是月初时，情况就变得复杂了。因为每个月的天数不同；新年的第一天是一个独特的挑战；闰年每四年发生一次，除了不发生的情况。

这个初步事件发生在 2 月，可能是 3 月，这是一个有趣的小练习，我写了代码并进行了测试。像所有优秀的程序员一样，我测试了每年第一天和最后一天。我还测试了几个极端月份的第一天，特别是像一月和十二月这样的月份。我测试了不同年份的 3 月 1 日，不是因为我是在 2 月编写这段代码，而是因为闰年的特殊性。不久后，我将代码交了出去，转向了其他项目。

代码在大约六个月内运行得很好。然后在 8 月 1 日，它突然就不工作了。我不记得后续的结果是什么，但我的同事花了大量时间追踪根本原因。他最终锁定了我的批处理文件，但无法弄清楚为什么它在那天停止工作。他的老板根本不听这些解释——代码能正常运行半年，然后突然崩溃。我的同事一定做了某种更改，导致了这个问题，他被挑战去找到这个更改。

这次搜索最终浪费了他半天的工作时间，但经过大量细致的工作，他终于将问题带给了我。我打开执行日志，找到了试图查找 08/01 之前日期的逻辑结果，然后...

我抬头望向天空，举起双手，带着沙特纳式的戏剧化大声喊道：“八进制！”我有些夸张——那一刻并没有像《星际迷航 II：可汗的愤怒》中可汗将柯克舰长（由威廉·沙特纳以莎士比亚式的风格演绎）困在一颗死去的星球中心那样戏剧化，但至少对我来说那一刻是相当难忘的。

执行日志中到底有什么让我不高兴的地方？让我们来看看，但在深入研究八进制之前，我将从整数开始。

### 整数

我们已经使用 set 命令处理字母数字值，但它也可以通过/A 选项用于算术运算。回想一下，像这样的语句会发生什么：

```
set x=4+5
```

x 所表示的变量被设置为文本 4+5。

使用/A 选项会将其转换为*算术*设置命令，因此以下命令将 x 变量设置为数字 9：

```
set /A x=4+5
```

/A 选项将 set 命令转化为执行加法和其他算术运算的工具。那些先前的值显然是硬编码为数字的。

一个稍微有趣一点的例子是将变量设置为数字值，然后通过 set /A 命令将它们相加，如列表 6-1 所示。

```
set nbr1=4
set nbr2=5
set /A sum = nbr1 + nbr2
> con echo The sum is %sum%. 
```

列表 6-1：通过 set /A 命令添加两个数字变量

控制台输出为“和是 9”，并且列表 6-1 展示了/A 选项显著改变了 set 命令——三次。首先也是最明显的，解锁了算术。第二，等号两边有空格，而在第二章中，我曾经大篇幅讲过空格带来的危险。为了说明这一点，这个没有使用/A 选项的命令：

```
set myVar = X
```

并没有将 myVar 设置为 X，而是将一个六字符的变量 myVar（后面跟着空格）设置为一个由空格和 X 组成的两字符值。相比之下，/A 选项使得 set 命令更像现代编程语言中的赋值操作符，因为命令中的空格不会被视为变量名或值的一部分；令人耳目一新的是，它们只是空格。

这三个命令在功能上是等价的；每个命令都将 myVar 设置为 7：

```
set myVar=7
set /A myVar=7
set /A myVar = 7 
```

若不使用/A 选项，等号两边不能有空格。然而，使用/A 选项时，可以有空格，但空格不是必需的，这就是/A 选项解锁的第二个重要区别。

列表 6-1 中的第三个区别在于变量 nbr1 和 nbr2 没有被百分号包围。因此，/A 选项允许你在没有常见分隔符的情况下解析变量。为了灵活性考虑，你仍然可以使用百分号和嵌入空格，或者不使用，所以这四个语句在逻辑上是等价的：

```
set /A result = nbr1 + nbr2
set /A result = %nbr1% + %nbr2%
set /A result=nbr1+nbr2
set /A result=%nbr1%+%nbr2% 
```

空格使代码更加易读，因此我不建议使用之前代码中的最后两个选项。第一个选项最简洁，但有些人已经习惯了变量周围有百分号，第二个选项或许能提供一种让人感到安稳的一致性。

让我们再来看一遍列表 6-1 中的 set /A 命令，不过这次它会在批处理文件的最开始执行：

```
set /A sum = nbr1 + nbr2
> con echo The sum is %sum%. 
```

控制台输出的求和结果将是 0。由于 nbr1 和 nbr2 尚未定义，未设置的变量在数值上下文中默认视为 0，而与此不同的是，在字母数字上下文中未设置的变量默认视为 null。由于两者都未设置，算术 0 + 0 的结果为 0。

警告

*允许的整数范围包括从-2,147,483,648 到 2,147,483,647（包含这两个值）。批量算术将数字存储为 32 位带符号字段，因此任何整数将属于这些 2**³²* *个值。这通常不会导致问题，但由于代码不是编译型的，需要注意确保正在处理的数据符合这一限制。代码不会中止，也不会挂起；它只是无法计算出正确的值。批量算术不是宏观经济学的首选语言。*

### 批量算术

批量算术不仅仅是简单的加法。以下列表展示了五种主要的算术运算（加法、减法、乘法、除法和取模运算）及其语法：

```
set /A sum = nbr1 + nbr2
set /A difference = nbr1 - nbr2
set /A product = nbr1 * nbr2
set /A quotient = nbr1 / nbr2
set /A modulo = nbr1 %% nbr2 
```

这些运算符与其他编程语言类似，但请注意取模除法的双百分号。帮助命令显示的是单一的百分号，但正确的批量语法需要两个百分号。（实际上，取模字符只是单一的百分号，但第一个百分号实际上是*转义*第二个。如果现在这并不太容易理解，可以等到第十四章再深入了解，但现在就使用两个符号。）

现在我们来执行这些算术命令，但首先我们要定义两个操作数 nbr1 和 nbr2。每条语句右侧的结果作为注释显示（如前所述，&符号用于分隔两个命令，第二个命令可以是 rem 命令）：

```
set nbr1=7
set nbr2=2

set /A sum = nbr1 + nbr2           &rem sum=9
set /A difference = nbr1 - nbr2    &rem difference=5
set /A product = nbr1 * nbr2       &rem product=14
set /A quotient = nbr1 / nbr2      &rem quotient=3
set /A modulo = nbr1 %% nbr2       &rem modulo=1 
```

加法、减法和乘法操作没有什么意外的结果，但将 7 除以 2 的结果是 3，而不是 3.5，因为批量算术只处理整数，并且会截断结果的小数部分。将 19 除以 10 不会得到 1.9，甚至不会返回四舍五入后的值 2\。1.9 的中间结果会被截断为 1。

取模是一个有用的运算符，用于返回余数。取模*n*返回的值是从 0 到*n* - 1，因此取模 2 操作对于偶数返回 0，因为 2/2、4/2、6/2 等的结果都是整数，不会产生余数。奇数则返回 1，因为 3/2、5/2、7/2 等都有余数 1。

奇怪的是，批量算术不支持指数或幂函数，这对一些人来说是一个令人沮丧的缺陷，但对另一些人来说则是激发创意的动力。你可以创建一个例程，输入基数和指数并返回指数结果（我将在第十八章中这样做）。

### 增强赋值运算符

*增量赋值运算符* 可以简化代码，当你希望将一个数字添加到变量并将结果存储在同一变量中时。最明显的例子是一个简单的计数器，可能在每次执行 set 命令时都想将一个变量递增 1，例如：

```
set /A veryVerboseTallyVariable = veryVerboseTallyVariable + 1
```

我故意选择了一个冗长且笨重的变量名，因为无论我们这些程序员怎么努力，它们有时几乎是不可避免的。

以下语法在逻辑上等价、简洁，并且更易理解：

```
set /A veryVerboseTallyVariable += 1
```

下一个命令将 17 添加到一个更简洁命名的变量中：

```
set /A nbr += 17
```

同样，以下的 set 命令分别减去 2、乘以 2、除以 2，并执行模 2 除法：

```
set /A nbr -= 2
set /A nbr *= 2
set /A nbr /= 2
set /A nbr %%= 2 
```

再次注意模除运算符的双百分号。许多经验丰富的 Batch 程序员并不知道增量赋值运算符在 Batch 中是可用的，错误地认为它们只存在于现代编程语言中，但它们确实存在，且在适当时应该使用。

### 运算顺序

你可以使用数学中的运算顺序规则进行更复杂的算术运算。你可能在代数预备课程中学过 PEMDAS 首字母缩略词（或者用“请原谅我亲爱的莎莉阿姨”作为记忆法），表示“括号、指数、乘法和除法、加法和减法”。在 Batch 中，我们使用 PMDAS，它发音困难，但如前所述，指数运算并不被支持（也许“请做甜点莎莉阿姨”这一记忆法能流行起来）。让我们看这个例子：

```
set /A nbr = 3 * (1 + 2) / 4 - 5
```

首先，1 和 2 被加在一起得 3，因为它们在括号内，尽管加法和减法在运算顺序中排在最后。乘法和除法具有相同的优先级，因此解释器会从左到右执行它们。表达式开头的 3 与加法中的 3 相乘，得到 9，然后 9 被 4 除，结果是 2.25。实际上，这是截断的，所以结果是 2。最后，减去 5，结果是 -3。

这个例子仅用于教学，因为直接将 nbr 设置为 -3 会更简单。实际上，会使用硬编码数字和变量的混合。例如：

```
set /A nbr = ((nbr1 + nbr2) * -10) / 4 
```

根据 PMDAS 规则，这里外部的括号是多余的，但它们使得语句更易读。

增量赋值运算符也可以与更复杂的表达式一起使用。以下两个语句在逻辑上是等价的：

```
set /A nbr = nbr + (2 * (4 + nbr) - -5)
set /A nbr += 2 * (4 + nbr) - -5 
```

在这两个命令中，变量 nbr 都在通过包含 nbr 的数学表达式递增，唯一的区别是第二个命令使用了增量赋值运算符。根据运算顺序，两者都会给变量加 4，然后将其翻倍，最后减去 -5（减去 -5 等同于加 5）。最终，这个表达式的结果是 nbr 增加的量。

### 八进制与十六进制运算

Batch 支持八进制和十六进制算术运算。这两种数制比十进制更接近计算机的*思维*方式，因此对程序员来说，理解它们并能够使用它们是很有帮助的。

十进制数系统是基数为 10，使用数字 0 到 9。没有代表 10 的数字；相反，有两个数字：新的位值从 1 开始，而个位数从 0 重新开始，因此是 10。与此相反，*八进制数系统*是基数为 8，使用数字 0 到 7。将 1 加到八进制的 7 不会得到 8，因为 8（和 9）在八进制数系统中是无意义的字符。相反，八进制数 10（发音为“一零”，因为它不是“十”）等于十进制数 8。同样，八进制数 11 等于十进制数 9，依此类推。

*十六进制数系统*是基数为 16，因此它和八进制面临相反的问题：它需要 16 个独特的数字，超过了大多数人类数制中使用的 10 个数字，因为我们是进化出了每只手有五个手指的结构。从 0 数到 9 后，我们有了“数字”A、B、C、D、E 和 F。十六进制的 B 等于十进制的 11，十六进制的 F 等于十进制的 15，十六进制的 10 等于十进制的 16。

Batch 可以进行八进制、十六进制和/或十进制输入的算术运算，并始终以十进制返回结果。十六进制数前面会加上 0x，而八进制数前面则单独加 0。因此，这两个变量分别被赋予八进制和十六进制的值：

```
set octalNbr=012
set hexadecimalNbr=0xB 
```

无论操作数的基数是十进制、八进制还是十六进制，Batch 总是将结果存储为十进制。为了演示，首先看这个例子：

```
set decimal7=7
set decimal1=1
set octal7=07
set octal1=01

set /A decimal = decimal7 + decimal1
set /A octal = octal7 + octal1 
```

数字 7 和 1 作为十进制和八进制相加。十进制的结果显然是 8。两个八进制数的和是八进制 10（“一零”，而不是十进制 10），但解释器会立即将其作为十进制 8 存储。在这个例子中，十进制和八进制表现相同，但这并不总是这样。

现在看这个例子：

```
set decimal11=11
set decimal2=2
set octal11=011
set octal2=02

set /A decimal = decimal11 + decimal2 
set /A octal = octal11 + octal2
> con echo The decimal sum is %decimal%.
> con echo The octal sum is %octal%. 
```

十进制加法得到十进制 13，而八进制加法得到八进制 13（“一三”，而不是十进制 13）。记住，八进制数系统没有 8 或 9。八进制 10 是十进制 8，在这个例子中，八进制 13 是十进制 11。因此，在 Batch 中，11 + 2 = 13，但 011 + 02 = 013 = 11，所以下面显示的结果是：

```
The decimal sum is 13.
The octal sum is 11. 
```

解释器甚至可以处理十进制和八进制值混合的算术运算。十进制的 10 + 10 是 20，而八进制的 010 + 010 是 16。当将十进制和八进制相加时，比如 10 + 010，Batch 会给出正确的结果 18。通常，这种类型的算术是偶然发生的，但有时精明的程序员会利用这一点，了解它是可能的也是很有用的。

以类似的方式，这些值被视为十六进制：

```
set /A hexadecimalNbr = 0xA * 0x14
```

通过这次乘法，0xA 等于十进制的 10，而 0x14 转换为十进制时比 16 大 4。执行此语句后，变量的值为 200，10 和 20 的积。

八进制和十六进制可以是强大的工具；但是，如果你打算做十进制算术运算，务必小心确保没有前导零。由于十六进制以 0x 开头，因此不小心进行十六进制运算要困难得多，但由于一个看似无害的前导零不知不觉地进行八进制运算则极为容易。

> 注意

*因为数学无处不在，你会在第十六章、第十八章和第二十一章中找到包含各种批处理文件算术运算示例的框。Batch 还具有用于位操作的算术运算符：按位与、按位或、按位异或、逻辑左移和逻辑右移。我将等到第三十章再探索它们，因为这些运算符使用一些具有其他用途的特殊字符，而且许多有经验的程序员从未在编译代码中操作过位，更不用说在批处理文件中了。*

### 浮动点数

批处理不明确处理浮动点数——也就是非整数有理数。事实上，如果需要对这样的数字进行大量处理，使用比批处理更好的工具。它就像是用铁锹挖掘房屋地基。虽然可以做到，但只有最严谨的苦行者才能完成。如果任务足够大，可以编写一些编译代码并从批处理文件中调用，但当只需要做一些轻量级的浮动点算术时，Batch 可以处理，就像你可以用铁锹在前院种几个郁金香球茎一样。

请记住，所有批处理变量实际上只是华丽的字符串。我们可以很容易地为几个变量赋予浮动点值——也就是带有小数点的数字。以下是两笔金额，单位为美元和美分：

```
set amt1=1.99
set amt2=2.50 
```

如果这些是整数，我们可以简单地使用 set /A 命令将它们相加。让我们试试，看会发生什么：

```
set /A sum = amt1 + amt2
```

结果是 3 存储在总和中，而不是期待的 4.49。每个数字的小数部分完全被忽略，导致 1 和 2 的整数和。

我们需要去掉小数点，进行算术运算，再恢复小数点。将每个金额乘以 100 就能解决这个问题，但批处理不会允许这样做。不过，由于浮动点值实际上只是一个伪装的字符串，我们可以使用上一章中描述的语法去掉小数点：

```
set amt1=%amt1:.=%
set amt2=%amt2:.=% 
```

现在金额是 199 和 250。此 set /A 命令的结果是 449：

```
set /A sum = amt1 + amt2
```

为了恢复小数，我们不能仅仅除以 100——这仅适用于整数——但是我们可以使用上一章中的字符串解析逻辑。通过子字符串提取，以下 set 命令将变量重置为三个项目的连接：数字的前面部分（去掉最后两个字节），一个硬编码的小数点（或点），以及数字的最后两个字节：

```
set sum=%sum:~0,-2%.%sum:~-2%
> con echo The sum is %sum%. 
```

最终，写入控制台的变量被设置为 4.49。

乘法的工作方式相同。如果你以 499 美元购买那台新电脑，第一年无需支付款项，利率为 19%，那么一年后你将欠多少钱？利率转化为 1.19 的倍数，但我们仍然必须去掉小数点。在找到两个整数的乘积后，我们通过在最后两个字节前插入小数点来恢复小数，如列表 6-2 所示。

```
set amt=499
set factor=1.19
set factor=%factor:.=%
set /A product = amt * factor
set product=%product:~0,-2%.%product:~-2%
> con echo The product is %product%. 
```

列表 6-2：整数与浮动小数的乘法

593.81 的乘积可能会让你重新考虑融资计划。

每个程序员的目标应该是编写“防弹”的代码。不幸的是，之前的代码更像是棉网而不是凯夫拉尔，并且有许多警告需要讨论。我们做了几个假设，如果其中任何一个被违反，代码就会出错。加法假设两个数字都包含两位小数；1.9 而不是 1.90 会使结果偏差 10 倍。除了小数点外，任何非数字字符都会引发问题，值前面的零会触发八进制运算。乘法则更为复杂。列表 6-2 包含一个整数，但如果 amt 表示的是美元和分，乘积将会有四个小数位，而不是两个。为了表示美元和分，应该去掉最后两个字节——或者更好地说，进行四舍五入。

我在这里不会深入探讨这些细节，原因很简单，如果输入数据不一致并且需要数据验证，批处理中的浮动小数算术可能不是最佳解决方案。为所有可能的情况编写代码无疑是繁琐的。重要的是，程序员要了解现有的选项。如果所有的值都具有一致的小数位数，则可以通过几行代码进行运算。当我在批处理程序中不得不使用浮动小数类型时，通常是为了处理涉及一致数据的特定任务。拿出那把铁锹，但只在合适的时候。

### 八进制案例研究，继续

那么，我在千年初的 8 月 1 日的执行日志中到底发现了什么呢？在批处理文件中，今天的日期被格式化为 CCYYMMDD，例如 20050801，并且被分解为三个独立的字段：

```
todaysYear = 2005
todaysMonth = 08
todaysDay = 01 
```

如果 todaysDay 不是 01，我们只需从八位数中减去 1，然后继续。但当它是 01 时，我们需要进行一些额外的算术运算。仅考虑月份的逻辑（并且理解 1 月会有一些特殊逻辑），我们必须减去 1 来确定前一个月份：

```
set /A month = todaysMonth - 1
```

当 todaysMonth 是 03 时，月份为 2；当 todaysMonth 是 07 时，月份为 6。但当 todaysMonth 是 08（即 8 月 1 日）时，前面的算术结果会变成-1。

解释器看到前导 0，并将算术视为八进制运算。八进制只理解数字 0 到 7，所以当解释器看到 8 时，它会认为这个字符像“ohkuh”一样陌生（“ohkuh”是瓦肯语言中数字 8 的发音），并简单地忽略它。最终，`set /A`命令将表达式中剩余部分的数学结果（即-1）赋值给月份变量。这个值最终打破了日期逻辑，导致我们无法获得所需的 7 月 31 日日期。

“八进制！”

使用子字符串提取和`if`命令，我插入了这一行代码来去除 todaysMonth 变量中可能存在的前导零：

```
if %todaysMonth:~0,1% equ 0  set todaysMonth=%todaysMonth:~1%
```

这段代码在未来几年都能正常工作，甚至在 8 月和 9 月的 1 号。如果原始代码没有在 8 月 1 日运行，那么在 9 月 1 日运行时会失败，因为 9 月被表示为 09。但如果代码没有在这两天运行呢？下一次会在哪一天失败呢？在 10 月 1 日，月份会被表示为 10。解释器会把它当作十进制数处理，因此代码会按预期执行。所以，8 月和 9 月的 1 号是唯一能打破这段代码的日期。

要非常注意八进制。

### 总结

在本章中，我讨论了数值数据类型以及它们在批处理中的处理方式。与大多数其他编程语言不同，批处理变量并没有定义为某种特定的数据类型。从本质上讲，所有变量都是简单的字符串，但当该字符串包含一个数字时，它可以被视为数值。

加法、减法、乘法、除法，甚至取模运算都能很容易地在十进制整数上进行，按照你可能在学校里学过的运算顺序规则。八进制和十六进制整数也得到了支持，尽管八进制运算容易被误用。根据我的个人经验，确保你的十进制整数没有前导零。增强赋值运算符提供了一种方便且未充分利用的工具，用于递增整数。

批处理中不支持浮动点数数据类型，但你已经学会了，通过一些小技巧，可以对带小数点的数字进行一些简单的算术运算。

转变话题，我将在下一章讨论文件操作。批处理的一个非常有用的功能是创建、复制、移动、重命名和删除文件和目录。
