## 第 14 章
# 微服务

在 2010 年代中期，微服务在软件工程行业掀起了一场风暴。其目的是为了解决现代系统需要快速变化、扩展和自然适应云计算的分布式性质。许多公司做出战略决定，解构他们的单体代码库，以支持基于微服务的架构所提供的灵活性。不幸的是，许多这样的努力都没有好结果。这些公司最终得到的不是灵活的架构，而是分布式的大泥球--这些设计比公司想要解构的单体更脆弱、更笨重、更昂贵。

从历史上看，微服务通常与 DDD 相关，特别是与限界上下文模式相关。许多人甚至可以交替使用 *限界上下文* 和 *微服务* 这两个术语。但它们真的是一回事吗？本章探讨了领域驱动设计方法和微服务架构模式之间的关系。你将了解这些模式之间的相互作用，更重要的是，你如何利用 DDD 来设计有效的基于微服务的系统。

让我们从基础知识开始，定义一下到底什么是服务和微服务。

## 什么是服务

根据 OASIS 的定义，服务是能够访问一种或多种能力的机制，其访问是通过规定的接口提供的。[^1] *规定的接口* 是用于将数据输入或输出服务的机制。它可以是同步的，如请求/响应模型，也可以是异步的，如生产及消费事件的模型。这就是服务的公共接口，如图 14-1 所示，它提供了与其它系统组件进行通信和集成的手段。

<img src=images/figure14-1.png />
图 14-1. 服务间的通信 <br><br>

