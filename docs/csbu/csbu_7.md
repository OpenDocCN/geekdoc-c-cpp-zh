

## 第八章. 过程背后的内容





## 1 可执行文件回顾

我们知道在内存中运行的程序有两个主要组件，*代码*（也常因历史原因被称为*文本*）和*数据*。然而，我们也知道，可执行文件并不生活在内存中，而是在磁盘上作为文件度过大部分生命，等待被加载和运行。由于文件本质上只是一个连续的位数组，所有系统都找到了在文件中组织代码和数据的办法，以便按需执行。这种文件格式通常被称为*二进制*或*可执行*。文件中的位和字节通常以可以直接放置在内存中并由处理器硬件直接解释的格式存在。





## 2 可执行文件的表示

### 2.1 三个标准部分

至少，任何可执行文件格式都需要指定代码和数据在二进制文件中的位置。这些是可执行文件中的两个主要部分。

我们之前没有提到的另一个组件是未初始化的全局变量的存储空间。如果我们声明一个变量并给它一个初始值，这个值需要存储在可执行文件中，以便在程序启动时可以将其初始化为正确的值。然而，许多变量在程序首次执行时是未初始化的（或为零）。在可执行文件中为这些变量预留空间，然后简单地存储零或 NULL 值是浪费空间，无谓地膨胀了磁盘上的可执行文件大小。因此，大多数二进制格式定义了`BSS`部分的概念，作为零初始化数据的占位符大小。在程序加载时，BSS 描述的额外内存可以被分配（并设置为零！）BSS*可能*代表“由符号开始的块”，这是旧 IBM 计算机的汇编命令；确切的起源可能已经失传于历史。

### 2.2 二进制格式

可执行文件是由工具链从源代码创建的。这个文件需要以明确定义的形式存在，以便编译器可以创建它，操作系统可以识别它并将其加载到内存中，将其转换为操作系统可以管理的运行进程。这种*可执行文件格式*可以特定于操作系统，因为我们通常不会期望为某个系统编译的程序可以在另一个系统上执行（例如，你不希望你的 Windows 程序在 Linux 上运行，或者你的 Linux 程序在 OS X 上运行）。

然而，所有可执行文件格式的共同点是它们都包含一个预定义、标准化的头文件，该头文件描述了程序代码和数据如何存储在文件的其余部分。用词来说，它通常会描述“程序代码从文件中的 20 个字节开始，长度为 50 千字节。程序数据随后，长度为 20 千字节”。

近年来，一种特定的格式已成为现代 UNIX 类型系统可执行表示的既定标准。它被称为“可执行和链接格式”，简称 ELF；我们很快将更详细地探讨它。

### 2.3 二进制格式历史

#### 2.3.1 a.out

ELF 并非一直是标准；原始 UNIX 系统使用一种名为`a.out`的文件格式。如果你不使用`-o`选项来指定输出文件名编译程序，你将看到这种格式的残留；可执行文件将以默认名称`a.out`创建。实际上，`a.out`是链接器的默认输出文件名。编译器通常使用随机生成的文件名作为汇编和目标代码的中间文件。

`a.out`是一个非常简单的头格式，只允许一个数据、代码和 BSS 部分。正如你将看到的，这对于具有动态库的现代系统来说是不够的。

#### 2.3.2 COFF

常见对象文件格式，或 COFF，是 ELF 的前身。其头格式更灵活，允许文件中有更多的（但有限的）部分。

COFF 在优雅地支持共享库方面也存在困难，因此 ELF 被选为 Linux 上的替代实现。

然而，COFF 在 Microsoft Windows 中以“可移植可执行文件”或 PE 格式继续存在。PE 对于 Windows 来说，就像 ELF 对于 Linux 一样。





## 3 ELF

ELF 是一种非常灵活的格式，用于在系统中表示二进制代码。通过遵循 ELF 标准，你可以像表示普通可执行文件或系统库一样轻松地表示内核二进制文件。相同的工具可以用于检查和操作所有 ELF 文件，并且理解 ELF 文件格式的开发者可以将他们的技能应用到大多数现代 UNIX 系统。

ELF 扩展了 COFF，并为头提供了足够的灵活性，可以定义任意数量的部分，每个部分都有其自己的属性。这有助于简化动态链接和调试。

<picture>![ELF 概述](img/elf-overview.svg)</picture>图 3.1 ELF 概述

### 3.1 ELF 文件头

总体而言，文件有一个*文件头*，它描述了文件的一般信息，然后指向构成文件的各个单独部分的指针。示例 3.1.1，ELF 头显示了 API 文档中给出的 ELF32（ELF 的 32 位形式）的描述。这是定义 ELF 头的 C 结构的布局。

```cpp
 1 |typedef struct { 
 | unsigned char e_ident[EI_NIDENT]; 
 | Elf32_Half    e_type; 
 | Elf32_Half    e_machine; 
 5 | Elf32_Word    e_version; 
 | Elf32_Addr    e_entry; 
 | Elf32_Off     e_phoff; 
 | Elf32_Off     e_shoff; 
 | Elf32_Word    e_flags; 
10 | Elf32_Half    e_ehsize; 
 | Elf32_Half    e_phentsize; 
 | Elf32_Half    e_phnum; 
 | Elf32_Half    e_shentsize; 
 | Elf32_Half    e_shnum; 
15 | Elf32_Half    e_shstrndx; 
 |} Elf32_Ehdr; 

```

示例 3.1.1 ELF 头

```cpp
 1 |$ readelf --header /bin/ls 
 |
 |ELF Header: 
 | Magic:   7f 45 4c 46 01 02 01 00 00 00 00 00 00 00 00 00` 
 5 | `Class:                             ELF32 
 | Data:                              2's complement, big endian 
 | Version:                           1 (current) 
 | OS/ABI:                            UNIX - System V 
 | ABI Version:                       0 
10 | Type:                              EXEC (Executable file) 
 | Machine:                           PowerPC 
 | Version:                           0x1 
 | Entry point address:               0x10002640 
 | Start of program headers:          52 (bytes into file) 
15 | Start of section headers:          87460 (bytes into file) 
 | Flags:                             0x0 
 | Size of this header:               52 (bytes) 
 | Size of program headers:           32 (bytes) 
 | Number of program headers:         8 
20 | Size of section headers:           40 (bytes) 
 | Number of section headers:         29 
 | Section header string table index: 28 
 |
 | [...] 

```

示例 3.1.2 readelf 显示的 ELF 头

示例 3.1.2，readelf 显示的 ELF 头显示了 readelf 程序显示的易读形式，该程序是 GNU binutils 的一部分。

`e_ident`数组是任何 ELF 文件开始处的第一件事，并且始终以几个“魔数”字节开始。第一个字节是 0x7F，然后接下来的三个字节是“ELF”。你可以使用类似`hexdump`命令的工具检查 ELF 二进制文件以查看这一点。

```cpp
 |ianw@mingus:~$ hexdump -C /bin/ls | more 
 |00000000  7f 45 4c 46 01 02 01 00  00 00 00 00 00 00 00 00  |.ELF............| 
 |
 |... (rest of the program follows) ... 

```

示例 3.1.3 检查 ELF 魔数

注意以 0x7F 开头，然后是 ASCII 编码的“ELF”字符串。查看标准，看看数组中定义了什么，以及二进制中的值是什么。

接下来，我们有针对为该二进制创建的机器类型的标志。我们可以看到的第一件事是，ELF 定义了不同大小的版本，一个用于 32 位版本，一个用于 64 位版本；这里我们检查 32 位版本。区别主要在于在 64 位机器上，地址显然需要存储在 64 位变量中。我们可以看到，该二进制是为一个大端机器创建的，该机器使用二进制补码表示负数。向下跳一点，我们可以看到`Machine`告诉我们这是一个 PowerPC 二进制。

表面上看似无害的入口点地址似乎足够直接；这是程序代码在内存中开始的位置。初学者程序员被告知`main()`是程序中第一个被调用的程序。使用入口点地址，我们实际上可以验证它*并不是*。

```cpp
 1 |$ cat test.c 
 |#include <stdio.h> 
 |
 |int main(void) 
 5 |{ 
 | printf("main is : %p\n", &main); 
 | return 0; 
 |} 
 |
10 |$ gcc -Wall -o test test.c 
 |
 |$ ./test 
 |main is : 0x10000430 
 |
15 |$ readelf --headers ./test | grep 'Entry point' 
 | Entry point address:               0x100002b0 
 |
 |$ objdump --disassemble ./test | grep 100002b0 
 |100002b0 <_start>: 
20 |100002b0:       7c 29 0b 78     mr      r9,r1 

```

示例 3.1.4 调查入口点

在示例 3.1.4，调查入口点中，我们可以看到入口点实际上是一个名为`_start`的函数。我们的程序根本未定义此函数，并且前导下划线表明它位于一个单独的*命名空间*中。我们详细探讨了程序如何启动，请参阅第 8.2 节，启动程序。

之后，头文件包含指向文件中其他重要部分的位置的指针，如目录表。

### 3.2 符号和重定位

ELF 规范提供了*符号表*，它只是将字符串（符号）映射到文件中的位置。符号对于链接是必需的；例如，将声明为`extern int foo;`的变量`foo`分配一个值，需要链接器找到`foo`的地址，这将涉及在符号表中查找`foo`并找到地址。

