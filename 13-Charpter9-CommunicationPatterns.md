## 第 9 章
# 通信模式

第 5-8 章介绍了战术设计模式，这些模式定义了实现系统组件的不同方式：如何对业务逻辑建模以及如何在架构上组织限界上下文的内部结构。在本章中，我们将超越单个组件的边界，去讨论跨越组织系统元素的通信流模式。

你在本章中将了解到的模式有助于跨限界上下文的通信，解决聚合设计原则带来的限制，并编排跨越多个系统组件的业务流程。

## 模型转译

限界上下文是模型的边界——也是通用语言的边界。正如你在第3章中所学到的，有不同的模式来设计跨不同限界上下文下的通信。假设实现两个限界上下文的团队正在有效地沟通并愿意协作。在这种情况下，限界上下文可以以合作关系（partnership）集成：集成协议可以以临时方式进行协调，任何集成问题都可以通过团队之间的沟通得到有效解决。另一种以合作驱动的集成方法是 *共享内核* ：不同团队提取并共同演化一个模型的有限的部分；例如，将限界上下文的集成契约提取到共同拥有的代码库中。

在客户-供应商关系中，权力的平衡倾向于上游（供应商）或下游（消费者）限界上下文。假设下游限界上下文不能符合上游限界上下文的模型。在这种情况下，需要一个更复杂的技术解决方案，通过转译限界上下文的模型来促进交流。

这种转译可由一方处理，有时也可以由双方共同处理：下游限界上下文可以使用防腐层 (ACL) 使上游限界上下文的模型适应其需求，而上游限界上下文可实现为开放主机服务（OHS）并通过使用特定于集成的发布语言来保护其消费者免受上游实现模型变更的影响。由于防腐层和开放主机服务的模型转译逻辑类似，故本章只涵盖了实现方案而没有区分模式，且只在特殊情况下会提及差异。

模型的转译逻辑可以是无状态的或有状态的。当传入 (OHS) 或传出 (ACL) 请求被发出时，*无状态转译* 会即时（on the fly）发生，而 *有状态转译* 涉及更为复杂的转换逻辑，需要数据库的支持。让我们看看实现这两种类型模型转译的设计模式。

### 无状态模型转译

对于无状态模型转译，拥有转译逻辑的限界上下文（上游为 OHS，下游为 ACL）会实现代理设计模式，以拦截（interject）传入和传出的请求并将源模型映射到限界上下文的目标模型。如图 9-1 所示。

<img src=images/figure9-1.png />
图 9-1. 通过代理行模型转译 <br><br>

代理的实现取决于限界上下文是同步通信还是异步通信。

#### 同步通信

同步通信中使用的模型的典型转译方法是将转译逻辑嵌入到限界上下文的代码库中，如图 9-2 所示。在开放主机服务中，转译成发布语言（译注：原文此处为 public language）是在处理传入的请求时进行的，而在防腐层中，转译是在调用上游限界上下文时进行的。

<img src=images/figure9-2.png />
图 9-2. 同步通信 <br><br>

在某些情况下，将转译逻辑卸装到一个外部组件，如 API 网关模式，可能更有成本效益且更方便。API 网关组件可以是基于开源软件的解决方案，例如 Kong 或 KrakenD，也可以是云供应商的托管服务，例如 AWS API Gateway、Google Apigee 或 Azure API Management。

对于实现开放主机模式的限界上下文，API 网关负责将内部模型转译为面向集成优化的发布语言。此外，拥有一个明确的 API 网关可以减轻管理和服务多版本限界上下文 API 的过程，如图 9-3 所示。

<img src=images/figure9-3.png />
图 9-3. 公开不同版本的发布语言 <br><br>

使用 API 网关实现的防腐层可以被多个下游限界上下文使用。在这种情况下，防腐层充当特定于集成目的的限界上下文，如图 9-4 所示。

<img src=images/figure9-4.png />
图 9-4. 共享防腐层 <br><br>

这种限界上下文主要负责转译模型，以便其他组件更方便地使用，通常被称为 *交互上下文*。

#### 异步通信