[Randy Shoup](https://oreil.ly/IU6xJ) 把一个服务的接口比作前门。所有进入或离开服务的数据都必须通过前门。此外，一个服务的公开接口定义了服务本身：服务所暴露的功能。一个表达清晰的接口足以描述服务所实现的功能。例如，图 14-2 所示的公开接口明确地描述了服务的功能。

<img src=images/figure14-2.png />
图 14-2. 服务的公开接口 <br><br>

这将我们带到微服务的定义。

## 什么是微服务

微服务的定义出奇地简单。由于一个服务是由其公共接口定义的，所以微服务是一个具有微公共接口的服务：一个微前门。

拥有一个微公共接口可以使人们更容易理解单个服务的功能以及它与其他系统组件的集成。减少服务的功能也限制了其变化的原因，并使服务在开发、管理和规模上更加自如。

此外，它还解释了为什么微服务不公开其数据库的做法。暴露出数据库，使其成为服务前门的一部分，会使其公开接口变得庞大。例如，你可以在一个关系型数据库上执行多少不同的 SQL 查询？由于 SQL 是一种相当灵活的语言，可能的估计是无穷大。因此，微服务封装了他们的数据库。数据只能通过一个更紧凑的、面向集成的公开接口来访问。

### 方法即服务：完美的微服务？

说微服务是微公开接口看似简单。听起来就好像把服务接口限制在一个单一的方法中就会产生完美的微服务。让我们看看如果我们在实践中应用这种天真的分解会发生什么。

考虑一下图 14-3 中的 backlog 管理服务。它的公开接口由 8 个公共方法组成，我们想应用 "每个服务一个方法" 的规则。

<img src=images/figure14-3.png />
图 14-3. 天真的分解 <br><br>

由于这些是行为良好的微服务，每个服务都封装了其数据库。任何一个服务都不允许直接访问另一个服务的数据库；只能通过其公开接口进行。但目前，还没有这样的公开接口。这些服务必须协同工作，并同步每个服务正在应用的变化。因此，我们需要扩展服务的接口，以考虑这些与整合相关的问题。此外，当被可视化时，组合服务之间的集成和数据流就像一个典型的分布式大泥球，如图 14-4 所示。

<img src=images/figure14-4.png />
图 14-4. 集成的复杂度 <br><br>

套用 Randy Shoup 的比喻，通过将系统分解为如此细化的服务，我们无疑最大限度地减少了每个服务的前门。然而，为了实现总体系统的功能，我们不得不为每个服务添加巨大的 "员工专用" 入口。让我们看看我们能从这个例子中学到什么。

### 设计目标

按照简单的分解启发式方法，让每个服务只暴露一个方法，出于许多原因，被证明是不理想的。首先，这根本不可能。由于这些服务必须协同工作，我们不得不用与集成有关的方法来扩展它们的公开接口。其次，我们赢得了战斗，却输掉了战争。每项服务最终都比最初的设计简单得多，然而，由此产生的系统却变得复杂得多。

微服务架构的目标是要产生一个灵活的系统。将设计工作集中在单一的组件上，却忽略了它与系统其他部分的相互作用，这违背了系统的定义：

* 一组协同运行的连接事物或设备
* 为特定目的一起使用的一组计算机设备和程序

因此，一个系统不能由独立的组件来构建。在一个适当的基于微服务的系统中，无论如何解耦，这些服务仍然必须是一体化的，并相互沟通。让我们来看看单个微服务的复杂性和总体系统的复杂性之间的相互作用。

### 系统复杂度

40年前，没有云计算，没有全局扩缩容的需求，也不需要每11.7秒部署一个系统。但是，工程师们仍然必须驯服系统的复杂性。尽管当时的工具不同，但这些挑战--更重要的是，这些解决方案--在今天仍然适用，并且可以应用于基于微服务的系统设计。

Glenford J. Myers 在他的《复合/结构化设计》一书中讨论了如何构建过程代码以降低其复杂性。在这本书的第一页，他写道：

> 复杂性的主题远不止是试图最小化程序中每个部分的局部复杂性那么简单。一个更重要的复杂性类型是全局复杂性：一个程序或系统的整体结构的复杂性（即一个程序的主要部分之间的关联或相互依赖程度）。

在我们的上下文中，*局部复杂性* 是指每个单独的微服务的复杂性，而 *全局复杂性* 是指整个系统的复杂性。局部复杂度取决于服务的实现；全局复杂度是由服务之间的相互作用和依赖关系决定的。在设计一个基于微服务的系统时，哪种复杂性更需要优化？让我们来分析一下这两个极端。

将全局的复杂性降低到最低限度是出乎意料的容易。我们所要做的就是消除系统组件之间的任何互动，也就是说，在一个单一的服务中实现所有的功能。正如我们前面所看到的，这种策略在某些情况下可能是有效的。在其他情况下，它可能会导致可怕的大泥球：这可能是最高水平的局部复杂性。

另一方面，我们知道当我们只优化局部复杂度而忽略了系统的全局复杂度--变成更可怕的分布式大泥球时，会发生什么。这种关系如图 14-5 所示。

<img src=images/figure14-5.png />
图 14-5. 服务粒度和系统复杂性 <br><br>

为了设计一个合适的基于微服务的系统，我们必须同时优化全局和局部的复杂性。将设计目标设定为单独优化任何一个方面都只是片面优化。整体优化则是平衡这两种复杂性。让我们看看微公开接口的概念是如何平衡全局和局部的复杂性的。

### 微服务作为深服务

软件系统中的一个模块，或者说任何系统，就此而言，都是由其功能和逻辑定义的。*功能* 是指模块应该做什么--它的业务功能。*逻辑* 是模块的业务逻辑，即模块如何实现其业务功能。

John Ousterhout 在他的《软件设计的哲学》一书中讨论了模块化的概念，并提出了一个简单而强大的视觉启发式方法来评估一个模块的设计：深度。

Ousterhout 建议将模块可视化为一个矩形，如图 14-6 所示。矩形的上边代表了模块的功能，或者说其公开接口的复杂性。一个较宽的矩形代表更广泛的功能，而一个较窄的矩形则提供有限的功能，因此有着更简单的公开接口。矩形的面积代表模块的逻辑，或其功能的实现。

<img src=images/figure14-6.png />
图 14-6. 深模块 <br><br>

根据这个模型，有效的模块是深模块：一个简单的公开接口封装了复杂的逻辑。无效的模块是浅模块：一个浅模块的公开接口封装的复杂性比深层模块要少得多。考虑一下下面列表中的方法。

```cs
int AddTwoNumbers(int a, int b)
{
  return a + b;
}
```

这是浅模块的极端情况：公开接口（方法的签名）和它的逻辑（方法）是完全一样的。拥有这样的模块会引入不相干的 "活动部件"，因此，它不是在封装复杂性，而是在总体系统中增加了意外的复杂性。

### 微服务作为深模块

除了术语不同外，深模块的概念与微服务模式不同，模块可以表示逻辑和物理边界，而微服务是严格的物理边界。除此之外，这两个概念和它们所依据的设计原则是相同的。

实现单一业务方法的服务，如图 14-3 所示，是浅模块。因为我们不得不引入与集成有关的公开方法，所以产生的接口比它们应该有的要 "宽"。

从系统复杂度的角度来看，深模块降低了系统的全局复杂度，而浅模块则通过引入一个没有封装其局部复杂度的组件来增加了全局复杂度。

浅服务也是许多面向微服务项目失败的原因。将微服务错误地定义为代码不超过 X 行的服务，或者定义为应该比修改更容易重写的服务，这些都集中在单个服务上，而忽略了架构中最重要的方面：系统。

一个系统可以被分解成微服务的门槛是由微服务所属的系统的用例决定的。如果我们把一个单体系统分解成服务，引入变化的成本就会下降。当系统被分解成微服务时，它的成本会降到最低。然而，如果你继续分解，超过了微服务的门槛，深服务会变得越来越浅。它们的接口会重新增长。这时，由于集成的需要，引入变化的成本也会上升，整个系统的架构会变成可怕的分布式大泥球。这在图 14-7 中得到了描述。

<img src=images/figure14-7.png />
图 14-7. 粒度和变更成本 <br><br>

现在我们已经了解了什么是微服务，让我们来看看领域驱动设计如何帮助我们找到深服务的边界。

## 领域驱动设计和微服务的边界

作为微服务，前几章讨论的许多领域驱动的设计模式都是关于边界的：限界上下文是模型的边界，子域是业务能力的边界，而聚合和值对象是事务的边界。让我们看看这些边界中有哪些适合于微服务的概念。

### 限界上下文

微服务和限界上下文模式有很多共同点，以至于这些模式经常被交替使用。让我们看看情况是否真的如此：限界上下文的边界是否与有效的微服务的边界相关？

微服务和限界上下文都是物理边界。微服务，作为限界上下文，由一个团队拥有。与限界上下文一样，相互冲突的模型不能在微服务中实现，从而产生复杂的接口。微服务的确是一个限界上下文。但是这种关系反过来也行得通吗？我们可以说，限界上下文就是微服务吗？

正如你在第 3 章中所学到的，限界上下文保护通用语言和模型的一致性。没有冲突的模型可以在同一个限界上下文中实现。假设你正在开发一个广告管理系统。在系统的业务领域中，业务实体 Lead 由Promotion 和 Sales 上下文中的不同模型表示。因此，Promotion 和 Sales 是限界上下文，每个上下文都定义了 Campaign 实体的唯一模型，该模型在上下文边界内有效，如图 14-8 所示。

<img src=images/figure14-8.png />
图 14-8. 限界上下文 <br><br>

为简单起见，我们假设系统中除了 Lead 之外没有其他冲突的模型。这使得所产生的限界上下文天然可以很宽--每个限界上下文可以包含多个子域。子域可以从一个限界上下文移动到另一个。只要子域不会带来冲突的模型，图 14-9 中的所有候选分解都是完全有效的限界上下文。

<img src=images/figure14-9.png />
图 14-9. 限界上下文的候选分解 <br><br>

对限界上下文的不同分解归因于不同的需求，例如不同的团队规模和结构、生命周期的依赖性等等。但我们能说这个例子中所有有效的限界上下文都一定是微服务吗？不能。特别是考虑到分解 1 中的两个限界上下文的功能相对比较较广。

因此，微服务和限界上下文之间的关系不是对称的。尽管微服务是限界上下文，但不是每个限界上下文都是微服务。另一方面，限界上下文表示最大的有效单体的边界。这样的单体不应该与一个大泥球相混淆，它是一个可行的设计方案，可以保护其通用语言的一致性，或其业务领域的模型。正如我们将在第 15 章讨论的那样，在某些情况下，这种宽泛的边界比微服务更有效。

图 14-10 直观地展示了限界上下文和微服务之间的关系。限界上下文和微服务之间的区域是安全的。这些都是有效的设计方案。然而，如果系统没有被分解成适当的限界上下文，或者分解超过了微服务的阈值，就会分别导致单体大泥球或分布式大泥球。

<img src=images/figure14-10.png />
图 14-10. 粒度和模块化 <br><br>

接下来，让我们看看另一个极端：聚合是否可以帮助找到微服务的边界。

### 聚合

限界上下文对最宽的有效边界施加限制，而聚合模式则相反。聚合的边界是可能的最窄边界。将一个聚合分解成多个物理服务或限界上下文，不仅是不理想的，而且正如你将在附录 A 中了解的那样，至少会导致不希望出现的后果。

就像限界上下文一样，聚合的边界也经常被认为是驱动微服务的边界。聚合是一个不可分割的业务功能单元，它封装了其内部业务规则、不变量和逻辑的复杂性。也就是说，正如你在本章前面所学到的，微服务不是针对于单个服务的。一个单独的服务必须在它与系统的其他组件的交互中加以考虑。

聚合是否真要与它的子域中的其他聚合进行通信？
* 它是否与其他聚合共享值对象？
* 聚合的业务逻辑变化会对子域的其他组件产生多大的影响，反之亦然？

聚合与它的子域中其他业务实体的关系越强，它作为一个单独的服务就越浅。

在有些情况下，将一个聚合作为服务会产生一个模块化的设计。然而，更多的时候，这种细粒度的服务会增加总体系统的整体复杂性。

### 子域

设计微服务的一个更平衡的启发式方法是使服务与业务子域的边界相一致。正如你在第 1 章中学到的，子域与细化的业务能力相关。这些是公司在其业务领域竞争所需的业务构件。从业务领域的角度来看，子域描述了能力--业务做什么--而没有解释能力是如何实现的。从技术角度看，子域代表了一组连贯的用例：使用相同的业务领域模型，处理相同或密切相关的数据，并具有强大的功能关系。如图 14-11 所示，其中一个用例的业务需求的变化可能会影响到其他用例。

<img src=images/figure14-11.png />
图 14-11. 子域 <br><br>

子域的粒度和对功能的关注 -- 关注 "是什么" 而不是 "怎么做" -- 使子域自然成为深模块。一个子域的描述--也就是功能--囊括了更复杂的实现细节--即逻辑。子域中包含用例的连贯性也确保了结果模块的深度。在许多情况下，将它们分开会导致更复杂的公开接口，从而导致更浅的模块。所有这些都使子域成为设计微服务的安全边界。

将微服务与子域对齐是一种安全的启发式方法，它能为大多数微服务产生最佳解决方案。不过，在某些情况下其他的边界也许会更有效；例如，保持更宽广的、语义边界的限界上下文，或者由于非功能性的需求，要诉诸于将聚合作为微服务。解决方案不仅取决于业务领域，而且还取决于组织的结构、业务战略和非功能性需求。正如我们在第 11 章所讨论的，不断调整软件架构和设计以适应环境的变化是至关重要的。

## 压缩微服务的公开接口

除了寻找服务边界外，领域驱动的设计可以帮助使服务更深入。本节展示了开放主机服务和防腐层模式如何简化微服务的公开接口。

### 开放主机服务

如图 14-12 所示，开放主机服务将限界上下文的业务领域模型与用于与系统的其他组件集成的模型解耦。

引入面向集成的模型，即发布语言，降低了系统的整体复杂性。首先，它允许我们在不影响消费者的情况下演化服务的实现：新的实现模型可以被翻译成现有的发布语言。第二，发布语言暴露了一个更受限的模型。它是围绕集成需求进行设计的。它封装了与服务的消费者无关的复杂的实现。例如，它可以为消费者暴露更少的数据和更方便的模型。

在相同的实现（逻辑）上有一个更简单的公开接口（功能）使服务更 "深"，有助于更有效的微服务设计。

<img src=images/figure14-12.png />
图 14-12. 通过发布语言集成服务 <br><br>

### 防腐层

防腐层（ACL）模式则反其道而行之。它减少了将服务与其他限界上下文整合的复杂性。传统上，防腐层属于它所保护的限界上下文。然而，正如我们在第 9 章所讨论的，这个概念可以更进一步，作为一个独立的服务来实现。

图 14-13 中的 ACL 服务既降低了消费者限界上下文的局部复杂性，也降低了系统的整体复杂性。消费者限界上下文的业务复杂度与集成复杂度是分开的。后者被卸装到 ACL 服务中。因为消费者的限界上下文正在使用更方便的、面向集成的模型，它的公开接口被压缩了--它不反映生产服务所暴露的集成复杂性。

<img src=images/figure14-13.png />
图 14-13. 防腐层作为独立服务 <br><br>

## 结论

从历史上看，基于微服务的架构风格与领域驱动设计有很深的联系，以至于 *微服务* 和 *限界上下文* 这两个词经常被交替使用。在这一章中，我们分析了两者之间的联系，看到它们并不是一回事。

所有的微服务都是限界上下文，但并非所有的限界上下文都是微服务。从本质上讲，微服务定义了服务的最小有效边界，而限界上下文保护了所包含的模型的一致性，代表了最宽的有效边界。将边界定义得比其限界上下文更宽，将导致一个单体大泥球，而比微服务更小的边界将导致一个分布式的大泥球。

尽管如此，微服务和领域驱动设计之间的联系是紧密的。我们看到领域驱动设计工具如何被用来设计有效的微服务边界。

在第 15 章，我们将继续讨论高层系统架构，但从不同的角度：通过事件驱动架构进行异步集成。你将学习如何利用不同种类的事件消息来进一步优化微服务的边界。

## 练习
1.	限界上下文和微服务之间有什么关系？<br>
A.	所有的微服务都是限界上下文。<br>
B.	所有的限界上下文都是微服务。<br>
C.	微服务和限界上下文是同一概念的不同术语。<br>
D.	微服务和限界上下文是完全不同的概念，不能相提并论。<br>

2.	微服务的哪一部分应该是“微”的？<br>
A.	为实现微服务的团队提供所需的比萨饼数量。该指标必须考虑团队成员的不同饮食偏好和平均每日卡路里摄入量。<br>
B.	实现服务的功能所需的代码行数。由于该指标与行的宽度无关，因此最好是在超宽显示器上实现微服务。<br>
C.	设计基于微服务的系统最重要的一点是获得对微服务友好的中间件和其他基础设施组件，最好是来自微服务认证的供应商。<br>
D.	业务领域的知识及其复杂性暴露在服务的边界上，并由其公开接口反映。<br>

3.	什么是安全的组件边界？<br>
A.	边界比限界上下文更宽。<br>
B.	边界比微服务更窄。<br>
C.	介于限界上下文（最宽）和微服务（最窄）之间的边界。<br>
D.	所有的边界都是安全的。<br>

4.	将微服务与聚合的边界对齐是一个好的设计决策吗？<br>
A.	是的，聚合总是会产生适当的微服务。<br>
B.	不，聚合永远不应该作为单独的微服务公开。<br>
C.	用一个聚合来制作微服务是不可能的。<br>
D.	该决定取决于业务领域。<br>

[^1]: 面向服务的架构参考模型 v1.0. (n.d.). 2021年6月14日，从 OASIS 检索。