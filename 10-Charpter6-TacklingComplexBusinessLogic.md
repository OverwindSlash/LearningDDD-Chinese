## 第 6 章
# 处理复杂业务逻辑

上一章讨论了两种解决相对简单的业务逻辑的模式：事务脚本和活动记录。本章继续讨论实现业务逻辑的话题，并介绍一个面向复杂业务逻辑的模式：领域模型模式。

## 历史
与事务脚本和活动记录模式一样，领域模型模式最初是在 Martin Fowler 的《*企业应用架构模式*》一书中引入的。Fowler 在结束对该模式的讨论时说：“Eric Evans 目前正在写一本关于构建领域模型的书。” 被提及的书就是 Evans 的开创性著作《*领域驱动设计：解决软件核心的复杂性*》。

在 Evans 的书中，他提出了一组模式，旨在将代码与业务领域的底层模型紧密联系起来：聚合、值对象、资源库等。这些模式紧随 Fowler 书中的收尾处，并且仿效出了一组用于实现领域模型模式的有效工具。

Evans 引入的模式通常被称为 *战术领域驱动设计*。实施领域驱动设计必然需要使用这些模式来实现业务逻辑，为了消除思维混乱，我更愿意坚持 Fowler 的原始术语。该模式就是 “领域模型”，而聚合和值对象是它的构建块。

## 领域模型
领域模型模式是为了应对复杂业务逻辑的情况。这时，我们处理的不是 CRUD 接口，而是复杂的状态转换、业务规则和不变量：这些规则必须始终受到保护。

假设我们正在实现一个帮助台系统（help desk system）。考虑下面的需求摘录，它描述了控制支持工单（support tickets）生命周期的逻辑。

* 客户打开支持工单，描述他们面临的问题。
* 客户和支持专员（support agent）都会对问题追加信息，所有的沟通都由支持工单来跟踪。
* 每张工单都有一个优先级：低、中、高或紧急。
* 支持专员应在设定的时限（SLA，Service-Level Agreement 服务等级协议）内提供解决方案，该时限基于工单的优先级。
* 如果支持专员未能在 SLA 内回复，客户可以将工单上报给专员的经理。
* 工单升级会使专员的响应时限减少 33%。
* 如果支持专员没有在 50% 的响应时限内打开已升级的工单，它就会被自动重新分配给另一个支持专员。
* 如果客户在 7 天内未回复支持专员的问题，那么工单将自动关闭。
* 已升级工单不能自动或由支持专员关闭，其只能由客户或支持专员的经理关闭。
* 客户只能重新打开在过去七天内关闭的工单。

这些需求在不同的规则之间形成了一个纠缠不清的依赖关系网，并且都影响着支持工单的生命周期管理逻辑。正如我们在上一章中讨论的那样，这不是一个 CRUD 数据输入屏幕。试图用活动记录对象来实现这个业务逻辑将很容易出现重复，并可能因为业务规则的错误实现而对系统状态造成破坏。

### 实现
领域模型是一个包含了行为和数据的领域中的对象模型。[^1] DDD 的战术模式——聚合、值对象、领域事件和领域服务——是此类对象模型的构建块。[^2]

所有这些模式都有一个共同的主题：它们将业务逻辑放在首位。让我们看看领域模型是如何处理不同的设计考量。

#### 复杂度
领域的业务逻辑本身已经很复杂了，所以用于建模的对象不应引入任何额外的和偶然的复杂性。该模型应该避免关联任何基础设施或技术关注点，例如实现对数据库或系统其他外部组件的调用。这种限制要求模型的对象必须是 *简单对象*（plain old objects），即实现业务逻辑的对象不依赖于也不直接包含任何基础设施组件或框架。[^3]

#### 通用语言
强调业务逻辑而不是技术问题，使得领域模型的对象更容易遵循限界上下文中通用语言的术语。换句话说，这种模式允许代码"说"通用语言，并遵循领域专家的心理模型。

### 构建块
让我们来看看 DDD 提供的主要领域模型构建块，或战术模式：值对象、聚合和领域服务。

#### 值对象
值对象是一个可以通过其值的组合来识别的对象。例如，考虑一个颜色对象。

```cs
class Color
{
    int _red;
    int _green;
    int _blue;
}
```

红、绿、蓝三个字段值的组合定义了一种颜色。改变其中一个字段的值会产生一种新的颜色。没有两种颜色可以有相同的值。而且，同一颜色的两个实例必须有相同的值。因此，不需要明确的 id 字段来标识颜色。

