## 第 2 章
# 发现领域知识

> 最终产物中发布的是开发人员的（错误）理解，而不是领域专家的知识。<br>
> —Alberto Brandolini

在上一章中，我们开始探索业务领域。你学习了如何识别公司的业务领域或活动领域，并分析公司的竞争策略；也就是业务领域中子域的边界和类型。

本章继续讨论业务领域分析的话题，但基于另一个不同的维度：深度。它侧重于子域内部发生的事情：它的业务功能和逻辑。你将学习用于有效沟通和知识共享的领域驱动设计工具：通用语言。在本章，我们将用它来学习业务领域里的复杂性。在本书的后续章节，我们将用它对软件建模并实现其业务逻辑。

## 业务问题
我们正在构建的软件系统是业务问题的解决方案。在这种情况下，*问题* 这个词并不像一个数学难题，也不是一个你可以猜出的谜语。在业务领域的上下文中，"问题"有着更广泛的含义。一个业务问题甚至可以是与优化工作流程和过程有关的挑战，从而最大限度地减少人工劳动，管理资源，支持决策，管理数据，等等。

业务问题同时出现在业务领域和子域两个层面。一个公司的目标就是为其客户所面临的问题提供解决方案。回到第 1 章中联邦快递的例子，该公司的客户需要在有限的时间内运送包裹，所以它们优化了配送过程。

子域是更细粒度的问题域，其目标是为特定的业务能力提供解决方案层面的支撑。比如：知识管理子域优化了存储和检索信息的过程。清算子域优化了执行金融交易的过程。会计子域保持对公司资金的跟踪。

## 知识发现
为了设计一个有效的软件解决方案，我们至少要掌握业务领域的基本知识。正如我们在第 1 章所讨论的，这些知识属于领域专家：他们的工作就是专门研究和理解业务领域里所有的复杂性。我们决不应该，也不可能成为领域专家。尽管如此，对我们来说，理解领域专家并使用他们使用的业务术语至关重要。

为了能有效，软件必须模仿领域专家思考问题的方式——他们的心智模型。如果不了解业务问题和需求背后的原因，我们的解决方案将仅限于将业务需求 "翻译" 成源代码。

如果需求漏掉了某个关键的边界情况（edge case）怎么办？或者未能描述某个业务概念，从而限制了我们实现支持未来需求的模型的能力？

正如 Alberto Brandolini 所说 [^1]，软件开发是一个学习的过程；可工作的代码只是一个副产品。软件项目的成功取决于领域专家和软件工程师之间知识共享的有效性。为了解决问题，我们必须了解问题。

## 沟通
可以肯定地说，几乎所有的软件项目都需要不同角色的利益相关者的合作：领域专家、产品所有者、工程师、UI 和 UX 设计师、项目经理、测试人员、分析师等。与任何协作努力的一样，结果取决于所有各方的合作程度。例如，所有利益相关者是否就正在解决的问题达成一致？关于他们正在构建的解决方案呢，他们对功能性和非功能性需求是否有任何相互矛盾的假设？在所有与项目有关的事项上达成共识和一致，对项目的成功至关重要。

对软件项目失败原因的研究表明，有效的沟通对于知识共享和项目成功至关重要。[^2] 然而，尽管它很重要，但在软件项目中却很少看到有效的沟通。通常情况下，业务人员和工程师之间没有直接的互动。取而代之的是，领域知识是从领域专家下推给工程师的，或是通过扮演调解人或"翻译"角色的人、比如：系统/业务分析师、产品所有者和项目经理来提供的。这种公共知识共享流程如图 2-1 所示。

<img src=images/figure2-1.png />
图 2-1. 软件项目中的知识共享流程 <br><br>


