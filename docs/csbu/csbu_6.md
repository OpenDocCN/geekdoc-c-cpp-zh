<main class="calibre3">

## 第七章. 工具链

</main>

<main class="calibre3">

## 1 编译型与解释型程序

### 1.1 编译型程序

到目前为止，我们已经讨论了程序如何加载到虚拟内存中，作为操作系统跟踪并通过系统调用来交互的进程。

可以直接加载到内存的程序需要以直 *二进制* 格式存在。将用如 C 语言编写的源代码转换为准备执行的二进制文件的过程称为 *编译*。毫不奇怪，这个过程是通过 *编译器* 来完成的；最广泛使用的例子是 gcc。

### 1.2 解释型程序

编译型程序在现代软件开发中存在一些缺点。每次开发者进行更改时，都必须调用编译器来重新创建可执行文件。设计一个可以读取 *另一个* 程序列表并逐行执行代码的编译型程序是一个逻辑上的扩展。

我们称这种类型的编译型程序为 *解释器*，因为它解释输入文件的每一行并将其作为代码执行。这样，程序就不需要编译，任何更改都会在解释器下次运行代码时显现。

为了方便起见，解释型程序通常比编译型程序运行得慢。对于编译型程序，每次读取和解释代码的开销只遇到一次，而对于解释型程序，每次运行都会遇到。

但解释型语言有许多积极的方面。许多解释型语言实际上是在从底层硬件抽象出来的 `虚拟机` 中运行的。Python 和 Perl 6 是实现虚拟机的语言，解释型代码在其上运行。

#### 1.2.1 虚拟机

编译型程序完全依赖于为其编译的机器的硬件，因为它必须能够简单地复制到内存中并执行。虚拟机是硬件到软件的抽象。

例如，Java 是一种部分编译和部分解释的混合语言。Java 代码被编译成一个在 *Java 虚拟机* 或更常见地称为 JVM 中运行的程序。这意味着编译后的程序可以在任何为其编写了 JVM 的硬件上运行；所谓 *一次编写，到处运行*。

</main>

<main class="calibre3">

## 2 构建可执行文件

当我们谈论编译器时，实际上涉及创建可执行文件的三个独立步骤。

1.  编译

1.  汇编

1.  链接

在这个过程中涉及到的组件统称为 *工具链*，因为这些工具 *链* 接一个工具的输出到另一个工具的输入，以创建最终的输出。

链中的每个环节都将源代码逐步转换为适合执行的二进制代码。

</main>

<main class="calibre3">

## 3 编译

### 3.1 编译过程

将源文件编译成可执行文件的第一步是将代码从高级、人类可理解的语言转换为*汇编代码*。我们从前面的章节中了解到，汇编代码直接与处理器提供的指令和寄存器交互。

编译器是整个过程中最复杂的步骤，原因有很多。首先，人类是非常不可预测的，他们的源代码形式多种多样。编译器只对实际代码感兴趣，然而人类需要诸如注释和空白（空格、制表符、缩进等）来理解代码。编译器将人类编写的源代码转换为内部表示的过程称为*解析*。

#### 3.1.1 C 语言代码

在 C 代码中，实际上在解析源代码之前有一个步骤叫做*预处理器*。预处理器本质上是一个文本替换程序。例如，任何声明为`#define variable text`的变量都将用`text`替换`variable`。然后，这个预处理后的代码被传递到编译器。

### 3.2 语法

任何计算语言都有特定的*语法*，它描述了语言的规则。你和编译器都知道语法规则，如果一切顺利，你们将互相理解。然而，人类常常会忘记规则或违反规则，导致编译器无法理解你的意图。例如，如果你在`if`条件中遗漏了闭合括号，编译器将不知道实际的条件在哪里。

语法通常用*巴科斯-诺尔范式*（BNF）来描述。实际上，最常见的形式是扩展巴科斯-诺尔范式，或 EBNF，因为它允许一些更适合现代语言的额外规则。这是一种你可以用来描述语言的工具！

### 3.3 代码生成

编译器的任务是将其转换为目标架构适合的汇编代码。显然，不同的架构有不同的指令集、不同的寄存器数量和不同的正确操作规则。

#### 3.3.1 对齐

<picture>![CPU 通常只能在特定对齐方式下从内存中加载值到寄存器。非对齐加载会导致，至多，性能下降。](img/alignment.svg)</picture>图 3.3.1.1 对齐

在内存中对变量进行对齐是编译器需要考虑的一个重要因素。系统程序员需要了解对齐约束，以帮助编译器生成最有效的代码。

CPU 通常不能从任意内存位置将值加载到寄存器中。它要求变量在特定的边界上*对齐*。在上面的例子中，我们可以看到在需要 4 字节对齐变量的机器上，如何将 32 位（4 字节）值加载到寄存器中。

第一个变量可以直接加载到寄存器中，因为它位于 4 字节边界之间。然而，第二个变量跨越了 4 字节边界。这意味着至少需要两次加载才能将变量放入单个寄存器；首先加载下半部分，然后加载上半部分。