图 6-1 中所示的 color-id 字段不仅是多余的，而且实际上为 bug 创造了机会。你可以创建具有相同红色、绿色和蓝色值的两行，但比较 color-id 的值不会反映出这是相同的颜色。

<img src=images/figure6-1.png />
图 6-1. 冗余的 color-id 字段，使得有可能出现具有相同值的两行 <br><br>

**通用语言** 完全依赖语言标准库的原生数据类型（例如 string、integer 或 dictionary）来表示业务领域概念被称为原生痴迷坏味道（primitive obsession）。[^4] 例如，考虑以下类：

```cs
class Person
{
    private int    _id;
    private string _firstName;
    private string _lastName;
    private string _landlinePhone;
    private string _mobilePhone;
    private string _email;
    private int    _heightMetric;
    private string _countryCode;

    public Person(...) {...}
}

static void Main(string[] args)
{
    var dave = new Person(
        id: 30217,
        firstName: "Dave",
        lastName: "Ancelovici",
        landlinePhone: "023745001",
        mobilePhone: "0873712503",
        email: "dave@learning-ddd.com",
        heightMetric: 180,
        countryCode: "BG");
}
```

在上述 Person 类的实现中，大部分的值都是 String 类型，它们根据惯例被赋予字符串形式的值。例如，landlinePhone 的输入应该是一个有效的固定电话号码，而 countryCode 应该是一个有效的、两个字母的、大写的国家代码。当然，系统不能迷信用户总是能提供正确的值，因此，这个类必须验证所有的输入字段。

这种方法存在多种设计风险。首先，验证逻辑往往是重复的。其次，很难在使用值之前强制调用验证逻辑。将来，当代码库转由其他工程师维护时，它将变得更具挑战性。

比较以下同一对象的替代设计，这次利用值对象：

```cs
class Person
{
    private PersonId     _id;
    private Name         _name;
    private PhoneNumber  _landline;
    private PhoneNumber  _mobile;
    private EmailAddress _email;
    private Height       _height;
    private CountryCode  _country;

    public Person(...) { ... }
}

static void Main(string[] args)
{
    var dave = new Person(
        id:       new PersonId(30217),
        name:     new Name("Dave", "Ancelovici"),
        landline: PhoneNumber.Parse("023745001"),
        mobile:   PhoneNumber.Parse("0873712503"),
        email:    EmailAddress.Parse("dave@learning-ddd.com"),
        height:   Height.FromMetric(180),
        country:  CountryCode.Parse("BG"));
}
```

首先，注意到明确程度的提高。例如，以 _country 变量为例。没有必要精心地把它叫做 "countryCode", 以传达它持有的是国家代码，而不是国家完整名称的意图。值对象使意图清晰，即使是较短的变量名。

其次，不需要在赋值之前进行验证，因为验证逻辑存在于值对象本身之中。然而，值对象的行为不仅限于验证。当值对象集中处理与值相关的业务逻辑时，它的光芒最耀眼。内聚性的逻辑在同一处实现，并且易于测试。最重要的是，值对象表达了业务领域的概念：它们让代码使用通用语言。

让我们看看如何将身高、电话号码和颜色等概念表示为值对象，从而使产生的类型系统使用起来更加丰富和直观。

与基于 integer 类型的值相比，Height 值对象既能使意图明确，又能使测量与特定的测量单位脱钩。例如，Height 对象可以使用公制和英制单位进行初始化，这样就很容易从一种单位转换到另一种单位，生成字符串形式表示，以及比较不同单位的值。

```cs
var heightMetric = Height.Metric(180);
var heightImperial = Height.Imperial(5, 3);

var string1 = heightMetric.ToString();             // "180 厘米"
var string2 = heightImperial.ToString();           // "5 英尺 3 英寸"
var string3 = height1.ToImperial().ToString();     // "5 英尺 11 英寸"

var firstIsHigher = heightMetric > heightImperial; // true
```

PhoneNumber 值对象可以封装字符串值解析、验证以及提取电话号码不同属性的逻辑；例如，它所属的国家和电话号码的类型——固定电话或手机：

```cs
var phone = PhoneNumber.Parse("+359877123503");
var country = phone.Country;                        // "BG"
var phoneType = phone.PhoneType;                    // "手机"
var isValid = PhoneNumber.IsValid("+972120266680"); // false
```

以下例子展示了值对象的强大，它封装了所有操作数据的业务逻辑，并生成了值对象的新实例。

