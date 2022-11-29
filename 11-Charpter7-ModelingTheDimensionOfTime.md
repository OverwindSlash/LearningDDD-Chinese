## 第 7 章
# 建立时间维度的模型

在上一章中，你了解了领域模型模式：它的构建块、用途和应用上下文。事件溯源领域模型模式基于领域模型模式相同的前提。同样，业务逻辑复杂，并且属于核心子域。此外，它使用与领域模型相同的战术模式：值对象、聚合和领域事件。

这些实现模式的区别在于聚合的状态被持久化的方式。事件溯源领域模型使用事件溯源模式来管理聚合的状态：该模型并不持久化聚合的状态，而是生成描述每个变化的领域事件，并将其作为聚合数据的真实来源。

本章首先介绍事件溯源的概念。然后介绍如何将事件溯源与领域模型模式相结合，使其成为事件溯源领域模型。

## 事件溯源

> 秀出你的流程图，但藏起你的表格，我依然迷惑不解。展示你的表格吧，通常我并不需要你的流程图；这显而易见。<br>
> —Fred Brooks [^1]

让我们用 Fred Brooks 的道理来定义事件溯源模式，并理解它与传统数据建模和持久化有什么不同。检查表 7-1 并分析你可以从该数据中了解到有关它所属系统的哪些信息。

表 7-1。基于状态的模型

|  lead-id |  first-name  |  last-name  |  status  |  phone-number  |  followup-on  |  created-on  |  updated-on  |
|  ----  |  ----  |  ----  |  ----  |  ----  |  ----  |  ----  |  ----  |
|  1  |  Sean  |  Callahan  |  CONVERTED  |  555-1246  |    |  2019-01-31T 10:02:40.32Z  |  2019-01-31T 10:02:40.32Z  |
|  2  |  Sarah  |  Estrada  |  CLOSED  |  555-4395  |    |  2019-03-29T 22:01:41.44Z  |  2019-03-29T 22:01:41.44Z  |
|  3  |  Stephanie  |  Brown  |  CLOSED  |  555-1176  |    |  2019-04-15T 23:08:45.59Z  |  2019-04-15T 23:08:45.59Z  |
|  4  |  Sami  |  Calhoun  |  CLOSED  |  555-1850  |    |  2019-04-25T 05:42:17.07Z  |  2019-04-25T 05:42:17.07Z  |
|  5  |  William  |  Smith  |  CONVERTED  |  555-3013  |    |  2019-05-14T 04:43:57.51Z  |  2019-05-14T 04:43:57.51Z  |
|  6  |  Sabri  |  Chan  |  NEW_LEAD  |  555-2900  |    |  2019-06-19T 15:01:49.68Z  |  2019-06-19T 15:01:49.68Z  |
|  7  |  Samantha  |  Espinosa  |  NEW_LEAD  |  555-8861  |    |  2019-07-17T 13:09:59.32Z  |  2019-07-17T 13:09:59.32Z  |
|  8  |  Hani  |  Cronin  |  CLOSED  |  555-3018  |    |  2019-10-09T 11:40:17.13Z  |  2019-10-09T 11:40:17.13Z  |
|  9  |  Sian  |  Espinoza  |  FOLLOWUP_SET  |  555-6461  |  2019-12-04T 01:49:08.05Z  |  2019-12-04T 01:49:08.05Z  |  2019-12-04T 01:49:08.05Z  |
|  10  |  Sophia  |  Escamilla  |  CLOSED  |  555-4090  |    |  2019-12-06T 09:12:32.56Z  |  2019-12-06T 09:12:32.56Z  |
|  11  |  William  |  White  |  FOLLOWUP_SET  |  555-1187  |  2020-01-23T 00:33:13.88Z  |  2020-01-23T 00:33:13.88Z  |  2020-01-23T 00:33:13.88Z  |
|  12  |  Casey  |  Davis  |  CONVERTED  |  555-8101  |    |  2020-05-20T 09:52:55.95Z  |  2020-05-27T 12:38:44.12Z  |
|  13  |  Walter  |  Connor  |  NEW_LEAD  |  555-4753  |    |  2020-04-20T 06:52:55.95Z  |  2020-04-20T 06:52:55.95Z  |
|  14  |  Sophie  |  Garcia  |  CONVERTED  |  555-1284  |    |  2020-05-06T 18:47:04.70Z  |  2020-05-06T 18:47:04.70Z  |
|  15  |  Sally  |  Evans  |  PAYMENT_FAILED  |  555-3230  |    |  2020-06-04T 14:51:06.15Z  |  2020-06-04T 14:51:06.15Z  |
|  16  |  Scott  |  Chatman  |  NEW_LEAD  |  555-6953  |    |  2020-06-09T 09:07:05.23Z  |  2020-06-09T 09:07:05.23Z  |
|  17  |  Stephen  |  Pinkman  |  CONVERTED  |  555-2326  |    |  2020-07-20T 00:56:59.94Z  |  2020-07-20T 00:56:59.94Z  |
|  18  |  Sara  |  Elliott  |  PENDING_PAYMENT  |  555-2620  |    |  2020-08-12T 17:39:43.25Z  |  2020-08-12T 17:39:43.25Z  |
|  19  |  Sadie  |  Edwards  |  FOLLOWUP_SET  |  555-8163  |  2020-10-22T 12:40:03.98Z  |  2020-10-22T 12:40:03.98Z  |  2020-10-22T 12:40:03.98Z  |
|  20  |  William  |  Smith  |  PENDING_PAYMENT  |  555-9273  |    |  2020-11-13T 08:14:07.17Z  |  2020-11-13T 08:14:07.17Z  |