与符号密切相关的是*重定位*。重定位简单地说就是留下一个空白空间，稍后进行修补。在先前的例子中，直到`foo`的地址已知，它才能被使用。然而，在 32 位系统上，我们知道`foo`的*地址*必须是一个 4 字节值，因此每当编译器需要使用该地址（例如，分配一个值）时，它只需留下 4 字节的空白空间，并保留一个重定位，这个重定位本质上是对链接器的指示：“将`foo`的实际值放入此地址的 4 个字节中”。如前所述，这需要解析符号`foo`。第 2.1 节，重定位包含有关重定位的更多信息。

### 3.3 节和段

ELF 格式指定了 ELF 文件的两种“视图”——用于链接的视图和用于执行的视图。这为系统设计者提供了显著的灵活性。

我们在对象代码中讨论*节*，这些节等待被链接到可执行文件中。一个或多个节映射到可执行文件中的*段*。

#### 3.3.1 段

正如我们之前所做的那样，在检查底层之前，有时查看更高层次的抽象（段）会更简单。

正如我们所提到的，ELF 文件有一个描述文件整体布局的头。ELF 头实际上指向另一组称为*程序头*的头。这些头描述了操作系统可能需要加载二进制文件到内存并执行的所有内容。段由程序头描述，但还有一些其他东西需要执行才能使可执行文件运行。

```cpp
 1 |typedef struct { 
 | Elf32_Word p_type; 
 | Elf32_Off  p_offset; 
 | Elf32_Addr p_vaddr; 
 5 | Elf32_Addr p_paddr; 
 | Elf32_Word p_filesz; 
 | Elf32_Word p_memsz; 
 | Elf32_Word p_flags; 
 | Elf32_Word p_align; 
10 |} 

```

示例 3.3.1.1 程序头

程序头的定义可以在示例 3.3.1.1，程序头中看到。你可能已经注意到，在上面的 ELF 头定义中，有字段`e_phoff`、`e_phnum`和`e_phentsize`；这些字段简单地表示程序头在文件中的偏移量，程序头的数量以及每个程序头的大小。有了这三块信息，你可以轻松地找到并读取程序头。

程序头不仅仅是段。`p_type`字段定义了程序头具体定义的内容。例如，如果这个字段是`PT_INTERP`，则该头定义为一个指向二进制文件*解释器*的字符串指针。我们之前讨论了编译语言与解释语言的区别，并指出编译器构建的二进制文件可以独立运行。为什么还需要解释器呢？正如往常一样，真实情况要复杂一些。现代系统在加载可执行文件时希望具有灵活性，为此，一些信息只能在程序实际设置运行时才能充分获取。我们将在后续章节中看到这一点，其中我们将探讨动态链接。因此，可能需要对二进制文件进行一些小的修改，以便它在运行时能够正常工作。因此，二进制文件的通常解释器是*动态加载器*，之所以称为动态加载器，是因为它完成加载可执行文件的最后一步，并为运行准备二进制映像。

段落由程序头中的 `p_type` 字段中的 `PT_LOAD` 值来描述。然后，每个段由程序头中的其他字段进行描述。`p_offset` 字段告诉您段的数据在磁盘文件中的位置。`p_vaddr` 字段告诉您数据在虚拟内存中的地址（`p_addr` 描述的是物理地址，这对于不实现虚拟内存的小型嵌入式系统来说非常有用）。两个标志 `p_filesz` 和 `p_memsz` 用来告诉您段在磁盘上的大小以及它在内存中应该有多大。如果内存大小大于磁盘大小，则重叠部分应填充零。这样，您可以通过不浪费空间为空全局变量腾出空间，从而在二进制文件中节省相当大的空间。最后，`p_flags` 指示段上的权限。执行、读取和写入权限可以以任何组合指定；例如，代码段应标记为只读和执行，数据段为可读可写且不可执行。

程序头中定义了少数几种其他段类型，它们在标准规范中有更详细的描述。

#### 3.3.2 部分

正如我们提到的，部分构成了段。部分是一种将二进制文件组织成逻辑区域的方法，以便在编译器和链接器之间传递信息。在某些特殊的二进制文件中，例如 Linux 内核，部分被用于更具体的方式（参见 部分 6.2，自定义部分）。

我们已经看到，段最终归结为磁盘文件中的一个数据块，其中包含有关其加载位置和权限的一些描述。部分与段有类似的头信息，如 示例 3.3.2.1，部分 所示。

```cpp
 1 |typedef struct { 
 | Elf32_Word sh_name; 
 | Elf32_Word sh_type; 
 | Elf32_Word sh_flags; 
 5 | Elf32_Addr sh_addr; 
 | Elf32_Off  sh_offset; 
 | Elf32_Word sh_size; 
 | Elf32_Word sh_link; 
 | Elf32_Word sh_info; 
10 | Elf32_Word sh_addralign; 
 | Elf32_Word sh_entsize; 
 |} 

```

示例 3.3.2.1 部分

部分在 `sh_type` 字段中定义了更多类型；例如，类型为 `SH_PROGBITS` 的部分被定义为包含程序使用的二进制数据的部分。其他标志表示此部分是否为符号表（例如，由链接器或调试器使用）或可能是用于动态加载器的某些内容。还有更多属性，例如 *allocate* 属性，表示此部分将需要为其分配内存。

在下面，我们将检查 示例 3.3.2.2，部分 中列出的程序。

```cpp
 1 |#include <stdio.h> 
 |
 |int big_big_array[10*1024*1024]; 
 |
 5 |char *a_string = "Hello, World!"; 
 |
 |int a_var_with_value = 0x100; 
 |
 |int main(void) 
10 |{ 
 | big_big_array[0] = 100; 
 | printf("%s\n", a_string); 
 | a_var_with_value += 20; 
 |} 

```

示例 3.3.2.2 部分

示例 3.3.2.3，readelf 输出部分 展示了 readelf 的输出，其中一些部分已被剥离以提高清晰度。使用此输出，我们可以分析我们简单程序中的每个部分，并查看它在最终输出二进制文件中的位置。

```cpp
 1 |$ readelf --all ./sections 
 |ELF Header: 
 | ... 
 | Size of section headers:           40 (bytes) 
 5 | Number of section headers:         37 
 | Section header string table index: 34 
 |
 |Section Headers: 
 | [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al 
10 | [ 0]                   NULL            00000000 000000 000000 00      0   0  0 
 | [ 1] .interp           PROGBITS        10000114 000114 00000d 00   A  0   0  1 
 | [ 2] .note.ABI-tag     NOTE            10000124 000124 000020 00   A  0   0  4 
 | [ 3] .hash             HASH            10000144 000144 00002c 04   A  4   0  4 
 | [ 4] .dynsym           DYNSYM          10000170 000170 000060 10   A  5   1  4 
15 | [ 5] .dynstr           STRTAB          100001d0 0001d0 00005e 00   A  0   0  1 
 | [ 6] .gnu.version      VERSYM          1000022e 00022e 00000c 02   A  4   0  2 
 | [ 7] .gnu.version_r    VERNEED         1000023c 00023c 000020 00   A  5   1  4 
 | [ 8] .rela.dyn         RELA            1000025c 00025c 00000c 0c   A  4   0  4 
 | [ 9] .rela.plt         RELA            10000268 000268 000018 0c   A  4  25  4 
20 | [10] .init             PROGBITS        10000280 000280 000028 00  AX  0   0  4 
 | [11] .text             PROGBITS        100002b0 0002b0 000560 00  AX  0   0 16 
 | [12] .fini             PROGBITS        10000810 000810 000020 00  AX  0   0  4 
 | [13] .rodata           PROGBITS        10000830 000830 000024 00   A  0   0  4 
 | [14] .sdata2           PROGBITS        10000854 000854 000000 00   A  0   0  4 
25 | [15] .eh_frame         PROGBITS        10000854 000854 000004 00   A  0   0  4 
 | [16] .ctors            PROGBITS        10010858 000858 000008 00  WA  0   0  4 
 | [17] .dtors            PROGBITS        10010860 000860 000008 00  WA  0   0  4 
 | [18] .jcr              PROGBITS        10010868 000868 000004 00  WA  0   0  4 
 | [19] .got2             PROGBITS        1001086c 00086c 000010 00  WA  0   0  1 
30 | [20] .dynamic          DYNAMIC         1001087c 00087c 0000c8 08  WA  5   0  4 
 | [21] .data             PROGBITS        10010944 000944 000008 00  WA  0   0  4 
 | [22] .got              PROGBITS        1001094c 00094c 000014 04 WAX  0   0  4 
 | [23] .sdata            PROGBITS        10010960 000960 000008 00  WA  0   0  4 
 | [24] .sbss             NOBITS          10010968 000968 000000 00  WA  0   0  1 
35 | [25] .plt              NOBITS          10010968 000968 000060 00 WAX  0   0  4 
 | [26] .bss              NOBITS          100109c8 000968 2800004 00  WA  0   0  4 
 | [27] .comment          PROGBITS        00000000 000968 00018f 00      0   0  1 
 | [28] .debug_aranges    PROGBITS        00000000 000af8 000078 00      0   0  8 
 | [29] .debug_pubnames   PROGBITS        00000000 000b70 000025 00      0   0  1 
40 | [30] .debug_info       PROGBITS        00000000 000b95 0002e5 00      0   0  1 
 | [31] .debug_abbrev     PROGBITS        00000000 000e7a 000076 00      0   0  1 
 | [32] .debug_line       PROGBITS        00000000 000ef0 0001de 00      0   0  1 
 | [33] .debug_str        PROGBITS        00000000 0010ce 0000f0 01  MS  0   0  1 
 | [34] .shstrtab         STRTAB          00000000 0011be 00013b 00      0   0  1 
45 | [35] .symtab           SYMTAB          00000000 0018c4 000c90 10     36  65  4 
 | [36] .strtab           STRTAB          00000000 002554 000909 00      0   0  1 
 |Key to Flags: 
 | W (write), A (alloc), X (execute), M (merge), S (strings) 
 | I (info), L (link order), G (group), x (unknown) 
50 | O (extra OS processing required) o (OS specific), p (processor specific) 
 |
 |There are no section groups in this file. 
 | ... 
 |
55 |Symbol table '.symtab' contains 201 entries: 
 | Num:    Value  Size Type    Bind   Vis      Ndx Name 
 |... 
 | 99: 100109cc 0x2800000 OBJECT  GLOBAL DEFAULT   26 big_big_array 
 |... 
60 | 110: 10010960     4 OBJECT  GLOBAL DEFAULT   23 a_string 
 |... 
 | 130: 10010964     4 OBJECT  GLOBAL DEFAULT   23 a_var_with_value 
 |... 
 | 144: 10000430    96 FUNC    GLOBAL DEFAULT   11 main 

```