```cs
var red = Color.FromRGB(255, 0, 0);
var green = Color.Green;
var yellow = red.MixWith(green);
var yellowString = yellow.ToString();               // "#FFFF00"
```

正如你在前面的示例中看到的那样，值对象消除了对约定的需要——例如，需要记住这个字符串是电子邮件而另一个字符串是电话号码——这样让使用对象模型不容易出错，且更直观。

**实现** 由于对值对象任何字段的改变都会导致不同的值，因此值对象被实现为不可变对象。对值对象某个字段的更改在概念上会创建出一个不同的值——也就是值对象的一个​​不同的实例。因此，当执行的操作产生新值时，就像下面这个使用 MixWith 方法的示例一样，它不会修改原始实例，而是实例化并返回一个新的实例。

```cs
public class Color
{
    public readonly byte Red;
    public readonly byte Green;
    public readonly byte Blue;

    public Color(byte r, byte g, byte b)
    {
        this.Red = r;
        this.Green = g;
        this.Blue = b;
    }

    public Color MixWith(Color other)
    {
        return new Color(
            r: (byte) Math.Min(this.Red + other.Red, 255),
            g: (byte) Math.Min(this.Green + other.Green, 255),
            b: (byte) Math.Min(this.Blue + other.Blue, 255)
        );
    }

    ...
}
```

由于值对象的相等性基于它们的值而不是基于 id 字段或引用，因此重写并正确实施相等性检查很重要。例如，在 C#:[^5]

```cs
public class Color
{
    ...
    
    public override bool Equals(object obj)
    {
        var other = obj as Color;
        return other != null &&
            this.Red == other.Red &&
            this.Green == other.Green &&
            this.Blue == other.Blue;
    }

    public static bool operator == (Color lhs, Color rhs)
    {
        if (Object.ReferenceEquals(lhs, null)) {
            return Object.ReferenceEquals(rhs, null);
        }
        return lhs.Equals(rhs);
    }

    public static bool operator != (Color lhs, Color rhs)
    {
        return !(lhs == rhs);
    }

    public override int GetHashCode()
    {
        return ToString().GetHashCode();
    }

    // ...
}
```

尽管使用核心类库中的 String 类型来表示特定领域中的值这种做法有违值对象的概念，但在 .NET、Java 和其他语言中，String 类型被完全实现为值对象。String 是不可变的，因为所有操作都会产生一个新的实例。此外，String 类型封装了丰富的行为，通过操作一个或多个 String 的值从而创建新的实例：修剪（trim）、连接（concatenate）多个字符串、替换（replace）字符、子串（substring）和其他方法。

**何时使用值对象** 简单的答案是，尽可能的使用。值对象不仅使代码更具表现力并封装了易于分散的业务逻辑，而且该模式还使代码更安全。由于值对象是不可变的，因此值对象的行为没有副作用并且是线程安全的。

从业务领域的角度来看，一个有用的经验法则是：对描述其他对象属性的领域元素使用值对象。这也适用于实体的属性，相关内容将在下一节中讨论。你前面看到的例子使用了值对象来描述一个人，包括他们的 ID、姓名、电话号码、电子邮件等。使用值对象的其他例子包括各种状态、密码和更多特定业务领域的概念，这些概念可以通过它们的值来识别，因此不需要明确的 id 字段。引入值对象的一个特别重要的时机是在为货币和其他货币化价值（monetary values）建模时。依靠原生类型来表示货币，不仅限制了你将所有与货币相关的业务逻辑封装在一个地方的能力，而且还经常导致危险的 bug，如舍入错误和其他与精度相关的问题。

#### 实体
*实体* 是值对象的对立面。它需要一个明确的 id 字段来区分实体的不同实例。实体的一个简单例子是人。考虑下面这个类：

```cs
class Person
{
    public Name Name { get; set; }
    
    public Person(Name name)
    {
        this.Name = name;
    }
}
```

该类只包含一个字段：Name（一个值对象）。然而，这种设计并不理想，因为不同的人可以同名同姓。当然，这并不能使他们成为同一个人。因此，需要一个 Id 字段来正确标识：

```cs
class Person
{
    public readonly PersonId Id;
    public Name Name { get; set; }
    
    public Person(PersonId id, Name name)
    {
        this.Id = id;
        this.Name = name;
    }
}
```

在上述代码中，我们引入了类型为 PersonId 的标识字段 Id。PersonId 是一个值对象，它可以使用任何符合业务领域需求的基础数据类型。例如，Id 可以是 GUID，数字，字符串，或特定领域的值，如社会安全号码。

