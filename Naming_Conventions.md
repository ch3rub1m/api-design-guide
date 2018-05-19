# 命名规范

为了能够长时间在跨APIs中为开发者提供一致性的体验，API的所有命名应该遵循以下几点：

*   简单
*   直观
*   一致

这包括接口，资源，集合，方法以及消息的命名。

因为大多数开发者都不是以英语作为母语的，这些命名规范的一个目的是确保大多数开发者可以很容易地理解一个API。规范通过鼓励在命名方法和资源时使用简单一致的小词汇量来达到这一目的。

*   APIs里面的名称应该是正确的美式英语。例如，license（而不是licence），color（而不是colour）。
*   为了简洁起见，可以使用常用的短语或长词的缩写。 例如，API就比Application Programming Interface更好一点。
*   如果有可能，使用直观而且人们熟悉的术语。举个例子，当要描述移除（并且删除）一个资源的时候，delete比erase更恰当。
*   为相同的概念使用相同的名称或术语，包括一些跨API共享的概念。
*   避免名字重载。 对不同的概念使用不同的名称。
*   避免在API上下文以及更大的Google APIs生态系统中使用过于笼统的含糊不清的名称。他们会导致API的一些概念被误解。相反，我们要选那些可以准确描述API概念的名称。这在定义最重要的API元素（如资源）的时候尤为重要。我们没有明确指出哪些是不能用来定义的名称，因为每一个名称都需要在上下文中被评估后确定。举个例子，instance，info和service这些名称在过去就被发现很有问题，因为选中的名字应该能够清晰地描述API的概念（例如：什么东西的instance）以及能够将它与一些相关的概念区分开来（例如“alert”指的是规则，还是型号，还是通知？）。
*   在使用那些可能与普通编程语言中的关键字冲突的名称时要小心。这些名字是可以被使用的，不过可能会在API review时接受额外的审查，所以应该谨慎并尽可能少地使用它们。

## 产品名称
产品指API的产品营销名称，例如Google Calendar API。产品名称一定要在APIs，UIs，文档，服务条款，账单报表以及商业合同等一系列东西里面保持一致。Google APIs必须使用那些被产品和市场营销团队允许的产品名称。

以下表格展示了不同API的名称如何保持一致性的例子。请在本篇文章后面获取更多关于每一个名字以及他们相关约定的详细内容。

| API 名称 | 示例                               |
|----------|------------------------------------|
| 产品名称 | Google Calendar API                |
| 服务名称 | calendar.googleapis.com            |
| 包名称   | google.calendar.v3                 |
| 接口名称 | google.calendar.v3.CalendarService |
| 源目录   | //google/calendar/v3               |
| API 名称 | calendar                           |