为了转译异步通信中使用的模型，你可以实现一个消息代理：一个中间组件，订阅来自源限界上下文的消息。代理将应用所需的模型转换，并将结果消息转发给目标订阅者（见图 9-5）。

<img src=images/figure9-5.png />
图 9-5. 在异步通信中转译模型 <br><br>

除了转译消息的模型外，拦截组件还可以通过过滤掉不相关的消息来减少目标限界上下文的噪音。

实现开放主机服务时，异步模型转译必不可少。一个常见的错误是，为模型的对象设计和公开一种发布语言并允许按原始内容直接发布领域事件，以至于暴露了限界上下文的实现模型。异步转译可以用来拦截领域事件，并将其转译为发布语言，从而对限界上下文的实现细节进行更好的封装（见图9-6）。

此外，将消息转译成发布语言，可以区分为限界上下文的内部需求而设计的私有事件和为与其他限界上下文整合而设计的公共事件。我们将在第 15 章重新讨论和扩展有关私有/公共事件的主题，讨论领域驱动设计和事件驱动架构之间的关系。

<img src=images/figure9-6.png />
图 9-6. 发布语言中的领域事件 <br><br>

### 有状态模型转译

对于更重要的模型转换——例如，当转译机制必须对源数据进行聚合或将多个来源的数据统一到一个模型中时——可能需要有状态的转译。让我们来依次详细讨论这些用例。

#### 对传入的数据进行聚合

假设一个限界上下文为了实现性能优化，有兴趣对传入的请求进行聚合及批处理。在这种情况下，同步和异步请求都可能需要执行聚合操作（见图9-7）。

<img src=images/figure9-7.png />
图 9-7. 批处理请求 <br><br>

对源数据进行聚合的另一个常见用例是将多个细粒度的消息合并成一个包含统一数据的消息，如图9-8所描述的。

<img src=images/figure9-8.png />
图 9-8. 对传入事件进行统一 <br><br>

对传入数据进行聚合的模型转译不能使用 API 网关来实现，因此需要更精细的、有状态的处理。为了跟踪传入的数据并进行相应的处理，转译逻辑需要自己的持久化存储（见图9-9）。

<img src=images/figure9-9.png />
图 9-9. 有状态模型转换 <br><br>

在一些用例中，你可以通过使用现成的产品来避免为有状态转移实现定制化的解决方案；例如，流处理平台（Kafka、AWS Kinesis等），或批处理解决方案（Apache NiFi、AWS Glue、Spark等）。

#### 统一多个数据源

限界上下文可能需要处理来自多个数据源的数据聚合，包括其他限界上下文。一个典型的例子是服务于前端的后端模式 [^1]，其中用户界面必须组合来自多个服务的数据。

另一个例子是限界上下文必须实现复杂的业务逻辑以处理来自其他多个上下文的所有数据。在这种情况下，将集成逻辑和业务逻辑的复杂性分离开来是有益的，如图 9-10 所示，限界上下文前面有一个防腐层，它将所有来自其他限界上下文的数据聚合起来。

<img src=images/figure9-10.png />
图 9-10. 使用防腐层模式简化集成模型 <br><br>


## 对聚合进行集成

在第 6 章中，我们讨论了聚合与系统其他部分的通信方式之一是发布领域事件。外部组件可以订阅这些领域事件并执行组件的业务逻辑。但是领域事件是如何发布到消息总线的呢？

在我们讨论解决方案之前，让我们研究一下事件发布过程中的几个常见错误以及每种做法的后果。考虑以下代码。

```cs
 1 public class Campaign
 2 {
 3     // ...
 4     List<DomainEvent> _events;
 5     IMessageBus _messageBus;
 6     // ...
 7 
 8     public void Deactivate(string reason)
 9     {
10         for (l in _locations.Values())
11         {
12             l.Deactivate();
13         }
14 
15         IsActive = false;
16 
17         var newEvent = new CampaignDeactivated(_id, reason);
18         _events.Append(newEvent);
19         _messageBus.Publish(newEvent);
20     }
21 } 
```