很明显，在电话营销系统中，该表被用来管理潜在客户或线索。对于每个线索，你可以看到他们的ID，他们的名字和姓氏，记录的创建和更新时间，他们的电话号码，以及该线索的当前状态。

通过检查各种状态，我们也可以推断出每个潜在客户经历的业务处理周期。

* 销售流程从处于 NEW_LEAD 状态的潜在客户开始。
* 销售电话可能以潜在客户对报价不感兴趣（销售线索为 CLOSED）、安排跟进电话 (FOLLOWUP_SET) 或接受报价 (PENDING_PAYMENT) 结束。
* 如果支付成功，该销售线索就被转化为客户。相反，支付可能会失败——PAYMENT_FAILED。

仅仅通过分析这张表的模式和其中存储的数据，我们就可以收集到相当多的信息。我们甚至可以假设在对数据进行建模时使用的是什么样的通用语言。但是，该表缺少什么信息呢？

该表的数据记录了线索的当前状态，但它忽略了每个线索是如何达到其当前状态的故事。我们无法分析在线索的生命周期内发生了什么。我们不知道在一个线索变成 CONVERTED 之前打了多少个电话。是马上就购买了，还是有一个漫长的销售过程？根据历史数据，在多次跟进后，是否值得继续尝试联系这个人，还是为了更有效而关闭该线索并转向更有前途的潜在客户？这些信息都没有。我们所知道的只是线索的当前状态。

这些问题反映了对优化销售流程至关重要的业务关注点。从业务的角度来看，分析数据并根据经验优化流程至关重要。填补缺失信息的方法之一是使用事件溯源。

事件溯源模式将时间维度引入数据模型。基于事件溯源的系统不是反映聚合的当前状态，而是持续记录聚合生命周期中每个变化的事件。

考虑表 7-1 中第 12 行的 CONVERTED 客户。下面的清单演示了这个人的数据将如何在事件溯源系统中表示：