一些架构，例如 x86，可以在硬件上处理未对齐的加载，唯一的症状将是性能降低，因为硬件需要做额外的工作将值放入寄存器。其他架构不能违反对齐规则，将会引发异常，通常由操作系统捕获，然后操作系统必须手动分部分加载寄存器，造成更多的开销。

##### 3.3.1.1 结构体填充

程序员在创建`struct`s 时需要考虑对齐。虽然编译器知道它为该架构构建的对齐规则，但有时程序员可能会引起次优行为。

C99 标准仅说明结构体在内存中的顺序将与声明中指定的顺序相同，并且结构体数组中所有元素的大小都相同。

```cpp
 1 |$ cat struct.c 
 |#include <stdio.h> 
 |
 |struct a_struct { 
 5 | char char_one; 
 | char char_two; 
 | int int_one; 
 |}; 
 |
10 |int main(void) 
 |{ 
 |
 | struct a_struct s; 
 |
15 | printf("%p : s.char_one\n" \ 
 | "%p : s.char_two\n" \ 
 | "%p : s.int_one\n", &s.char_one, 
 | &s.char_two, &s.int_one); 
 |
20 | return 0; 
 |
 |} 
 |
 |$ gcc -o struct struct.c 
25 |
 |$ gcc -fpack-struct -o struct-packed struct.c 
 |
 |$ ./struct 
 |0x7fdf6798 : s.char_one 
30 |0x7fdf6799 : s.char_two 
 |0x7fdf679c : s.int_one 
 |
 |$ ./struct-packed 
 |0x7fcd2778 : s.char_one 
35 |0x7fcd2779 : s.char_two 
 |0x7fcd277a : s.int_one 

```

示例 3.3.1.1.1 结构体填充示例

在上面的例子中，我们构造了一个结构体，它有两个字节（`chars`后跟一个 4 字节整数。编译器按照以下方式填充结构体。

<picture>![编译器填充结构体以对齐整数到 4 字节边界。](img/padding.svg)</picture>图 3.3.1.1.1 对齐

在另一个例子中，我们指导编译器*不要*填充结构体，相应地，我们可以看到整数直接跟在两个`chars`之后。

##### 3.3.1.2 缓存行对齐

我们之前讨论了缓存中的别名问题，以及多个地址可能映射到相同的缓存行。程序员需要确保在编写程序时，他们不会导致缓存行的*弹跳*。

这种情况发生在程序不断访问映射到相同缓存行的两个内存区域时。这实际上浪费了缓存行，因为它被加载进来，短暂使用后，必须将其踢出，并将另一个缓存行加载到缓存中的相同位置。

显然，如果这种情况重复出现，性能将显著降低。如果将冲突数据以稍微不同的方式组织，以避免缓存行冲突，则可以缓解这种情况。

检测这种状况的一种可能方法是*分析*。当你分析你的代码时，你“监视”它以分析采取的代码路径以及它们执行所需的时间。通过*分析指导优化*（PGO），编译器可以在构建的第一个二进制文件中放置特殊的额外代码，该代码运行并记录分支等。然后你可以使用额外的信息重新编译二进制文件，以可能创建性能更好的二进制文件。否则，程序员可以查看分析输出，并可能检测到缓存行弹跳等情况。（XXX 其他地方？）

##### 3.3.1.3 空间-速度权衡

编译器在上面所做的是通过使用一些额外的内存来换取运行我们代码的速度提升。编译器了解架构的规则，可以做出关于最佳数据对齐方式的决定，可能通过牺牲少量浪费的内存来换取（或甚至只是正确的）性能提升。

因此，作为一个程序员，你不应该对变量和数据将由编译器如何布局做出假设。这样做是不可移植的，因为不同的架构可能有不同的规则，编译器可能会根据显式命令或优化级别做出不同的决定。

##### 3.3.1.4 做出假设

因此，作为一个 C 程序员，你需要熟悉你可以假设编译器会做什么以及可能是什么变量。你可以假设和不能假设的确切内容在 C99 标准中有详细说明；如果你在用 C 编程，熟悉这些规则以避免编写不可移植或存在错误的代码是非常值得的。

```cpp
 1 |$ cat stack.c 
 |#include <stdio.h> 
 |
 |struct a_struct { 
 5 | int a; 
 | int b; 
 |}; 
 |
 |int main(void) 
10 |{ 
 | int i; 
 | struct a_struct s; 
 | printf("%p\n%p\ndiff %ld\n", &i, &s, (unsigned long)&s - (unsigned long)&i); 
 | return 0; 
15 |} 
 |$ gcc-3.3 -Wall -o stack-3.3 ./stack.c 
 |$ gcc-4.0 -o stack-4.0 stack.c 
 |
 |$ ./stack-3.3 
20 |0x60000fffffc2b510 
 |0x60000fffffc2b520 
 |diff 16 
 |
 |$ ./stack-4.0 
25 |0x60000fffff89b520 
 |0x60000fffff89b524 
 |diff 4 

```

示例 3.3.1.4.1 栈对齐示例