在第17行，一个新的事件被实例化了。在接下来的两行中，它被追加到聚合的内部领域事件列表（第18行），并且此事件被发布到消息总线上（第19行）。这种发布领域事件的实现虽然简单，但却是错误的。直接从聚合中发布领域事件有两个不好的原因。首先，在聚合的新状态被提交到数据库之前，事件就有可能被分发。这会导致订阅者收到活动被停用的通知，但这与活动的实际状态相矛盾。其次，如果数据库事务由于竞争条件、或仅仅是因为技术问题而未能提交，后续的聚合逻辑导致无效操作该如何处理？即使数据库事务被回滚，但事件已经被发布并推送给了订阅者，且没有办法被回收。

让我们试试其他方法：

```cs
 1 public class ManagementAPI
 2 {
 3     // ...
 4     private readonly IMessageBus _messageBus;
 5     private readonly ICampaignRepository _repository;
 6     // ...
 7     public ExecutionResult DeactivateCampaign(CampaignId id, string reason)
 8     {
 9         try
10         {
11             var campaign = repository.Load(id);
12             campaign.Deactivate(reason);
13             _repository.CommitChanges(campaign);
14 
15             var events = campaign.GetUnpublishedEvents();
16             for (IDomainEvent e in events)
17             {
18                 _messageBus.publish(e);
19             }
20             campaign.ClearUnpublishedEvents();
21         }
22         catch(Exception ex)
23         {
24             // ...
25         }
26     }
27 } 
```

在前面的代码列表中，发布新领域事件的责任被转移到了应用层。在第 11 行到第 13 行，Campaign 聚合的相关实例被加载，它的 Deactivate 命令被执行，只有在更新的状态被成功提交到数据库后，在第 15 行到第 20 行，新的领域事件才会被发布到消息总线上。我们能相信这段代码吗？并不能。

在这种情况下，运行业务逻辑的进程由于某种原因未能发布领域事件。也许是消息总线发生了故障。或者是运行代码的服务器在提交数据库事务后，发布事件之前，随即失败了。系统仍将以不一致的状态结束，这意味着数据库事务被提交，但领域事件永远不会被发布。

这些极端场景可以使用发件箱模式来解决。

### 发件箱模式

发件箱模式（图 9-11）使用以下算法确保领域事件的可靠发布：

* 更新后的聚合状态和新领域事件都在同一个原子事务中提交。
* 消息中继从数据库中获取新提交的领域事件。
* 消息中继将领域事件发布到消息总线。
* 成功发布后，消息中继要么在数据库中将事件标记为发布，要么将其完全删除

<img src=images/figure9-11.png />
图 9-11. 发件箱模式 <br><br>

使用关系型数据库时，可以方便地利用数据库以原子方式提交到两张表的功能，并使用专用表来存储消息，如图 9-12 所示。

<img src=images/figure9-12.png />
图 9-12. 发件箱表 <br><br>

当使用不支持多文档事务的 NoSQL 数据库时，传出的领域事件必须嵌入到聚合记录中。例如：

```
{
    "campaign-id": "364b33c3-2171-446d-b652-8e5a7b2be1af",
    "state": {
        "name": "Autumn 2017",
        "publishing-state": "DEACTIVATED",
        "ad-locations": [
            "..."
        ],
        "...": "..."
    },
    "outbox": [
        {
            "campaign-id": "364b33c3-2171-446d-b652-8e5a7b2be1af",
            "type": "campaign-deactivated",
            "reason": "Goals met",
            "published": false
        }
    ]
}
```

在这个例子中，你可以看到 JSON 文档的附加属性 outbox，它包含了一个需要发布的领域事件的列表。

#### 获取未发布的事件

发布中继可以以基于拉取或基于推送的方式获取新的领域事件：

*拉取：轮询发布者*

中继可以持续查询数据库中未发布的事件。需要设置适当的索引以最小化由持续轮询引起的数据库负载。

*推送：事务日志追踪*

在这里，我们可以利用数据库的功能集，在新的事件被追加时主动调用发布中继。例如，某些关系数据库可以通过跟踪数据库的事务日志来获取有关更新/插入记录的通知。一些 NoSQL 数据库将提交的更改公开为事件流（例如，AWS DynamoDB Streams）。