对标识字段的核心要求是，它对实体的每个实例都应该是唯一的：在我们的例子中，对每个人都是如此（图6-2）。此外，除了非常罕见的例外情况，实体的标识字段的值应该在实体的整个生命周期内保持不可改变。这就给我们带来了值对象和实体之间的第二个概念上的区别。

<img src=images/figure6-2.png />
图 6-2. 引入显式标识字段，允许区分对象的实例，即使所有其他字段的值都相同 <br><br>

与值对象相反，实体并不是不可改变的，预期就会有变化。实体和值对象之间的另一个区别是，值对象描述实体的属性。在本章的前面，你看到了一个 Person 实体的例子，它有两个描述实例的值对象。PersonId 和 Name。

实体是任何业务领域的基础构建块。你可能已经注意到，在本章的前面，我没有在领域模型的构建块列表中包含“实体”。这并不是一个错误。省略“实体”的原因是因为我们不是独立地实现实体，而是仅在聚合模式的上下文中实现它。

#### 聚合
聚合是一个 *实体*：它需要一个明确的标识字段，并且它的状态预计会在实例的生命周期中发生变化。然而，聚合远不止是一个实体。该模式的目标是保护其数据的一致性。由于聚合的数据是可变的，因此该模式必须处理一些影响和挑战，以保持聚合的状态在任何时候都具有一致性。

**一致性增强** 由于聚合的状态是可以改变的，这就为多种破坏聚合内数据的方式创造了条件。为了确保数据的一致性，聚合模式在聚合和它的外部范围之间画了一个明确的边界：聚合是一致性的强制边界。聚合的逻辑必须验证所有传入的修改，并确保这些修改不会与聚合的业务规则相矛盾。

从实现的角度来看，一致性是通过只允许聚合内部的业务逻辑修改其状态来实现的。聚合外部的所有进程或对象只允许读取聚合的状态。若想从外部改变聚合的状态，只有通过执行其公开接口中的相应方法来实现。

聚合的公开接口中会暴露修改聚合状态的方法，通常这些方法被称之为 *命令*，就像 "做某事的命令"一样。命令可以通过两种方式实现。首先，它可以作为聚合对象的普通公有方法来实现：

```cs
public class Ticket
{
    // ...
 
    public void AddMessage(UserId from, string subject, string body)
    {
        var message = new Message(from, body);
        _messages.Append(message);
    }
 
    // ...
}
```

或者，命令可以表示为参数对象，它封装了执行命令所需的所有输入：

```cs
public class Ticket
{
    // ...
 
    public void Execute(AddMessage cmd)
    {
        var message = new Message(cmd.from, cmd.body);
        _messages.Append(message);
    }

    // ...
}
```

命令如何在聚合的代码中表达是一个偏好问题。我倾向于使用更明确的方式来定义命令结构，并将它们以多态的方式传递给相关的 Execute 方法。

聚合的公开接口负责验证输入并确保所有相关的业务规则和不变性。这种严格的边界也保证了所有与聚合有关的业务逻辑都实现在同一个地方：聚合本身。

这使得在聚合之上编排操作的应用层（application layer）[^6] 相当简单：[^7] 它所要做的就是加载聚合的当前状态，执行所需的操作，持久化修改过的状态，并将操作的结果返回给调用者。

```cs
 1 public ExecutionResult Escalate(TicketId id, EscalationReason reason)
 2 {
 3     try
 4     {
 5         var ticket = _ticketRepository.load(id);
 6         var cmd = new Escalate(reason);
 7         ticket.Execute(cmd);
 8         _ticketRepository.Save(ticket);
 9         return ExecutionResult.Success();
10     }
11     catch (ConcurrencyException ex)
12     {
13         return ExecutionResult.Error(ex);
14     }
15 } 
```

请注意上述代码（第 11 行）中的并发检查。这对于保护聚合状态的一致性至关重要。[^8] 如果多个进程同时更新同一聚合，我们必须防止后一个事务盲目地覆盖前一个事务所提交的修改。在这种情况下，当第二个进程进行决策所依据的状态已经过时，必须通知它重试操作。

因此，用于存储聚合的数据库必须支持并发管理。在其最简单的形式中，聚合应该持有一个版本字段，该字段在每次更新后都会被递增。

```cs
class Ticket
{
    TicketId _id;
    int      _version;

    // ...
}
```