```json
[
    {
        "lead-id": 12,
        "event-id": 0,
        "event-type": "lead-initialized",
        "first-name": "Casey",
        "last-name": "David",
        "phone-number": "555-2951",
        "timestamp": "2020-05-20T09:52:55.95Z"
    },
    {
        "lead-id": 12,
        "event-id": 1,
        "event-type": "contacted",
        "timestamp": "2020-05-20T12:32:08.24Z"
    },
    {
        "lead-id": 12,
        "event-id": 2,
        "event-type": "followup-set",
        "followup-on": "2020-05-27T12:00:00.00Z",
        "timestamp": "2020-05-20T12:32:08.24Z"
    },
    {
        "lead-id": 12,
        "event-id": 3,
        "event-type": "contact-details-updated",
        "first-name": "Casey",
        "last-name": "Davis",
        "phone-number": "555-8101",
        "timestamp": "2020-05-20T12:32:08.24Z"
    },
    {
        "lead-id": 12,
        "event-id": 4,
        "event-type": "contacted",
        "timestamp": "2020-05-27T12:02:12.51Z"
    },
    {
        "lead-id": 12,
        "event-id": 5,
        "event-type": "order-submitted",
        "payment-deadline": "2020-05-30T12:02:12.51Z",
        "timestamp": "2020-05-27T12:02:12.51Z"
    },
    {
        "lead-id": 12,
        "event-id": 6,
        "event-type": "payment-confirmed",
        "status": "converted",
        "timestamp": "2020-05-27T12:38:44.12Z"
    }
]
```

列表中的事件讲述了客户的故事。该线索在系统中被创建（事件0），大约两小时后被销售人员联系上（事件1）。在通话过程中，双方同意销售专员在一周后回电（事件2），但要换一个电话号码（事件3）。销售专员还修正了姓氏中的拼写错误（事件 3）。在商定的日期和时间（事件 4）联系了线索客户并提交了订单（事件 5）。该订单应在三天内支付（事件5），但大约半小时后就收到了付款（事件6），并且该线索被转化为了一个新客户。

正如我们之前看到的，客户的状态可以很容易地从这些领域事件中投影出来。我们所要做的就是对每个事件顺序应用简单的转换逻辑：

```cs
public class LeadSearchModelProjection
{
    public long LeadId { get; private set; }
    public HashSet<string> FirstNames { get; private set; }
    public HashSet<string> LastNames { get; private set; }
    public HashSet<PhoneNumber> PhoneNumbers { get; private set; }
    public int Version { get; private set; }

    public void Apply(LeadInitialized @event)
    {
        LeadId = @event.LeadId;
        FirstNames = new HashSet < string > ();
        LastNames = new HashSet < string > ();
        PhoneNumbers = new HashSet < PhoneNumber > ();
        FirstNames.Add(@event.FirstName);
        LastNames.Add(@event.LastName);
        PhoneNumbers.Add(@event.PhoneNumber);
        Version = 0;
    }

    public void Apply(ContactDetailsChanged @event)
    {
        FirstNames.Add(@event.FirstName);
        LastNames.Add(@event.LastName);
        PhoneNumbers.Add(@event.PhoneNumber);
        Version += 1;
    }

    public void Apply(Contacted @event)
    {
        Version += 1;
    }

    public void Apply(FollowupSet @event)
    {
        Version += 1;
    }

    public void Apply(OrderSubmitted @event)
    {
        Version += 1;
    }

    public void Apply(PaymentConfirmed @event)
    {
        Version += 1;
    }
}
```

迭代聚合的事件并将它们按顺序提供给 Apply 方法的相应重载实现，将精确地生成表 7-1 中表格中建模的状态表征（state representation）。

注意应用每个事件后被递增的 Version 字段。它的值代表了对业务实体进行修改的总次数。此外，假设我们应用了一个事件的子集。在这种情况下，我们可以 "穿越时空"：我们可以通过只应用相关的事件来投影实体在其生命周期任何一点的状态。例如，如果我们需要实体在版本5中的状态，我们可以只应用前五个事件。

最后，我们并不局限于只投影事件的单一状态表征！我们可以考虑以下情况。

