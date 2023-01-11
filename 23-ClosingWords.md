# 结束语

为了完成我们对领域驱动设计的探索，我想回到我们开始的那句话。

> 在我们就问题达成一致之前谈论解决方案是没有意义的，在我们就解决方案达成一致之前谈论实施步骤也是没有意义的。<br>
> &emsp;&emsp;&emsp;&emsp;—Efrat Goldratt-Ashlag<br>

这句话巧妙地总结了我们的 DDD 之旅。

## 问题

要提供一个软件解决方案，我们首先要了解问题：我们所处的业务领域是什么，业务目标是什么，以及实现这些目标的策略是什么。

我们使用通用语言来深入了解我们必须在软件中实现的业务领域和其逻辑。

你学会了通过将业务问题分解成限界上下文来管理它的复杂性。每个限界上下文都实现了业务领域的单一模型，旨在解决一个特定的问题。

我们讨论了如何识别和分类业务领域的构件：核心子域、支撑子域和通用子域。表 E-1 对这三种类型的子域进行了比较。

表 E-1。三种类型的子域

|  子域类型  |  竞争优势  |  复杂度  |  易变性  |  实现  |  问题  |
|  ----  |  ----  |  ----  |  ----  |  ----  |  ----  |
|  核心  |  是  |  高  |  高  |  内部  |  有意思的  |
|  通用  |  否  |  高  |  低  |  购买/引进  |  已解决的  |
|  支撑  |  否  |  低  |  低  |  内部/外包  |  显而易见的  |

## 解决方案

你学会了利用这些知识来设计针对每种类型的子域的优化解决方案。我们讨论了四种业务逻辑实现模式--事务脚本、活动记录、领域模型和事件溯源领域模型，以及每种模式所能发挥的场景。你还看到了为业务逻辑的实现提供所需脚手架的三种架构模式：分层架构、端口和适配器以及 CQRS。图 E-1 总结了使用这些模式进行战术决策的启发式方法。

<img src=images/figureE-1.png />
图 E-1. 总结战术决策启发式方法的决策树 <br><br>

## 实现

在第三部分，我们讨论了如何将理论转化为实践。你学到了如何通过举行事件风暴会议来有效地建立一个通用语言，如何在业务领域的发展中保持设计的形状，以及如何在既有项目中引入并开始使用领域驱动设计。

在第四部分，我们讨论了领域驱动设计与其他方法论和模式之间的相互作用：微服务、事件驱动架构和数据网格。我们看到，领域驱动设计不仅可以与这些技术同时使用，而且它们实际上是相互补充的。

## 延伸阅读

我希望这本书能让你对领域驱动设计感兴趣。如果你想继续学习，这里有一些我衷心推荐的书。

### 进阶领域驱动设计

* Evans, E. (2003). Domain-Driven Design: Tackling Complexity in the Heart of Software. Boston: Addison-Wesley.

介绍领域驱动设计方法的 Eric Evans 的原著。尽管它没有反映 DDD 的最新方面，例如领域事件和事件溯源，但它仍然是成为 DDD 黑带的必备读物。

* Martraire, C. (2019). Living Documentation: Continuous Knowledge Sharing by Design. Boston: Addison-Wesley.

在这本书中，Cyrille Martraire 提出了一种基于领域驱动设计的知识共享、文档和测试的方法。

* Vernon, V. (2013). Implementing Domain-Driven Design. Boston: AddisonWesley.

又一个永恒的 DDD 经典。Vaughn Vernon 对领域驱动设计思想及其战略和战术工具集的使用进行了深入的讨论和详细的实例。作为学习的基础，Vaughn 使用了一个真实的 DDD 实施失败的例子，以及团队通过应用基本的路线修正而获得的复兴之旅。

* Young, G. (2017). Versioning in an Event Sourced System. Leanpub.

在第 7 章中，我们讨论了演化事件溯源系统可能是一个挑战。本书就是专门讨论这个话题的。

### 架构和集成模式

* Dehghani, Z. (Expected to be published in 2022). Data Mesh: Delivering DataDriven Value at Scale. Boston: O’Reilly.

Zhamak Dehghani 是我们在第 16 章讨论的数据网格模式的作者。在这本书中，Dehghani 解释了数据管理架构背后的原理，以及如何在实践中实现数据网格架构。

* Fowler, M. (2002). Patterns of Enterprise Application Architecture. Boston: Addison-Wesley.

我在第 5 章和第 6 章多次引用的经典应用架构模式书。事务脚本、活动记录和领域模型模式最初就是在这本书中定义的。

* Hohpe, G., & Woolf, B. (2003). Enterprise Integration Patterns: Designing, Building, and Deploying Messaging Solutions. Boston: Addison-Wesley.

第 9 章中讨论的许多模式最初是在本书中介绍的。阅读本书可以了解更多的组件集成模式。

* Richardson, C. (2019). Microservice Patterns: With Examples in Java. New York: Manning Publications.

在这本书中，Chris Richardson 提供了许多在架构基于微服务的解决方案时经常使用的模式的详细例子。其中被讨论的模式有 saga、流程管理器和 outbox，我们在第 9 章中讨论了这些模式。

### 遗留系统的现代化

* Kaiser, S. (Expected to be published in 2022). Adaptive Systems with Domain Driven Design, Wardley Mapping, and Team Topologies. Boston: Addison-Wesley.

Susanne Kaiser 分享了她通过利用领域驱动设计、Wardley 映射和团队拓扑结构实现遗留系统现代化的经验。

* Tune, N. (Expected to be published in 2022). Architecture Modernization: Prod‐ uct, Domain, & Team Oriented. Leanpub.

在这本书中，Nick Tune 深入讨论了如何利用领域驱动设计和其他技术来使棕地项目的架构现代化。

* Vernon, V., & Jaskula, T. (2021). Implementing Strategic Monoliths and Microservices. Boston: Addison-Wesley

这是一本实践性很强的书，作者在书中展示了不老的软件工程工具，包括快速发现和学习、领域驱动的方法，以及处理正确实施基于单体和微服务的解决方案的复杂性，同时关注最重要的方面：交付创新的商业战略。

* Vernon, V., & Jaskula, T. (2021). Strategic Monoliths and Microservices. Boston: Addison-Wesley.

在本书中，Vaughn 和 Tomasz 通过探讨如何利用基于发现的学习和领域驱动的方法实现所有重要的创新，以及如何为工作选择目的性最强的架构和工具：微服务、单体或混合，以及如何使它们一起工作，来促进软件战略思维。

### 事件风暴

* Brandolini, A. (尚未出版). Introducing EventStorming. Leanpub.

Alberto Brandolini 是事件风暴研讨会的创建者，在这本书中，他详细解释了事件风暴背后的过程和原理。

* Rayner, P. (尚未出版). The EventStorming Handbook. Leanpub.

Paul Rayner 解释了他如何在实践中使用事件风暴，包括促进成功会议的许多技巧和窍门。

## 结论

就是这些! 非常感谢你阅读这本书。我希望你喜欢它，并希望你能使用你从这里学到的东西。

我希望你从这本书中得到的是领域驱动设计工具背后的逻辑和原则。不要把领域驱动设计当作教条而盲目追随，而是要理解它所基于的道理。这种理解将大大增加你应用 DDD 的机会，并从中获得价值。理解领域驱动设计的理念也是通过逐步融入该方法的概念来发挥价值的关键，特别是在既有项目中。

最后，始终注意你的通用语言，当有疑问时，就尝试做事件风暴。祝您好运!