当提交对数据库的更改时，我们必须确保被更新记录的版本与最初读取的版本相一致。例如，在 SQL 中：

```sql
 1 UPDATE tickets
 2 SET ticket_status = @new_status,
 3 agg_version = agg_version + 1,
 4 WHERE ticket_id=@id and agg_version=@expected_version; 
```

此 SQL 语句应用对聚合实例状态所做的更改（第 2 行），并增加其版本计数（第 3 行），但前提是记录的当前版本必须等于更新操作之前所读取的版本（第 4 行）。

当然，除了关系型数据库，并发管理还可以在其他地方实现。此外，文档数据库更适合与聚合一起工作。总而言之，确保用于存储聚合数据的数据库支持并发管理是至关重要的。

**事务边界** 由于聚合的状态只能由其自身的业务逻辑修改，因此聚合也充当事务边界。对聚合状态的所有更改都应作为一个原子操作以事务方式提交。如果聚合的状态被修改，则要么提交所有更改，要么都不提交。

此外，没有任何系统操作可以假设多聚合事务。对聚合状态的更改只能单独提交，每个数据库事务一个聚合。

每个事务只有一个聚合实例，这迫使我们仔细设计聚合的边界，确保设计能解决业务领域的不变量和业务规则。对于在多个聚合中提交更改的需求意味着错误的事务边界，因此也是错误的聚合边界。

这似乎强加了对于建模的限制。如果我们需要在同一个事务中修改多个对象怎么办？让我们看看该模式如何处理这样的场景。

**实体层次结构** 正如我们在本章前面讨论的那样，我们不将实体用作独立模式，而只是作为聚合的一部分。让我们看看实体和聚合之间的根本区别，以及为什么实体是聚合的构建块而不是总体领域模型的构建块。

在一些业务场景中，多个对象应该共享一个事务边界；例如，当两者可以同时被修改，或者一个对象的业务规则取决于另一个对象的状态时。

DDD 规定，一个系统的设计应该由其业务领域驱动。聚合也不例外。为了支持在一个原子事务中应用对多个对象的更改，聚合模式类似于实体的层次结构，所有实体都共享事务上的一致性，如图6-3所示。

<img src=images/figure6-3.png />
图 6-3. 将聚合作为实体的层次结构 <br><br>

层次结构包含实体和值对象，如果它们受领域的业务逻辑所约束，那么它们就同属于一个聚合。

这就是该模式被命名为“聚合”的原因：它将属于同一事务边界的业务实体和值对象聚合在一起。

以下代码示例展示了一个业务规则，该规则跨越属于聚合边界的多个实体——“如果支持专员没有在 50% 的响应时限内打开已升级的工单，它就会被自动重新分配给另一个支持专员。”：

```cs
 1 public class Ticket
 2 {
 3     // ...
 4     List<Message> _messages;
 5     // ...
 6 
 7     public void Execute(EvaluateAutomaticActions cmd)
 8     {
 9         if (this.IsEscalated && this.RemainingTimePercentage < 0.5 &&
10                 GetUnreadMessagesCount(forAgent: AssignedAgent) > 0)
11         {
12             _agent = AssignNewAgent();
13         }
14     }
15 
16     public int GetUnreadMessagesCount(UserId id)
17     {
18         return _messages.Where(x => x.To == id && !x.WasRead).Count();
19     }
20 
21     // ...
22 } 
```

该方法检查工单的属性以查看它是否升级以及剩余处理时间是否小于定义的阈值 50%（第 9 行）。此外，它还检查当前支持专员尚未读取的信息（第10行）。如果所有条件都满足，则请求将该工单重新分配给不同的支持专员。

聚合确保所有的条件都是针对强一致性数据进行检查的，并且在检查完成后不会发生变化，以保证对聚合数据的所有变更都是作为一个原子事务进行的。

**引用其他聚合** 由于聚合包含的所有对象共享相同的事务边界，因此如果聚合增长的太大，可能会出现性能和可伸缩性问题。

数据的一致性可以成为设计聚合边界的简易指导原则。只有那些被聚合的业务逻辑要求为强一致性的信息才应该是聚合的一部分。所有可接受最终一致性的信息都应该在聚合的边界之外；比如，作为另一个聚合的一部分，如图 6-4 所示。

<img src=images/figure6-4.png />
图 6-4. 将聚合作为一致性边界 <br><br>

经验法则是保持聚合尽可能小，仅包括聚合的业务逻辑要求处于强一致状态的对象。