### 查询

你必须实现搜索功能。然而，由于线索的联系信息可以被更新--姓、名和电话号码--销售专员可能并不知道其他专员所做的更新，他可能想通过潜在客户的联系信息，包括历史值来定位线索。我们可以很容易地投影出历史信息。

```cs
public class LeadSearchModelProjection
{
    public long LeadId { get; private set; }
    public HashSet<string> FirstNames { get; private set; }
    public HashSet<string> LastNames { get; private set; }
    public HashSet<PhoneNumber> PhoneNumbers { get; private set; }
    public int Version { get; private set; }

    public void Apply(LeadInitialized @event)
    {
        LeadId = @event.LeadId;
        FirstNames = new HashSet < string > ();
        LastNames = new HashSet < string > ();
        PhoneNumbers = new HashSet < PhoneNumber > ();
        FirstNames.Add(@event.FirstName);
        LastNames.Add(@event.LastName);
        PhoneNumbers.Add(@event.PhoneNumber);
        Version = 0;
    }

    public void Apply(ContactDetailsChanged @event)
    {
        FirstNames.Add(@event.FirstName);
        LastNames.Add(@event.LastName);
        PhoneNumbers.Add(@event.PhoneNumber);
        Version += 1;
    }

    public void Apply(Contacted @event)
    {
        Version += 1;
    }

    public void Apply(FollowupSet @event)
    {
        Version += 1;
    }

    public void Apply(OrderSubmitted @event)
    {
        Version += 1;
    }

    public void Apply(PaymentConfirmed @event)
    {
        Version += 1;
    }
}
```

投影逻辑使用 LeadInitialized 和 ContactDetailsChanged 事件来填充相应的潜在客户的个人详细信息。其他事件将被忽略，因为它们不会影响特定模型的状态。

将此投影逻辑应用于之前示例中的 Casey Davis 事件将导致以下状态：

```json
LeadId: 12
FirstNames: ['Casey']
LastNames: ['David', 'Davis']
PhoneNumbers: ['555-2951', '555-8101']
Version: 6
```

### 分析

你的商业智能部门要求你提供一个更有利于分析的线索数据的表示形式。对于他们目前的研究，他们希望得到为不同线索安排的跟进电话的数量。他们将过滤已转化和已关闭的线索数据，并使用该模型来优化销售流程。让我们投影一下他们所要求的数据。

```cs
public class AnalysisModelProjection
{
    public long LeadId { get; private set; }
    public int Followups { get; private set; }
    public LeadStatus Status { get; private set; }
    public int Version { get; private set; }

    public void Apply(LeadInitialized @event)
    {
        LeadId = @event.LeadId;
        Followups = 0;
        Status = LeadStatus.NEW_LEAD;
        Version = 0;
    }

    public void Apply(Contacted @event)
    {
        Version += 1;
    }

    public void Apply(FollowupSet @event)
    {
        Status = LeadStatus.FOLLOWUP_SET;
        Followups += 1;
        Version += 1;
    }

    public void Apply(ContactDetailsChanged @event)
    {
        Version += 1;
    }

    public void Apply(OrderSubmitted @event)
    {
        Status = LeadStatus.PENDING_PAYMENT;
        Version += 1;
    }

    public void Apply(PaymentConfirmed @event)
    {
        Status = LeadStatus.CONVERTED;
        Version += 1;
    }
}
```

前面的逻辑维护了一个计数器，记录了线索所有事件中 FollowupSet 事件出现的次数。如果我们将此投影应用于聚合事件的例子中，它将生成以下状态：

```json
LeadId: 12
Followups: 1
Status: Converted
Version: 6
```

前面的例子中实现的逻辑是在内存中投影搜索优化和分析优化的模型。然而，为了实现实际所需的功能，我们必须在数据库中持久化投影模型。在第8章中，你将了解到一种可以让我们这样做的模式：命令-查询职责分离模式（CQRS）。