在上面的例子中，取自 Itanium 机器，我们可以看到 gcc 版本之间栈的填充和对齐发生了相当大的变化。这种事情是可以预料的，程序员必须考虑这一点。

通常，你应该确保不要对类型的大小或对齐规则做出假设。

##### 3.3.1.5 对齐的 C 语言习语

有一些常见的代码序列处理对齐；通常大多数程序都会以某种方式考虑它。你可能会在内核之外许多地方看到这些“代码习语”，当处理以某种形式处理数据块的程序时，所以值得调查。

我们可以从 Linux 内核中举一些例子，内核经常需要处理系统内内存页的对齐。

```cpp
 1 |[ include/asm-ia64/page.h ] 
 |
 |/* 
 | * PAGE_SHIFT determines the actual kernel page size. 
 5 | */ 
 |#if defined(CONFIG_IA64_PAGE_SIZE_4KB) 
 |# define PAGE_SHIFT     12 
 |#elif defined(CONFIG_IA64_PAGE_SIZE_8KB) 
 |# define PAGE_SHIFT     13 
10 |#elif defined(CONFIG_IA64_PAGE_SIZE_16KB) 
 |# define PAGE_SHIFT     14 
 |#elif defined(CONFIG_IA64_PAGE_SIZE_64KB) 
 |# define PAGE_SHIFT     16 
 |#else 
15 |# error Unsupported page size! 
 |#endif 
 |
 |#define PAGE_SIZE               (__IA64_UL_CONST(1) << PAGE_SHIFT) 
 |#define PAGE_MASK               (~(PAGE_SIZE - 1)) 
20 |#define PAGE_ALIGN(addr)        (((addr) + PAGE_SIZE - 1) & PAGE_MASK) 

```

示例 3.3.1.5.1 页面对齐操作

在上面，我们可以看到内核内有多种页面大小的选项，从 4KB 到 64KB。

`PAGE_SIZE` 宏相当直观，通过将 1 的值左移给定的移位次数（记住，这相当于说 `2^n`，其中 `n` 是 `PAGE_SHIFT`）来给出系统内当前选定的页面大小。

接下来，我们有一个 `PAGE_MASK` 的定义。`PAGE_MASK` 允许我们找到当前页面内的那些位，即地址在其页面内的 `offset`。

XXX 继续简短讨论

### 3.4 优化

一旦编译器有了代码的内部表示，编译器的真正有趣部分就开始了。编译器想要找到给定输入代码的最优化的汇编语言输出。这是一个庞大且多样化的问题，需要了解从基于计算机科学的效率算法到特定处理器的深入知识。

在生成输出时，编译器可以查看一些常见的优化。有更多、更多的策略用于生成最佳代码，并且这始终是一个活跃的研究领域。

#### 3.4.1 通用优化

编译器通常能够识别出某些代码片段无法使用，因此会将其省略，以优化特定的语言结构，将其转换为具有相同结果但更小的形式。

#### 3.4.2 展开循环

如果代码包含循环，例如 `for` 或 `while` 循环，并且编译器对循环将执行多少次有一些了解，那么将循环展开以顺序执行可能更有效。这意味着，而不是执行循环内部代码然后跳转回开始处重复这个过程，循环内部的代码会被复制以再次执行。

虽然这会增加代码的大小，但它可能允许处理器更有效地执行指令，因为分支可能会在进入处理器的指令管道中造成低效。

#### 3.4.3 内联函数

与展开循环类似，可以在被调用函数中嵌入调用函数。程序员可以通过在函数定义中将函数指定为 `inline` 来指示编译器尝试这样做。再次强调，通过这样做，您可能会以代码大小为代价换取代码的顺序性。

#### 3.4.4 分支预测

每当计算机遇到一个 `if` 语句时，都有两种可能的结果：真或假。处理器希望尽可能保持其输入管道充满，因此它不能在将代码放入管道之前等待测试的结果。

因此，编译器可以预测测试可能的发展方向。编译器可以使用一些简单的规则来猜测这类事情，例如 `if (val == -1)` 很可能*不*为真，因为-1 通常表示错误代码，并且希望这种情况不会频繁触发。

一些编译器实际上可以编译程序，让用户运行它，并注意在真实条件下分支的走向。然后，它可以基于所看到的情况重新编译它。

</main>

<main class="calibre3">

## 4 汇编器

编译器输出的汇编代码仍然是可读的，如果您了解处理器的汇编代码细节。开发者通常会查看汇编输出，以手动检查代码是否是最优化的，或者发现编译器中的任何错误（这比人们想象的更常见，尤其是在编译器非常积极地优化时）。

汇编器是一个更机械的过程，将汇编代码转换为二进制形式。本质上，汇编器维护一个包含每个可能的指令及其二进制对应物（称为*操作码*）的大表。它将这些操作码与汇编中指定的寄存器结合起来，生成一个二进制输出文件。

这段代码被称为*目标代码*，在这个阶段，它不可执行。目标代码仅仅是特定输入源代码文件的二进制表示。良好的编程实践规定，程序员不应该“把所有的鸡蛋放在一个篮子里”，即将所有的源代码放在一个文件中。

</main>

<main class="calibre3">