```cs
public class Ticket
{
    private UserId          _customer;
    private List<ProductId> _product;
    private UserId          _assignedAgent;
    private List<Message>   _messages;

    // ...
}
```

在上面的示例中，Ticket 聚合引用了属于聚合边界内的消息集合。另一方面，客户、与工单相关的产品集合以及被分配的支持专员不属于此聚合，所以由其 ID 引用。

通过 ID 来引用外部聚合的理由是为了重申这些对象不属于聚合的边界，并确保每个聚合有自己的事务边界。

要确定一个实体是否属于某个聚合，请检查如果此实体工作于最终一致性数据时，聚合是否包含可能导致无效系统状态的业务逻辑。让我们回到之前的例子，如果当前代理没有在 50% 的响应时限内阅读新消息，则重新分配工单。如果有关已读/未读消息的信息是最终一致性的怎么办？换句话说，在一定延迟之后接收到已读确认是可能的。在这种情况下，可以预见会有相当多的工单被不必要地重新分配。这当然会破坏系统的状态。因此，已读/未读消息中的数据属于聚合的边界内。

**聚合根** 我们之前看到聚合的状态只能通过执行它的命令来修改。由于聚合代表了实体的层次结构，因此只有其中的一个实体应该被指定为聚合的公开接口——也就是聚合根，如图 6-5 所示。

<img src=images/figure6-5.png />
图 6-5. 聚合根 <br><br>

考虑以下 Ticket 聚合的源代码摘录：

```cs
public class Ticket
{
   // ...
   List<Message> _messages;
   // ...
 
   public void Execute(AcknowledgeMessage cmd)
   {
      var message = _messages.Where(x => x.Id == cmd.id).First();
      message.WasRead = true;
   }
   // ...
}
```

在这个例子中，聚合暴露了一个命令，此命令可将一个特定的消息标记为已读。尽管该操作修改了 Message 实体的实例，但它只能通过其聚合根 Ticket 访问。

除了聚合根的公开接口之外，还有另一种机制：领域事件，外部世界可以通过它与聚合进行交流。

**领域事件** 领域事件是一种消息，它描述了业务领域中发生的重大事件。比如说。

* 工单已分配
* 工单已升级
* 消息已接收

由于领域事件描述的是已经发生的事情，因此它们的名称应该使用过去式来表述。

领域事件的目标是描述业务领域中所发生的事情，并提供与该事件相关的所有必要数据。例如，以下领域事件表明特定工单已升级、包含升级的时间和原因：

```json
{
    "ticket-id": "c9d286ff-3bca-4f57-94d4-4d4e490867d1",
    "event-id": 146,
    "event-type": "ticket-escalated",
    "escalation-reason": "missed-sla",
    "escalation-time": 1628970815
}
```

与软件工程中几乎所有的事情一样，命名很重要。要确保领域事件的名称简洁地反映业务领域中所发生的事情。

领域事件是聚合公开接口的一部分。聚合发布它的领域事件。其他进程、聚合、甚至外部系统可以订阅并执行他们自己的逻辑来响应领域事件，如图6-6所示。

<img src=images/figure6-6.png />
图 6-6. 领域事件发布流 <br><br>

如下 Ticket 聚合的源代码摘录中，一个新的领域事件被实例化（第12行）并被附加到 Ticket 的领域事件集合中（第13行）。

```cs
 1 public class Ticket
 2 {
 3     // ...
 4     private List<DomainEvent> _domainEvents;
 5     // ...
 6 
 7     public void Execute(RequestEscalation cmd)
 8     {
 9         if (!this.IsEscalated && this.RemainingTimePercentage <= 0)
10         {
11             this.IsEscalated = true;
12             var escalatedEvent = new TicketEscalated(_id);
13             _domainEvents.Append(escalatedEvent);
14         }
15     }
16 
17     // ...
18 } 
```

在第 9 章中，我们将讨论如何将领域事件可靠地发布给感兴趣的订阅者。

**通用语言** 最后但并非最不重要的，聚合应该反映通用语言。聚合的名称、数据成员、行为和领域事件所使用的术语都应该使用限界上下文中的通用语言来表述。正如 Eric Evans 所说，源代码必须基于开发人员在与彼此和领域专家交谈时使用的相同语言。这对于实现复杂的业务逻辑尤为重要。

现在让我们来看看领域模型的第三个也是最后一个构建块。

#### 领域服务
或早或迟，你可能会遇到这样的业务逻辑：它要么不属于任何聚合或值对象，要么似乎与多个聚合相关。这种情况下，领域驱动设计建议将逻辑作为 *领域服务* 来实现。

