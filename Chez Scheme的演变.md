# 1. 简介

Chez Scheme Version 1于1984年完成并于1985年发布。我惊讶地发现自己在二十多年后仍然在研究它。如果在1985年被问到期待二十年，我会说Chez Scheme和Scheme本身早就进入了历史的一段时间。毕竟，最古老的语言并不比今天的Scheme早，而且许多语言已经过去了。从那以后，许多语言都来去匆匆，但是根据大约1960年的Lisp，Scheme仍然存在。用户社区现在比以往更大，更多样化，所以运气好的话，语言和实现将持续至少二十年。这是一个可怕的想法。

长寿与适应性有关，目前Chez Scheme的版本与1985年的版本大不相同。它实现了更大，更不同的语言，体现了更丰富的编程环境。编译器和存储管理系统一样复杂得多。初始版本在一个架构上运行并在一个操作系统下运行，而系统现在支持各种不同的计算平台，并且一次又支持许多其他计算平台。

尽管如此，版本7背后的原理与第1版背后的原则相同。我们的主要目标仍然是可靠性和效率。可靠的系统是正确实现整个语言的系统，并且不会因编译器或运行时环境中的故障​​而崩溃。

一个高效的系统是在其操作的各个方面都表现出一致良好性能的系统，具有快速编译器，可以生成快速代码，并且可以实现最广泛的程序和编程风格。虽然我们多年来添加了许多新功能，并通过更好的反馈和调试支持来提高系统的可用性，但我们始终以考虑主要目标的方式这样做。

上面的段落首先出现在Chez Scheme Version 7用户指南[21]的序言中，该指南于2005年出版。当然，用户指南继续记录当前存在的语言，并且没有更多关于系统的历史。本文的目的是探索历史，回答系统如何以及为何成为现在的样子。

本文的其余部分首先简要介绍了这些系统，这些系统以某种方式是Chez Scheme的前身（第2节）。然后描述了Chez Scheme的初始版本和后续版本背后的动机以及这些版本中出现的一些更重要的新语言特性或实现技术（第3至10节）。本文最后提出了一些分离言论（第11节）。

# 2. 前言

Chez Scheme不是从真空中产生。 以下是我在Chez Scheme之前研究过的几个Scheme或Lisp系统的描述，这些系统以某种方式影响了Chez Scheme的设计和实现。

## 2.1 SDP

Scheme Distributed Processes [13]（SDP）是由印第安纳大学研究生Rex Dwyer和我在1980-81编写的Scheme的多线程实现。该系统主要是研究Per Brinch Hansen [34]提出的分布式过程并发模型的工具。它源于丹·弗里德曼（Dan Friedman）在他的研究生编程语言研讨会上给我们的任务，他用他和鲍勃·菲尔曼（Bob Filman）撰写的关于并发编程技术的书进行了教学[32]。

SDP支持1978年（修订报告）版本的大部分Scheme[52]。除了并行处理扩展外，SDP还支持数组，部分应用程序机制，用于加载和保存定义的dskin和dskout函数，甚至是结构编辑器。 SDP通过区分false和空列表以及要求列表的cdr也是列表来偏离Scheme的语义。 SDP完全是用Simura [3]编写的，并利用了Simula的运行时系统，其中最重要的是它的垃圾收集器。

虽然SDP的代码都没有存入我后来写的任何Scheme系统中，但它为我提供了我第一次使用Scheme和实现Lisp方言的经验。

## 2.2 Z80计划

1981年，当我在IU的学术计算中心担任系统程序员时，George Cohn和我决定为Z80微处理器创建Scheme的实现。通过这样做，乔治可以教我如何在Z80微处理器中编程，我可以教他Scheme。乔治是一个令人难以置信的程序员，我从他那里学到了很多东西，我确信这是讨价还价的最好结局。

我们在CP / M [48]操作系统下完全用Z80组装编码了Z80 Scheme系统。该系统支持大多数（如果不是全部）1978版本的方案，包括完整延续。所有值恰好占用32位（两个16位字）或由32位值的链接组成。所有对象在32位边界上对齐，我们能够使用每个单词的低位两位进行标记和垃圾收集。该系统被解释并包括一个简单的标记扫描收集器和自由列表分配器。

大约一年后，我们创建了Z80 Scheme系统的第二个版本，进行了两次重大更改。首先，我们将收集器转换为标记扫描紧凑型收集器，重新发明了显然由Daniel Edwards首先提出的“双指”压缩算法[49]。该算法实际上比原始算法运行得更快，并且它允许我们使用更快的内联分配。其次，我们取消了对完全延续的支持，以便我们可以使用传统的递归堆栈。因为堆栈和堆相互增长，所以压缩堆也可以确保在内存真正耗尽之前系统没有耗尽堆栈或堆空间。总的来说，这个系统比原来快得多，但我很遗憾失去对完整延续的支持。

我们还尝试了Z80 Scheme系统的编译器。我们设计了一个类似于Lisp 1.5 Lisp汇编语言（LAP）的方案汇编语言[45]，将其实现为一组库例程以节省代码空间，并从Scheme源生成了对这些库的一系列调用。例程。遗憾的是，由于库调用开销，系统运行速度并不比直接解释器快，并且生成的代码比原始源代码大，所以我们放弃了编译器。

## 2.3 C-Scheme
1982年，我也开始实施一种新方案Curry Scheme（后来缩写为C-Scheme）[14]。该系统采用预处理器，用Scheme编写，并通过Z80 Scheme系统启动。预处理器执行了宏扩展，并且还负责处理应用程序和lambda表达式。这是我第一次编写宏扩展器的经历，也是我第一次使用bootstrapping。运行时系统和解释器最初是在Z80上的Pascal中实现的，但后来在VAX上用C语言重新编码。存储管理系统采用存储器的大包页（Bi-BOP）表示，其中存储器被分解为固定大小的段，并且单独的段表用于识别每个段中包含的对象的类型[51]。 。通过分配两个或更多个连续段来支持大于段的对象。通过将虚拟内存地址空间的最低和最高部分留空并将相应的段表条目设置为fixnums的类型代码来支持未装箱的本机整数（fixnums）。

虽然Chez Scheme尚未构思，但C-Scheme是向Chez Scheme迈出的重要一步。 Chez Scheme的初始运行时系统大量借用了C-Scheme，而C-Scheme用于引导Chez Scheme编译器的第一个版本。


## 2.4 Data General Common Lisp
令人难以置信的巧合，我在1982年前往北卡罗来纳大学研究生院之前就碰到了丹·弗里德曼的办公室，当时三角研究园数据通用的Jed Harris打电话询问Dan是否知道任何可能感兴趣的人来北卡罗来纳州帮助他们发起Common Lisp的努力。丹把电话给我，杰德和我在安顿下来后安排见面。我作为一名合同雇员聘用了我自己作为整个DG Common Lisp小组服务了大约一年，Jed不时地越过我的肩膀查看（我的工作成果）。计划是将我为C-Scheme所做的工作调整为运行时系统，使用Common Lisp核心的解释器，然后在稍后的时间移植编译器。到今年年底，我已经用DG的专有系统编程语言编写了存储管理系统，I / O系统，解释器和各种原语。此时，其他几个人被聘用，Spice Lisp编译器从CMU引入，与运行时系统结合使用，我退回到偶尔的咨询，这样我就可以更多地关注我的博士研究。
在1984年的夏天，我被要求全职工作在DG修理存储管理系统，该系统已被替换为比原来慢两个数量级的存储管理系统，并在第二个收集周期中崩溃。简单地回到我之前的代码不是一个选项，因为现在添加了新的对象类型，并且其他一些代表的表示已经改变。此外，虽然我的旧（垃圾）收集器比它的速度快，但仍然很慢，花了一两分钟时间在DG的旗舰MV / 10000计算机上收集8MB的堆。 （别笑。我们有报道称除了DG以外的其他更慢的（垃圾）收集器。）所以我被要求做一个更快的（垃圾）收集器。不幸的是，直到7月底我还没有完成任务，因为DG计划在8月初的同一个Lisp和Functional Programming以及AAAI会议上演示他们的Common Lisp。另一方面，我获得了一个出色的合作伙伴Rob Vollum，这使得任务变得更加轻松，更加愉快。工作了18个小时，我们有一个坚实的存储管理系统，快速收集器及时进行演示。

