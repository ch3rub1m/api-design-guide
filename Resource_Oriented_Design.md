## 面向资源的设计
这份设计指导的目标是帮助开发者设计出简洁、一致并且易于使用的网络API。

在过去，人们参考像 CORBA 和 Windows COM 这样的 API 接口和方法来设计 RPC APIs。随着时间推移，越来越多的接口和方法被引入。这就导致了接口和方法的泛滥，并且他们的设计各异，互不相同。开发者们必须小心翼翼地学习每一个接口才能正确使用它，这不仅导致了时间的浪费，还容易出错。

REST的架构风格在2000年首次被引入，与HTTP/1.1有着良好的相性。他的核心原则在于定义一些可以被少量方法操作的资源。毫无疑问资源是个名词，方法是个动词。在HTTP协议中，资源名称很自然地被映射到URLs上，方法则被映射成了HTTP的POST、GET、PUT和DELETE四种方法。

在互联网上，HTTP REST APIs毫无疑问获得了巨大的成功，在2010年，有大约74%的开放网络API采用了REST设计。

虽然HTTP REST APIs在互联网上非常流行，但是他们承载的流量还是比传统的RPC APIs要少。举个例子，在高峰期，大约一半的互联网流量来自视频内容，而出于性能原因，几乎不会有人会考虑用REST API去传输这样的内容。在数据中心内部，很多的公司使用基于socket的RPC APIs去承载大部分的网络流量，这比开放的REST APIs可能要多好几个数量级。

现实点来说，RPC APIs和HTTP REST APIs因为不同的原因都不可或缺。在理想情况下，一个API平台应该为所有APIs提供最佳支持。这份设计指导将帮助你设计与构建遵从这个原则的APIs，并且会定义一些共用的设计模式以提高可用性并降低复杂度。

> 提示：这份文档解释了怎样将REST原则运用于API设计，并且这与编程语言、操作系统及网络协议无关。这不是一个教你怎样创建REST APIs的文档。

### 什么是REST API？
REST API被建模为可以被单独寻址的资源（API的名词）的集合。资源可以被他们的[资源名称](Resource_Names.md)引用，并通过一小组方法（也称为动词或操作）来进行操作。

REST Google APIs的标准方法（也被称为REST方法）是`List`, `Get`, `Create`, `Update`和`Delete`。自定义方法（也被称为自定义动词或自定义操作）也可供API设计人员使用，以应对那些不易映射到任何一种标准方法的功能，例如数据库事务。

> 提示：自定义动词并不意味着创建自定义的HTTP动词来支持自定义方法。对于基于HTTP的API，它们只是映射到最合适的HTTP动词。

### 设计流程
这份设计指南建议设计面向资源的APIs的时候按照以下几个步骤（以下具体的章节涵盖了更多的细节）：

*   确定API所提供资源的类型
*   确定资源之间的关系
*   基于类型和关系决定资源名称的结构
*   决定资源的结构
*   为资源附加上方法的最小集合

### 资源
一个面向资源的API通常被建模为资源层级，其中每个节点都是一个简单资源或是一个集合资源。为了方便起见，它们通常分别被称为资源和集合。

*   集合包含相同类型的资源列表。例如，用户拥有一个联系人集合。
*   资源有一些状态和零个或多个子资源。 每个子资源可以是简单资源或集合资源。

例如，Gmail API包含一个用户集合，每个用户都有一个消息集合，一个邮件线集合，一个标签集合，一个个人资料资源以及多个设置资源。

尽管存储系统和REST API之间存在一些概念上的一致性，但具有面向资源的API的服务不一定是数据库，并且在解释资源和方法方面具有极大的灵活性。例如，创建一个日历事件（资源）可能为与会者创建其他事件，向与会者发送电子邮件邀请，预定会议室以及更新视频会议日程安排。

### 方法
一个面向资源的API的关键特征是它强调资源（数据模型）而不是在资源上执行的方法（功能）。一个典型的面向资源的API会暴露大量携带少量方法的资源。这些方法可以是标准方法或自定义方法。对于本指南而言，标准方法是：`List`, `Get`, `Create`, `Update`和`Delete`。

如果API功能可以自然地映射到其中一种标准方法，那么应该在API设计中使用该方法。对于不能自然地映射到其中一种标准方法的功能，可以使用自定义方法。自定义方法提供与传统RPC API相同的设计自由度，可用于实现常见编程模式，如数据库事务或数据分析。

### 例子
以下部分介绍了一些关于如何将面向资源的API设计应用于大规模服务的现实世界示例。你可以在[Google API](https://github.com/googleapis/googleapis)仓库中找到更多示例。

#### Gmail API
Gmail API服务实现了Gmail API并暴露了大量的Gmail功能。 它有以下资源模型：

*   API服务：`gmail.googleapis.com`
*   一个用户集合：`users/*`。每个用户都有以下资源。
    *   消息集合：`users/*/messages/*`。
    *   邮件线集合：`users/*/threads/*`。
    *   标签集合：`users/*/labels/*`。
    *   更改历史记录的集合：`users/*/history/*`。
    *   表示用户个人资料的资源：`users/*/profile`。
    *   表示用户设置的资源：`users/*/settings`。

#### Cloud Pub/Sub API

`pubsub.googleapis.com`服务实现了[Cloud Pub/Sub API](https://cloud.google.com/pubsub)，该API定义了以下资源模型：

*   API服务：`pubsub.googleapis.com`
*   主题集合：`projects/*/topics/*`。
*   订阅集合：`projects/*/subscriptions/*`。

注意：Pub/Sub API的其他实现可能会选择不同的资源命名方案。

Cloud Spanner API

spanner.googleapis.com服务实现了[Cloud Spanner API](https://cloud.google.com/spanner)，该API定义了以下资源模型：

*   API服务：`spanner.googleapis.com`
*   一个实例的集合：`projects/*/instances/*`。
    *   实例操作的集合：`projects/*/instances/*/operations/*`。
    *   数据库的集合：`projects/*/instances/*/databases/*`。
    *   数据库操作的集合：`projects/*/instances/*/databases/*/operations/*`。
    *   数据库会话的集合：`projects/*/instances/*/databases/*/sessions/*`。