示例 3.3.2.3 部分 readelf 输出

首先，让我们看看变量 `big_big_array`，正如其名称所暗示的那样，这是一个相当大的全局数组。如果我们跳到符号表，我们可以看到该变量位于地址 `0x100109cc`，我们可以将其与段列表中的 `.bss` 段相关联，因为它正好位于其下方 `0x100109c8`。注意其大小，以及它相当大。我们提到 BSS 是二进制图像的标准部分，因为要求磁盘上的二进制文件分配 10 兆字节的空间是愚蠢的，因为所有这些空间都将为零。注意这个部分有一个类型为 `NOBITS`，这意味着它没有在磁盘上的任何字节。

因此，`.bss` 段是为全局变量定义的，这些变量的值在程序开始时应为零。我们在讨论段时已经看到内存大小可以与磁盘大小不同；变量位于 `.bss` 段是它们将在程序开始时赋予零值的指示。

变量 `a_string` 位于 `.sdata` 段，代表 *小数据*。小数据（以及相应的 `.sbss` 段）是在某些架构上可用的段，其中数据可以通过从某个已知指针的偏移量访问。这意味着可以将一个固定值添加到基址，这使得访问这些段中的数据更快，因为没有额外的查找和将地址加载到内存中所需的加载。大多数架构限制于可以添加到寄存器的立即值的大小（例如，如果执行指令 `r1 = add r2, 70;`，70 是一个 *立即值*，而不是说，将存储在寄存器中的两个值相加 `r1 = add r2,r3`）因此只能从地址偏移一定的“小”距离。我们还可以看到我们的 `a_var_with_value` 也位于同一位置。

然而，`main` 位于 `.text` 段，正如我们所期望的（记住“text”和“code”这两个名称可以互换用来指代内存中的程序）。

#### 3.3.3 段和段一起

```cpp
 1 |$ readelf --segments /bin/ls 
 |
 |Elf file type is EXEC (Executable file) 
 |Entry point 0x100026c0 
 5 |There are 8 program headers, starting at offset 52 
 |
 |Program Headers: 
 | Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align 
 | PHDR           0x000034 0x10000034 0x10000034 0x00100 0x00100 R E 0x4 
10 | INTERP         0x000154 0x10000154 0x10000154 0x0000d 0x0000d R   0x1 
 | [Requesting program interpreter: /lib/ld.so.1] 
 | LOAD           0x000000 0x10000000 0x10000000 0x14d5c 0x14d5c R E 0x10000 
 | LOAD           0x014d60 0x10024d60 0x10024d60 0x002b0 0x00b7c RWE 0x10000 
 | DYNAMIC        0x014f00 0x10024f00 0x10024f00 0x000d8 0x000d8 RW  0x4 
15 | NOTE           0x000164 0x10000164 0x10000164 0x00020 0x00020 R   0x4 
 | GNU_EH_FRAME   0x014d30 0x10014d30 0x10014d30 0x0002c 0x0002c R   0x4 
 | GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RWE 0x4 
 |
 | Section to Segment mapping: 
20 | Segment Sections... 
 | 00 
 | 01     .interp 
 | 02     .interp .note.ABI-tag .hash .dynsym .dynstr .gnu.version .gnu.version_ r .rela.dyn .rela.plt .init .text .fini .rodata .eh_frame_hdr 
 | 03     .data .eh_frame .got2 .dynamic .ctors .dtors .jcr .got .sdata .sbss .p lt .bss 
25 | 04     .dynamic 
 | 05     .note.ABI-tag 
 | 06     .eh_frame_hdr 
 | 07 

```

示例 3.3.3.1 段和段

示例 3.3.3.1，段和段 展示了 `readelf` 如何显示二进制文件 `/bin/ls` 中的段和段映射。

跳到输出的底部，我们可以看到哪些段被移动到了哪些段。例如，`.interp` 段被放置在带有 `INTERP` 标志的段中。注意 readelf 告诉我们它正在请求解释器 `/lib/ld.so.1`；这是运行以准备二进制文件执行动态链接器。

查看两个 `LOAD` 段，我们可以看到文本和数据之间的区别。注意第一个只有一个“读取”和“执行”权限，而下一个有读取、写入和执行权限？这些描述了代码（r/w）和数据（r/w/e）段。

但是数据不应该需要可执行！实际上，在大多数架构（例如，最常用的 x86）中，数据段不会被标记为具有可执行的数据段。然而，上面的示例输出是从一个 PowerPC 机器上获取的，它有一个略微不同的编程模型（ABI，见下文），要求数据段可执行。对于那些好奇的人来说，PowerPC ABI 直接在 GOT 中调用动态库中的函数的存根，而不是通过单独的 PLT 条目进行跳转。因此，处理器需要 GOT 段的执行权限，您可以看到它嵌入在数据段中。在阅读了动态链接章节后，这应该是有意义的！这就是系统程序员的日常生活，规则就是为了被打破！

另一个值得注意的有趣之处是，文件大小与代码段的内存大小相同，然而内存大小大于数据段的文件大小。这来自 BSS 部分，它包含零初始化的全局变量。





## 4 ELF 可执行文件

可执行文件当然是 ELF 格式的主要用途之一。在 *二进制* 中包含了操作系统执行代码所需的所有内容。

由于可执行文件设计为在一个具有唯一地址空间的过程中运行（参见第六章，虚拟内存），代码可以假设程序的各种部分将在内存中的何处加载。示例 4.1，可执行文件的段展示了使用 readelf 工具检查可执行文件段的示例。我们可以看到 `LOAD` 段必须放置的虚拟地址。我们还可以看到，一个段是用于代码的——它只有读取和执行权限——另一个是用于数据的，不出所料，具有读取和写入权限，但重要的是没有执行权限（没有执行权限，即使存在漏洞允许攻击者引入任意数据，支持这些页面的页面也不会标记为具有执行权限，并且大多数处理器将因此禁止在这些页面中执行代码）。

```cpp
 1 |$ readelf --segments /bin/ls 
 |
 |Elf file type is EXEC (Executable file) 
 |Entry point 0x4046d4 
 5 |There are 8 program headers, starting at offset 64 
 |
 |Program Headers: 
 | Type           Offset             VirtAddr           PhysAddr 
 | FileSiz            MemSiz              Flags  Align 
10 | PHDR           0x0000000000000040 0x0000000000400040 0x0000000000400040 
 | 0x00000000000001c0 0x00000000000001c0  R E    8 
 | INTERP         0x0000000000000200 0x0000000000400200 0x0000000000400200 
 | 0x000000000000001c 0x000000000000001c  R      1 
 | [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2] 
15 | LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000 
 | 0x0000000000019ef4 0x0000000000019ef4  R E    200000 
 | LOAD           0x000000000001a000 0x000000000061a000 0x000000000061a000 
 | 0x000000000000077c 0x0000000000001500  RW     200000 
 | DYNAMIC        0x000000000001a028 0x000000000061a028 0x000000000061a028 
20 | 0x00000000000001d0 0x00000000000001d0  RW     8 
 | NOTE           0x000000000000021c 0x000000000040021c 0x000000000040021c 
 | 0x0000000000000044 0x0000000000000044  R      4 
 | GNU_EH_FRAME   0x0000000000017768 0x0000000000417768 0x0000000000417768 
 | 0x00000000000006fc 0x00000000000006fc  R      4 
25 | GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000 
 | 0x0000000000000000 0x0000000000000000  RW     8 
 |
 | Section to Segment mapping: 
 | Segment Sections... 
30 | 00` 
 | `01     .interp` 
 | `02     .interp .note.ABI-tag .note.gnu.build-id .hash .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt .text .fini .rodata .eh_frame_hdr .eh_frame` 
 | `03     .ctors .dtors .jcr .dynamic .got .got.plt .data .bss` 
 | `04     .dynamic` 
35 | `05     .note.ABI-tag .note.gnu.build-id` 
 | `06     .eh_frame_hdr` 
 | `07` 