我们通过将所有系统数据结构保留在所谓的堆的静态部分中而获得了大部分性能（但仍然在跟踪）。这是生成清除的一个很差的变种[43,45]，我还没有听说过，但它仍然相当有效。从C-Scheme继承的BiBOP表示允许我们避免跟踪不包含指针的段，例如包含字符串的段，并且我们也不经常收集这些段以避免在它们被页面调出时将它们带入内存。最终结果是收集器平均大约15秒收集8MB的堆 - 按照今天的标准非常慢，但当时可观。

从各方面来看，LFP和AAAI的演示都取得了成功。不幸的是，据我所知，DG内部的营销部门出价并赢得了销售Common Lisp产品的权利，此后不久获得了一份利润丰厚的合同，并且Common Lisp产品从未真正看到过光明的一天。然而，我从该项目中学到了很多东西，我能够在我的Chez Scheme工作中使用。除了获得有关存储管理的宝贵经验之外，我还记得DG对质量保证的认真态度以及对广泛测试套件开发的坚定信念印象深刻。此外，虽然我没有机会直接使用编译器，但我确实编写了处理lambda列表的代码，这是一项令人不快的任务，这有​​助于推动我在以后的几年中采用更简约的语言设计方法。

# 3. Chez Scheme Version 1

在1983年秋季和1984年春季DG的假期，我认真地为我的顾问（Gyula Mago）的蜂窝计算机[44]设计并行实施Scheme，这是我从我决定的那天起的意图。去UNC上学。由于实际上还没有机器，我也开始编写模拟器（自然地在Scheme中）。不幸的是，C-Scheme不够快，虽然它比我尝试的其他Scheme系统更快。我将模拟器移植到Franz Lisp [33]，这是一个很好的系统，但是由于它对函数参数的不良处理感到沮丧 - 编译器似乎对解释器的支持 - 以及编译器之间缺乏一致的语义 - 解释。因此，我决定与我的研究工作同时进行Scheme的编译器的设计和构建，最终成为Chez Scheme。

作为设计过程的一部分，我分析了C-Scheme的实现，发现大部分时间都花在了可变查找和堆栈帧创建上。我突然意识到Scheme的典型实现模型都是错误的：通过堆分配环境和调用框架，它以更常见的变量引用为代价快速创建闭包，并且它以更多的代价快速地进行延续操作通用程序调用。在我们的第二个Z80 Scheme实现中，我们选择牺牲完整的延续来实现堆栈帧的堆栈分配，并且T的设计者也做了同样的事[47]，但我不愿意采用Chez Scheme的那条路线。相反，我开始考虑如何使延续“按自己的方式付费”，同时将负担从闭合访问转移到闭包创建。

延续的解决方案似乎显而易见：使用堆栈进行过程调用，通过将堆栈复制到堆分配的数据结构来实现连续捕获，并通过将堆栈副本复制回堆栈来实现连续恢复。在环境仍然分配堆的情况下，变量值永远不会直接存储在堆栈中，并且不会担心制作可变变量的多个副本。另一方面，我的最终目标是使用传统的堆栈帧，其中局部变量存储在堆栈中，我不确定这是如何解决的。

解决关闭问题有点棘手。在研究其他系统如何应对类似问题时，我遇到了Brian Randell和Lawford Russell [46]关于Algol 60实现的书，该书描述了使用显示来加速访问本地函数的自由变量。显示器是一组存储器位置或寄存器，每个存储器位置或寄存器指向其变量构成当前词法环境的帧之一。显示器不能直接用于我的目的，但我能够进行多次调整，并从显示模型中派生出一个显示闭包的概念，一个堆分配的矢量类对象，其中包含一个代码指针和自由变量的值[16]。除了允许对所有变量进行恒定时间访问之外，它还具有额外的好处，即闭包不会占用环境而不是它们需要的环境，这有可能使垃圾收集更有效。

赋值变量是这种表示的一个问题，因为变量的值可能会出现在多个修饰中。我通过“装箱”分配变量来处理这个问题，即用指向堆分配的单细胞对象或装箱的指针替换每个赋值变量的值，并保持实际值。 （如果变量出现在其范围某处的分配的左侧，则假定变量被分配。）后来我了解到Luca Cardelli在他的ML实现中使用了类似的闭合平面表示[11] 。在ML中，变量是不可变的，因此编译器不需要引入装箱。可以将引入的装箱视为ML ref单元的一种形式，但不同之处在于，在ML中，程序员必须明确地引入ref单元，而在Scheme中，编译器隐式地引入装箱。

装箱分配的变量也解决了堆栈问题，因为它允许局部变量的值（或装箱值）直接存储在堆栈帧中，而不用担心由于连续捕获而可能复制帧的事实。

使用新的闭包和延续模型，创建闭包的成本与自由变量的数量成比例，但访问变量值的成本变得很小且恒定 - 如果没有赋值给变量，则需要一个内存参考能够出现在其范围内，否则两个。创建或恢复延续的成本与堆栈的大小成比例，但调用帧变为堆栈而不是堆分配，节省了链接开销，减少了垃圾收集的频率，并允许重用帧的公共部分当多次非尾调用时。分配变量的使用也变得更加昂贵，但是已经（并且应该）在Scheme中不经常使用分配的变量，因此这并不是很重要。最令人担忧的是尾部呼叫的正确处理可能会成本更高。被调用者的参数必须放在与调用者的局部变量相同的位置，以防止堆栈增长，但通常需要调用者的本地生成被调用者的参数。版本1中使用的简单解决方案是将被调用者的参数置于调用者的本地上方，并在将控制权转移给被调用者之前将其调低。

当我对新的封闭和延续模型感到兴奋时，我开发Chez Scheme的主要动机从构建用于我的研究的工具转变为证明我的新想法可用于构建快速可靠的Scheme实现供别人和我使用。因为我发现其他系统缺乏可靠性和速度，所以我重新集中精力建立一个不仅快速而且可靠的系统，包括完整的类型和边界检查，包括堆栈溢出检查，甚至在编译时也是如此码。新的焦点被证明是一个强大的激励因素，到1984年夏天，我有一个解决方案和运行时系统，其中大部分都是从C-Scheme复制而来，但是包含了从我的DG Common Lisp存储管理器中借来的想法。我还编写了编译器的前端，包括实现新模型所需的赋值和闭包转换过程。不幸的是，当Data General打电话给他们的存储管理器时，我不得不休息一下，直到那个秋天我才能再次工作。

当我在秋季回到Chez Scheme工作时，我支持运行时系统，在UNC研究生Bruce Smith的帮助下，添加了一个bignum算法包。这只剩下一个主要任务，编译后端的构造。我本来打算遵循Lisp的传统并提供一个交互式使用的翻译，但是Luca Cardelli访问并展示了他为ML制作的光滑的交互式增量编译器，我受到启发，为Chez Scheme走同样的路线。这使得后端更加困难，因为我无法使用系统汇编程序和链接程序，即使没有进程创建开销也会过慢，但是必须编写自己的程序集。最后，这些结果非常简单，但当时看起来是一个巨大的挑战，我把它推迟了很长一段时间。然而，最后，到1984年底，Chez Scheme的第1版已经完成。

编译器和运行时系统的部分是用Scheme编写的，并使用C-Scheme引导。运行时系统的其余部分是用C和汇编语言编写的。该系统仅支持一种架构，即VAX和一种操作系统BSD Unix。

从高级意义上讲，编译器是天真的。它处理了一小组核心形式，并且只做了一件可以称之为高级“优化：”的东西，它将直接lambda应用程序绑定的变量视为局部变量，以避免分配闭包和调用该变量的成本关闭。正如我现在告诉我的编译器学生，“优化”和“不是愚蠢的”之间存在一条界线。这实际上是后者的一个实例。

我的重点是低级细节，比如选择有效的表示和生成良好的指令序列，编译器确实包括一个窥视孔优化器。高级优化很重要，我们后来做了很多，但是低级细节通常具有更大的影响力，因为它们通常会影响更广泛的程序类，如果不是所有程序的话。