值得注意的是，发件箱模式保证消息至少传递一次：如果在消息发布之后，但在数据库中标记为已发布之前，中继发生故障，那么同一消息将在下一次迭代中再次被发布。

接下来，我们将看到我们如何利用领域事件的可靠发布来克服聚合设计原则带来的一些限制。

### Saga

聚合设计的核心原则之一是将每个事务限制在聚合的单一实例中。这确保了一个聚合的边界是经过仔细考虑的，并封装了一组连贯的业务功能。但在有些情况下，你必须实现一个跨越多个聚合的业务流程。

考虑以下例子：当一个广告活动被激活时，它应该自动将活动的广告材料提交给发布者。在收到发布者的确认后，活动的发布状态应改为 Published。在被发布者拒绝的情况下，该活动应被标记为 Rejected。

这一流程跨越了两个业务实体：广告活动和发布者。将这些实体放在同一个聚合边界中肯定是矫枉过正，因为这些显然是不同的业务实体，它们有不同的责任，可能属于不同的限界上下文。换言之，这个流程可以作为 saga 来实现。

saga 是一个长期运行的业务流程。它的长期运行不一定是指时间，因为 saga 运行可以从几秒钟到几年，长期指的是事务方面：也就是跨越多个事务的业务流程。事务不仅可以由聚合处理，而且可以由任何发出领域事件和响应命令的组件处理。saga 监听相关组件发出的事件，并向其他组件发出后续命令。如果其中一个执行步骤失败，saga 负责发布相关的补偿动作，以确保系统状态保持一致。

让我们看看前面例子中的广告活动发布流程如何以 saga 的形式实现，如图9-13所示。

<img src=images/figure9-13.png />
图 9-13. Saga <br><br>

为了实现发布过程，saga 需要从 Campaign 聚合中侦听 Campaign Activated 事件，并从 AdPublishing 限界上下文中侦听 PublishingConfirmed 和 PublishingRejected 事件。saga 要在 AdPublishing 上执行 Submit Advertisement 命令，并在 Campaign 聚合上执行 TrackPublishingConfirmation 和 TrackPublishingRejection 命令。在这个例子中，TrackPublishingRejection 命令作为一个补偿动作，将确保广告活动没有被列为已激活。下面是代码：

```cs
public class CampaignPublishingSaga
{
    private readonly ICampaignRepository _repository;
    private readonly IPublishingServiceClient _publishingService;
    // ...

    public void Process(CampaignActivated @event)
    {
        var campaign = _repository.Load(@event.CampaignId);
        var advertisingMaterials = campaign.GenerateAdvertisingMaterials);
        _publishingService.SubmitAdvertisement(@event.CampaignId,
                                              advertisingMaterials);
    }

    public void Process(PublishingConfirmed @event)
    {
        var campaign = _repository.Load(@event.CampaignId);
        campaign.TrackPublishingConfirmation(@event.ConfirmationId);
        _repository.CommitChanges(campaign);
    }

    public void Process(PublishingRejected @event)
    {
        var campaign = _repository.Load(@event.CampaignId);
        campaign.TrackPublishingRejection(@event.RejectionReason);
        _repository.CommitChanges(campaign);
    }
}
```

前面的示例依赖于消息传递基础设施来传递相关事件，并通过执行相关命令来对事件做出反应。这是一个相对简单的 saga 的例子：它没有状态。你会遇到确实需要状态管理的 saga；例如，跟踪已执行的操作，以便在发生故障时发出相关的补偿性动作。在这种情况下，saga 可以作为事件溯源聚合来实现，对接收到的事件以及发出的命令的完整历史记录进行持久化。然而，命令的执行逻辑应该被从 saga 本体中移出，并以异步方式执行，类似于领域事件在发件箱模式中的调度方式。