```

示例 4.1 可执行文件的段

程序段必须在这些地址上加载；链接器的最后一步是解决大多数重定位（第 3.2 节，符号和重定位）并将它们与假定的绝对地址修补——然后最终二进制中丢弃描述重定位的数据，并且不再有找到这些信息的方法。

在现实中，可执行文件通常依赖于**共享库**的外部依赖，或者说是整个系统中抽象和共享的通用代码——示例 4.1，可执行文件的段中的大部分复杂部分都与共享库的使用有关。库在第五部分，库中讨论，动态库在第九章动态链接中讨论。





## 5 库

开发者很快就厌倦了必须从头开始编写一切，因此计算机科学中的第一个发明之一就是**库**。

库简单来说就是一组你可以从你的程序中调用的函数。显然，库有很多优点，其中最重要的可能是你可以通过重用别人已经完成的工作来节省大量时间，并且通常可以更有信心它有更少的错误（因为可能有很多其他人也使用这些库，并且你可以从他们找到和修复错误中受益）。库与可执行文件完全一样，只是库函数不是直接运行，而是通过从你的可执行文件中传递参数来调用。

### 5.1 静态库

使用库函数最直接的方法是将库的对象文件直接链接到你的最终可执行文件中，就像你自己编译的那样。当以这种方式链接时，库被称为**静态**库，因为除非程序重新编译，否则库将保持不变。

这是最直接使用库的方法，因为最终结果是一个简单的可执行文件，没有依赖项。

#### 5.1.1 静态库内部

静态库简单来说是一组对象文件。这些对象文件保存在一个**归档**中，这导致了它们通常的`.a`后缀扩展名。你可以把归档想象成类似于`zip`文件，但没有压缩。

下面我们将展示基本静态库的创建，并介绍一些用于处理库的常用工具。

```cpp
 1 |$ cat library.c 
 |`/* Library Function */ 
 |int function(int input) 
 |{ 
 5 | return input + 10; 
 |} 
 |
 |$ cat library.h 
 |/* Function Definition */ 
10 |int function(int); 
 |
 |$ cat program.c 
 |#include <stdio.h> 
 |/* Library header file */ 
15 |#include "library.h" 
 |
 |int main(void) 
 |{ 
 | int d = function(100); 
20 |
 | printf("%d\n", d); 
 |} 
 |
 |$ gcc -c library.c 
25 |$ ar rc libtest.a library.o 
 |$ ranlib ./libtest.a 
 |$ nm --print-armap ./libtest.a 
 |
 |Archive index: 
30 |function in library.o 
 |
 |library.o: 
 |00000000 T function 
 |
35 |$ gcc -L . program.c -ltest -o program 
 |
 |$ ./program 
 |110 

```

示例 5.1.1.1 创建和使用静态库

首先，我们将编译我们的库到一个对象文件，就像我们在上一章中看到的那样。

注意，我们在头文件中定义了库 API。API 由库中函数的定义组成；这样，编译器在构建引用库的对象文件（例如`program.c`，它包含了头文件）时，就知道函数的类型。

我们创建了一个名为`ar`（代表“归档”）的库命令。按照惯例，静态库文件名以`lib`为前缀，并具有`.a`扩展名。`c`参数告诉程序创建归档，而`a`参数告诉归档将指定的目标文件添加到库文件中。使用`ar`创建的归档在 Linux 系统中出现在几个不同的地方，而不仅仅是创建静态库。一个广泛使用的例子是 Debian、Ubuntu 和一些其他 Linux 系统中使用的`.deb`打包格式。`debs`使用归档将所有应用程序文件集中在一个包文件中。RedHat RPM 包使用一个类似但不同的格式，称为 cpio。当然，将文件集中起来的规范应用是`tar`文件，这是一种常见的源代码分发格式。

接下来，我们使用 ranlib 应用程序在库中创建一个包含目标文件内容符号的头文件。这有助于编译器快速引用符号；如果我们只有一个符号，这一步可能看起来有点多余；然而，大型库可能有数千个符号，这意味着索引可以显著加快查找引用的速度。我们使用 nm 应用程序检查这个新的头文件。我们看到`function()`函数的`function`符号位于偏移量零处，正如我们所期望的。

然后，您使用`-lname`指定库给编译器，其中 name 是库的文件名，不包括前缀`lib`。我们还提供了一个额外的库搜索目录，即当前目录（`-L .`），因为默认情况下，当前目录不会被搜索库。

最终结果是包含我们的新库的单个可执行文件。

#### 5.1.2 静态链接的缺点

静态链接非常直接，但存在一些缺点。

有两个主要缺点；首先，如果库代码被更新（比如修复一个错误），您必须重新编译您的程序到一个新的可执行文件中；其次，系统中使用该库的每个程序在其可执行文件中都包含一个副本。这非常低效（如果您发现一个错误并需要重新编译，这会非常痛苦，正如第一点所述）。

例如，C 库（glibc）包含在所有程序中，并提供所有常见函数，如`printf`。

### 5.2 共享库

共享库是解决静态库带来的问题的优雅方式。共享库是一个在运行时动态加载的库，每个需要它的应用程序都会加载它。

应用程序简单地留下指针，表明它将需要某个库，当函数调用发生时，该库被加载到内存中并执行。如果库已经被另一个应用程序加载，那么代码可以在两者之间共享，从而节省大量资源，特别是对于常用库。

这个过程，称为动态链接，是现代操作系统更复杂部分之一。因此，我们将在下一章中专门研究动态链接过程。





## 6 扩展 ELF 概念

### 6.1 调试

传统上，死后调试的主要方法被称为 *核心转储*。术语 *core* 来自于磁性核心存储器的原始物理特性，它使用小磁环的取向来存储状态。

因此，核心转储（core dump）仅仅是程序在特定时间运行时的完整快照。然后可以使用调试器（*debugger*）来检查这个转储并重建程序状态。示例 6.1.1，使用 gdb 创建核心转储的示例 展示了一个将数据写入随机内存位置的程序示例，以强制程序崩溃。在此点，进程将被停止，并记录当前状态。

```cpp
 1 |$ cat coredump.c 
 |int main(void) { 
 | char *foo = (char*)0x12345; 
 | *foo = 'a'; 
 5 |
 | return 0; 
 |} 
 |
 |$ gcc -Wall -g -o coredump coredump.c 
10 |
 |$ ./coredump 
 |Segmentation fault (core dumped) 
 |
 |$ file ./core 
15 |./core: ELF 32-bit LSB core file Intel 80386, version 1 (SYSV), SVR4-style, from './coredump' 
 |
 |$ gdb ./coredump 
 |... 
 |(gdb) core core 
20 |[New LWP 31614] 
 |`Core was generated by `./coredump'.` 
 |Program terminated with signal 11, Segmentation fault. 
 |#0  0x080483c4 in main () at coredump.c:3 
 |3		*foo = 'a'; 
25 |(gdb) 

```

示例 6.1.1 创建核心转储并使用 gdb 的示例

因此，核心转储（core-dump）只是另一个具有一系列调试器理解的段的 ELF 文件，这些段代表正在运行的程序的部分。

#### 6.1.1 符号和调试信息

如 示例 6.1.1，使用 gdb 创建核心转储的示例 所示，调试器 gdb 需要原始可执行文件和核心转储来重建调试会话的环境。请注意，原始可执行文件是用 `-g` 标志构建的，该标志指示编译器包含所有 *调试信息*。这些额外的调试信息被保留在 ELF 文件的特殊部分中。它详细描述了诸如当前哪些寄存器值持有代码中使用的变量、变量的大小、数组长度等。它通常是标准的 *DWARF* 格式（几乎与 ELF 同义的玩笑）。

包含调试信息可以使可执行文件和库变得非常大；尽管这些数据在实际运行时不需要驻留在内存中，但仍然可能占用相当大的磁盘空间。因此，通常的过程是从 ELF 文件中 *剥离* 这些信息。虽然可以安排同时发送已剥离和未剥离的文件，但大多数当前的二进制分发方法都提供单独的调试信息文件。可以使用 objcopy 工具提取调试信息（`--only-keep-debug`），然后通过添加链接到原始可执行文件中的这些已剥离信息（`--add-gnu-debuglink`）来执行此操作。完成此操作后，原始可执行文件中将出现一个名为 `.gnu_debuglink` 的特殊部分，其中包含一个散列值，以便在调试会话开始时，调试器可以确信它将正确的调试信息与正确的可执行文件关联起来。

```cpp
 1 |$ gcc -g -shared -o libtest.so libtest.c 
 |$ objcopy --only-keep-debug libtest.so libtest.debug 
 |$ objcopy --add-gnu-debuglink=libtest.debug libtest.so 
 |$ objdump -s -j .gnu_debuglink libtest.so 
 5 |
 |libtest.so:     file format elf32-i386 
 |
 |Contents of section .gnu_debuglink: 
 | 0000 6c696274 6573742e 64656275 67000000  libtest.debug... 
10 | 0010 52a7fd0a                             R...` 

```

示例 6.1.1.1 使用 objcopy 将调试信息剥离到单独文件中的示例

符号占用的空间要小得多，但也是最终输出中需要删除的目标。一旦可执行文件的各个单独对象文件链接到单个最终映像中，大多数符号通常就没有保留的必要了。如第 3.2 节，符号和重定位所述，符号是用于修复重定位条目的，但一旦完成这项工作，符号对于运行最终程序就不是严格必要的了。在 Linux 上，GNU 工具链的 strip 程序提供了删除符号的选项。请注意，一些符号在运行时需要解决（对于 *动态链接*，见第九章，动态链接），但这些被放在单独的 *动态* 符号表中，因此它们不会被删除，也不会使最终输出变得无用。

#### 6.1.2 核心转储内部

coredump 实际上只是另一个 ELF 文件；这说明了 ELF 作为二进制格式的灵活性。