*领域服务* 是一个实现了业务逻辑的无状态对象。在绝大多数情况下，这种逻辑编排了对系统中各种组件的调用，以执行一些计算或分析。

让我们再回到工单聚合的例子。回顾一下，被分配的支持专员有一个有限的时间段来向客户提出一个解决方案。时间段不仅取决于工单的属性（其优先级和升级状态），还取决于支持专员部门关于每个优先级的 SLA 政策以及专员的工作时间表（班次）—— 我们不能指望专员在下班时间响应。

响应时间段的计算逻辑需要基于多个信息来源：工单、被分配专员所在部门和工作日程。这使得它成为一个理想的候选者，可以作为领域服务来实现。

```cs
public class ResponseTimeFrameCalculationService
{
    // ...

    public ResponseTimeframe CalculateAgentResponseDeadline(UserId agentId,
        Priority priority, bool escalated, DateTime startTime)
    {
        var policy = _departmentRepository.GetDepartmentPolicy(agentId);
        var maxProcTime = policy.GetMaxResponseTimeFor(priority);

        if (escalated) {
            maxProcTime = maxProcTime * policy.EscalationFactor;
        }

        var shifts = _departmentRepository.GetUpcomingShifts(agentId,
            startTime, startTime.Add(policy.MaxAgentResponseTime));
        
        return CalculateTargetTime(maxProcTime, shifts);
    }

    // ...
}
```

领域服务使编排多个聚合的工作变得很容易。然而，重要的是要始终牢记聚合模式的限制，即在一个数据库事务中只能修改聚合的一个实例。领域服务不是绕过这一限制的漏洞。每个事务一个实例的规则仍然有效。相反，领域服务适合于实现需要 *读取* 多个聚合数据的计算逻辑。

同样重要的是要指出，领域服务与微服务、面向服务的架构或软件工程中几乎所有其他 *服务* 一词的用法都没有关系。它只是一个用于承载业务逻辑的无状态对象。

### 管理复杂度
正如本章介绍中所指出的，聚合和值对象模式是作为解决业务逻辑实现中复杂性的一种手段而被引入的。让我们来看看这背后的原因。

商业管理大师 Eliyahu M. Goldratt 在他的《选择》一书中概述了系统复杂性的一个简洁而有力的定义。根据 Goldratt 的说法，在讨论一个系统的复杂性时，我们关注的是评估关于控制和预测系统行为的难度。这两个方面由系统的自由度来反映。

一个系统的自由度是描述其状态所需的数据点。考虑以下两类。

```cs
public class ClassA
{
    public int A { get; set; }
    public int B { get; set; }
    public int C { get; set; }
    public int D { get; set; }
    public int E { get; set; }
}

public class ClassB
{
    private int _a, _d;

    public int A
    {
        get => _a;
        set
        {
            _a = value;
            B = value / 2;
            C = value / 3;
        }
    }

    public int B { get; private set; }
    
    public int C { get; private set; }
    
    public int D
    {
        get => _d;
        set
        {
            _d = value;
            E = value * 2
        }
    }

    public int E { get; private set; }
}
```

乍一看，ClassB 似乎比 ClassA 要复杂得多。它们拥有相同数量的变量，但在此基础上，ClassB 还实现了额外的计算逻辑。它比 ClassA 更复杂吗？

让我们从自由度的角度分析一下这两个类。你需要多少个数据元素来描述 ClassA 的状态？答案是五个：它的五个变量。因此，ClassA 有五个自由度。

你需要多少个数据元素来描述 ClassB 的状态？如果你看一下属性 A 和 D 的赋值逻辑，你会发现 B、C 和 E 的值是 A 和 D 值的函数。如果你知道了 A 和 D 是什么，那么你就可以推断出其余变量的值。因此，ClassB 只有两个自由度。你只需要两个值来描述它的状态。

回到最初的问题，在控制和预测其行为方面，哪一个类更难？答案是自由度较高的那一个，即 ClassA。ClassB 中引入的不变量降低了它的复杂性。这就是聚合和值对象模式的作用：封装不变量，从而降低复杂性。

所有与值对象状态相关的业务逻辑都位于其边界内。聚合也是如此。聚合只能通过其自身的方法进行修改。它的业务逻辑对业务不变量进行了封装和保护，从而降低了自由度。

由于领域模型模式仅适用于具有复杂业务逻辑的子域，因此可以放心的假设这些是核心子域——也就是软件的核心。