```cs
public class CampaignPublishingSaga
{
    private readonly ICampaignRepository _repository;
    private readonly IList<IDomainEvent> _events;
    // ...

    public void Process(CampaignActivated activated)
    {
        var campaign = _repository.Load(activated.CampaignId);
        var advertisingMaterials = campaign.GenerateAdvertisingMaterials);
        var commandIssuedEvent = new CommandIssuedEvent(
            target: Target.PublishingService,
            command: new SubmitAdvertisementCommand(activated.CampaignId,
            advertisingMaterials));
        
        _events.Append(activated);
        _events.Append(commandIssuedEvent);
    }

    public void Process(PublishingConfirmed confirmed)
    {
        var commandIssuedEvent = new CommandIssuedEvent(
            target: Target.CampaignAggregate,
            command: new TrackConfirmation(confirmed.CampaignId,
                                           confirmed.ConfirmationId));

        _events.Append(confirmed);
        _events.Append(commandIssuedEvent);
    }

    public void Process(PublishingRejected rejected)
    {
        var commandIssuedEvent = new CommandIssuedEvent(
            target: Target.CampaignAggregate,
            command: new TrackRejection(rejected.CampaignId,
                                        rejected.RejectionReason));

        _events.Append(rejected);
        _events.Append(commandIssuedEvent);
    }
}
```

在此示例中，发件箱中继必须针对每个 CommandIssuedEvent 实例在相关端点上执行命令。与发布领域事件的情况一样，将 saga 的状态转换与命令的执行分开，即使该过程在任何阶段失败了，也可以确保命令的可靠执行。

#### 一致性

尽管 saga 模式编排了一个多组件事务，但所涉及组件的状态是具有最终一致性的。虽然 saga 最终会执行相关命令，但没有两个事务可以被认为是原子的。这与聚合设计原则相关：

> *只有聚合边界内的数据才能被认为是强一致性的。边界之外的一切都是最终一致性的。*

将此作为一个指导原则，以确保你不会滥用 saga 来弥补不恰当的聚合边界。属于同一聚合的业务操作需要强一致性的数据。

saga 模式经常与另一种模式混淆：流程管理器。虽然实现方式相似，但这是不同的模式。在下一节中，我们将讨论流程管理器模式的目的以及它与 saga 模式的不同之处。

### 流程管理器

saga 模式管理着简单的、线性的流程。严格来说，saga 将事件与相应的命令相匹配。在我们用来演示 saga 实现的示例中，我们实际上实现了事件与命令的简单匹配：

* CampaignActivated 事件 对 PublishingService.SubmitAdvertisement 命令
* PublishingConfirmed 事件 对Campaign.TrackConfirmation 命令
* PublishingRejected 事件 对 Campaign.TrackRejection 命令

如图 9-14 所示，流程管理器模式旨在实现基于业务逻辑的流程。它被定义为一个中央处理单元，用于维护流程的状态并确定接下来的处理步骤。[^2]

<img src=images/figure9-14.png />
图 9-14. 流程管理器 <br><br>

作为一个简单的经验法则，如果一个 saga 包含 if-else 语句来选择正确的行动方案，那么它可能是一个流程管理器。

流程管理器和 saga 之间的另一个区别是，当侦测到特定事件时，saga 是隐式实例化的，如前面示例中的 CampaignActivated。另一方面，流程管理器并不能被绑定到单个源事件（source event）上。相反，它是一个由多个步骤组成的连贯业务​​流程。因此，必须显式的实例化流程管理器。考虑以下示例：

预订商务旅行要从路线算法开始，选择最具成本效益的航班路线并要求员工确认。如果员工倾向于不同的路线，则需要他们的直接经理进行批准。预订航班后，必须在适当的日期预订预先核准的酒店之一。如果没有酒店可用，则必须取消机票。

在此示例中，没有核心实体来触发旅行预订流程。旅行预订是一个流程，它必须作为流程管理器来实现（见图 9-15）。

<img src=images/figure9-15.png />
图 9-15. 行程预订流程管理器 <br><br>

从实现的角度来看，流程管理器通常被实现为基于状态或事件溯源的聚合。例如：