### 真相之源

为了使事件溯源模式发挥作用，针对对象状态的所有变化都应该以事件的形式表示和持久化。这些事件成为系统的真相之源（该模式因此而得名）。这个过程如图7-1所示。

<img src=images/figure7-1.png />
图 7-1. 事件溯源聚合 <br><br>

存储系统事件的数据库是独一无二的强一致性存储：系统的真相之源。用于持久化事件的数据库其公认名称叫 *事件存储* （event store）。

### 事件存储

事件存储不应允许修改或删除事件 [^2]，因为它是仅追加的存储。为了支持事件溯源模式的实现，事件存储至少必须支持以下功能：获取属于特定业务实体的所有事件并可以追加事件。例如：

```cs
interface IEventStore
{
    IEnumerable<Event> Fetch(Guid instanceId);
    void Append(Guid instanceId, Event[] newEvents, int expectedVersion);
}
```

Append 方法中的 expectedVersion 参数是实现乐观并发管理所必须的：当追加新事件时，你也指定了你的操作决策所依据的实体版本。如果它是 *过期* （stale） 的，也就是说，在预期版本之后有新事件被添加，则事件存储应该引发并发异常。

在大多数系统中，需要额外的端点来实现 CQRS 模式，我们将在下一章讨论。

&emsp;&emsp;&emsp;&emsp;<img src=images/general-note.png width=50 />&emsp;&emsp; 本质上，事件溯源模式并不是什么新鲜事。金融业使用事件来代表账本的变化。账本是一个仅支持追加交易记录的日志。

## 事件溯源领域模型

原版的领域模型维护着聚合的状态表示，并发布选定的领域事件。事件溯源领域模型完全使用领域事件来对聚合的生命周期进行建模。聚合状态的所有变化都必须被表达为领域事件。

对事件溯源聚合的每个操作都遵循以下脚本：

* 加载聚合的领域事件。
* 重组状态表征——将事件投影到一个可用于做出商业决策的状态表征对象中。
* 执行聚合的命令来运行业务逻辑，从而产生新的领域事件。
* 将新的领域事件提交到事件存储。

回到第 6 章中 Ticket 聚合的例子，让我们看看如何将其作为一个事件溯源聚合来实现。

```cs
 1 public class TicketAPI
 2 {
 3     private ITicketsRepository _ticketsRepository;
 4     // ...
 5 
 6     public void RequestEscalation(TicketId id)
 7     {
 8         var events = _ticketsRepository.LoadEvents(id);
 9         var ticket = new Ticket(events);
10         var originalVersion = ticket.Version;
11         var cmd = new RequestEscalation();
12         ticket.Execute(cmd);
13         _ticketsRepository.CommitChanges(ticket, originalVersion);
14     }
15 
16     // ...
17 } 
```

Ticket 聚合在构造函数中的补水（rehydration）逻辑（第 27 行到第 31 行）实例化状态投影类 TicketState 的实例，并依次为每个工单事件调用其 AppendEvent 方法：

```cs
18 public class Ticket
19 {
20     // ...
21     private List<DomainEvent> _domainEvents = new List<DomainEvent>();
22     private TicketState _state;
23     // ...
24 
25     public Ticket(IEnumerable<IDomainEvents> events)
26     {
27         _state = new TicketState();
28         foreach (var e in events)
29         {
30             AppendEvent(e);
31         }
32     } 
```

AppendEvent 函数将传入事件传递给 TicketState 投影逻辑，从而生成工单当前状态的内存表示：

```cs
33     private void AppendEvent(IDomainEvent @event)
34     {
35         _domainEvents.Append(@event);
36         // Dynamically call the correct overload of the “Apply” method.
37         ((dynamic)state).Apply((dynamic)@event);
38     } 
```