## 5 链接器

在大型程序中，你通常会分离代码到多个文件中，以保持相关函数在一起。这些文件中的每一个都可以编译成目标代码：但你的最终目标是创建一个单独的可执行文件！需要有一种方法将每个目标文件组合成一个单独的可执行文件。我们称这个过程为链接。

注意，即使你的程序可以放在一个文件中，它仍然需要链接到某些系统库才能正确运行。例如，`printf`调用被保存在一个库中，必须与你的可执行文件结合才能工作。所以尽管在这种情况下你不必担心链接，但确实仍然有一个链接过程在创建你的可执行文件。

在接下来的章节中，我们将解释一些理解链接所必需的术语。

### 5.1 符号

#### 5.1.1 符号

变量和函数在源代码中都有名称，我们通过这些名称来引用它们。一种思考声明变量`int a`的方法是，你是在告诉编译器“留出一些`sizeof(int)`大小的内存，从现在开始，当我使用`a`时，它将指向这块分配的内存”。同样，一个函数会说“将这段代码存储在内存中，当我调用`function()`时，跳转到并执行这段代码”。

在这种情况下，我们将`a`和`function`称为*符号*，因为它们是内存区域的符号表示。

符号帮助人类理解编程。你可以这样说，编译过程的主要任务就是移除符号——处理器并不知道`a`代表什么，它只知道在某个特定的内存地址有一些数据。编译过程需要将`a += 2`转换为类似于“将内存地址`0xABCDE`中的值增加 2”的操作。

#### 5.1.2 符号可见性

在某些 C 程序中，你可能见过`static`和`extern`这些术语与变量一起使用。这些修饰符可以影响我们所说的符号的可见性。

假设你将程序分成两个文件，但某些函数需要共享一个变量。你只希望有一个共享变量的*定义*（即内存位置），但两个文件都需要引用它。

为了实现这一点，我们在一个文件中声明变量，然后在另一个文件中声明一个具有相同名称但前缀为`extern`的变量。`extern`代表*外部*，对人类来说意味着这个变量是在其他地方声明的。

`extern` 对编译器说的话是，它不应该为这个变量在内存中分配任何空间，并将这个符号留在对象代码中，稍后将其修复。编译器不可能知道符号实际定义在哪里，但 *链接器* 知道，因为它的任务是查看所有对象文件，并将它们组合成一个单独的可执行文件。所以链接器会看到第二个文件中留下的符号，并说“我在文件 1 中见过这个符号，我知道它指的是内存位置 `0x12345`”。因此，它可以修改符号值，使其成为第一个文件中变量的内存值。

`static` 几乎是 `extern` 的对立面。它对它修改的符号的可见性施加限制。如果你用 `static` 声明一个变量，这意味着告诉编译器“不要在对象代码中为这个变量留下任何符号”。这意味着当链接器链接对象文件时，它永远不会看到那个符号（因此不能做出“我之前见过这个！”的连接）。`static` 对分离和减少冲突很有用--通过声明一个 `static` 变量，你可以在其他文件中重用变量名，而不会出现符号冲突。我们说我们正在 *限制符号的可见性*，因为我们不允许链接器看到它。这与更可见的符号（未用 `static` 声明的符号）形成对比，链接器可以看到它。

### 5.2 链接过程

因此，链接过程实际上是两个步骤；将所有对象文件组合成一个可执行文件，然后遍历每个对象文件以 *解析* 任何符号。这通常需要两次遍历；一次是读取所有符号定义并注意未解决的符号，另一次是将所有这些未解决的符号修复到正确的位置。

最终的可执行文件应该没有未解决的符号；如果有任何错误，链接器将失败。我们称之为 *静态链接*。动态链接是一个类似的概念，在可执行文件运行时完成，稍后会有所描述。

</main>

<main class="calibre3">

## 6 实际示例

我们可以一步一步地走过构建一个简单应用程序的步骤。

注意，当你输入 `gcc` 时，实际上运行的是一个驱动程序，它隐藏了大多数步骤。在正常情况下，这正是你想要的，因为要在真实系统上获得一个真正工作的可执行文件的确切命令和选项可能相当复杂，并且与架构相关。

我们将通过以下两个示例展示编译过程。这两个都是 C 源文件，一个定义了初始程序入口点的 `main()` 函数，另一个声明了一个辅助类型函数。还有一个全局变量，仅用于说明。

```cpp
 1 |#include <stdio.h> 
 |
 |/* We need a prototype so the compiler knows what types function() takes */ 
 |int function(char *input); 
 5 |
 |/* Since this is static, we can define it in both hello.c and function.c */ 
 |static int i = 100; 
 |
 |/* This is a global variable */ 
10 |int global = 10; 
 |
 |int main(void) 
 |{ 
 | /* function() should return the value of global */ 
15 | int ret = function("Hello, World!"); 
 | exit(ret); 
 |} 

```

示例 6.1 Hello World

```cpp
 1 |#include <stdio.h> 
 |
 |static int i = 100; 
 |
 5 |/* Declard as extern since defined in hello.c */ 
 |extern int global; 
 |
 |int function(char *input) 
 |{ 
10 | printf("%s\n", input); 
 | return global; 
 |} 

```