## 服务名称
服务名称应该是语法上有效的DNS名(参照[RFC 1035](http://www.ietf.org/rfc/rfc1035.txt)），它可以被解析成一个或者多个网络ip地址。Google公共APIs的服务名称遵循以下模式：`xxx.googleapis.com`。例如，Google Calendar的服务名称是`calendar.googleapis.com`。

如果一个API是由几个服务组成的，这些服务应该以一种让他们容易被发现的方式进行命名。实现这的其中一个方式是让这些服务名称都共享一个前缀。例如`build.googleapis.com`和`buildresults.googleapis.com`都是Google Build API的组成部分。

## 包名称
在API的.proto文件中定义的包名称应该与产品和服务名称保持一致。那些拥有版本号的APIs的包名称必须以版本号结尾。例如：
```
// Google Calendar API
package google.calendar.v3;
```
那些不直接和某一个服务关联的抽象API名称应该使用和产品名称一致的包名称，例如Google Watcher API:
```
// Google Watcher API
package google.watcher.v1;
```
在API .proto名称中定义的Java包名称一定要匹配标准的Java包名称前缀（com., edu., net., etc）。例如：
```
package google.calendar.v3;

// 使用标准的 "com."来指定Java的包名称
option java_package = "com.google.calendar.v3";
```

## 集合IDs
[集合IDs](https://cloud.google.com/apis/design/resource_names#collection_id)应该使用复数和驼峰命名法，而且要使用美式英语的拼写和语义。例如：`events`，`children`，和`deletedEvents`。

## 接口名称
为了不与诸如`pubsub.googleapis.com`的[服务名称](https://cloud.google.com/apis/design/naming_convention#service_names)冲突，接口名称指的是在.proto文件中定义的`service`名称：
```
// Library就是接口名称
service Library {
	rpc ListBooks(...) returns (...);
	rpc ...
}
```
您可以将服务名称视为对一组API的实际实现的引用，而接口名称是对一个API的抽象定义。

接口名称应该使用比较直观的名词，例如Calendar和Blob。该名称不应与编程语言及其运行时库（例如File）中的任何已建立的概念相冲突。

在极少数情况下，接口名称会与API中的其他名称发生冲突，应使用后缀（例如Api或Service）来消除歧义。

## 方法名称
服务可能在其IDL规范中定义一个或多个对应于集合和资源上的RPC方法。 方法名称应该遵循`VerbNoun`的命名规范，其中名词`Noun`通常是资源类型。

| 动词   | 名词 | 方法名称   | 请求消息          | 响应消息           |
|--------|------|------------|-------------------|--------------------|
| List   | Book | ListBooks  | ListBooksRequest  | ListBooksResponse  |
| Get    | Book | GetBook    | GetBookRequest    | Book               |
| Create | Book | CreateBook | CreateBookRequest | Book               |
| Update | Book | UpdateBook | UpdateBookRequest | Book               |
| Rename | Book | RenameBook | RenameBookRequest | RenameBookResponse |
| Delete | Book | DeleteBook | DeleteBookRequest | DeleteBookResponse |

方法名称的动词应该是一种 [祈使语气](https://zh.wikipedia.org/wiki/%E7%A5%88%E4%BD%BF%E8%AA%9E%E6%B0%A3)，它是用来下达命令的而不是那种带有提问感觉的指示性语气。

这样当动词是询问一个关于API子资源的问题的时候就有点容易让人疑惑了。举个例子，创建一本书的API方法名称很明显叫做`CreateBook`（祈使语气），但是如果是要询问书出版商的状态可能就会使用指示性语气，例如`IsBookPublisherApproved`或者`NeedsPublisherApproval`。为了在这种情况下也可以保持祈使语气，我们可以把这些命令替换为“check”（CheckBookPublisherApproved）和”validate”（ValidateBookPublisher）。

## 消息名称
RPC方法的请求和响应消息应该分别以带有`Request`和`Response`后缀的方法名称命名，除非方法的请求和响应类型是：

*   一个空消息（使用`google.protobuf.Empty`）
*   一个资源名称
*   一个代表一个操作的资源

这通常适用于在标准方法`Get`，`Create`，`Update`或`Delete`中使用的请求或响应。

## 枚举名称
枚举名称一定要使用驼峰命名法。

枚举值一定要使用`大写的以下横线分割`的命名方法。每个枚举值一定要以分号而不是逗号结尾。第一个值必须命名为ENUM_TYPE_UNSPECIFIED，这个值会在枚举值不被明显地指定的时候返回。

```
enum FooBar {
	// 第一个值表示默认值，而且必须等于0
	FOO_BAR_UNSPECIFIED = 0;
	FIRST_VALUE = 1;
	SECOND_VALUE = 2;
}
```

## 字段名称
在.proto文件里面的字段名称必须使用`小写以下横线分割`的命名方法。这些名称会根据每种编程语言的原生命名约定映射到其生成代码中。

字段名称应该避免使用介词（例如 “for”，“during”和“at”），例如：

*   使用`error_reason`而不是`error_for_reason`。
*   使用`failure_time_cpu_usage`而不是`cpu_usage_at_time_of_failure`。

字段名称应该避免使用后置形容词（名词后面的装饰词），例如：

*   使用`collected_items`而不是`items_collected`。
*   使用`imported_objects`而不是`objects_imported`。

### 重复字段名称

APIs的重复字段名称必须使用正确的复数形式。这和现存的Google APIs和外部开发者的普遍期待保持一致。

### 时间和持续时间

应该使用`google.protobuf.Timestamp`来表示一个与时区和日历无关的时间点，而且时间点的字段名称应该以`time`来结尾，例如`created_time`或者`last_updated_time`。

如果时间点和某一个活动相关，字段的名称应该是符合`动词下横线time`的形式，例如`create_time`，`update_time`。在名称里面避免使用动词的过去式，例如`created_time`和`last_updated_time`。

应该使用`google.protobuf.Duration`来表示一个和任何日历和诸如”日”，“月”等概念无关的时间跨度。

```
message FlightRecord {
	google.protocolbuf.Timestamp takeoff_time = 1;
	google.protocolbuf.Duration flight_duration = 2;
}
```

如果由于遗留或兼容性的原因，必须使用整数类型来表示与时间相关的字段，则包括挂钟时间，持续时间和延时等字段名称必须具有以下格式：

```
xxx_{time|duration|delay|latency}_{seconds|millis|micros|nanos}
```

```
message Email {
	int64 send_time_millis = 1;
	int64 receive_time_millis = 2;
}
```

如果由于遗留或者兼容性原因，必须使用字符串来表示时间戳，字段名称中不应该包含任何单位后缀。字符串的表达应该符合RFC 3339格式，例如，“2014-07-30T10:43:17Z”。

### 日期和时间

对于那些和时区和时间无关的日期，应该使用`google.type.Date`， 而且应该在名称后面带上`_date`后缀。如果日期必须用字符串来表示，它应该符合ISO 8601的YYYY-MM-DD格式，例如 2014-07-30。

对于和时区和日期无关的一天的时间段，应该使用`google.type.TimeOfDay`， 而且应该在名称后面加上`_time`后缀。如果这个时间段必须用字符串表示，它应该符合ISO 8601 24小时制的格式 HH:MM:SS[.FFF]，例如 14:55:01.672。

```
message StoreOpening {
	google.type.Date opening_date = 1;
	google.type.TimeOfDay opening_time = 2;
}
```

### 数量

用整数表示的数量字段必须带上计量单位。

```
xxx_{bytes|width_pixels|meters}
```

如果数量是物品的数目，字段名称一定要带上`_count`后缀，例如`node_count`。

### 列表过滤器字段

如果API支持通过过滤器来过滤`List`方法返回的资源，包含filter表达式的字段应该被命名为`filter`。例如：
```
message ListBooksRequest {
	// 父级资源
	string parent = 1;
	// filter表达式
 	string filter = 2;
}
```

### 列表响应

在`List`方法响应消息里面的包含资源列表数据的字段必须是资源名的复数形式。例如`CalendarApi.ListEvents()`必须定义一个`ListEventsResponse`的响应消息，这个响应消息包含一个对应于返回资源的叫做`events`的字段。

```
service CalendarApi {
  rpc ListEvents(ListEventsRequest) returns (ListEventsResponse) {
    option (google.api.http) = {
      get: "/v3/{parent=calendars/*}/events";
    };
  }
}

message ListEventsRequest {
  string parent = 1;
  int32 page_size = 2;
  string page_token = 3;
}

message ListEventsResponse {
  repeated Event events = 1;
  string next_page_token = 2;
}
```

## 驼峰命名

除了字段和枚举值，`.proto`文件内部的所有定义必须使用在[Google Java Style](https://google.github.io/styleguide/javaguide.html#s5.3-camel-case)中定义的`大写字母开头的驼峰法`来命名。

## 名字缩写
对于诸如`config`和`spec`等在软件开发里面被大家熟悉的名字缩写，在API定义的时候，比较偏向于使用缩写而不是全称。这样会使源代码更容易被阅读和编写。在正式的文档里面，就应该使用全拼了。例如：

*   config (configuration)
*   id (identifier)
*   spec (specification)
*   stats (statistics)