与我们在上一章中看到的实现相比，事件溯源聚合的 RequestEscalation 方法没有明确地将 IsEscalated 标志设置为 true。相反，它实例化适当的事件并将其传递给 AppendEvent 方法（第 43 和 44 行）：

```cs
39     public void Execute(RequestEscalation cmd)
40     {
41         if (!_state.IsEscalated && _state.RemainingTimePercentage <= 0)
42         {
43             var escalatedEvent = new TicketEscalated(_id, cmd.Reason);
44             AppendEvent(escalatedEvent);
45         }
46     }
47 
48     // ...
49 } 
```

添加到聚合的事件集合中的所有事件都会被传递到 TicketState 类中的状态投影逻辑，其中相关字段的值根据事件的数据会发生变化：

```cs
50 public class TicketState
51 {
52     public TicketId Id { get; private set; }
53     public int Version { get; private set; }
54     public bool IsEscalated { get; private set; }
55     // ...
56     public void Apply(TicketInitialized @event)
57     {
58         Id = @event.Id;
59         Version = 0;
60         IsEscalated = false;
61         // ....
62     }
63 
64     public void Apply(TicketEscalated @event)
65     {
66         IsEscalated = true;
67         Version += 1;
68     }
69 
70     // ...
71 } 
```

现在让我们看看在实现复杂业务逻辑时利用事件溯源的一些优势。

--
### 为什么要称之为“事件溯源领域模型”？
我觉得有必要解释一下，为什么我使用 *事件溯源领域模型* 模型这个术语，而不是仅仅使用 *事件溯源*。无论是否使用领域模型构建块，使用事件来表示状态转换--也就是事件溯源模式--都是可能的。因此，我更倾向于在较长的时间内明确指出，我们正在使用事件溯源来表示领域模型中的聚合在其生命周期内的变化。

--

### 优势

与更传统的模式相比，即聚合的当前状态被持久化保存在数据库中，事件溯源领域模型需要更多的努力来为聚合建模。不过，这种方法带来了显著的优势，使得该模式在许多场景中值得被考虑使用。

*时间旅行*

正如领域事件可以用来重建一个聚合的当前状态一样，它们也可以用来恢复聚合的所有过去状态。换句话说，你总是可以重新构建一个聚合的所有过去的状态。换句话说，你始终可以重新构建聚合的所有过去的状态。

这通常在分析系统行为、检查系统决策和优化业务逻辑时完成。

重组过去状态的另一个常见情况是追溯性调试：你可以将聚合恢复到发现错误时的确切状态。

*深入洞察*

在本书的第 I 部分中，我们看到优化核心子域对于业务具有重要的战略意义。事件溯源提供了对系统状态和行为的深入洞察。正如你在本章之前了解到的，事件溯源提供了灵活的模型，允许将事件转换为不同的状态表示——你总是可以添加新的投影，利用现有事件的数据来提供额外的洞察力。

*审计日志*

持久化的领域事件代表了发生在聚合状态上的每件事的强一致性审计日志。法律规定一些业务领域必须实现这样的审计日志，而事件溯源提供了这种开箱即用的服务。

这种模式对于管理货币或货币交易的系统来说特别方便。它使我们能够轻松地追踪系统的决策和账户之间的资金流动。

*高级乐观并发处理*

当读取的数据在写入时变得过时（被另一个进程覆盖）时，经典的乐观并发模型会引发异常。

当使用事件溯源时，我们可以更深入地了解在读取现有事件和编写新事件之间到底发生了什么。你可以查询 并发追加到事件存储中的确切事件，并做出业务领域驱动的决定，即新的事件是否与尝试的操作相冲突，或者追加的事件是不相关的，可以安全地进行。

### 劣势

到目前为止，似乎事件溯源领域模型是实现业务逻辑的终极模式，因此应该尽可能多地使用。当然，这与让业务领域的需求来驱动设计决策的原则相矛盾。所以，让我们来讨论一下该模式所带来的一些挑战。

*学习曲线*