在传统的软件开发生命周期中，领域知识被“转化”为一种工程师友好的形式，称为*分析模型*，它是对系统需求的描述，而不是对背后业务领域的理解。虽然意图可能是好的，但这种转换对知识共享其实是有害的。在任何翻译中，信息都会丢失。在这种情况下，对于解决业务问题至关重要的领域知识会在传递给软件工程师的过程中丢失。在典型的软件项目中，类似的翻译并不罕见。分析模型被翻译成软件设计模型（一种软件设计文档，它又被翻译成实现模型或源代码）。正如经常发生的那样，文档很快就会过时。源代码用于将业务领域知识传达给之后将维护项目的软件工程师。 图2-2 说明了在代码中实现领域知识所需的不同翻译。

<img src=images/figure2-2.png />
图 2-2. 模型转换 <br><br>

这样的软件开发过程类似于儿童游戏 打电话：[^3] 信息或领域知识经常被曲解。这些信息导致软件工程师实现了错误的解决方案，或者虽然是正确的解决方案，但却针对了错误的问题。无论哪种情况，结果都是一样的：软件项目失败。

领域驱动设计提出了一种更好的方法将知识从领域专家那里传递给软件工程师：通过使用通用语言。

## 什么是通用语言
使用通用语言是领域驱动设计的基础实践。这个想法简单明了：如果各方需要有效沟通，而不是依赖翻译，那他们必须说同一种语言。

虽然这个道理勉强算（borderline）是个常识，但正如伏尔泰所说，"常识并不那么普通"。传统的软件开发生命周期意味着以下的翻译。

* 领域知识转化为分析模型
* 分析模型转化为需求
* 需求转化为系统设计
* 系统设计转化为源代码

与不断翻译领域知识不同，领域驱动设计需要培育一种描述业务领域的单一语言：通用语言。

所有与项目相关的利益相关者——软件工程师、产品所有者、领域专家、UI/UX 设计师——在描述业务领域时都应该使用通用语言。最重要的是，领域专家在对业务领域进行演绎时，必须能自如地使用通用语言；这种语言将代表业务领域和领域专家的心智模型。

只有通过不断使用通用语言及其术语，才能在项目的所有利益相关者中培育出共识。

## 业务的语言
重要的是要强调通用语言是业务的语言。因此，它应该只包含与业务领域相关的术语。没有技术术语！教给领域专家单例模式和抽象工厂并不是你的目标。通用语言旨在以易于理解的术语来构建领域专家对于业务领域的理解和心理模型。

### 场景
假设我们正在开发一个广告活动管理系统。考虑以下陈述：

* 一个广告活动可以展示不同的创意材料。
* 仅当至少有一个展示位处于可用状态时，才能发布活动。
* 销售佣金在交易被批准后进行核算。

所有这些陈述都是用业务语言制定的。也就是说，它们反映了领域专家对业务领域的看法。

另一方面，以下陈述是绝对技术性的，因此不符合通用语言的概念：

* 广告 iframe 可以显示一个 HTML 文件。
* 广告活动只有在活动展示表（active-placements table）中至少有一条关联记录时，才能被发布。
* 销售佣金基于交易记录和已批准销售表中的相关记录。

这些陈述纯粹是技术性的，对领域专家来说非常的不清晰。假设工程师只熟悉业务领域中这种技术性的、面向解决方案的观点。在这种情况下，他们将无法完全理解业务逻辑或为什么它以这种方式运作，这将限制他们建模和实现有效解决方案的能力。

### 一致性
通用语言必须是精确和一致的。它应该消除对假设的需要，并应使业务领域的逻辑明确。

由于歧义会阻碍交流，通用语言的每个术语都应该有且只有一个含义。让我们来看看几个术语不明确的例子，以及如何进行改进。

#### 模棱两可的术语
假设在某些业务领域中，*policy* 一词具有多种含义：它可以表示监管规则或保险合同。虽然确切的含义可以在人与人的互动中，根据当时的上下文明确出来。然而，软件并不能很好地应对模棱两可的问题，在代码中对 "policy" 实体进行建模可能会很麻烦，而且具有挑战性。