一个重要的表示方法是在每个符号中包含一个代码指针槽以及一个值槽，以提高对全局绑定过程（包括基元）的调用速度。通过维护一个单独的代码指针槽，全局调用通过该槽无条件地跳转，编译器不必生成过程？检查每次全球通话。在创建符号时将槽初始化，或者将其分配给陷阱处理程序的地址，该陷阱处理程序修补代码指针并完成调用（如果值是过程），或者发出错误信号，如果值未绑定或不是过程。在第一次成功调用之后，陷阱处理程序因此被绕过并且控制直接传递给被调用的过程。

可以将这个想法扩展到不同接口的多个代码指针，从而避免参数计数检查。我决定不在版本1中执行此操作有以下几个原因：（a）额外的代码指针会膨胀符号大小或需要使用辅助数据结构，（b）处理多个代码指针会增加编译器的复杂性和存储管理系统，（c）运行时间的节省会不那么重要，因为参数计数检查比类型检查便宜，后者涉及段表间接，以及（d）代码大小的节省将是不太重要，因为参数计数检查是在入口点而不是在呼叫站点进行的，并且呼叫站点通常比入口点更多。

这就是Chez Scheme决策的方式，特别是在早期。 我们采摘低悬的水果，这很容易，产生最大的节省，剩下的时间用于以后，如果有的话，继续选择其他树上的低悬果实。 当一个更容易获得更大利益的东西等待完成时，尝试在性能的一个方面实现完美是没有意义的。
虽然我现在不记得细节，但是版本1优于我可以访问的其他Scheme和Lisp系统，尽管有完整的类型和边界检查，所以我觉得新的闭包和延续表示已经证明了自己。 我在1985年春季通过UNC分发了6份版本1，用户的反馈也是积极的。

# 4. Chez Scheme Version 1.1

在1985年春季，布鲁斯史密斯和我创建了一个参考手册[31]。我将参考手册中的许多示例程序转换为测试套件的开头，然后通过许多其他测试扩充了测试套件。在此过程中我发现并修复了一些错误，并在夏季完成了系统的1.1版本。与此同时，我的妻子Susan Dybvig和我创立了Cadence Research Systems来分发和进一步开发该系统。 Susan的MBA学位以及之前与一家小型软件公司的经验补充了我的培训和经验，并且我们在夏天成功实现了1.1版的第一次商业发货。我们当时和之后的所有利润都经过再投资，以支付与发展相关的成本 - 主要是劳动力。

Susan还与Prentice-Hall签订了合同，以发布我们的参考手册。 Prentice-Hall选择了“计划编程语言”这一标题，显然是为了利用他们也发表的C编程语言[39]的成功。我很满意这一点，但它在我的脑海里大大提高了标准，最后我在1987年最终出版之前重写了大部分文本。不幸的是，我的合着者布鲁斯史密斯无法花太多时间在努力，我最终要求并获得他自己完成项目的许可。我永远感谢他对系统和书籍的早期重要贡献。

最初，我编写了所有的代码和文档，我们使用自制的Z80 PC来管理我们的业务，它配备8Mhz Z80H处理器，128K RAM和20MB硬盘，实际上比任何一个都要快。新的IBM PC，甚至是我在UNC使用过的VAX系统。 （公平地说，我已经定制了我的Gosling的Emacs副本来模拟Word88，这是我们在Z80上使用的单词处理系统。这大大减慢了Emacs的速度，并且可能解释了为什么VAX似乎变慢了。）构建和测试这是在北卡罗来纳州微电子中心（MCNC）的VAX计算机上完成的，作为交换，我们为Chez Scheme提供了免费许可。我们最终购买了一个Sun工作站并将我们的开发工作转移到该平台，我们不时购买或提供其他系统，但我们继续以客户为单位进行易货交易或借用时间进行构建并在我们所做的平台上进行测试没有内部。

我们购买的第一台Sun工作站是他们最低端的系统。我本来希望有一个更快的系统和更多的内存，当然，如果我们能够证明成本合理。另一方面，我认为让低端机器具有各种优势：如果我能让Chez Scheme运行良好，Chez Scheme在任何事情上都能运行良好。

|第1版亮点|
|-------|
|发行年份：1984(版本1.0),1985(版本1.1)|
|语言：R2RS兼容性，fixnums，bignums，比率，flonums，简单宏，保存的堆，保存的可执行文件，引擎[27,35]，跟踪支持，多个服务员和咖啡馆，原始宏系统，计时器和键盘中断，属性列表， 原始格式，动态风，流体绑定|
|实现：增量编译器（无解释器），自定义链接器，平面（“显示”）闭包，基于堆栈的控件表示，已分配变量的装箱，带保留（16位）fixnum范围的BiBOP类型，停止和复制收集器 ，代码指针缓存在符号代码指针槽中，窥孔优化|
|文档：Chez Scheme参考手册版本1.0 [31]，Unix手册页|
|平台：Vax BSD Unix|


# 5. Chez Scheme Version 2

1985年秋天，我们搬到印第安纳州布卢明顿，在那里我加入了印第安纳大学的教师队伍。在学年期间，我完成了开发工作，同时我准备并教授了一整年的编译课程和另一门课程。然而，在春季学期的某个时候，桑迪亚国家实验室的George Davidson曾使用过Chez Scheme几个月，他们要求我们创建一个从VAX到嵌入式MC68000系统的交叉编译器。这是一个很好的机会，可以使用MC68000代码生成器，后来我们可以将系统移植到Sun，Apollo和其他几个平台上。我决定从我的编译器课程中聘请一名研究生来帮助我度过这个项目。

编译器课程中充满了非常优秀的学生，但其中一个在其他学生中脱颖而出（字面上和形象上都是如此）。 Bob Hieb身高6英尺6英寸，宽肩，浓密的头发，留着浓密的胡须，尖锐的特征，以及由法兰绒衬衫，牛仔裤和靴子组成的典型服装。他看起来更像一个伐木工人而不是一个计算机科学家（我后来才知道他已经花了十年时间作为木匠），但他在课堂上的表现显示出巨大的潜力，所以我给了他这份工作。他很高兴地接受了，并且在他1992年不幸去世之前，我们最终共同工作了七年。

在完成MC68000交叉编译器之后，Bob和我一起研究了实现的其他方面。我们实现了不同的优化级别，这些级别实际上只是标志告诉编译器它可以做某些事情。在优化级别1，允许花费更多时间。在优化级别2，允许假设基元的全局名称确实绑定到那些基元。在优化级别3，允许生成不安全的代码。我们还在编译器中引入了一些代码来优化letrec表达式，优化循环和内联大多数简单的原语
由于编译器在Scheme中编码并在引导后从其自身的优化中受益，因此这些改进使编译器本身更快。实际上，优化不仅弥补了编译器为实现优化所做的额外工作。这是一件好事，因为我们担心编译器速度（编译器毕竟是交互式使用和加载源文件）以及有效性。在某些时候，我们实际上制定了以下规则以限制编译开销：如果优化不能使编译器本身足够快以弥补执行优化的成本，则优化将被丢弃。这排除了我们尝试的几个优化，包括早期尝试源优化器。

我们还在收集器上工作，看看我们是否可以改善其性能。在算法方面有很小的选择，这已经和我们知道如何制作一样紧。但是，通过将关键例程定义为C预处理器宏而不是C函数，我们能够从“10％到33％的改进”中获取（根据代码中的注释）。

除了移植和加快系统工作外，我们还采用了一些新的语言功能，包括支持低级扩展传递式宏定义[24,25]和高级扩展语法宏定义[41] 。直到很久以后我们才采用卫生宏观扩张[41,42]，因为二次扩张成本以及我们直到很久以后才解决的机制的其他局限性。

我们还添加了一个用于创建“可变arity”过程的新功能，即具有多个接口的过程，可选参数的泛化。 case-lambda表达式类似于lambda表达式，但有多个子句将形式参数列表与正文配对。当调用使用case-lambda创建的过程时，将根据接收的实际参数数选择相应的子句。我们的原始设计要求使用不同的语法替换点接口，允许一个或多个案例接受无限数量的参数而不承诺对这些参数的任何特定表示。这有效地从界面中删除了列表，以及列表可以通过优化对无限期程序的调用而导致的各种困难[26,28]。我们退回了这个功能，而是选择了一个不那么激进的版本，其中每个子句的形式参数列表是一个普通的lambda形式参数列表。

