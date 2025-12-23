# 参考文献

> 原文：[`enhancedformysql.github.io/The-Art-of-Problem-Solving-in-Software-Engineering_How-to-Make-MySQL-Better/References.html`](https://enhancedformysql.github.io/The-Art-of-Problem-Solving-in-Software-Engineering_How-to-Make-MySQL-Better/References.html)


[1] Zeitz, P. (1999). 问题解决的艺术与技巧。纽约：John Wiley。

[2] C. Mohan, D.L. Haderle, B. Lindsay, H. Pirahesh, 和 P. Schwarz。ARIES：使用写入前日志支持细粒度锁定和部分回滚的事务恢复方法。ACM TODS，第 17 卷，第 1 期，第 94–162 页，1992 年。

[3] Johnson, R. 等。多核和多插槽硬件上写入前日志的可扩展性。VLDBJ，第 239–263 页，2012 年。

[4] Collin McCurdy 和 Jeffrey Vetter。2010。Memphis：在多核平台上寻找和修复 NUMA 相关的性能问题。在 IEEE 国际系统与软件性能分析会议（ISPASS’10），第 87–96 页。

[5] T. Kiefer, B. Schlegel, 和 W. Lehner。NUMA 对数据库管理系统影响的实验评估。在 BTW，2013 年。

[6] A. Alquraan, H. Takruri, M. Alfatafta, 和 S. Al-Kiswany。云系统中网络分区故障分析。在第 13 届 USENIX 操作系统设计与实现会议，OSDI’18，第 51–68 页，Carlsbad, CA, USA，2018 年。USENIX 协会。

[7] Adnan Alhomssi 和 Viktor Leis. 2023. 可扩展且鲁棒的快照隔离技术用于高性能存储引擎。VLDB Endowment 16, 6 (2023), 1426–1438.

[8] Y. Wang, M. Yu, Y. Hui, F. Zhou, Y. Huang, R. Zhu, 等。2022。数据库性能对实验设置的敏感性研究，VLDB Endowment，第 15 卷，第 7 期。

[9] M. Raasveldt, P. Holanda, T. Gubner, 和 H. Muhleisen。公平基准测试困难重重：数据库性能测试中的常见陷阱。在第 7 届国际数据库系统测试研讨会，DBTest，第 2:1–2:6 页，2018 年。

[10] M. Harchol-Balter。计算机系统性能建模与设计：排队论的实际应用。剑桥大学出版社，纽约，NY，USA，第 1 版，2013 年。

[11] Hao Xu, Qingsen Wang, Shuang Song, Lizy Kurian John, 和 Xu Liu. 2019. 我们能相信分析结果吗？理解并修复现代分析器中的不准确之处。在 ACM 国际超级计算会议论文集，第 284–295 页。

[12] T. Mytkowicz, A. Diwan, M. Hauswirth, 和 P. F. Sweeney。评估 Java 分析器的准确性。在 PLDI，第 187–197 页。ACM，2010 年。

[13] https://dev.mysql.com/doc/refman/8.0/en/

[14] 阿里云。2019. OceanBase 在 TPC-C 基准测试中优于任何其他数据库。alibaba-cloud.medium.com。

[15] B. H. Dowden. 逻辑推理（加利福尼亚州立大学，萨克拉门托，CA，2019 年）。

[16] C. Hong, D. Zhou, M. Yang, C. Kuo, L. Zhang, 和 L. Zhou. KuaFu: 缩小数据库复制中的并行性差距。在 2013 年 ICDE 会议论文集，第 1186–1195 页，2013 年。

[17] 伊·萨鲁达基斯，T·舒尔，N·梅，和 A·艾拉马基。高度并发分析和事务内存工作负载的任务调度。在 ADMS 研讨会，2013。

[18] 戴勤，布朗，和戈尔。2017。快速数据库的可扩展重放式复制。VLDB Endowment (2017)。

[19] 俊昊，韩，费克特，海瑟，和姚恩。多核的可扩展锁管理器。在 SIGMOD，2013。

[20] 西蒙：布鲁尔器的 CAP 定理。巴塞尔大学 CS341 分布式信息系统，HS2012。

[21] 哈里佐波罗斯，S.，和艾拉马基，A. 2003。关于分阶段数据库系统的案例。在创新数据系统研究会议（CIDR）上发表论文。阿索拉马尔，CA。哈里佐波罗斯，S.，和艾拉马基，A. 2003。关于分阶段数据库系统的案例。在创新数据系统研究会议（CIDR）上发表论文。阿索拉马尔，CA。

[22] 郁祥耀。对一千核心并发控制的评估。麻省理工学院博士论文，2015。

[23] 任科，Faleiro，J.M.，和 Abadi，D.J.。在高竞争下扩展多核 OLTP 的设计原则。在 2016 年 ACM SIGMOD 国际数据管理会议上发表，2016。

[24] 天博，黄杰，莫扎法里，和舒恩贝克。针对事务数据库的竞争感知锁调度。PVLDB，11(5)，2018。

[25] 彼得·巴利斯，艾伦·费克特，迈克尔·J·富兰克林，阿里·戈德斯，约瑟夫·M·赫勒斯坦，和伊恩·斯托卡。2014。数据库系统中的协调避免。VLDB Endow. 3 (Nov. 2014)，185–196。

[26] 拉奥，谢基塔，和塔塔。使用 paxos 构建可扩展、一致且高度可用的数据存储。VLDB，2011。

[27] [帕维尔·奥尔查瓦](https://dev.mysql.com/blog-archive/?author=Pawe%C5%82%20Olchawa)。2018。MySQL 8.0：新的无锁、可扩展 WAL 设计。dev.mysql.com/blog-archive。

[28] 马丁·克莱普曼。设计数据密集型应用。英文版。1 版。奥雷利媒体，2017 年 1 月。ISBN：978-1-4493-7332-0。

[29] 海迪·豪沃德。分布式一致性修订。剑桥大学博士论文，2019。URL：https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-935.pdf。

[30] 查德拉，T.D.，格里塞梅尔，R.，雷德斯坦，J.：Paxos Made Live：工程视角。在第 26 届 ACM 分布式计算原理年会上发表。纽约，美国：ACM，第 398–407 页，（2007）。

[31] https://dev.mysql.com/blog-archive/the-new-mysql-thread-pool/

[32] 毛宇，朱恩奎拉，和马鲁祖洛。Mencius：为 WAN 构建高效的复制状态机。在第 8 届 USENIX OSDI 上发表论文，第 369–384 页，圣地亚哥，CA，2008。

[33] 米尔斯普。关于性能、队列的清晰思考，Queue，第 8 卷，第 9 期，第 10-20 页，2010。

[34] Leonidas Galanis, Supiti Buranawatanachoke, Romain Colle, [Benoît Dageville](https://dblp.org/pid/59/847.html), Karl Dias, Jonathan Klein, Stratos Papadomanolakis, Leng Leng Tan, Venkateshwaran Venkataramani, Yujun Wang, 和 Graham Wood. 2008\. Oracle 数据库回放。在第 2008 年 ACM SIGMOD 国际数据管理会议上，第 1159–1170 页。

[35] G. Moerkotte 和 T. Neumann. 动态规划反击。在第 2008 年 ACM SIGMOD 国际数据管理会议上，第 539–552 页。ACM，2008 年。

[36] Suntorn Sae-eung. 2010\. 多核 CPU 上虚假缓存行共享效应的分析，硕士项目，第 01 卷。

[37] D. Terry. 通过棒球解释复制数据一致性，微软技术报告 MSR-TR-2011-137，2011 年 10 月。即将发表在《ACM 通讯》上，2013 年 12 月。

[38] Anirban Rahut, Abhinav Sharma, Yichen Shen, Ahsanul Haque. 2023\. 在 Meta 构建和部署 MySQL Raft。engineering.fb.com。

[39] Taipalus T. 数据库管理系统性能比较：系统综述。在线发布于 2023 年 1 月 3 日。2023 年 7 月 31 日访问。http://arxiv.org/abs/2301.01095。

[40] Holger Pirk. 2022\. https://co339.pages.doc.ic.ac.uk/decks/Profiling.pdf.

[41] M Poke. 2019\. 高性能状态机复制算法。博士论文。赫尔穆特·施密特大学。

[42] Ritwik Yadav 和 Anirban Rahut. 2023\. FlexiRaft：Raft 的灵活法定人数。创新数据系统研究会议（CIDR）(2023 年)。

[43] Jung-Sang Ahn, Woon-Hak Kang, Kun Ren, Guogen Zhang, 和 Sami Ben-Romdhane. 2019\. 使用共识协议设计一个高效的复制日志存储。在第 11 届 USENIX 云计算热点话题研讨会（HotCloud 19）上。

[44] Arunprasad P. Marathe, Shu Lin, Weidong Yu, Kareem El Gebaly, Per-Åke Larson, 和 Calvin Sun. 2022\. 将 Orca 优化器集成到 MySQL 中。在 25 届国际扩展数据库技术会议（EDBT 2022）论文集中，爱丁堡，英国，2022 年 3 月 29 日至 4 月 1 日。OpenProceedings.org，第 2 卷，第 511–2:523 页。

[45] https://en.wikipedia.org/wiki/.

[46] Urbán, P., Défago, X., Schiper, A.: 追寻 FLP 不可行性结果在局域网中的可能性或一个容错服务器可以有多强大？在：第 20 届 IEEE 可靠分布式系统研讨会（SRDS），第 190–193 页。美国新奥尔良（2001 年）。

[47] https://www.candtsolution.com/news_events-detail/what-is-the-difference-between-arm-and-x86/.

[48] N. Santos 和 A. Schiper. 通过批处理和流水线调整 Paxos 以实现高吞吐量，在第 13 届国际分布式计算和网络会议（ICDCN 2012）上，2012 年 1 月。

[49] J. Kończak, N. Santos, T. Zurkowski, P. T. Wojciechowski, 和 A. Schiper. JPaxos：基于 Paxos 协议的状态机复制。技术报告，EPFL，2011 年。

[50] R. N. Avula 和 C. Zou. 在各种云服务提供商上对 TPC-C 基准的性能评估，第 11 届 IEEE 年度通用计算、电子和移动通信会议（UEMCON），第 226-233 页，2020 年 10 月.

[51] Guna Prasaad，Alvin Cheung，和 Dan Suciu. 2018. 通过事务调度提高高冲突 OLTP 性能. https://arxiv.org/abs/1810.01997v1.

[52] Sergey Blagodurov 和 Alexandra Fedorova. 2011. 在 Linux 下 NUMA 多核系统上的用户级调度. 在 Linux 研讨会论文集中.

[53] https://www.ibm.com/docs/en/linux-on-systems?topic=management-linux-scheduling.

[54] 王艳华. 2010. 广域网中的状态机复制. 计算机科学博士论文. 加利福尼亚大学圣地亚哥分校.

[55] 周翔，蔡超，李刚，孙杰. 数据库与人工智能的融合：综述. IEEE 知识数据工程 Transactions (2020).

[56] Prince Samuel. 2023. 编程中逻辑推理的作用. medium.com.

[57] Sunny Bains. 2017. 基于冲突感知的事务调度即将在 InnoDB 中推出以提升性能. https://dev.mysql.com/blog-archive/.

[58] Andres Freund. 2020. 提高 Postgres 连接可扩展性：快照. techcommunity.microsoft.com.

[59] J. M. Hellerstein, M. Stonebraker, 和 J. R. Hamilton. 数据库系统架构. 数据库基础与趋势. 第 1 卷第 2 期，第 141-259 页，2007 年.

[60] 李传鹏，丁晨，沈凯. 量化上下文切换的成本. 在 2007 年实验计算机科学研讨会论文集中，ExpCS ‘07，美国纽约，2007 年. ACM. [61] 郝向鹏，周新静，余翔耀，Michael Stonebraker. 2024. 向缓冲管理引入分层主存储. ACM 管理数据 2，1（2024 年 2 月），文章 31 号. SIGMOD.

[62] Guoliang Jin，Linhai Song，Xiaoming Shi，Joel Scherpelz，和 Shan Lu. 2012. 理解和检测现实世界的性能错误. 在 ACM SIGPLAN 编程语言设计与应用会议，PLDI ‘12，中国北京 - 2012 年 6 月 11 日至 16 日，第 77-88 页.

[63] Jelena Antic, Georgios Chatzopoulos, Rachid Guerraoui, 和 Vasileios Trigonakis. 2016. 简化锁机制. 在国际中间件会议（Middleware）论文集中，第 1-14 页.

[64] D. Dice 和 A. Kogan, “通过限制并发来避免可扩展性崩溃”在 Euro-Par 2019：并行处理，Cham：Springer 国际出版社，第 363-376 页，2019 年.

[65] https://github.com/session-replay-tools/tcpcopy.

[66] https://github.com/session-replay-tools/cetus.

[67] https://github.com/enhancedformysql/enhancedformysql.

[68] https://github.com/enhancedformysql/benchmarksql.

[69] https://github.com/enhancedformysql/tech-explorer-hub.

[70] Graefe G (2010) B 树锁定技术的综述. ACM 数据库系统 Transactions 35(3):16

[71] 让-皮埃尔·洛齐（Jean-Pierre Lozi）、巴蒂斯特·勒佩尔（Baptiste Lepers）、贾斯汀·芬斯顿（Justin Funston）、法比恩·高德（Fabien Gaud）、维维安·克玛（Vivien Quéma）和亚历山德拉·费多罗娃（Alexandra Fedorova）。Linux 调度器：十年浪费的核心。载于第十一届欧洲计算机系统会议论文集，EuroSys ‘16，纽约，纽约，美国，2016 年。计算机制造协会。

下一页