```cs
public class BookingProcessManager
{
    private readonly IList<IDomainEvent> _events;
    private BookingId _id;
    private Destination _destination;
    private TripDefinition _parameters;
    private EmployeeId _traveler;
    private Route _route;
    private IList<Route> _rejectedRoutes;
    private IRoutingService _routing;
    // ...

    public void Initialize(Destination destination,
                           TripDefinition parameters,
                           EmployeeId traveler)
    {
        _destination = destination;
        _parameters = parameters;
        _traveler = traveler;
        _route = _routing.Calculate(destination, parameters);

        var routeGenerated = new RouteGeneratedEvent(
            BookingId: _id,
            Route: _route);

        var commandIssuedEvent = new CommandIssuedEvent(
            command: new RequestEmployeeApproval(_traveler, _route)
        );

        _events.Append(routeGenerated);
        _events.Append(commandIssuedEvent);
    }

    public void Process(RouteConfirmed confirmed)
    {
        var commandIssuedEvent = new CommandIssuedEvent(
            command: new BookFlights(_route, _parameters)
        );

        _events.Append(confirmed);
        _events.Append(commandIssuedEvent);
    }

    public void Process(RouteRejected rejected)
    {
        var commandIssuedEvent = new CommandIssuedEvent(
            command: new RequestRerouting(_traveler, _route)
        );

        _events.Append(rejected);
        _events.Append(commandIssuedEvent);
    }

    public void Process(ReroutingConfirmed confirmed)
    {
        _rejectedRoutes.Append(route);
        _route = _routing.CalculateAltRoute(destination,
                                            parameters, rejectedRoutes);
        var routeGenerated = new RouteGeneratedEvent(
            BookingId: _id,
            Route: _route);
        
        var commandIssuedEvent = new CommandIssuedEvent(
            command: new RequestEmployeeApproval(_traveler, _route)
        );

        _events.Append(confirmed);
        _events.Append(routeGenerated);
        _events.Append(commandIssuedEvent);
    }

    public void Process(FlightBooked booked)
    {
        var commandIssuedEvent = new CommandIssuedEvent(
            command: new BookHotel(_destination, _parameters)
        );
    
        _events.Append(booked);
        _events.Append(commandIssuedEvent);
    }

    // ...
}
```

在这个例子中，流程管理器有它的显式 ID 和持久化状态，描述了需要预订的行程。如同前面的 saga 模式的例子，流程管理器订阅了控制工作流的事件（如：RouteConfirmed、RouteRejected、ReroutingConfirmed 等）。并且它实例化了类型为 "Command Issued" 的事件，这些事件将被一个发件箱中继处理，以执行实际的命令。

## 结论

在本章中，你学到了整合系统组件的不同模式。本章首先探讨了模型转译的相关模式，这些模式可用于实现防腐层或开放主机服务。我们看到，转译可以是即时处理的，也可以遵循更复杂的逻辑，并且需要状态跟踪。

发件箱模式是发布聚合的领域事件的可靠方式。它确保领域事件总是会被发布，即使面对不同的进程故障。

saga 模式可以用来实现简单的跨组件的业务流程。更复杂的业务流程可以用流程管理器模式来实现。这两种模式都依赖于对领域事件的异步相应以及对命令的发布。

## 练习
1. 哪种限界上下文集成模式需要实现模型转译逻辑？<br>
A. 遵奉者 <br>
B. 防腐层 <br>
C. 开放主机服务 <br>
D. B 和 C <br>

2. 发件箱模式的目标是什么？<br>
A. 将消息传递基础设施与系统的业务逻辑层解耦 <br>
B. 可靠地发布消息 <br>
C. 支持事件溯源领域模型模式的实现 <br>
D. A 和 C <br>

3. 除了将消息发布到消息总线之外，发件箱模式还有哪些其他可能的用例？

4. saga 和流程管理器模式之间有什么区别？<br>
A. 流程管理器需要显式实例化，而 saga 是在发布相关领域事件时实例化的。 <br>
B. 与流程管理器相反，saga 从不需要持久化其执行状态。 <br>
C. saga 要求它所操作的组件实现事件溯源模式，而进程管理器则不需要。 <br>
D. 流程管理器模式适用于复杂的业务工作流。<br>
E. A 和 D 是正确的


[^1]: Richardson, C.（2019 年）。微服务架构设计模式：Java 示例。纽约：Manning
[^2]: Hohpe, G., & Woolf, B. (2003)。企业集成模式：设计、构建和部署消息传递解决方案。波士顿：Addison-Wesley