版本2于1987年发布，与Prentice-Hall出版的The Scheme Programming Language大致相同。

# 6. Chez Scheme Version 3

在版本2和版本3的版本之间，我们继续调整编译器和运行时系统以提高性能，但我们大部分时间都花在移植上用于一对新的RISC架构，从而提高了Chez Scheme与其他语言的互操作性和流程，并提高系统的整体可用性。

改善性能和可用性的一个变化是采用了新的延续机制。在版本1和版本2中，捕获延续意味着复制整个堆栈，并重新启动延续意味着将其全部复制回来，这意味着延续操作可能变得非常昂贵。我们用一种新的分段堆栈方法解决了这个问题，鲍勃和我与另一位毕业生Carl Bruggeman一起开发了这种方法[36]。这种机制消除了在捕获延续时复制堆栈的需要，并且在恢复延续时减少复制到少量恒定字数（或最多为最大帧的大小）的数量，而不添加任何开销正常的过程调用和返回。该机制还支持自动堆栈溢出恢复，只要堆空间仍然可用于分配，堆栈空间永远不会耗尽。

通过好运而不是设计，新的延续机制还使我们能够根据Olivier Danvy的请求修改跟踪包，以反映非尾部调用之间的差异（通过增加跟踪显示嵌套级别和显示返回值）和尾调用（两者都没有）。挑战在于识别对跟踪过程的哪些调用是非尾调用，哪些调用是尾调用，最初我们感到难过。如果跟踪包被内置到解释器中，则解释器可以跟踪此信息，但我们没有解释器。相反，跟踪包通过将每个跟踪过程嵌入另一个过程来操作，该过程是一个负责打印跟踪信息的跟踪包装器。解决方案是跟踪包以维护跟踪连续变量，该变量始终保持最后一个跟踪调用的延续。每个跟踪包装器将其自身的延续与跟踪延续进行比较。如果它们相同，则发生尾调用，如果它们不同，则发生非尾调用。对于非尾部调用，跟踪包装器在应用嵌入式过程之前将跟踪延续设置为新的当前延续，并在之后恢复旧的跟踪延续。使用continuation的分段堆栈表示，可以使用单个eq? test进行连续的比较，即指针比较。

我们的第一个RISC移植是摩托罗拉MC88000，是在摩托罗拉的Sam Daniel的要求下完成的，摩托罗拉多年来一直支持摩托罗拉的Chez Scheme，甚至设法说服摩托罗拉在其Delta系列MC68000和MC88000系列上销售Chez Scheme机器一会儿。 MC88000是我们承担的最困难的移植之一，在一个夏天的大部分时间里，我花了三个人的共同努力（我招募了Carl和Bob一起帮忙）。部分原因是它的RISC架构与CISC VAX和MC68000架构完全不同，但主要是因为操作系统，C编译器甚至硬件在移植时仍处于活跃开发阶段。当然，这意味着要处理几个编译器和操作系统错误，但最大的挑战是硬件错误。一个是挑战，部分是因为它在我们设置断点或单步执行代码时进入隐藏状态。事实证明这是一个竞争条件，在从硬件返回地址寄存器移动到我们自己的返回地址寄存器中时缺少记分板（忙）检查。当我们从断点开始或单步执行代码时，寄存器不再繁忙，我们的代码运行没有问题。但是，当全速运行时，我们的代码偶尔会尝试过早访问寄存器，而不是被迫等待，会得到错误的（旧）值，通常会导致程序返回（最终）给调用者的调用者而不是自己的调用者。

另一个移植是Sparc架构，而且更顺畅，因为我们已经移植到一个RISC架构，操作系统基本上与MC68000 Sun系统相同，工具和硬件更稳定。在同一过程中与其他流程和其他语言的互操作性是我们的一些用户的优先事项。这促使我们添加了用于加载外部目标文件和在这些文件中输入入口点的工具，并自动将语言和返回值数据类型转换为C和Scheme表示。这些工具是在其他Lisp方言的实现中的类似特征之后建模的。我们还添加了对子进程中运行其他程序以及通过Unix管道与它们进行交互的支持。

我们为版本3和所有其他新版本编写的大多数新代码都是用Scheme编写的，而一些最初用C编写的代码已经转换为Scheme，它可以在更多代码中编写抽象风格不太可能受系统变化的影响，例如，对象表示。我们在使用版本3时改变的最多代码是Chez Scheme读取器，它实现了Scheme读取过程，并在加载或编译源程序时使用。我们期望读取器会有点慢，但不足以证明让读取器留在C中。尽管Scheme版本使用了相互尾递归的例程，但是Scheme编码的读取器实际上变得更快了。词汇分析，而C语言的时间关键部分在循环时使用。虽然差异可能部分是由于我们可以相对容易地调整Scheme代码，但很高兴知道Chez Scheme的编译器开始达到可以与C编译器竞争的程度，C编译器具有更容易完成的工作。当我们对寄存器分配进行了重大改进，包括将寄存器分配给过程参数时，Scheme编码的读取器在下一版本中变得更快。

版本3于1989年发布。

# 7. Chez Scheme Version 4

版本3发布后不久，我们开始对系统进行重大改革，改变它表示Scheme值的方式，并重写编译器和存储管理系统的主要部分。在这样做时，我们打算删除一些对象大小的限制，并且一如既往地提高性能和可靠性。我们还希望让系统具有更强大的基础，考虑到我们已经制定或知道我们希望在未来进行的系统修改。随着新功能被移植到最初并非旨在支持它们的基础上，不时过度运行的系统可能会受到自身的影响。

我们坚持使用C-Scheme通过Chez Scheme的前三个主要版本采用的BiBOP机制。由于类型信息存储在单独的表中，因此BiBOP机制允许我们对fixnums和指针使用本机表示，并避免在任何数据结构中占用类型信息的空间。另一方面，进行类型检查的成本相当高。它涉及从对象的地址中提取段地址，使用它作为段表的索引，并进行测试和分支。
我们也只是感受到另一个问题的影响，这是由典型物理内存和后备存储大小的快速增加引起的。这些增加产生了相应的压力，以增加弦和矢量的最大尺寸。这意味着增加fixnum范围的大小，因为这些对象的索引表示为效率的fixnums。与此同时，典型物理内存和后备存储大小的增加也意味着进程使用了​​更大比例的虚拟内存地址空间。不幸的是，我们的fixnum范围是通过牺牲一部分地址空间来获得的，如2.3节所述。增加fixnum范围以允许更大的字符串和向量将减少放入它们的空间量。这个问题迫使我们考虑从BiBOP模型转向标记指针模型。

然而，BiBOP模型有许多我们不愿意失去的好处。通过将包含指针的对象与不包含指针的对象隔离，它允许垃圾收集器避免不包含指针的扫描对象（如字符串）。更一般地说，它允许收集器使用不同的方法来扫描不同类型的对象，这反过来允许对某些对象使用更聪明的表示，例如包含到其字符串缓冲区中间的“下一个”指针的I / O端口或包含代码指针到代码对象中间的闭包。它允许将动态生成或加载的代码放置在堆中的不同段中，以便可以从仅包含数据的页面为代码页提供不同的保护。它还允许存储管理器优雅地处理虚拟地址空间中的“漏洞”;当O / S或其他语言运行时为某些其他目的保留空间时，可能会出现漏洞。

最终，我们决定改用混合模型[23]，它使用标记指针来区分特定类型的对象，并使用BiBOP来获得各种好处。我们不再使用BiBOP机制按特定类型隔离对象，而是使用收集器感兴趣的特性，例如它们是否包含指针以及它们是否可变。对于我们标记指针的实现，我们采用了T [47]使用的低标签模型，具有不同的标签分配。混合机制允许将fixnum大小增加到30位并且在许多情况下减少类型检查开销，而不牺牲虚拟地址空间或上述BiBOP机制的益处。

更改对象和指针表示以各种可预测的方式影响编译器。例如，它必须生成不同的代码来访问或变异对象，执行算术和类型检查。此更改还允许我们切换到内联分配。因为不同类型的对象不再被它们所驻留的段的类型区分，所以我们能够将大多数对象（除了代码对象之外的所有对象）分配到单个内存区域中，然后隔离在第一次收集后幸存的对象。这意味着我们只需要一个分配指针，我们可以将其放入寄存器中，并且分配代码序列变得足够小以证明内联某些分配操作是合理的，例如对和闭包。这大大提高了许多程序的性能。