## 结论
领域模型模式针对复杂业务逻辑的情况。它由三个主要构建块组成：

*值对象*

业务领域中的概念，可以完全通过它们的值来识别，因此不需要一个明确的 ID 字段。由于其中一个字段的变化在语义上创造了新的值，所以值对象是不可改变的。

值对象不仅为数据建模，也为行为建模：方法操作值并因此初始化新的值对象。

*聚合*

一个共享了事务边界的实体层次结构。聚合边界所包含的所有数据都必须是强一致性的，以实现其业务逻辑。

聚合的状态以及其内部对象只能通过其公开接口，由执行聚合的命令来修改。数据字段对外部组件来说是只读的，以确保与聚合相关的所有业务逻辑都位于其边界内。

聚合作为事务的边界。它的所有数据，包括它的所有内部对象，都必须作为一个原子事务提交到数据库。

聚合可以通过发布领域事件（描述聚合生命周期中重要业务事件的消息）与外部实体通信。其他组件可以订阅这些事件并使用它们来触发业务逻辑的执行。

*领域服务*

一个无状态对象，其承载的业务逻辑，天然不属于任何领域模型中的聚合或值对象。

领域模型的构建块通过将业务逻辑封装在值对象和聚合的边界中来解决业务逻辑的复杂性。无法从外部修改对象的状态，确保了所有相关的业务逻辑都在聚合和值对象的边界内实现，不会在应用层重复。

在下一章中，你将学习实现领域模型模式的高级方法，这次是让时间这个维度成为模型的固有部分。

## 练习
1. 下面哪个描述是正确的？<br>
A. 值对象只能包含数据。 <br>
B. 值对象只能包含行为。 <br>
C. 值对象是不可变的。 <br>
D. 值对象的状态可以改变。 <br>

2. 设计聚合边界的一般性指导原则是什么？<br>
A. 一个聚合只能包含一个实体，因为单个数据库事务中只能包含一个聚合实例。<br>
B. 只要业务领域的数据一致性需求不受影响，聚合就应该被设计得尽可能小。 <br>
C. 聚合代表了实体的层次结构。因此，为了最大限度地提高系统数据的一致性，聚合应该被设计得尽可能宽。 <br>
D. 看情况而定：对于某些业务领域来说，小聚合是最好的，而在其他领域，使用尽可能大的聚合会更有效率。 <br>

3. 为什么一个事务中只能提交一个聚合实例？<br>
A. 确保模型能够在高负载下执行。<br>
B. 确保正确的事务边界。<br>
C. 没有这样的要求；这取决于业务领域。<br>
D. 使系统能够与不支持多记录事务的数据库一起工作，如键值和文档存储。<br>

4. 以下哪项陈述最能描述领域模型的构建块之间的关系？<br>
A. 值对象描述实体的属性。<br>
B. 值对象可以发出域事件。<br>
C. 聚合包含一个或多个实体。<br>
D. A 和 C。<br>

5. 以下哪项关于活动记录和聚合之间的差异的说法是正确的？<br>
A. 活动记录只包含数据，而聚合还包含行为。<br>
B. 聚合封装了它的所有业务逻辑，但操作活动记录的业务逻辑可以位于其边界之外。<br>
C. 聚合只包含数据，而活动记录包含数据和行为。<br>
D. 聚合包含一组活动记录。<br>

[^1]: Fowler, M. (2002). 企业应用架构的模式。波士顿。Addison-Wesley
[^2]: 本章中的所有代码示例都将使用面向对象的编程语言。然而，所讨论的概念并不局限于 OOP，与函数式编程范式同样相关.
[^3]: .NET 中的 POCO、Java 中的 POJO、Python 中的 POPO 等。
[^4]: "原生痴迷"。(n.d.) 2021年6月13日，来自 https://wiki.c2.com/?PrimitiveObsession
[^5]: 在 C# 9.0 中，新的 record 类型实现了基于值的相等性，因此不需要重写相等性运算符。
[^6]: 也被称为服务层，是将公开 API 操作转发到领域模型的系统组件
[^7]: 从本质上讲，应用层的操作实现了事务脚本模式。它必须将操作作为一个原子事务进行编排。对整个聚合的改变要么成功，要么失败，绝不会提交部分更新的状态。
[^8]: 回顾一下，应用层是事务脚本的集合，正如我们在第 5 章中所讨论的，为了防止竞争性的更新破坏系统的数据，并发管理是必不可少的。