该模式的明显缺点是它与传统的数据管理技术有着鲜明的区别。成功实现该模式需要对团队进行培训，并需要时间来适应新的思维方式。除非团队已经有实现事件溯源系统的经验，否则必须考虑到学习曲线。

*演化模型*
演化事件溯源模型可能具有挑战性。事件溯源的严格定义是说事件是不可变的。但是，如果你需要调整事件的架构怎么办？该过程不像更改数据库表的架构那么简单。事实上，仅针对这个主题就写了一整本书：Greg Young 所著的 [《事件溯源系统中的版本控制》](https://leanpub.com/esversioning)。

*架构复杂性*
事件溯源的实现引入了许多架构上的 "活动部件"，使得整个设计更加复杂。这个主题将在下一章讨论 CQRS 架构时更详细地介绍。

如果手头的任务没有理由使用该模式，而可以通过更简单的设计来解决，那么所有这些挑战就会更加严峻。在第 10 章中，你将学习简单的经验法则，它可以帮助你决定使用哪种业务逻辑实现模式。

## 常见问题

当向工程师介绍事件溯源模式时，他们经常会问几个常见的问题，因此我觉得有必要在本章中解决这些问题。

### 性能

*从事件中重建一个聚合的状态将对系统的性能产生负面影响。性能将随着事件的增加而降低。这怎么能行得通呢？*

将事件投影到状态表征中确实需要计算能力，而且随着越多的事件被添加到聚合的列表中，这种需求就会越大。

投影对性能的影响需要进行基准测试是很重要的：需要观测处理成百上千个事件的效果。应将结果与聚合的预期寿命进行比较——预期聚合在平均寿命期间记录的事件总数。

在大多数系统中，只有在每个聚合具有 10,000 个以上的事件后，性能才会受到明显的影响。一般来说，在绝大多数的系统中，一个聚合的平均生命周期中不会超过 100 个事件。

在极少数情况下，当投影状态确实成为一个性能问题时，可以实施另一种模式：快照（snapshot）。这个模式如图7-2所示，实现了以下步骤。

* 一个过程不断地迭代事件存储中的新事件，生成相应的投影，并将它们存储在缓存中。
* 需要一个内存中的投影来执行聚合的动作，在这种情况下：

	— 该过程从缓存中获取当前状态投影（快照）。
	
	— 该过程从事件存储中获取快照版本之后的事件。
	
	— 将额外的事件应用在内存中的快照上。

<img src=images/figure7-2.png />
图 7-2. 对聚合的事件进行快照 <br><br>

值得重申的是，快照模式是一种必须证明其合理性的优化。如果系统中的聚合不会持久化超过 10,000 个事件，那么实现快照模式只会引入意料之外的复杂性。但在你继续实施快照模式之前，我建议你后退一步，仔细检查聚合的边界。

*这个模型产生了大量的数据。它能被扩展吗？*

事件溯源模型易于扩展。由于所有与聚合相关的操作都是在单个聚合的上下文中完成的，因此事件存储可以按聚合 ID 进行分片：属于聚合实例的所有事件都应驻留在单个分片中（参见图 7-3）。

<img src=images/figure7-3.png />
图 7-3. 分片事件存储 <br><br>

### 删除数据

*事件存储是一个只能追加的数据库，但是如果我确实需要物理删除数据怎么办？例如，遵守 GDPR？[^3]*

这一需求可以使用可遗忘有效载荷（forgettable payload）模式 [^4] 来解决：所有敏感信息都以加密的形式包含在事件中。加密密钥存储在外部键值存储中：密钥存储，其中键是特定聚合的 ID，值是加密密钥。当敏感数据必须被删除时，加密密钥将从密钥存储中删除。因此，事件中包含的敏感信息将不能再被访问。

### 为什么我不能……？

*为什么我不能把日志写到一个文本文件中，并把它作为审计日志使用？*

将数据同时写入操作数据库和日志文件是一个容易出错的操作。从本质上讲，它是针对两种存储机制的事务：数据库和文件。如果第一个失败，则第二个必须回滚。例如，如果一个数据库事务失败，没有人会关心删除之前的日志消息。因此，这样的日志不具有一致性，而且不具备最终一致性。

*为什么我不能继续使用基于状态的模型，比如在同一个数据库事务中，将日志追加到日志表中？*

从基础架构的角度来看，这种方法确实提供了状态和日志记录之间的一致性同步。但是，它仍然容易出错。如果将来要在代码库上工作的工程师忘记追加适当的日志记录怎么办？

此外，当使用基于状态的表征作为真相之源时，追加日志表的模式通常会迅速退化为一团乱麻。没有办法强制要求所有需要的信息都被写下来，而且是以正确的格式写下来。

*为什么我不能继续使用基于状态的模型，比如添加一个数据库触发器，该触发器将拍摄记录的快照并将其复制到专用的“历史”表中？*

这种方法克服了前一种方法的缺点：不需要明确的手动调用来追加记录到日志表中。尽管如此，所产生的历史记录只包括干巴巴的事实：哪些字段被改变了。它忽略了业务背景：为什么这些字段会被改变。缺少 "为什么" 极大地限制了投影出额外模型的能力。

## 结论

本章解释了事件溯源模式及其在领域模型的聚合中对时间维度建模的应用。

在事件溯源领域模型中，对聚合状态的所有变更都表示为一系列领域事件。这与更传统的方法形成对比，在传统方法中，状态变更只是更新数据库中的记录。生成的领域事件可用于投影聚合的当前状态。此外，基于事件的模型使我们能够灵活地将事件投影到多个展现模型中，每个模型都针对特定任务进行了优化。

这种模式适合于对系统数据有深入了解是非常关键的情况，无论是为了分析和优化，还是因为法律要求的审计日志。

本章展示了我们对建模和实现业务逻辑的不同方法的探索。在下一章中，我们将把注意力转移到属于更高层级的模式：架构模式。

## 练习
1. 关于领域事件和值对象之间的关系，以下哪项陈述是正确的？<br>
A. 领域事件使用值对象来描述业务领域中所发生的事情。 <br>
B. 当实现一个事件溯源领域模型时，值对象应该被重构为事件溯源聚合。 <br>
C. 值对象与领域模型模式有关，在事件溯源领域模型中被领域事件所取代。 <br>
D. 以上所有陈述都不正确。 <br>

2. 关于从一系列事件中投影状态的选项，下列哪项陈述是正确的？<br>
A. 可以从聚合的事件中投影出单一的状态表示。 <br>
B. 可以投影多个状态表征，但领域事件必须以支持多种投影的方式被建模。 <br>
C. 可以投影多个状态表征，并且你可以在未来随时添加其他投影。<br>
D. 以上所有陈述都不正确。 <br>

3. 关于基于状态的聚合和基于事件的聚合之间的区别，以下哪项陈述是正确的？<br>
A. 事件溯源聚合可以产生领域事件，而基于状态的聚合不能产生领域事件。 <br>
B. 聚合模式的两种变体都会产生领域事件，但只有事件溯源聚合才会使用领域事件作为真相之源。 <br>
C. 事件溯源聚合确保每个状态转换都会产生领域事件。<br>
D. B 和 C 都正确。 <br>

4. 回到本书序言中描述的 WolfDesk 公司，该系统的哪些功能适合作为事件溯源领域模型来实现？

[^1]: Brooks, F. P. Jr. (1974). 人月神话：软件项目管理之道。Reading, MA: AddisonWesley
[^2]: 例外情况除外，例如数据迁移。
[^3]: 通用数据保护条例。（n.d.）2021 年 6 月 14 日从 [维基百科](https://oreil.ly/08px7) 检索。
[^4]: 译注：https://verraes.net/2019/05/eventsourcing-patterns-forgettable-payloads/