这些更改只是我们在重写编译器时所执行的几项任务之一。另一项任务是用编译器特定的记录结构或c记录替换用于表示中间代码的列表结构。因为每个c记录都是扁平结构而不是链表，这减小了中间代码的大小和访问每个中间形式的子表达式的成本。同时，它允许我们减少调度开销。可靠性也提高了，因为每个记录的形状在创建和调度站点都是静态检查的，并且因为c记录是不可变的，所以防止了无意的修改。

另一个任务是用一个过程内寄存器分配器替换我们的本地寄存器分配器。同时，我们更改了Chez Scheme的调用约定，以允许过程参数在寄存器中传递。在版本4之前，所有参数都在堆栈位置传递。虽然这在CISC体系结构上很常见，它直接在内存上支持许多操作并且具有相对的效率，但在RISC体系结构上肯定不是一个好主意。使用一种强调过程调用循环作为重复的基本机制的语言，我们需要一种能很好地处理调用的机制。不幸的是，寄存器分配文献几乎专注于为传统（Fortran）基于循环的程序生成良好的代码，并且要么没有解决过程调用，要么以粗略的方式进行。我们简要地考虑了使图形着色寄存器分配[12]适应我们的需求，但它没有为调用提供特殊帮助，并且潜在的编译时成本超出了基于增量编译的交互式系统的合理性。

所以Bob和我开发了自己的寄存器分配算法。该算法首先将寄存器分配给传入的参数，然后按先到先得的原则分配给自下而上传递中找到的绑定，从过程的抽象语法树的叶开始。因此，包含不再需要的值的寄存器可用于其他值，该算法仅对寄存器值采用昂贵的实时分析形式，允许使用廉价的fixnum逻辑运算。该算法的初始版本保存并恢复了每个非尾调用的实时寄存器值。算法运行后，我们为一组基准程序收集了各种动态信息，包括我们自己的编译器。基于这些信息，我们切换到“延迟保存”策略，其中尽快保存实时注册值，但不是在呼叫不可避免之前保存。我们还添加了每次通话时使用的一种避免机制。洗牌器在每个调用站点重新排序参数以减少寄存器保存的数量，并允许将参数值直接放在它们的传出寄存器或堆栈位置，几乎没有或没有额外的移动。 Shuffling避免了第3节中描述的尾调用参数的移位。寄存器分配算法虽然主要是为了快速设计，却出人意料地有效。算法的精炼版本在几年后发布[7]。

另一项任务是改进编译器对内联基元运算符的支持，以允许部分内联无法完全内联的运算符。我们使用它来内联许多基元的安全版本，这些基元遵循外部错误处理程序并内联各种通用数字运算符（如+）的fixnum情况。同样的机制允许我们引入一些平凡但有用的程序转换，比如取代eqv？用更便宜的eq打电话？当一个参数不变并且与eq？相容时调用。

最终的编译任务是改进对浮点运算的支持，并增加对复杂运算的支持。我们添加了一组新的flonum运算符，如fl +，它们以不安全的代码内联，并以安全代码部分内联。我们添加了一组类似的“复杂的flonum”运算符，如cfl +，它们用于组合flonums和不精确的complexnums。在支持复杂的操作方面，一个主要的挑战是设计一个有效的不精确复杂的表示。我们不希望额外的间接参与将它们表示为一对指向flonums的指针，但我们也想避免在提取实部或虚部时分配flonum对象的成本。

我们能够通过一个简单的黑客来避免间接和分配的开销，这种黑客也将我们的flonums的大小减少了一半。因为收集器在对象工作时移动对象，所以每个对象需要空间留下转发标记和地址。这是浮点数的问题，因为我们选择的任何标记都可能与原始浮点数据无法区分。在支持IEEE浮点数的系统上，我们考虑将前向地址编码为与NaN对应的位模式，但发现某些体系结构和操作系统没有记录可能由其指令和库产生的NaN集。因此，每个flonum中包含额外的空间用于转发标记和地址。黑客攻击是删除此空间并修改收集器，以便它永远不会在flonum中存储转发地址。当然，如果存在两个指针，可能会复制一个flonum，但不会造成特别的困难。复制可以通过eq？检测到，但这并不违反eq？的语义，当给定两个数字参数时，总是允许返回#f。一旦我们做了这个改变，我们就能够将不精确的复合体表示为双浮点数（又称为flonums），并使用简单的指针算法来提取实部和虚部。

Scheme值的表示形式的变化，新的复数类型的添加以及从flonums中消除转发地址导致我们对收集器进行一些调整，但这些都相对较小。我们为自己设定的主要挑战是将我们的停止和复制收集器转换为世代收藏家[43,45]。与我们的价值表示变化一样，这种转换的动机是典型物理内存大小的增加。随着内存大小的增加，典型堆的大小也随之增加，并且收集成本随之增长。我们转向世代收藏以降低成本。分代收集基于这样的理论：通过多个集合幸存下来的旧对象比年轻对象更不可能是垃圾，因此不需要经常收集。

我们的变体[23]采用了几代，其数量是在系统建立时确定的。默认情况下使用五代，其中第0代是最年轻的，第3代是最早的可收集生成，4是在加载系统和应用程序“引导”代码后在初始堆中包含代码的静态生成。新对象被分配到第0代，当对象在年轻一代的一定数量的集合中存活时，它们被提升到更高的一代。程序员可以设置收集每一代的频率和一个对象在升级到更高级别之前必须存活的数量。默认是每运行收集器4n次收集生成n，因此每次收集生成0，每隔四次收集第1代，第16次生成第2代，第64次生成第3代。这个相当随意的策略最初只是一个测试的黑客，但事实证明它运行良好，确实比我们尝试过的几个更精细的策略更好，因此它一直是默认的。它在减少收集的平均时间方面做得很好，没有与动态分配的对象“过早使用”[54]相关的潜在问题，这些问题从未被收集过。

我没有记录我们在大修期间所做的各种改变带来的性能优势，但是运行时速度的总体改进大约是50％。也许更令人印象深刻的是，尽管添加了新的寄存器分配通道，但编译器的速度提高了30％。

虽然我们对版本4的大多数更改都与大修有关，但我们还添加了几个新功能。其中一个功能是一个新的检查员，具有程序和交互界面。检查器允许程序员在适当的情况下查看和修改任何对象的内容，包括最重要的是继续发生错误或键盘中断。为了支持源信息的列表以及通过延续或闭包中保存的值的变量名进行适当标记，我们必须对编译器进行其他更改，以便通过各种传递跟踪源信息。事实证明，这是一个向每个c记录添加源字段，通过每个传递传播信息，最后将信息与条目地址相关联并在生成的代码对象中返回点的过程。在不牺牲性能的情况下支持检查，因此我们能够在解决系统中明显缺陷的同时保持对我们的优先级的忠诚。

# 8. Chez Scheme Version 5

我们都准备在1992年夏天开始研究第5版，当时Bob Hieb在车祸中丧生，同时夺走了女儿Iva的生命。这一悲惨事件可能会破坏Chez计划的发展，但我通过忙着完成我们为Chez Scheme开始或计划的开发工作以及IU的一些未完成的研究工作部分地处理了我的悲痛。

Carl Bruggeman对后者特别有帮助，因为他介入与我一起完成了syntax-case宏系统，这是Bob博士研究的核心。我们完成了该系统的工作以及同年描述该系统的一对技术报告[17,17]，并发表了一篇期刊文章，该文章将一年后两份技术报告的部分内容结合起来[29]。我将syntax-case系统整合到Chez Scheme中，并且还发布了一个便携版本，从那时起就与Chez Scheme版本保持同步。

我在1992年做出的大多数其他系统更改都是代码生成，编译时检查等的常规改进，但一个主要任务是优化本地调用。到目前为止，大多数本地电话涉及一个程序？检查并间接跳过程序的闭包。唯一的例外是直接调用被认为是let表达式和调用参与循环的等价，编译器认为这是循环。全局调用实际上更便宜，因为第3节中描述的代码指针黑客避免了这个过程？校验。