```cpp
 1 |$ readelf --all ./core 
 |`ELF Header: 
 | Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00` 
 | `Class:                             ELF32 
 5 | Data:                              2's complement, little endian 
 | Version:                           1 (current) 
 | OS/ABI:                            UNIX - System V 
 | ABI Version:                       0 
 | Type:                              CORE (Core file) 
 10 | Machine:                           Intel 80386 
 | Version:                           0x1 
 | Entry point address:               0x0 
 | Start of program headers:          52 (bytes into file) 
 | Start of section headers:          0 (bytes into file) 
 15 | Flags:                             0x0 
 | Size of this header:               52 (bytes) 
 | Size of program headers:           32 (bytes) 
 | Number of program headers:         15 
 | Size of section headers:           0 (bytes) 
 20 | Number of section headers:         0 
 | Section header string table index: 0 
 |
 |There are no sections in this file. 
 |
 25 |There are no sections to group in this file. 
 |
 |Program Headers: 
 | Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align 
 | NOTE           0x000214 0x00000000 0x00000000 0x0022c 0x00000     0 
 30 | LOAD           0x001000 0x08048000 0x00000000 0x01000 0x01000 R E 0x1000 
 | LOAD           0x002000 0x08049000 0x00000000 0x01000 0x01000 RW  0x1000 
 | LOAD           0x003000 0x489fc000 0x00000000 0x01000 0x1b000 R E 0x1000 
 | LOAD           0x004000 0x48a17000 0x00000000 0x01000 0x01000 R   0x1000 
 | LOAD           0x005000 0x48a18000 0x00000000 0x01000 0x01000 RW  0x1000 
 35 | LOAD           0x006000 0x48a1f000 0x00000000 0x01000 0x153000 R E 0x1000 
 | LOAD           0x007000 0x48b72000 0x00000000 0x00000 0x01000     0x1000 
 | LOAD           0x007000 0x48b73000 0x00000000 0x02000 0x02000 R   0x1000 
 | LOAD           0x009000 0x48b75000 0x00000000 0x01000 0x01000 RW  0x1000 
 | LOAD           0x00a000 0x48b76000 0x00000000 0x03000 0x03000 RW  0x1000 
 40 | LOAD           0x00d000 0xb771c000 0x00000000 0x01000 0x01000 RW  0x1000 
 | LOAD           0x00e000 0xb774d000 0x00000000 0x02000 0x02000 RW  0x1000 
 | LOAD           0x010000 0xb774f000 0x00000000 0x01000 0x01000 R E 0x1000 
 | LOAD           0x011000 0xbfeac000 0x00000000 0x22000 0x22000 RW  0x1000 
 |
 45 |There is no dynamic section in this file. 
 |
 |There are no relocations in this file. 
 |
 |There are no unwind sections in this file. 
 50 |
 |No version information found in this file. 
 |
 |Notes at offset 0x00000214 with length 0x0000022c: 
 | Owner                 Data size	Description 
 55 | CORE                 0x00000090	NT_PRSTATUS (prstatus structure) 
 | CORE                 0x0000007c	NT_PRPSINFO (prpsinfo structure) 
 | CORE                 0x000000a0	NT_AUXV (auxiliary vector) 
 | LINUX                0x00000030	Unknown note type: (0x00000200) 
 |
 60 |$ eu-readelf -n ./core 
 |
 |Note segment of 556 bytes at offset 0x214: 
 | Owner          Data size  Type 
 | CORE                 144  PRSTATUS 
 65 | info.si_signo: 11, info.si_code: 0, info.si_errno: 0, cursig: 11 
 | sigpend: <> 
 | sighold: <> 
 | pid: 31614, ppid: 31544, pgrp: 31614, sid: 31544 
 | utime: 0.000000, stime: 0.000000, cutime: 0.000000, cstime: 0.000000 
 70 | orig_eax: -1, fpvalid: 0 
 | ebx:     1219973108  ecx:     1243440144  edx:              1 
 | esi:              0  edi:              0  ebp:     0xbfecb828 
 | eax:          74565  eip:     0x080483c4  eflags:  0x00010286 
 | esp:     0xbfecb818 
 75 | ds: 0x007b  es: 0x007b  fs: 0x0000  gs: 0x0033  cs: 0x0073  ss: 0x007b 
 | CORE                 124  PRPSINFO 
 | state: 0, sname: R, zomb: 0, nice: 0, flag: 0x00400400 
 | uid: 1000, gid: 1000, pid: 31614, ppid: 31544, pgrp: 31614, sid: 31544 
 | fname: coredump, psargs: ./coredump` 
 80 | `CORE                 160  AUXV 
 | SYSINFO: 0xb774f414 
 | SYSINFO_EHDR: 0xb774f000 
 | HWCAP: 0xafe8fbff  <fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov clflush dts acpi mmx fxsr sse sse2 ss tm pbe> 
 | PAGESZ: 4096 
 85 | CLKTCK: 100 
 | PHDR: 0x8048034 
 | PHENT: 32 
 | PHNUM: 8 
 | BASE: 0 
 90 | FLAGS: 0 
 | ENTRY: 0x8048300 
 | UID: 1000 
 | EUID: 1000 
 | GID: 1000 
 95 | EGID: 1000 
 | SECURE: 0 
 | RANDOM: 0xbfecba1b 
 | EXECFN: 0xbfecdff1 
 | PLATFORM: 0xbfecba2b 
100 | NULL 
 | LINUX                 48  386_TLS 
 | index: 6, base: 0xb771c8d0, limit: 0x000fffff, flags: 0x00000051 
 | index: 7, base: 0x00000000, limit: 0x00000000, flags: 0x00000028 
 | index: 8, base: 0x00000000, limit: 0x00000000, flags: 0x00000028 

```

示例 6.1.2.1 使用 readelf 和 eu-readelf 检查 coredump 的示例。

在示例 6.1.2.1，使用 readelf 和 eu-readelf 检查 coredump 的示例中，我们可以看到使用 readelf 工具对由示例 6.1.1，创建 coredump 并与 gdb 一起使用的示例产生的核心文件进行检查。文件中没有包含加载可执行文件或库可能需要的部分、重定位或其他无关信息；它仅仅由一系列描述 `LOAD` 段的程序头部组成。这些段是内核创建的当前内存分配的原始数据转储。

核心转储的另一个组成部分是 `NOTE` 部分，它包含调试所需的数据，但不一定包含在内存分配的直接快照中。图中第二部分使用的 eu-readelf 程序通过解码提供了更完整的数据视图。

`PRSTATUS` 注释提供了关于正在运行的过程的一些有趣信息；例如，我们可以从 `cursig` 中看到程序收到了信号 11，即段错误，正如我们所预期的那样。除了进程号信息外，它还包括所有当前寄存器的转储。根据寄存器值，调试器可以重建堆栈状态，从而提供 *回溯*；结合来自原始二进制的符号和调试信息，调试器可以显示您是如何到达当前执行点的。

另一个有趣的输出是当前的 *辅助向量* (`AUXV`)，在 第 8.1 节，内核与程序的通信 中讨论。`386_TLS` 描述了用于 x86 实现的 *全局描述符表* 条目，用于 *线程局部存储*（有关分段使用的更多信息，请参阅 第 4.1.1.3 节，快速系统调用；有关线程的信息，请参阅 第 4.3.1.1 节，线程）。对于多线程应用程序，每个运行的线程都会有重复的条目。调试器会理解这一点，这也是 gdb 实现显示和切换线程的 `thread` 命令的方式。

内核在当前 `ulimit` 设置的范围内创建核心转储文件——由于使用大量内存的程序可能会导致非常大的转储，从而可能填满磁盘并使问题更加严重，通常 `ulimit` 设置得较低，甚至为零，因为大多数非开发者很少使用核心转储文件。然而，核心转储仍然是调试意外情况的最有用方式之一。

### 6.2 自定义部分

对于代码、数据和符号的组织，程序员通常可以将其留给工具链默认设置。然而，有时扩展或自定义部分及其内容是有意义的。一个常见的例子是与 Linux 内核 *模块* 相关，这些模块用于动态地将驱动程序和其他功能加载到正在运行的内核中。因为这些模块不可移植，即它们只与一个固定的内核构建版本一起工作，所以模块与内核之间的接口可以是灵活的，并且不受特定标准的约束。这意味着存储诸如许可信息、作者身份、依赖关系和模块参数的方法可以由内核独特地完全定义。

`modinfo` 工具可以在模块内部检查这些信息并向用户展示。以下我们以 `FUSE` Linux 内核模块为例，它允许用户空间库向内核提供文件系统实现。

```cpp
 1 |$ cd /lib/modules/$(uname -r) 
 |
 |$ sudo modinfo ./kernel/fs/fuse/fuse.ko` 
 |`filename:       /lib/modules/3.2.0-4-amd64/./kernel/fs/fuse/fuse.ko 
 5 |alias:          devname:fuse 
 |alias:          char-major-10-229 
 |license:        GPL 
 |description:    Filesystem in Userspace 
 |author:         Miklos Szeredi <miklos@szeredi.hu> 
10 |depends:` 
 |`intree:         Y 
 |vermagic:       3.2.0-4-amd64 SMP mod_unload modversions` 
 |`parm:           max_user_bgreq:Global limit for the maximum number of backgrounded requests an unprivileged user can set (uint) 
 |parm:           max_user_congthresh:Global limit for the maximum congestion threshold an unprivileged user can set (uint) 
15 |
 |$ objdump -s -j .modinfo ./kernel/fs/fuse/fuse.ko` 
 |
 |`./kernel/fs/fuse/fuse.ko:     file format elf64-x86-64 
 |