示例 6.2 函数示例

### 6.1 编译

所有编译器都有一个选项，只执行编译的第一步。通常这类似于 `-S`，输出通常会被放入与输入文件同名但扩展名为 `.s` 的文件中。

因此，我们可以通过以下示例中的 `gcc -S` 来展示第一步。

```cpp
 1 |$ gcc -S hello.c 
 |$ gcc -S function.c 
 |$ cat function.s 
 | .file   "function.c" 
 5 | .pred.safe_across_calls p1-p5,p16-p63 
 | .section        .sdata,"aw",@progbits 
 | .align 4 
 | .type   i#, @object 
 | .size   i#, 4 
10 |i: 
 | data4   100 
 | .section        .rodata 
 | .align 8 
 |.LC0: 
15 | stringz "%s\n" 
 | .text 
 | .align 16 
 | .global function# 
 | .proc function# 
20 |function: 
 | .prologue 14, 33 
 | .save ar.pfs, r34 
 | alloc r34 = ar.pfs, 1, 4, 2, 0 
 | .vframe r35 
25 | mov r35 = r12 
 | adds r12 = -16, r12 
 | mov r36 = r1 
 | .save rp, r33 
 | mov r33 = b0 
30 | .body 
 | ;; 
 | st8 [r35] = r32 
 | addl r14 = @ltoffx(.LC0), r1 
 | ;; 
35 | ld8.mov r37 = [r14], .LC0 
 | ld8 r38 = [r35] 
 | br.call.sptk.many b0 = printf# 
 | mov r1 = r36 
 | ;; 
40 | addl r15 = @ltoffx(global#), r1 
 | ;; 
 | ld8.mov r14 = [r15], global# 
 | ;; 
 | ld4 r14 = [r14] 
45 | ;; 
 | mov r8 = r14 
 | mov ar.pfs = r34 
 | mov b0 = r33 
 | .restore sp 
50 | mov r12 = r35 
 | br.ret.sptk.many b0 
 | ;; 
 | .endp function# 
 | .ident  "GCC: (GNU) 3.3.5 (Debian 1:3.3.5-11)" 

```

示例 6.1.1 编译示例

汇编过程相对复杂，难以完全描述，但你应该能够看到 `i` 被定义为 `data4`（即 4 字节或 32 位，`int` 的大小），`function` 被定义（`function:`）以及 `printf()` 的调用。

现在我们有两个汇编文件，准备将其汇编成机器代码！

### 6.2 汇编

汇编是一个相当直接的过程。汇编器通常被称为 `as`，并且以与 `gcc` 类似的方式接受参数。

```cpp
 |$ as -o function.o function.s 
 |$ as -o hello.o hello.s 
 |$ ls 
 |function.c  function.o  function.s  hello.c  hello.o  hello.s 

```

示例 6.2.1 汇编示例

组装完成后，我们得到的是*目标代码*，它已经准备好被链接成最终的可执行文件。通常，你可以通过使用带有 `-c` 选项的编译器来跳过手动使用汇编器的步骤，这将直接将输入文件转换为目标代码，并将其放入具有相同前缀但扩展名为 `.o` 的文件中。

我们不能直接检查目标代码，因为它是以二进制格式存在的（在未来的几周里，我们将学习这种二进制格式）。然而，我们可以使用一些工具来检查目标文件，例如 `readelf --symbols` 将会显示目标文件中的符号。

```cpp
 1 |$ readelf --symbols ./hello.o 
 |
 |Symbol table '.symtab' contains 15 entries: 
 | Num:    Value          Size Type    Bind   Vis      Ndx Name 
 5 | 0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
 | 1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS hello.c 
 | 2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
 | 3: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
 | 4: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
10 | 5: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
 | 6: 0000000000000000     4 OBJECT  LOCAL  DEFAULT    5 i 
 | 7: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
 | 8: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
 | 9: 0000000000000000     0 SECTION LOCAL  DEFAULT    8 
15 | 10: 0000000000000000     0 SECTION LOCAL  DEFAULT   10 
 | 11: 0000000000000004     4 OBJECT  GLOBAL DEFAULT    5 global 
 | 12: 0000000000000000    96 FUNC    GLOBAL DEFAULT    1 main 
 | 13: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND function 
 | 14: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND exit 
20 |
 |$ readelf --symbols ./function.o 
 |
 |Symbol table '.symtab' contains 14 entries: 
 | Num:    Value          Size Type    Bind   Vis      Ndx Name 
25 | 0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
 | 1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS function.c 
 | 2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
 | 3: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
 | 4: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
30 | 5: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
 | 6: 0000000000000000     4 OBJECT  LOCAL  DEFAULT    5 i 
 | 7: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
 | 8: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
 | 9: 0000000000000000     0 SECTION LOCAL  DEFAULT    8 
35 | 10: 0000000000000000     0 SECTION LOCAL  DEFAULT   10 
 | 11: 0000000000000000   128 FUNC    GLOBAL DEFAULT    1 function 
 | 12: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND printf 
 | 13: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND global 

```