为了优化本地调用，我修改了编译器，以便在未指定的变量被let-或letrec直接绑定到lambda或case-lambda表达式时进行记录，然后为通过这些变量进行的调用生成更高效的代码。特别是，编译器删除了程序？检查，通过跳转到lambda体（通过参数计数检查或直接进入适当的case-lambda体）替换闭包间接，仅当过程具有自由变量时传递闭包指针，并且，对于“rest”参数，在调用参数数量的调用站点分配列表，不需要循环。在许多情况下，仅以这种方式调用过程。如果它没有自由变量，则永远不会使用闭包，因此编译器会同时删除闭包创建代码和let或letrec绑定。编译器通过识别在一组多个递归过程中对闭包的唯一需要是何时保持其他过程的闭包来安排更多过程属于此类别。

本地调用优化产生了很大的差异 - 从15-50％ - 与我们的基准测试版本相比，通过将每个基准测试包装在let表达式中，全局调用用户定义的过程已转换为本地调用。再一次，自举编译器受益于自己的优化，因为编译器的平均速度提高了约25％。

回想起来，这些优化似乎并不困难，我的研究生编译器课程的学生能够在每周几次的任务中实施它们。然而，当时我发现它们极具挑战性。当然，我没有分配说明告诉我如何继续，而我的中间语言更复杂。造成这种困难的另一个原因是，当我可能应该添加一个或多个单独的传递时，我已经将优化归结为现有的编译器传递。额外的通行证并不像我当时认为的那样昂贵，并且编译器在以后几年可以修改的复杂性和相对容易性的降低将弥补编译时间成本的适当增加。

1993年，我招募了一名研究生Mike Ashley，与我合作，支持多种回报价值。我们想要一种机制，就像我们的延续机制一样，在正常的，单一价值的回报和回报点的效率不受影响的意义上支付自己的方式。我们还希望机制在收到错误数量的值时发出错误信号。我们能够通过一种能够有效处理多值和单值返回以及返回点的机制来实现这两个目标[2]。

我还招募了另一位研究生Oscar Waddell将Chez Scheme移植到运行Digital OSF / 1操作系统的Alpha处理器。这是奥斯卡与我共同合作的众多项目中的第一个。除了Alpha端口本身之外，该项目最有用的功能是从系统源中删除汇编代码。我们一直在使用机器相关的m4 [40]宏和大多数与机器无关的汇编文件的组合，以减少移植到新架构所涉及的汇编代码量，但这很难处理，并没有完全解除我们从必须处理汇编语法的许多特性。我们使用以机器无关的汇编语言编写的代码替换了汇编代码，该代码通过将代码输入到编译器的代码生成器中实现。

对于那些喜欢简单而有效的黑客攻击的人来说，值得一提的另一个版本5更改。这种特殊的黑客攻击将创建生成的符号或gensym的成本降低了25倍。黑客是将gensym的名称生成延迟到第一次打印，这样（gensym）就变成了廉价的内联分配操作。例如，在宏观扩展期间创建的大多数gensyms从不打印，因此节省是真实的，并且对使用gensyms的程序产生重大影响。

# 9. Chez Scheme版本6

系统开发在版本5和版本6之间继续快速发展。我们通过读取器，扩展器和编译过程添加了对源文件信息跟踪的支持，以便检查员可以显示原始源代码而不是扩展器输出。我们还增加了对不相交记录类型，一次性延续[4]，模块[56]以及我最喜欢的新语言功能之一的支持，即基准注释。使用语法＃;的基准注释会注释掉整个S表达式，例如，（a＃; b c）读为（a c）。在看到人们使用引号来评论顶级表达式之后，我发现需要这样的事情，并且当同一个技巧并不总是远离顶层时会感到沮丧。

我们还添加了对从C调用Scheme的支持，它建立了嵌套Scheme / C调用的可能性。通过在调用Scheme之前保存C堆栈上下文（使用setjmp），我们避免了延续的困难。这个上下文在从Scheme返回的第一次返回时被恢复（使用longjmp），此时上下文也被标记为无效以及任何动态下属上下文。尝试返回无效的上下文会导致发出错误信号。这允许在Scheme侧执行任意连续操作，只要不尝试将第二次返回到C帧或在C堆栈中动态嵌套在其中的帧。

我的另一名研究生Bob Burger被聘请将Chez Scheme移植到两个新的架构，HP / UX下的HP PA-RISC架构和AIX下的PowerPC架构。我还根据Guy Steele和Jon White [53]的一篇论文，结合了Bob在1990年写的一个浮点打印机的几个改进。 Bob的改进使算法更简单，更有效[5]。 Bob的博士研究是关于轮廓驱动的动态重新编译[6]，尽管我们从未采用动态重新编译支持，但我们在Mike Ashley的帮助下将Bob的分析支持纳入了Chez Scheme。

我们的第6版目标之一是提供完整的运行时系统，以简化已编译的Chez Scheme程序的交付。如果没有eval，Scheme的运行时系统就不会完整，所以我们决定在第2版中包含一个我们已经悄悄地进入系统的解释器，以帮助进行交叉编译。使用解释器和运行时系统的其余部分，我们有99％的工作Scheme系统，所以我们决定包含read-eval-print循环并释放运行时系统，将解释器作为一个完整的独立系统。该系统是使用与Chez Scheme相同的源构建的，只剩下编译器。我们将系统命名为Petite Chez Scheme，因为没有编译器它比整个系统小。我们无法拒绝对解释器进行一些调整以在释放系统之前加快速度。我们得到了很多关于翻译速度的积极反馈，人们常常惊讶地发现它是用Scheme编写的。出于某种原因，他们认为用C编写的解释器应该更快，尽管用Scheme编写并由一个好的Scheme编译器编译的解释器可以利用内置支持来正确处理尾调用和连续，以及编译库代码。

当然，与往常一样，我们也致力于提高编译代码的速度。我们多年前编写了一个源优化器，但放弃了它，因为它未通过我们的性能测试，即没有使引导编译器的代码足够快以克服所涉及的额外通道。然而，在1996年初，Suresh Jagannathan和Andrew Wright [38]的论文“Flow-directed inlining”的预先复制品促使我们采取行动。这篇论文很有意思，因为它声称一系列实质性计划项目的速度有了令人印象深刻的改进。令它特别有趣的是，这些改进是通过优化器完成的，该优化器作为Chez Scheme编译器的预处理器运行。一些收益原因是由于全局调用转换为本地调用，触发了第8节中描述的本地调用优化，但即使在对此进行调整之后，结果也太好了，无法忽略。虽然他们的分析的编译时成本对于我们的编译器来说是不切实际的，但我们现在知道如果我们能够弄清楚如何获得好的收益。
迈克阿什利已经开始研究他自己的流量分析仪，并且他采用了它来实现类似于Jagannathan和Wright的算法。他实际上能够稍微改善他们的结果，但是尽管他的流量分析仪速度要快得多，但对于我们的目的来说仍然是不切实际的。

所以Oscar Waddell和我开始创建我们自己的内联器，其约束条件应该是快速和线性的。我们希望它也能产生与流动导向内衬所达到的结果“相同的结果”。经过几个月的工作，我们有一个比我们希望的更好的内衬，为所有基准产生至少可比较的代码，并且在少数情况下实际上击败了流向内置的内衬[55]。一个原因是我们的算法的“在线”性质，它允许内联器基于它已经转换的子表达式做出决策，而流向内联的内联者“离线”运行分析，即完全在任何转换发生之前。另一个原因是我们在决定何时内联和何时不使用时使用的方法。内联器简单地进行所有内联尝试，并在剩余代码的大小或尝试所花费的时间超过预定限制时将其切断。在确定这种严谨的方法之前，我们尝试了几种启发式方法来限制代码大小和内联时间，但启发式方法不可避免地会抑制或允许更多内容。我们将内联器合并到Chez Scheme编译器中，并将其用作Petite Chez Scheme中的解释器预先通道。