通用语言要求每个术语都有单一的含义，所以 "policy" 应该明确地使用 *监管规则* 和 *保险合同* 这两个术语来建模。

#### 同义词
在通用语言中，两个术语不能互换使用。例如，许多系统使用 *用户* 这个术语。然而，仔细研究领域专家的行话可能会发现，*用户* 和其他术语是可以互换使用的：例如，*访客*、*管理员*、*账户*等等。

同义词起初可能看起来无害。然而，在大多数情况下，它们表示的是不同的概念。在这个例子中，*访客* 和 *账户* 在技术上都是指系统的用户；然而，在大多数系统中，未注册用户和注册用户代表不同的角色，有不同的行为。例如，"访客" 数据主要用于分析目的，而 "账户" 则实际使用该系统及其功能。

最好是在特定的上下文中明确地使用每个术语。了解所使用的术语之间的差异，可以为业务领域的实体建立更简单、更清晰的模型和实现。

## 业务领域的模型
现在让我们从不同的角度来看待通用语言：从建模的角度。

### 什么是模型
> 模型是对事物或现象的简化表示，它有意强调某些方面而忽略其他方面。模型考虑的是特定用途的抽象。<br>
> —Rebecca Wirfs-Brock

模型不是现实世界的副本，而是帮助我们理解现实世界系统的人工构造。模型的典型示例是地图。任何地图都是一个模型，包括导航地图、地形地图、世界地图、地铁地图等，如图2-3所示。

<img src=images/figure2-3.png />
图 2-3. 不同类型的地图显示不同的地球模型：道路、时区、航海导航、地形、航空导航和地铁路线。<br><br>

这些地图都没有表现我们星球的所有细节。相反，每张地图只包含足够的数据来支持其特定的目的：它要解决的问题。

### 有效的建模
所有的模型都有一个目的，而一个有效的模型只包含实现其目的所需的细节。例如，你不会在世界地图上看到地铁站。另一方面，你也不能用地铁地图来估计距离。每张地图都只包含它应该提供的信息。

这一点值得重申：有用的模型并不是现实世界的复制品。相反，模型旨在解决问题，并且应该为此目的提供足够的信息。或者，正如统计学家 George Box 所说，“所有模型都是错误的，但有些模型是有用的。”

就其本质而言，模型是一种抽象。抽象的概念允许我们通过省略不必要的细节并只留下解决手头问题所需的东西来处理复杂性。相反，无效的抽象会删除必要的信息或通过留下不需要的信息而产生噪音。正如 Edsger W. Dijkstra 在他的论文“谦卑的程序员”[^4]中所指出的 ，抽象的目的不是为了模糊，而是为了创造一个新的语义层次，在这个层次上，人们可以 *绝对精确*。

### 为业务领域建模
在培育通用语言时，我们也是在有效地构建业务领域模型。该模型应该捕捉领域专家的心智模型——也就是他们关于企业如何工作以实现其功能的思维过程。该模型必须反映所涉及的业务实体及其行为、因果关系和不变量（invariants）。

我们使用的通用语言不应涵盖领域中所有可能的细节。那就相当于让每个利益相关者都成为领域专家。相反，该模型应该只包括业务领域中足够的方面，使其有可能实现所需的系统；也就是说，解决软件要解决的特定问题。在接下来的章节中，你将看到通用语言是如何推动低层设计和实现决策的。

工程团队和领域专家之间的有效沟通至关重要。这种沟通的重要性随着业务领域的复杂性而增加。业务领域越复杂，在代码中建模和实现其业务逻辑就越困难。即使对复杂的业务领域或其基本原理有轻微的误解，也会在不经意间导致容易出现严重错误的实现。验证对业务领域认知的唯一可靠方法就是与领域专家交谈，并使用他们所理解的语言：业务语言。