示例 6.2.2 Readelf 示例

尽管输出相当复杂（又是！）但你应该能够理解其中大部分内容。例如

+   在 `hello.o` 的输出中，看看名为 `i` 的符号。注意它说它是 `LOCAL` 吗？这是因为我们将其声明为 `static`，因此它被标记为仅在此目标文件中本地。

+   在相同的输出中，注意 `global` 变量被定义为 `GLOBAL`，这意味着它在文件外部是可见的。同样，`main()` 函数也是外部可见的。

+   注意到 `function` 符号（对于 `function()` 的调用）左侧有 `UND` 或 *未定义*。这意味着它被留给了链接器来查找函数的地址。

+   看看 `function.c` 文件中的符号以及它们如何与输出匹配。

### 6.3 链接

实际上调用链接器，称为 `ld`，在真实系统中是一个非常复杂的过程（你厌倦了听到这一点吗？）。这就是为什么我们将链接过程留给 `gcc`。

当然，我们可以使用 `-v`（详细）标志来监视 `gcc` 在底层执行的操作。

```cpp
 1 | /usr/lib/gcc-lib/ia64-linux/3.3.5/collect2 -static` 
 |/usr/lib/gcc-lib/ia64-linux/3.3.5/../../../crt1.o` 
 |`/usr/lib/gcc-lib/ia64-linux/3.3.5/../../../crti.o` 
 |`/usr/lib/gcc-lib/ia64-linux/3.3.5/crtbegin.o` 
 5 |`-L/usr/lib/gcc-lib/ia64-linux/3.3.5` 
 |`-L/usr/lib/gcc-lib/ia64-linux/3.3.5/../../..` 
 |`hello.o` 
 |`function.o` 
 |`--start-group` 
10 |`-lgcc` 
 |`-lgcc_eh` 
 |`-lunwind` 
 |`-lc` 
 |`--end-group` 
15 |`/usr/lib/gcc-lib/ia64-linux/3.3.5/crtend.o` 
 |`/usr/lib/gcc-lib/ia64-linux/3.3.5/../../../crtn.o 