在此期间，Oscar Waddell一直致力于开发GUI API，以支持在教学中使用Scheme作为NSF支持的教育基础设施项目的一部分。这个系统的初始名称是“Bob。”我认为这是同名短命的微软产品的起飞，但我不确定。在任何情况下，名称都没有坚持，而是采用名称Scheme Widget Library。缩写的SWL和公布的“swill”，这个名字仍然反映了奥斯卡在早期对系统的不良看法。然而，该系统最终非常好，包括用于Scheme的交互式开发环境以及用于构建图形应用程序的工具。奥斯卡投入了大量的工作，其他几个人也做出了贡献，特别是Carl Bruggeman，Bob Burger，Erik Hilsdale和John Zuckerman。 Abstrax公司的John LaLonde也支持这个项目，我们从几年前在摩托罗拉工作时所写的系统中借鉴了一些想法。该系统现在与Chez Scheme一起分发，我自己做了一些小的贡献。许多程序员仍然使用Chez Scheme和emacs，像我这样的真正的高级用户将它与vi一起使用，但是许多其他人使用SWL作为Chez Scheme或Petite Chez Scheme的接口，特别是在Windows和Mac下。

# 10. Chez Scheme Version 7

虽然在版本6和版本7的版本之间经过了多年，但我们并没有闲置并发布了几个小版本。我们添加了各种新功能，如bignums上的逻辑操作，文件压缩，支持Scheme shell脚本（包括脚本编译），Windows注册表原语，eq？哈希表和apropos。我们扩展了gensyms以允许创建全局唯一标识符，添加记录继承，并添加了对基于应用程序可执行名称自动加载堆和引导文件的支持。在疯狂的情况下，我还实现了Common Lisp格式，这可能非常有用，但即使在实现之后我也永远无法记住哪个指令是哪个和哪个
特别是用于完成特定任务的冒号和符号修饰符的组合。我原本打算实施一些不太精细但不知道在哪里停下来的东西。

除了这些相对较小的变化之外，Oscar和我首次将系统移植到64位架构，并基于Posix Threads创建了一个多线程版本的系统，允许应用程序使用多个处理器和处理器内核。针对64位Sparc架构的64位端口应该相对简单，但我们被许多32位依赖项绊倒了。您可能认为经历过从16位微处理器地址空间到32位地址空间转换的人会避免这种依赖性，但当然，我从未想过Chez Scheme会在很长时间内将其作为一个问题。至少我们现在已准备好过渡到128位地址空间。

线程系统带来了许多挑战，特别是在使系统的某些部分成为线程安全的时候。幸运的是，我们早期的一些设计决策简化了项目的其他方面。我们坚持使用BiBOP和分段存储器的决定使每个线程的个别分配区域变得微不足道。个别分配区域对于避免在每个分配操作上进行同步至关重要，这将是一个性能障碍。我们的分段表示也有所帮助，因为堆栈溢出恢复机制允许每个线程以一个小堆栈开始，该堆栈通过根据需要添加新段来增长。我们启动收藏的模式也有所帮助。集合通过中断触发，中断由段级分配器根据自上次集合后分配的字节数设置。这很好地概括了允许多个线程同步进行收集。当线程收到收集中断并且其他线程仍处于活动状态时，它会等待其他线程。当最后一个活动线程同步，阻止或退出时，将启动该集合。

然而，跟踪对老一代的分配证明是一个问题。只要老一代对象中的某个位置包含指向年轻一代的指针，就必须通知世代收藏家，以便在年轻一代的收集过程中对其进行追踪。我们的收集器使用经过修改的卡片标记[50]系统，其中此信息存储在全局表中。不幸的是，存储信息不是原子操作，并且对每个变异操作进行同步都过于昂贵，即使有人认为程序应该尽可能没有变异。我们通过在本地分配区域的末尾维护变异位置的日志来解决这个问题，日志指针朝着分配指针增长，因为Z80 Scheme系统的第二个版本已经为堆栈和位置指针完成了。在日志和分配指针满足之前不需要进行同步，此时扫描记录的条目以记录关于从较旧代到较年代的指针的信息，并且如果需要则获得新的分配区域。

# 11. 其他

开发像Chez Scheme这样的系统是一个连续改进的过程。即使我在1983年知道我今天所知道的事情，在今天的一项持续发展努力中创建Chez Scheme仍然是难以想象的困难。如果我尝试过这样的事情，我可能会在它开始之前放弃努力。相反，我开始使用一个表示Scheme值的简单模型，一个执行很少优化的小型编译器，一个简单的停止和复制垃圾收集器，以及一个小型运行时系统。随着功能的添加，扩展，删除和替换，它随之增长并缩小。

当然，为了这个长达数十年的成功发展努力，我们不得不做一些经常不愉快的事情，比如我们认为很聪明的丢弃代码，重写我们希望再也不用看的代码，即使在不方便时也能及时修复bug，并扩展添加新功能时的测试套件。如果我们不这样做，系统就会充满无用的，狡猾的，错误的和未经测试的代码，使得难以维护和扩展，甚至使工作的有趣部分变得像编写新代码一样令人不快。

随着时间的推移，我们的优先事项也变得更加精细，特别是在效率方面。我一直认为效率不仅仅取决于编译代码的速度，还取决于编译时间，内存使用情况和整体系统响应能力。多年来，我开始相信性能的一致性和连续性也很重要。我也更关心处理大量数据的大型程序或程序。除了效率和可靠性之外，还出现了额外的优先级，例如标准一致性，与其他语言编写的代码优雅交互的能力以及整体系统可用性。

即使经过二十年的改进，该系统也无法满足我的需求。有许多方法可以改善性能，代码可以更清洁的众多地方，以及我想添加或扩展的众多功能。我们的待办事项清单有数百个条目，从简单的家务到研究项目。我没有抱怨。如果待办事项清单是空的，我会知道是时候打包了。

# 致谢

除了本文中提到的直接参与Chez Scheme工作的人们之外，还有更多，无数提及，以其他方式支持开发。 没有他们的贡献，Chez Scheme就不会是今天的样子。 如果不是因为朱莉娅·劳尔和其他计划委员会在ICFP '06上就这一主题发表演讲的邀请，这篇论文就不存在了。 Oscar Waddell，Julia Lawall，Susan Dy-bvig，Dan Friedman和Olivier Danvy的建议引发了该论文的许多改进。

# 引用

[1] Harold Abelson, Gerald Jay Sussman, and Julie Sussman. Structure and interpretation of computer programs. MIT Press, Cambridge, MA, USA, 1985.

[2] J. Michael Ashley and R. Kent Dybvig. An efficient implementation of multiple return values in Scheme. In Proceedings of the 1994 ACM Conference on Lisp and Functional Programming, pages 140–149, June 1994.

[3] G.M. Birtwhistle, O.J. Dahl, B. Myhrhaug, and K. Nygaard. Simula Begin. Chartwell-Bratt Ltd, 1979.

[4] Carl Bruggeman, Oscar Waddell, and R. Kent Dybvig. Representing control in the presence of one-shot continuations. In Proceedings of the SIGPLAN ’96 Conference on Programming Language Design and Implementation, pages 99–107, May 1996.

[5] Robert G. Burger and R. Kent Dybvig. Printing floating-point numbers quickly and accurately. In Proceedings of the SIGPLAN ’96 Conference on Programming Language Design and Implementation, pages 108–116, May 1996.

[6] Robert G. Burger and R. Kent Dybvig. An infrastructure for profile- driven dynamic recompilation. In Proceedings of the IEEE Computer Society 1998 International Conference on Computer Languages, pages 240–251, May 1998.

[7] Robert G. Burger, Oscar Waddell, and R. Kent Dybvig. Register allocation using lazy saves, eager restores, and greedy shuffling. In Proceedings of the SIGPLAN ’95 Conference on Programming Language Design and Implementation, pages 130–138, June 1995.

[8] Cadence Research Systems, Bloomington, Indiana. Chez Scheme System Manual, August 1989.

[9] Cadence Research Systems, Bloomington, Indiana. Chez Scheme System Manual, Rev. 2.0, December 1990.

[10] Cadence Research Systems. Chez Scheme System Manual, Rev. 2.5, October 1994.

[11] Luca Cardelli. Compiling a functional language. In Proceedings of the 1984 ACM Symposium on LISP and functional programming, pages 208–217, New York, NY, USA, 1984. ACM Press.