### 持续投入
通用语言的制定需要与它的天然持有者——领域专家进行互动。只有与真实的领域专家互动，才能发现不准确的地方、错误的假设或对业务领域总体上有缺陷的理解。利益相关者应在所有与项目相关的沟通中始终使用通用语言，以传播关于业务领域的知识并促进对业务领域的共同理解。该语言应该在整个项目中不断得到加强：需求、测试、文档，甚至源代码本身都应该使用这种语言。

最重要的是，培育通用语言是一个持续的过程。它应该不断得到验证和发展。随着时间的推移，通用语言的频繁使用将揭示对业务领域更深入的见解。当这些突破发生时，通用语言必须发展以跟上新获得的领域知识的步伐。

### 工具
有一些工具和技术可以缓解捕获和管理通用语言的过程。

例如，wiki 可以用作捕获和记录通用语言的 *词汇表*。该词汇表可以缓解新团队成员学习过程的难度，因为它是了解有关业务领域术语信息的首选之地。

让词汇表的维护成为一项公共的工作是很重要的。当通用语言被更改时，应该鼓励任何团队成员都可以去更新词汇表。这与集中式的方法相反，在那种方法中，只有团队领导或架构师负责维护词汇表。

尽管维护与项目相关的术语表具有明显的优势，但它也有固有的局限性。词汇表最适合“名词”：实体、流程、角色等的名称。尽管名词很重要，但对行为的捕捉也同样关键。行为不是一个单纯的与名词相关联的动词列表，而是实际的业务逻辑，包括其规则、假设和不变量。这样的概念在词汇表中往往更难记录。因此，词汇表最好与其他更适合捕捉行为的工具结合使用；例如，用例或 *Gherkin 测试*。