20 |Contents of section .modinfo: 
 | 0000 616c6961 733d6465 766e616d 653a6675  alias=devname:fu 
 | 0010 73650061 6c696173 3d636861 722d6d61  se.alias=char-ma 
 | 0020 6a6f722d 31302d32 32390070 61726d3d  jor-10-229.parm= 
 | 0030 6d61785f 75736572 5f636f6e 67746872  max_user_congthr 
25 | 0040 6573683a 476c6f62 616c206c 696d6974  esh:Global limit 
 | 0050 20666f72 20746865 206d6178 696d756d   for the maximum 
 | 0060 20636f6e 67657374 696f6e20 74687265   congestion thre 
 | 0070 73686f6c 6420616e 20756e70 72697669  shold an unprivi 
 | 0080 6c656765 64207573 65722063 616e2073  leged user can s 
30 | 0090 65740070 61726d74 7970653d 6d61785f  et.parmtype=max_ 
 | 00a0 75736572 5f636f6e 67746872 6573683a  user_congthresh: 
 | 00b0 75696e74 00706172 6d3d6d61 785f7573  uint.parm=max_us 
 | 00c0 65725f62 67726571 3a476c6f 62616c20  er_bgreq:Global` 
 | `00d0 6c696d69 7420666f 72207468 65206d61  limit for the ma 
35 | 00e0 78696d75 6d206e75 6d626572 206f6620  ximum number of` 
 | `00f0 6261636b 67726f75 6e646564 20726571  backgrounded req 
 | 0100 75657374 7320616e 20756e70 72697669  uests an unprivi 
 | 0110 6c656765 64207573 65722063 616e2073  leged user can s 
 | 0120 65740070 61726d74 7970653d 6d61785f  et.parmtype=max_ 
40 | 0130 75736572 5f626772 65713a75 696e7400  user_bgreq:uint. 
 | 0140 6c696365 6e73653d 47504c00 64657363  license=GPL.desc 
 | 0150 72697074 696f6e3d 46696c65 73797374  ription=Filesyst 
 | 0160 656d2069 6e205573 65727370 61636500  em in Userspace. 
 | 0170 61757468 6f723d4d 696b6c6f 7320537a  author=Miklos Sz 
45 | 0180 65726564 69203c6d 696b6c6f 7340737a  eredi <miklos@sz 
 | 0190 65726564 692e6875 3e000000 00000000  eredi.hu>....... 
 | 01a0 64657065 6e64733d 00696e74 7265653d  depends=.intree= 
 | 01b0 59007665 726d6167 69633d33 2e322e30  Y.vermagic=3.2.0 
 | 01c0 2d342d61 6d643634 20534d50 206d6f64  -4-amd64 SMP mod 
50 | 01d0 5f756e6c 6f616420 6d6f6476 65727369  _unload modversi 
 | 01e0 6f6e7320 00                          ons .` 

```

示例 6.2.1 `modinfo` 输出示例

如上所示，`modinfo` 正在解析模块文件中嵌入的 `.modinfo` 部分，以展示模块的详细信息。示例 6.2.2，将模块信息放入部分 展示了如何将一个字段，“作者”，放入模块中。代码主要来自 `include/linux/module.h`。

```cpp
 1 |/* 
 | * Start at the bottom, and work your way up! 
 | */ 
 |
 5 |`/* Indirect macros required for expanded argument pasting, eg. __LINE__. */ 
 |#define ___PASTE(a,b) a##b 
 |#define __PASTE(a,b) ___PASTE(a,b) 
 |
 |
10 |#define __UNIQUE_ID(prefix) __PASTE(__PASTE(__UNIQUE_ID_, prefix), __COUNTER__) 
 |
 |/* Indirect stringification.  Doing two levels allows the parameter to be a 
 | * macro itself.  For example, compile with -DFOO=bar, __stringify(FOO) 
 | * converts to "bar". 
15 | */ 
 |
 |#define __stringify_1(x...)     #x 
 |#define __stringify(x...)       __stringify_1(x) 
 |
20 |#define __MODULE_INFO(tag, name, info)                                    \ 
 |static const char __UNIQUE_ID(name)[]                                     \ 
 | __used __attribute__((section(".modinfo"), unused, aligned(1)))         \ 
 | = __stringify(tag) "=" info 
 |
25 |/* Generic info of form tag = "info" */ 
 |#define MODULE_INFO(tag, info) __MODULE_INFO(tag, tag, info) 
 |
 |/* 
 | * Author(s), use "Name <email>" or just "Name", for multiple 
30 | * authors use multiple MODULE_AUTHOR() statements/lines. 
 | */ 
 |#define MODULE_AUTHOR(_author) MODULE_INFO(author, _author) 
 |
 |/* ---- */ 
35 |
 |MODULE_AUTHOR("Your Name <your@name.com>"); 

```

示例 6.2.2 将模块信息放入部分

起初，这看起来像是一个宏噩梦，但它可以一步一步地解开。从底部开始，我们看到 `MODULE_AUTHOR` 是一个围绕更通用的 `__MODULE_INFO` 宏的包装器，那里大部分的魔法发生。在那里，我们可以看到我们正在构建一个 `static const char []` 变量来保存字符串 `"author=Your Name <your@name.com>"`。值得注意的是，该变量有一个额外的参数 `__attribute__((section(".modinfo")))`，它告诉编译器不要将此放入与所有其他变量一起的 `data` 部分，而是将其存储在其自己的 ELF 部分 `.modinfo` 中。其他参数阻止变量被优化掉，因为它看起来未使用，并确保通过指定对齐来将变量紧密打包在一起。

在这里广泛使用了 *字符串化* 宏，这些是 C 预处理器中使用的相当晦涩的技巧，用于确保字符串和定义可以共存。另一个技巧是使用 `gcc` 提供的 `__COUNTER__` 特殊定义，它在每次调用时提供一个唯一的递增值；这允许在一个文件中对多个 `MODULE_AUTHOR` 调用，而不会导致相同的变量名。

我们可以检查放置在最终模块中的符号，以查看最终结果：

```cpp
 1 |$ objdump --syms ./fuse.ko | grep modinfo 
 |
 |0000000000000000 l    d  .modinfo	0000000000000000 .modinfo 
 |0000000000000000 l     O .modinfo	0000000000000013 __UNIQUE_ID_alias1 
 5 |0000000000000013 l     O .modinfo	0000000000000018 __UNIQUE_ID_alias0 
 |000000000000002b l     O .modinfo	0000000000000011 __UNIQUE_ID_alias8 
 |000000000000003c l     O .modinfo	000000000000000e __UNIQUE_ID_alias7 
 |000000000000004a l     O .modinfo	0000000000000068 __UNIQUE_ID_max_user_congthresh6 
 |00000000000000b2 l     O .modinfo	0000000000000022 __UNIQUE_ID_max_user_congthreshtype5 
10 |00000000000000d4 l     O .modinfo	000000000000006e __UNIQUE_ID_max_user_bgreq4 
 |0000000000000142 l     O .modinfo	000000000000001d __UNIQUE_ID_max_user_bgreqtype3 
 |000000000000015f l     O .modinfo	000000000000000c __UNIQUE_ID_license2 
 |000000000000016b l     O .modinfo	0000000000000024 __UNIQUE_ID_description1 
 |000000000000018f l     O .modinfo	000000000000002a __UNIQUE_ID_author0 
15 |00000000000001b9 l     O .modinfo	0000000000000011 __UNIQUE_ID_alias0 
 |00000000000001d0 l     O .modinfo	0000000000000009 __module_depends 
 |00000000000001d9 l     O .modinfo	0000000000000009 __UNIQUE_ID_intree1 
 |00000000000001e2 l     O .modinfo	000000000000002f __UNIQUE_ID_vermagic0 

```

示例 6.2.3 `.modinfo` 中的模块符号

### 6.3 链接脚本

在 示例 3.3.2.2，段 中，我们描述了段如何组成最终输出中的段。这是链接器的任务，将这些部分构建成段；为了实现这一点，它使用一个 *链接脚本*，该脚本描述了段开始的位置、哪些部分放入其中以及各种其他参数。

示例 6.3.1，默认链接脚本 展示了默认链接脚本的摘录，当通过指定 `-Wl,--verbose` 到 gcc 时，链接器会显示此脚本。默认脚本内置于链接器中，并基于标准 API 定义来创建适用于构建平台的用户空间程序。

```cpp
 1 |$ gcc -Wl,--verbose -o test test.c 
 |GNU ld (GNU Binutils for Debian) 2.26 
 |... 
 |using internal linker script: 
 5 |================================================== 
 |OUTPUT_FORMAT("elf64-x86-64", "elf64-x86-64", 
 | "elf64-x86-64") 
 |OUTPUT_ARCH(i386:x86-64) 
 |ENTRY(_start) 