[12] Gregory J. Chaitin. Register allocation & spilling via graph coloring. In Proceedings of the 1982 ACM SIGPLAN symposium on Compiler construction, pages 98–105, New York, NY, USA, 1982. ACM Press.

[13] Rex A. Dwyer and R. Kent Dybvig. A SCHEME for distributed processes. Computer Science Department Technical Report #107, Indiana University, Bloomington, Indiana, April 1981.

[14] R. Kent Dybvig. C-Scheme. Master’s thesis, Indiana University Computer Science Department Technical Report #149, 1983.

[15] R. Kent Dybvig. The Scheme Programming Language. Prentice-Hall, 1987.

[16] R. Kent Dybvig. Three Implementation Models for Scheme. PhD thesis, University of North Carolina Technical Report #87-011, Chapel Hill, April 1987.

[17] R. Kent Dybvig. Writing hygienic macros in Scheme with syntax- case. Technical Report 356, Indiana University, June 1992.

[18] R. Kent Dybvig. The Scheme Programming Language. Prentice Hall, second edition, 1996.

[19] R. Kent Dybvig. Chez Scheme User’s Guide. Cadence Research Systems, 1998.

[20] R. Kent Dybvig. The Scheme Programming Language. MIT Press, third edition, 2003.

[21] R. Kent Dybvig. Chez Scheme Version 7 User’s Guide. Cadence Research Systems, 2005.

[22] R. Kent Dybvig, Carl Bruggeman, and David Eby. Guardians in a generation-based garbage collector. In Proceedings of the SIGPLAN ’93 Conference on Programming Language Design and Implementation, pages 207–216, June 1993.

[23] R. Kent Dybvig, David Eby, and Carl Bruggeman. Don’t stop the BiBOP: Flexible and efficient storage management for dynamically- typed languages. Technical Report 400, Indiana Computer Science Department, March 1994.

[24] R. Kent Dybvig, Daniel P. Friedman, and Christopher T. Haynes. Expansion-passing style: Beyond conventional macros. In Pro- ceedings of the 1986 ACM Conference on LISP and Functional Programming, pages 143–150, 1986.

[25] R. Kent Dybvig, Daniel P. Friedman, and Christopher T. Haynes. Expansion-passing style: A general macro mechanism. Lisp and Symbolic Computation, 1(1):53–75, 1988.

[26] R. Kent Dybvig and Robert Hieb. A variable-arity procedural interface. In Proceedings of the 1988 ACM Conference on Lisp and Functional Programming, pages 106–115, July 1988.

[27] R. Kent Dybvig and Robert Hieb. Engines from continuations. Computer Languages, 14(2):109–123, 1989.

[28] R. Kent Dybvig and Robert Hieb. A new approach to procedures with variable arity. Lisp and Symbolic Computation, 3(3):229–244, September 1990.

[29] R. Kent Dybvig, Robert Hieb, and Carl Bruggeman. Syntactic abstraction in Scheme. Lisp and Symbolic Computation, 5(4):295– 326, 1993.

[30] R. Kent Dybvig, Robert Hieb, and Tom Butler. Destination-driven code generation. Technical Report 302, Indiana Computer Science Department, February 1990.

[31] R. Kent Dybvig and Bruce T. Smith. Chez Scheme Reference Manual, Version 1.0. Cadence Research Systems, Chapel Hill, North Carolina, May 1985.

[32] Robert E. Filman and Daniel P. Friedman. Coordinated computing: tools and techniques for distributed software. McGraw-Hill, Inc., New York, NY, USA, 1984.

[33] John K. Foderaro, Keith L. Sklower, and Kevin Layer. The Franz LISP Manual. University of California, Berkeley, 1983.

[34] Per Brinch Hansen. Distributed processes: a concurrent programming concept. Communications of the ACM, 21(11):934–941, 1978.

[35] Christopher T. Haynes and Daniel P. Friedman. Abstracting timed preemption with engines. Computer Languages, 12(2):109–121, 1987.

[36] Robert Hieb, R. Kent Dybvig, and Carl Bruggeman. Representing control in the presence of first-class continuations. In Proceedings of the SIGPLAN ’90 Conference on Programming Language Design and Implementation, pages 66–77, June 1990.

[37] Robert Hieb, R. Kent Dybvig, and Carl Bruggeman. Syntactic abstraction in Scheme. Technical Report 355, Indiana University, June 1992.

[38] Suresh Jagannathan and Andrew Wright. Flow-directed inlining. In
Proceedings of the ACM SIGPLAN ’96 Conference on Programming Language Design and Implementation, pages 193–205, 1996.

[39] B. W. Kernighan and D. M. Ritchie. The C Programming Language. Prentice-Hall, 1978.

[40] Brian W. Kernighan and Dennis M. Ritchie. The M4 Macro Processor, 1979.

[41] Eugene Kohlbecker. Syntactic Extensions in the Programming Language Lisp. PhD thesis, Indiana University, Bloomington, August 1986.

[42] Eugene Kohlbecker, Daniel P. Friedman, Matthias Felleisen, and Bruce Duba. Hygienic macro expansion. In Proceedings of the 1986 ACM Conference on LISP and Functional Programming, pages 151–161, 1986.

[43] Henry Lieberman and Carl Hewitt. A real-time garbage collector based on the lifetimes of objects. Communications of the ACM, 26(6):419–429, 1983.

[44] Gyula Mago. A cellular computer architecture for functional programming. In Proc. COMPCON Spring, IEEE Comp. Soc. Conf., pages 179–187, 1980.

[45] John McCarthy, Paul W. Abrahams, Daniel J. Edwards, Timothy P. Hart, and Michael I. Levin. LISP 1.5 Programmer’s Manual. The MIT Press, Cambridge, Mass., 1966. second edition.

[46] Brian Randell and Lawford J. Russell. ALGOL 60 Implementation. Academic Press, London, 1964.

[47] Jonathan A. Rees and Norman I. Adams IV. T: a dialect of Lisp or LAMBDA: The ultimate software tool. In Proceedings of the 1982 ACM symposium on LISP and functional programming, pages 114–122, New York, NY, USA, 1982. ACM Press.

[48] Digital Research. CP/M Operating System Manual. Pacific Grove, CA, USA, 1976.

[49] Robert A. Saunders. The LISP system for the q-32 computer. In Edmund C. Berkeley and Daniel G. Bobrow, editors, The Programming Language LISP: Its Operation and Applications. Information International, Inc. and MIT Press, 1964.

[50] Patrick G. Sobalvarro. A lifetime-based garbage collector for LISP systems on general-purpose computers. B.S. Thesis, Massachusetts Institute of Technology, Electrical Engineering and Computer Science Department, Cambridge, MA., September 1988.

[51] Guy L. Steele Jr. Data representation in PDP-10 MACLISP. MIT AI Memo 421, Massachusetts Institute of Technology, September 1977.

[52] Guy L. Steele Jr. and Gerald J. Sussman. The revised report on Scheme, a dialect of Lisp. MIT AI Memo 452, Massachusetts Institute of Technology, January 1978.

[53] Guy L. Steele Jr. and Jon L White. How to print floating-point numbers accurately. Proceedings of the ACM SIGPLAN ’90 Conference on Programming Language Design and Implementation, 25(6):112–126, June 1990.

[54] David Ungar. Generation scavenging: A non-disruptive high performance storage reclamation algorithm. In Proceedings of the first ACM SIGSOFT/SIGPLAN software engineering symposium on Practical software development environments, pages 157–167, New York, NY, USA, 1984. ACM Press.

[55] Oscar Waddell and R. Kent Dybvig. Fast and effective procedure inlining. In Proceedings of the Fourth International Symposium on Static Analysis, volume 1302 of Lecture Notes in Computer Science, pages 35–52. Springer-Verlag, September 1997.

[56] Oscar Waddell and R. Kent Dybvig. Extending the scope of syntactic abstraction. In Conference Record of the Twenty Sixth Annual ACM Symposium on Principles of Programming Languages, pages 203– 213, January 1999.

[57] Oscar Waddell, Dipanwita Sarkar, and R. Kent Dybvig. Fixing letrec: A faithful yet efficient implementation of Scheme’s recursive binding construct. Higher-order and and symbolic computation, 18(3/4):299– 326, 2005.