用 [Gherkin 语言](https://oreil.ly/WJw3C) 编写的自动化测试不仅是捕捉通用语言的好工具，而且它还充当了弥合领域专家和软件工程师之间鸿沟的额外工具。领域专家可以阅读测试并验证系统的预期行为[^5]。例如，请参阅以下用 Gherkin 语言编写的测试：

---
**Senario**：通知专员有一个新的支持工单. <br>
&emsp;&emsp;**Given** Vincent Jules 提交了一个新的支持工单，他说：<br>
&emsp;&emsp;""" <br>
&emsp;&emsp;我需要帮助来配置 AWS Infinidash <br>
&emsp;&emsp;""" <br>
&emsp;&emsp;**When** 工单据被分配给 Wolf 先生时 <br>
&emsp;&emsp;**Then** 专员收到关于新工单的通知 <br>

---


管理基于 Gherkin 的测试集有时可能具有挑战性，尤其是在项目的早期阶段。但是，对于复杂的业务领域来说，这绝对是值得的。

最后，甚至还有静态代码分析工具，可以验证通用语言中术语的使用。这种工具中值得一提的是 NDepend。

虽然这些工具很有用，但它们与在日常交互中实际使用通用语言相比是次要的。使用工具支持通用语言的管理，但不要指望文档能代替实际的应用。正如[《敏捷宣言》](https://agilemanifesto.org/)所说，"个人和互动高于流程和工具"。

### 挑战
从理论上讲，培育通用语言听起来是一个简单、直接的过程。在实践中，事实却并非如此。收集领域知识的唯一可靠方法是与领域专家交谈。很多时候，最重要的知识是隐性的。它没有被记录或编纂，只存在于领域专家的头脑中。获取这些知识的唯一途径是提出问题。

当你在这个实践中获得经验时，你会注意到，这个过程经常涉及到的不仅仅是发现已经存在的知识，而是一个与领域专家共同创造模型的过程。在领域专家自己对业务领域的理解中，也可能存在模糊不清甚至是白点（white spots）。例如，只定义了“快乐路径”的情况，而不考虑会挑战默认假设的边缘用例。此外，你可能会遇到缺乏明确定义的业务领域概念。提出关于业务领域本质的问题往往会使这种隐含的冲突和白点变得更加明确。这对于核心子域尤为常见。在这种情况下，学习过程是相互的——你也在帮助领域专家更好地了解他们的领域。

当把领域驱动的设计实践引入一个既有（brownfield）项目时，你会发现已经有一种成型的语言在描述业务领域了，而且利益相关者也在使用这种语言。然而，由于 DDD 原则并不驱动这种语言，所以它不一定能有效地反映业务领域。例如，它可能使用技术术语，例如数据库表名。更改组织中已经使用的语言并不容易。在这种情况下，必不可少的工具是耐心。你需要确保在易于控制的地方使用正确的语言：比如在文档和源代码中。

最后，我在会议上经常被问到的关于通用语言的问题：如果公司不在英语国家，我们应该使用什么语言。我的建议是，至少要使用英文名词来命名业务领域的实体。这将促使在代码中使用相同的术语。

## 结论
有效的沟通和知识共享对于一个成功的软件项目至关重要。软件工程师必须了解业务领域，以便设计和建立软件解决方案。

领域驱动设计的通用语言是弥合领域专家和软件工程师之间知识差距的有效工具。它通过培育一种可以被所有利益相关者在整个项目中（在对话、文档、测试、图表、源代码等方面）使用的共享语言来促进沟通和知识共享。

为了确保有效的沟通，通用语言必须消除歧义和隐含的假设。通用语言的所有术语都必须是一致的——没有歧义，也没有同义词。

培育通用语言是一个持续的过程。随着项目的发展，更多的领域知识将被发现。把这种洞见反映在通用语言中是很重要的。

诸如基于 wiki 的词汇表和 Gherkin 测试之类的工具可以极大地缓和记录和维护通用语言的过程。然而，有效的通用语言的主要先决条件是使用它：必须在所有与项目相关的沟通中持续地使用该语言。

## 练习
1. 谁应该可以为通用语言的定义做出贡献？<br>
A. 领域专家 <br>
B. 软件工程师 <br>
C. 最终用户 <br>
D. 所有项目的利益相关者 <br>

2. 通用语言应该被用在哪里？<br>
A. 当面交流 <br>
B. 文档 <br>
C. 源代码 <br>
D. 以上所有 <br>

3. 请回顾序言中对虚构的 WolfDesk 公司的描述。你能在描述中发现哪些业务领域的术语？

4. 考虑你目前正在从事或过去从事过的软件项目：<br>
A. 尝试提出可以在与领域专家对话中使用的业务领域概念。<br>
B. 试着找出不一致术语的例子：业务领域的概念要么有不同的含义，要么有着相同的概念却由不同的术语表示。<br>
C. 你是否遇到过因沟通不畅而导致的软件开发效率低下？<br>

5. 假设你正在进行一个项目，你注意到来自不同组织单位的领域专家使用同一个术语，例如，*policy*，来描述业务领域不相关的概念。
由此产生的通用语言的确是基于领域专家的心智模型，但未能满足一个术语只有单一含义的要求。
在你继续阅读下一章之前，你将如何解决这样的难题？

[^1]: Brandolini, Alberto. 《介绍事件风暴》Leanpub。

[^2]: Sudhakar, Goparaju Purna. (2012). "软件项目的关键成功因素模型"。企业信息管理杂志，25(6), 537-558。

[^3]: 玩家排成一列，第一名玩家想出一个信息，并在第二名玩家耳边轻声说出来。第二个人向第三个人重复这个信息，以此类推。最后一名玩家向整个小组宣布他们听到的信息。然后第一名玩家将原始信息与最终版本进行比较。虽然目标是传达相同的信息，但通常会出现混乱，最后一名玩家收到的信息与原来的信息有很大的不同。

[^4]: 谦卑的程序员(The Humble Programmer)(https://oreil.ly/LXd4W)

[^5]: 但请不要误以为领域专家会编写 Gherkin 测试。