10 |SEARCH_DIR("=/usr/local/lib/x86_64-linux-gnu"); ... 
 |SECTIONS 
 |{ 
 | /* Read-only sections, merged into text segment: */ 
 | PROVIDE (__executable_start = SEGMENT_START("text-segment", 0x400000)); . = SEGMENT_START("text-segment", 0x400000) + SIZEOF_HEADERS; 
15 | .interp         : { *(.interp) } 
 | .note.gnu.build-id : { *(.note.gnu.build-id) } 
 | .hash           : { *(.hash) } 
 | .gnu.hash       : { *(.gnu.hash) } 
 | .dynsym         : { *(.dynsym) } 
20 | .dynstr         : { *(.dynstr) } 
 | .gnu.version    : { *(.gnu.version) } 
 | .gnu.version_d  : { *(.gnu.version_d) } 
 | .gnu.version_r  : { *(.gnu.version_r) } 
 | .rela.dyn       : 
25 | { 
 | ... 
 | } 
 | PROVIDE (etext = .); 
 | .rodata         : { *(.rodata .rodata.* .gnu.linkonce.r.*) } 
30 | .rodata1        : { *(.rodata1) } 
 |... 

```

示例 6.3.1 默认链接脚本

你可以大致了解链接脚本如何指定诸如起始位置和将哪些部分组合到各种段中等内容。同样，使用 `-Wl` 通过 gcc 将 `--verbose` 传递给链接器，可以通过标志提供自定义的链接脚本。普通用户空间开发者不太可能需要覆盖默认链接脚本。然而，通常需要非常定制的应用程序，如内核构建，则需要自定义链接脚本。





## 7 ABIs

当进行系统编程时，你将经常听到 ABI 这个术语。我们已经广泛讨论了 *API*，这是程序员看到的与你的代码的接口。

ABI（应用程序二进制接口）指的是编译器、操作系统以及在一定程度上处理器必须达成一致的低级接口，以便相互通信。下面我们将介绍一些对理解 ABI 考虑因素很重要的概念。

### 7.1 字节序

字节序

### 7.2 调用约定

#### 7.2.1 传递参数

寄存器还是栈？

#### 7.2.2 函数描述符

在许多架构中，你必须通过一个*函数描述符*来调用一个函数，而不是直接调用。

例如，在 IA64 架构中，函数描述符由两个组件组成；函数的地址（这是一个 64 位，或 8 字节的值）和全局指针（gp）的地址。ABI 指定 r1 应该始终包含函数的 gp 值。这意味着当你调用一个函数时，保存自己的 gp 值、将 r1 设置为新的值（来自函数描述符）并`然后`调用函数是调用者的责任。

这种方法可能看起来有些奇怪，但它在实际应用中非常有用，你将在下一章关于全局偏移表的章节中看到。在 IA64 架构中，`add`指令只能接受最大 22 位的*立即值*。技术上这是因为 IA64 指令捆绑的方式。每个捆绑包中放入三个指令，而捆绑包中只有足够的空间来保持一个 22 位值，以保持捆绑包在一起。立即值是直接指定的值，而不是在寄存器中指定的（例如，在`add r1 + 100`中，100 是立即值）。

你可能已经注意到 22 位可以表示 4194304 字节，即 4MB。因此，每个函数可以直接偏移到 4MB 大小的内存区域，而无需承担将任何值加载到寄存器的惩罚。如果编译器、链接器和加载器都同意全局指针指向的内容（如 ABI 中指定），则可以通过减少加载来提高性能。





## 8 启动进程

我们之前提到，简单地说程序从`main()`函数开始并不完全准确。下面我们将检查一个典型的动态链接程序在加载和运行时会发生什么（静态链接程序类似但有所不同 XXX 我们是否应该深入探讨这个问题？）。

首先，内核响应`exec`系统调用，为新进程分配结构，并从磁盘读取指定的 ELF 文件。

我们提到，ELF 有一个程序解释器字段，`PT_INTERP`，可以设置为`interpret`程序。对于动态链接的应用程序，这个解释器是动态链接器，即 ld.so，它允许在程序开始之前动态地完成一些链接过程。

在这种情况下，内核*还*读取动态链接器代码，并从它指定的入口点地址启动程序。我们将在下一章深入探讨动态链接器的角色，但简而言之，它执行一些设置，例如加载应用程序所需的任何库（如二进制文件的动态部分中指定），然后从入口点地址启动程序二进制文件的执行（即`_init`函数）。

### 8.1 内核与程序的通信

当程序启动时，内核需要将一些信息传递给程序；即程序的参数、当前环境变量以及一个称为`辅助向量`或`auxv`的特殊结构（你可以通过指定环境值`LD_SHOW_AUXV=1`来请求动态链接器显示一些`auxv`的调试输出）。

参数和环境变量相当直接，`exec`系统调用的各种实现允许你为程序指定这些参数。

内核通过将所有必要信息放在新创建程序的栈上，以便程序可以拾取这些信息来传达这一点。因此，当程序启动时，它可以使用其栈指针找到所有必需的启动信息。

辅助向量是一个特殊结构，用于将信息直接从内核传递到新运行的程序。它包含可能需要的一些系统特定信息，例如系统上虚拟内存页的默认大小或*硬件能力*；即内核已识别的底层硬件具有的特定功能，用户空间程序可以利用这些功能。

#### 8.1.1 内核库

我们之前提到系统调用很慢，现代系统有机制来避免调用处理器陷阱的开销。

在 Linux 中，这是通过动态加载器和内核之间的一种巧妙技巧实现的，所有通信都通过 AUXV 结构进行。内核实际上将一个小型共享库添加到每个新创建的进程的地址空间中，该库包含一个为你执行系统调用的函数。这个系统的美妙之处在于，如果底层硬件支持快速系统调用机制，内核（作为库的创建者）可以使用它；否则，它可以使用生成陷阱的旧方案。这个库被命名为`linux-gate.so.1`，之所以这样命名，是因为它是进入内核内部工作原理的*门户*。

当内核启动动态链接器时，它会在`auxv`中添加一个条目，称为`AT_SYSINFO_EHDR`，这是特殊内核库在内存中的地址。当动态链接器启动时，它可以查找`AT_SYSINFO_EHDR`指针，如果找到，则为程序加载该库。程序并不知道这个库的存在；这是动态链接器和内核之间的一个私人安排。

我们提到，程序员通过调用系统库中的函数间接进行系统调用，即调用 libc。libc 可以检查是否已加载特殊的内核二进制文件，如果是，则使用其中的函数进行系统调用。正如我们提到的，如果内核确定硬件具备条件，它将使用快速系统调用方法。

### 8.2 程序启动

一旦内核加载了解释器，它将解释器传递给解释器文件中给出的入口点（注意，在此阶段不会检查动态链接器的启动方式；有关动态链接的完整讨论，请参阅第九章，动态链接）。动态链接器将跳转到 ELF 二进制中给出的入口点地址。

```cpp
 1 |$ cat test.c 
 |
 |int main(void) 
 |{ 
 5 | return 0; 
 |} 
 |
 |$ gcc -o test test.c 
 |
10 |$ readelf --headers ./test | grep Entry 
 | Entry point address:               0x80482b0 
 |
 |$ objdump --disassemble ./test 
 |
15 |[...] 
 |
 |080482b0 <_start>: 
 | 80482b0:       31 ed                   xor    %ebp,%ebp 
 | 80482b2:       5e                      pop    %esi 
20 | 80482b3:       89 e1                   mov    %esp,%ecx 
 | 80482b5:       83 e4 f0                and    $0xfffffff0,%esp 
 | 80482b8:       50                      push   %eax 
 | 80482b9:       54                      push   %esp 
 | 80482ba:       52                      push   %edx 
25 | 80482bb:       68 00 84 04 08          push   $0x8048400 
 | 80482c0:       68 90 83 04 08          push   $0x8048390 
 | 80482c5:       51                      push   %ecx 
 | 80482c6:       56                      push   %esi 
 | 80482c7:       68 68 83 04 08          push   $0x8048368 
30 | 80482cc:       e8 b3 ff ff ff          call   8048284 <__libc_start_main@plt> 
 | 80482d1:       f4                      hlt 
 | 80482d2:       90                      nop 
 | 80482d3:       90                      nop 
 |
35 |08048368 : 
 | 8048368:       55                      push   %ebp 
 | 8048369:       89 e5                   mov    %esp,%ebp 
 | 804836b:       83 ec 08                sub    $0x8,%esp 
 | 804836e:       83 e4 f0                and    $0xfffffff0,%esp 
40 | 8048371:       b8 00 00 00 00          mov    $0x0,%eax 
 | 8048376:       83 c0 0f                add    $0xf,%eax 
 | 8048379:       83 c0 0f                add    $0xf,%eax 
 | 804837c:       c1 e8 04                shr    $0x4,%eax 
 | 804837f:       c1 e0 04                shl    $0x4,%eax 
45 | 8048382:       29 c4                   sub    %eax,%esp 
 | 8048384:       b8 00 00 00 00          mov    $0x0,%eax 
 | 8048389:       c9                      leave 
 | 804838a:       c3                      ret 
 | 804838b:       90                      nop 
50 | 804838c:       90                      nop 
 | 804838d:       90                      nop 
 | 804838e:       90                      nop 
 | 804838f:       90                      nop 
 |
55 |08048390 <__libc_csu_init>: 
 | 8048390:       55                      push   %ebp 
 | 8048391:       89 e5                   mov    %esp,%ebp 
 | [...] 
 |
60 |08048400 <__libc_csu_fini>: 
 | 8048400:       55                      push   %ebp 
 | [...] 

```

示例 8.2.1 程序启动的拆解

如上所述，我们研究了非常简单的程序。使用 readelf 我们可以看到，入口点是二进制中的 `_start` 函数。在这个点上，我们可以看到在反汇编中一些值被推入栈中。第一个值 `0x8048400` 是 `__libc_csu_fini` 函数；`0x8048390` 是 `__libc_csu_init`，然后最后是 `0x8048368`，即 `main()` 函数。之后，调用 `__libc_start_main` 函数。

`__libc_start_main` 定义在 glibc 源文件 `sysdeps/generic/libc-start.c` 中。该文件中的函数相当复杂，隐藏在大量的定义之间，因为它需要在 glibc 可以运行的非常广泛的系统和架构之间保持可移植性。它执行了与设置 C 库相关的一系列特定操作，而普通程序员不需要担心这些操作。库回调到程序的下一个点是处理 `init` 代码。

`init` 和 `fini` 是两个特殊的概念，它们调用共享库中的代码部分，这些代码部分可能在库开始之前或库卸载时需要调用。你可以看到这对于库程序员来说可能很有用，可以在库启动时设置变量，或者在结束时进行清理。最初，在库中寻找 `_init` 和 `_fini` 函数；然而，这变得有些限制，因为所有内容都必须包含在这些函数中。下面我们将考察 `init`/`fini` 过程是如何工作的。

在这个阶段，我们可以看到 `__libc_start_main` 函数将在栈上接收相当多的输入参数。首先，它将能够访问来自内核的程序参数、环境变量和辅助向量。然后，初始化函数将把处理 `init`、`fini` 的函数地址以及主函数本身的地址推入栈中。

我们需要在源代码中找到一种方法来指示函数应由 `init` 或 `fini` 调用。使用 gcc，我们使用 *属性* 来标记主程序中的两个函数作为 *构造函数* 和 *析构函数*。这些术语在面向对象的语言中更常用，用于描述对象的生命周期。

```cpp
 1 |$ cat test.c 
 |#include <stdio.h> 
 |
 |void __attribute__((constructor)) program_init(void)  { 
 5 | printf("init\n"); 
 |} 
 |
 |void  __attribute__((destructor)) program_fini(void) { 
 | printf("fini\n"); 
 10 |} 
 |
 |int main(void) 
 |{ 
 | return 0; 
 15 |} 
 |
 |$ gcc -Wall  -o test test.c 
 |
 |$ ./test 
 20 |init 
 |fini 
 |
 |$ objdump --disassemble ./test | grep program_init 
 |08048398 <program_init>: 
 25 |
 |$ objdump --disassemble ./test | grep program_fini 
 |080483b0 <program_fini>: 
 |
 |$ objdump --disassemble ./test` 
 30 |
 |`[...] 
 |08048280 <_init>: 
 | 8048280:       55                      push   %ebp 
 | 8048281:       89 e5                   mov    %esp,%ebp 
 35 | 8048283:       83 ec 08                sub    $0x8,%esp 
 | 8048286:       e8 79 00 00 00          call   8048304 <call_gmon_start> 
 | 804828b:       e8 e0 00 00 00          call   8048370 <frame_dummy> 
 | 8048290:       e8 2b 02 00 00          call   80484c0 <__do_global_ctors_aux> 
 | 8048295:       c9                      leave 
 40 | 8048296:       c3                      ret 
 |[...] 
 |
 |080484c0 <__do_global_ctors_aux>: 
 | 80484c0:       55                      push   %ebp 
 45 | 80484c1:       89 e5                   mov    %esp,%ebp 
 | 80484c3:       53                      push   %ebx 
 | 80484c4:       52                      push   %edx 
 | 80484c5:       a1 2c 95 04 08          mov    0x804952c,%eax 
 | 80484ca:       83 f8 ff                cmp    $0xffffffff,%eax 
 50 | 80484cd:       74 1e                   je     80484ed <__do_global_ctors_aux+0x2d> 
 | 80484cf:       bb 2c 95 04 08          mov    $0x804952c,%ebx 
 | 80484d4:       8d b6 00 00 00 00       lea    0x0(%esi),%esi 
 | 80484da:       8d bf 00 00 00 00       lea    0x0(%edi),%edi 
 | 80484e0:       ff d0                   call   *%eax 
 55 | 80484e2:       8b 43 fc                mov    0xfffffffc(%ebx),%eax 
 | 80484e5:       83 eb 04                sub    $0x4,%ebx 
 | 80484e8:       83 f8 ff                cmp    $0xffffffff,%eax 
 | 80484eb:       75 f3                   jne    80484e0 <__do_global_ctors_aux+0x20> 
 | 80484ed:       58                      pop    %eax 
 60 | 80484ee:       5b                      pop    %ebx 
 | 80484ef:       5d                      pop    %ebp 
 | 80484f0:       c3                      ret 
 | 80484f1:       90                      nop 
 | 80484f2:       90                      nop 
 65 | 80484f3:       90                      nop 
 |
 |
 |$ readelf --sections ./test 
 |There are 34 section headers, starting at offset 0xfb0: 
 70 |
 |Section Headers: 
 | [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al 
 | [ 0]                   NULL            00000000 000000 000000 00      0   0  0 
 | [ 1] .interp           PROGBITS        08048114 000114 000013 00   A  0   0  1 
 75 | [ 2] .note.ABI-tag     NOTE            08048128 000128 000020 00   A  0   0  4 
 | [ 3] .hash             HASH            08048148 000148 00002c 04   A  4   0  4 
 | [ 4] .dynsym           DYNSYM          08048174 000174 000060 10   A  5   1  4 
 | [ 5] .dynstr           STRTAB          080481d4 0001d4 00005e 00   A  0   0  1 
 | [ 6] .gnu.version      VERSYM          08048232 000232 00000c 02   A  4   0  2 
 80 | [ 7] .gnu.version_r    VERNEED         08048240 000240 000020 00   A  5   1  4 
 | [ 8] .rel.dyn          REL             08048260 000260 000008 08   A  4   0  4 
 | [ 9] .rel.plt          REL             08048268 000268 000018 08   A  4  11  4 
 | [10] .init             PROGBITS        08048280 000280 000017 00  AX  0   0  4 
 | [11] .plt              PROGBITS        08048298 000298 000040 04  AX  0   0  4 
 85 | [12] .text             PROGBITS        080482e0 0002e0 000214 00  AX  0   0 16 
 | [13] .fini             PROGBITS        080484f4 0004f4 00001a 00  AX  0   0  4 
 | [14] .rodata           PROGBITS        08048510 000510 000012 00   A  0   0  4 
 | [15] .eh_frame         PROGBITS        08048524 000524 000004 00   A  0   0  4 
 | [16] .ctors            PROGBITS        08049528 000528 00000c 00  WA  0   0  4 
 90 | [17] .dtors            PROGBITS        08049534 000534 00000c 00  WA  0   0  4 
 | [18] .jcr              PROGBITS        08049540 000540 000004 00  WA  0   0  4 
 | [19] .dynamic          DYNAMIC         08049544 000544 0000c8 08  WA  5   0  4 
 | [20] .got              PROGBITS        0804960c 00060c 000004 04  WA  0   0  4 
 | [21] .got.plt          PROGBITS        08049610 000610 000018 04  WA  0   0  4 
 95 | [22] .data             PROGBITS        08049628 000628 00000c 00  WA  0   0  4 
 | [23] .bss              NOBITS          08049634 000634 000004 00  WA  0   0  4 
 | [24] .comment          PROGBITS        00000000 000634 00018f 00      0   0  1 
 | [25] .debug_aranges    PROGBITS        00000000 0007c8 000078 00      0   0  8 
 | [26] .debug_pubnames   PROGBITS        00000000 000840 000025 00      0   0  1 
100 | [27] .debug_info       PROGBITS        00000000 000865 0002e1 00      0   0  1 
 | [28] .debug_abbrev     PROGBITS        00000000 000b46 000076 00      0   0  1 
 | [29] .debug_line       PROGBITS        00000000 000bbc 0001da 00      0   0  1 
 | [30] .debug_str        PROGBITS        00000000 000d96 0000f3 01  MS  0   0  1 
 | [31] .shstrtab         STRTAB          00000000 000e89 000127 00      0   0  1 
105 | [32] .symtab           SYMTAB          00000000 001500 000490 10     33  53  4 
 | [33] .strtab           STRTAB          00000000 001990 000218 00      0   0  1 
 |Key to Flags: 
 | W (write), A (alloc), X (execute), M (merge), S (strings) 
 | I (info), L (link order), G (group), x (unknown) 
110 | O (extra OS processing required) o (OS specific), p (processor specific) 
 |
 |$ objdump --disassemble-all --section .ctors ./test 
 |
 |./test:     file format elf32-i386 
115 |
 |Contents of section .ctors: 
 | 8049528 ffffffff 98830408 00000000           ............ 

```

示例 8.2.2 构造函数和析构函数

最后压入 `__libc_start_main` 堆栈的值是初始化函数 `__libc_csu_init`。如果我们从 `__libc_csu_init` 跟踪调用链，我们可以看到它进行了一些设置，然后调用可执行文件中的 `_init` 函数。`_init` 函数最终调用一个名为 `__do_global_ctors_aux` 的函数。查看这个函数的反汇编代码，我们可以看到它似乎从地址 `0x804952c` 开始，循环读取值并调用它。我们可以看到这个起始地址位于文件的 `.ctors` 部分；如果我们查看它，我们可以看到它包含第一个值 `-1`，一个函数地址（以大端格式），以及值零。

大端格式的地址是 `0x08048398`，或者 `program_init` 函数的地址！所以 `.ctors` 部分的格式首先是一个 `-1`，然后是初始化时需要调用的函数的地址，最后是一个零，表示列表结束。每个条目都将被调用（在这种情况下我们只有一个函数）。

一旦 `__libc_start_main` 通过 `_init` 调用完成，它最终会调用 `main()` 函数！记住，它最初是通过内核的参数和环境指针设置了堆栈；这就是 `main` 获取其 `argc, argv[], envp[]` 参数的方式。现在进程开始运行，设置阶段完成。

当程序退出时，对 `.dtors` 进行类似的过程来执行析构函数。`__libc_start_main` 在 `main()` 函数完成后调用这些函数。

正如你所见，在程序开始之前做了很多工作，甚至在你想它已经完成之后，还有一些工作要做！