```

示例 6.3.1 链接示例

你首先注意到被调用的程序是 collect2。这是一个简单的 `ld` 包装器，由 `gcc` 内部使用。

下一个你注意到的是，指定给链接器的 `crt` 开头的目标文件。这些函数由 gcc 和系统库提供，包含启动程序所需的代码。实际上，当程序运行时，`main()` 函数并不是第一个被调用的，而是 `crt` 目标文件中的 `_start` 函数。这个函数执行一些通用的设置，应用程序员不需要担心。

路径层次结构相当复杂，但本质上我们可以看到最终步骤是链接一些额外的目标文件，即

+   `crt1.o` : 由系统库（libc）提供，此目标文件包含 `_start` 函数，实际上这是程序中首先被调用的东西。

    `crti.o` : 由系统库提供

    `crtbegin.o`

    `crtsaveres.o`

    `crtend.o`

    `crtn.o`

我们稍后会讨论这些是如何被用来启动程序的。

接下来，你可以看到我们链接了两个目标文件，`hello.o` 和 `function.o`。之后，我们使用 `-l` 标志指定了一些额外的库。这些库是系统特定的，并且对于每个程序都是必需的。主要的一个是 `-lc`，它引入了 C 库，其中包含所有常见的函数，如 `printf()`。

之后，我们再次链接一些更多的系统目标文件，这些文件在程序退出后进行清理。

虽然细节很复杂，但概念很简单。所有目标文件都将链接成一个单独的可执行文件，准备运行！

### 6.4 可执行文件

我们将在不久的将来更详细地介绍可执行文件，但我们可以以类似对象文件的方式进行检查，看看发生了什么。

```cpp
 1 |`ianw@lime:~/programs/csbu/wk7/code$ gcc -o program hello.c function.c 
 |ianw@lime:~/programs/csbu/wk7/code$ readelf --symbols ./program 
 |
 |Symbol table '.dynsym' contains 11 entries: 
 5 | Num:    Value          Size Type    Bind   Vis      Ndx Name 
 | 0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
 | 1: 6000000000000de0     0 OBJECT  GLOBAL DEFAULT  ABS _DYNAMIC 
 | 2: 0000000000000000   176 FUNC    GLOBAL DEFAULT  UND printf@GLIBC_2.2 (2) 
 | 3: 600000000000109c     0 NOTYPE  GLOBAL DEFAULT  ABS __bss_start 
 10 | 4: 0000000000000000   704 FUNC    GLOBAL DEFAULT  UND exit@GLIBC_2.2 (2) 
 | 5: 600000000000109c     0 NOTYPE  GLOBAL DEFAULT  ABS _edata 
 | 6: 6000000000000fe8     0 OBJECT  GLOBAL DEFAULT  ABS _GLOBAL_OFFSET_TABLE_     7: 60000000000010b0     0 NOTYPE  GLOBAL DEFAULT  ABS _end 
 | 8: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _Jv_RegisterClasses 
 | 9: 0000000000000000   544 FUNC    GLOBAL DEFAULT  UND __libc_start_main@GLIBC_2.2 (2) 
 15 | 10: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__ 
 |
 |Symbol table '.symtab' contains 127 entries: 
 | Num:    Value          Size Type    Bind   Vis      Ndx Name 
 | 0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
 20 | 1: 40000000000001c8     0 SECTION LOCAL  DEFAULT    1 
 | 2: 40000000000001e0     0 SECTION LOCAL  DEFAULT    2 
 | 3: 4000000000000200     0 SECTION LOCAL  DEFAULT    3 
 | 4: 4000000000000240     0 SECTION LOCAL  DEFAULT    4 
 | 5: 4000000000000348     0 SECTION LOCAL  DEFAULT    5 
 25 | 6: 40000000000003d8     0 SECTION LOCAL  DEFAULT    6 
 | 7: 40000000000003f0     0 SECTION LOCAL  DEFAULT    7 
 | 8: 4000000000000410     0 SECTION LOCAL  DEFAULT    8 
 | 9: 4000000000000440     0 SECTION LOCAL  DEFAULT    9 
 | 10: 40000000000004a0     0 SECTION LOCAL  DEFAULT   10 
 30 | 11: 40000000000004e0     0 SECTION LOCAL  DEFAULT   11 
 | 12: 40000000000005e0     0 SECTION LOCAL  DEFAULT   12 
 | 13: 4000000000000b00     0 SECTION LOCAL  DEFAULT   13 
 | 14: 4000000000000b40     0 SECTION LOCAL  DEFAULT   14 
 | 15: 4000000000000b60     0 SECTION LOCAL  DEFAULT   15 
 35 | 16: 4000000000000bd0     0 SECTION LOCAL  DEFAULT   16 
 | 17: 4000000000000ce0     0 SECTION LOCAL  DEFAULT   17 
 | 18: 6000000000000db8     0 SECTION LOCAL  DEFAULT   18 
 | 19: 6000000000000dd0     0 SECTION LOCAL  DEFAULT   19 
 | 20: 6000000000000dd8     0 SECTION LOCAL  DEFAULT   20 
 40 | 21: 6000000000000de0     0 SECTION LOCAL  DEFAULT   21 
 | 22: 6000000000000fc0     0 SECTION LOCAL  DEFAULT   22 
 | 23: 6000000000000fd0     0 SECTION LOCAL  DEFAULT   23 
 | 24: 6000000000000fe0     0 SECTION LOCAL  DEFAULT   24 
 | 25: 6000000000000fe8     0 SECTION LOCAL  DEFAULT   25 
 45 | 26: 6000000000001040     0 SECTION LOCAL  DEFAULT   26 
 | 27: 6000000000001080     0 SECTION LOCAL  DEFAULT   27 
 | 28: 60000000000010a0     0 SECTION LOCAL  DEFAULT   28 
 | 29: 60000000000010a8     0 SECTION LOCAL  DEFAULT   29 
 | 30: 0000000000000000     0 SECTION LOCAL  DEFAULT   30 
 50 | 31: 0000000000000000     0 SECTION LOCAL  DEFAULT   31 
 | 32: 0000000000000000     0 SECTION LOCAL  DEFAULT   32 
 | 33: 0000000000000000     0 SECTION LOCAL  DEFAULT   33 
 | 34: 0000000000000000     0 SECTION LOCAL  DEFAULT   34 
 | 35: 0000000000000000     0 SECTION LOCAL  DEFAULT   35 
 55 | 36: 0000000000000000     0 SECTION LOCAL  DEFAULT   36 
 | 37: 0000000000000000     0 SECTION LOCAL  DEFAULT   37 
 | 38: 0000000000000000     0 SECTION LOCAL  DEFAULT   38 
 | 39: 0000000000000000     0 SECTION LOCAL  DEFAULT   39 
 | 40: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 60 | 41: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 42: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 43: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 44: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 45: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 65 | 46: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <command line> 
 | 47: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 48: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <command line> 
 | 49: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <built-in> 
 | 50: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS abi-note.S 
 70 | 51: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 52: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS abi-note.S 
 | 53: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 54: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS abi-note.S 
 | 55: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <command line> 
 75 | 56: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 57: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <command line> 
 | 58: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <built-in> 
 | 59: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS abi-note.S 
 | 60: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS init.c 
 80 | 61: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 62: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 63: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS initfini.c 
 | 64: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 65: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <command line> 
 85 | 66: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 67: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <command line> 
 | 68: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <built-in> 
 | 69: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 70: 4000000000000670   128 FUNC    LOCAL  DEFAULT   12 gmon_initializer 
 90 | 71: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 72: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 73: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS initfini.c 
 | 74: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 75: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <command line> 
 95 | 76: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 77: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <command line> 
 | 78: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <built-in> 
 | 79: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /build/buildd/glibc-2.3.2 
 | 80: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS auto-host.h 
100 | 81: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <command line> 
 | 82: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <built-in> 
 | 83: 6000000000000fc0     0 NOTYPE  LOCAL  DEFAULT   22 __CTOR_LIST__ 
 | 84: 6000000000000fd0     0 NOTYPE  LOCAL  DEFAULT   23 __DTOR_LIST__ 
 | 85: 6000000000000fe0     0 NOTYPE  LOCAL  DEFAULT   24 __JCR_LIST__ 
105 | 86: 6000000000001088     8 OBJECT  LOCAL  DEFAULT   27 dtor_ptr 
 | 87: 40000000000006f0   128 FUNC    LOCAL  DEFAULT   12 __do_global_dtors_aux` 
 | `88: 4000000000000770   128 FUNC    LOCAL  DEFAULT   12 __do_jv_register_classes 
 | 89: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS hello.c 
 | 90: 6000000000001090     4 OBJECT  LOCAL  DEFAULT   27 i 
110 | 91: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS function.c 
 | 92: 6000000000001098     4 OBJECT  LOCAL  DEFAULT   27 i 
 | 93: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS auto-host.h 
 | 94: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <command line> 
 | 95: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS <built-in> 
115 | 96: 6000000000000fc8     0 NOTYPE  LOCAL  DEFAULT   22 __CTOR_END__ 
 | 97: 6000000000000fd8     0 NOTYPE  LOCAL  DEFAULT   23 __DTOR_END__ 
 | 98: 6000000000000fe0     0 NOTYPE  LOCAL  DEFAULT   24 __JCR_END__ 
 | 99: 6000000000000de0     0 OBJECT  GLOBAL DEFAULT  ABS _DYNAMIC 
 | 100: 4000000000000a70   144 FUNC    GLOBAL HIDDEN   12 __do_global_ctors_aux 
120 | 101: 6000000000000dd8     0 NOTYPE  GLOBAL DEFAULT  ABS __fini_array_end 
 | 102: 60000000000010a8     8 OBJECT  GLOBAL HIDDEN   29 __dso_handle 
 | 103: 40000000000009a0   208 FUNC    GLOBAL DEFAULT   12 __libc_csu_fini 
 | 104: 0000000000000000   176 FUNC    GLOBAL DEFAULT  UND printf@@GLIBC_2.2 
 | 105: 40000000000004a0    32 FUNC    GLOBAL DEFAULT   10 _init 
125 | 106: 4000000000000850   128 FUNC    GLOBAL DEFAULT   12 function 
 | 107: 40000000000005e0   144 FUNC    GLOBAL DEFAULT   12 _start 
 | 108: 6000000000001094     4 OBJECT  GLOBAL DEFAULT   27 global 
 | 109: 6000000000000dd0     0 NOTYPE  GLOBAL DEFAULT  ABS __fini_array_start 
 | 110: 40000000000008d0   208 FUNC    GLOBAL DEFAULT   12 __libc_csu_init 
130 | 111: 600000000000109c     0 NOTYPE  GLOBAL DEFAULT  ABS __bss_start 
 | 112: 40000000000007f0    96 FUNC    GLOBAL DEFAULT   12 main 
 | 113: 6000000000000dd0     0 NOTYPE  GLOBAL DEFAULT  ABS __init_array_end 
 | 114: 6000000000000dd8     0 NOTYPE  WEAK   DEFAULT   20 data_start 
 | 115: 4000000000000b00    32 FUNC    GLOBAL DEFAULT   13 _fini 
135 | 116: 0000000000000000   704 FUNC    GLOBAL DEFAULT  UND exit@@GLIBC_2.2 
 | 117: 600000000000109c     0 NOTYPE  GLOBAL DEFAULT  ABS _edata 
 | 118: 6000000000000fe8     0 OBJECT  GLOBAL DEFAULT  ABS _GLOBAL_OFFSET_TABLE_` 
 | `119: 60000000000010b0     0 NOTYPE  GLOBAL DEFAULT  ABS _end 
 | 120: 6000000000000db8     0 NOTYPE  GLOBAL DEFAULT  ABS __init_array_start 
140 | 121: 6000000000001080     4 OBJECT  GLOBAL DEFAULT   27 _IO_stdin_used 
 | 122: 60000000000010a0     8 OBJECT  GLOBAL DEFAULT   28 __libc_ia64_register_back 
 | 123: 6000000000000dd8     0 NOTYPE  GLOBAL DEFAULT   20 __data_start 
 | 124: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _Jv_RegisterClasses 
 | 125: 0000000000000000   544 FUNC    GLOBAL DEFAULT  UND __libc_start_main@@GLIBC_ 
145 | 126: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__ 

```

示例 6.4.1 可执行示例

一些需要注意的事项

+   注意我是通过“简单”的方式构建可执行文件的！

+   注意到存在两个符号表；`dynsym` 和 `symtab`。我们很快会解释 `dynsym` 符号的工作原理，但请注意，其中一些通过 `@` 符号进行了版本控制。

+   注意从额外目标文件中包含的许多符号。许多符号以 `__` 开头，以避免与程序员可能选择的任何名称冲突。阅读并通过目标文件挑选出我们之前提到的符号，看看它们是否有所变化。

</main>
