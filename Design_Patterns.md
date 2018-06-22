# 通用设计模式
## 空响应
为了全局的一致性，标准的`Delete`方法必须返回`google.protobuf.Empty`作为响应的内容。这同时也可以防止客户端依赖那些在重试期间不可用的额外元数据。对于自定义方法，他们必须返回他们自己的`XxxResponse`字段，即使这个字段现在是空的也要被带上，因为随着时间的推移这些自定义方法的功能可能会增加，那时候就有需要返回额外的数据了。

## 表示范围
表示范围的字段应该使用符合命名规范的半开半闭区间`[start_xxx, end_xxx)`，例如`[start_key, end_key)`或者`[start_time, end_time)`。半开半闭区间通常在C++ STL和Java的标准库里面被用到。API应该避免使用其他方法去代表范围，例如`(index, count)`，或者`[first, last]`。

## 资源标签
在面向资源的API里面，资源的结构通常由API定义。为了让客户端在资源上带上少量简单的元数据（举个例子，标记一台虚拟机资源为数据库服务器），APIs应该使用在`google.api.LabelDescriptor`里面描述的`资源标签设计模式`(resource label design pattern)。

为了实现这一点，API设计应该在资源定义里面添加一个`map<string, string>`字段。

```
message Book {
	string name = 1;
	map<string, string> labels = 2;
}
```

## 耗时操作（Long Running Operations）
如果某个API方法需要很长时间去完成，你可以返回一个耗时操作（Long Running Operation）资源给客户端，客户端可以用这个资源来追踪执行的过程和接收执行的结果。[Operation](https://github.com/googleapis/googleapis/blob/master/google/longrunning/operations.proto)里面定义了一个耗时操作的标准接口。为了避免不一致性，单独的API一定不可以为耗时操作定义他们自己的接口。

资源必须作为响应消息被直接返回，并且对资源操作的结果应该反映在 API 中。例如，当创建一个资源的时候，资源应该出现在`LIST`和`GET`方法中，但是一定要标明这个资源现在还不能被使用。如果该方法不是耗时的，当操作完成后，响应的`Operation.response`字段一定要包含那个已经被立即返回的信息。

操作可以通过使用`Operation.meta`字段来提供关于其执行状况的信息。API应该为这个元数据定义一个消息，即使初始实现没有生成这个消息。

## 列表分页
即使结果集很小，可分页的集合都应该支持分页操作。

原因：如果API一开始不支持分页操作，往后对分页的支持可能会带来麻烦，因为这对于API来说是一个破坏性操作。那些不知道API支持分页操作的客户端，在他们收到集合的结果时可能会错误地假设他们已经获得API的所有结果了，可是实际上他们只是拿到第一页的结果。

为了支持`List`的分页操作，API应该：

*   在方法的请求信息里面定义一个叫做`page_token`的`string`类型的字段。客户端用来向服务器请求关于集合某一页的内容。
*   在方法的请求信息里面定义一个叫做`page_size`的`int32`类型的字段。客户端用来指定服务器给其返回集合的最大资源个数。服务端可能会自己判断单页里面应该返回的资源个数，例如，当`page_size`的值为0时，服务端将会自己决定返回结果的最大资源数是多少。
*   在`List`方法的响应信息中定义一个叫做`next_page_token`的`string`类型字段。这个字段用来指明用于接受下一页结果的token。如果这个字段的值是`""`，这表明没有更多的请求结果了。

为了获取下一页的结果，客户端应该在下一个`List`请求时带上上一个请求的`next_page_token`：

```
rpc ListBook(ListBookRequest) returns (ListBooksResponse);

message ListBookRequest {
	string name = 1;
	int32 page_size = 2;
	string page_token = 3;
}

message ListBooksResponse {
	repeated Book books = 1;
	string next_page_token = 2;
}
```

当客户端在请求里面携带了除page token以外的请求参数（query parameters）时，如果参数与page token不一致，服务必须拒绝此请求。

page token的内容应该是基于一个url安全的protocol buffer进行base64编码后的结果。这样在内容变动的时候就不会有兼容性问题。page token中存在敏感信息时，应该将其加密。服务端必须通过以下方法来防止通过篡改page token来获取敏感信息：

*   后续请求要重新指定请求参数
*   在page token中仅引用服务端的会话状态
*   在page token中加密并签名请求参数，并且在每次调用中对这些参数进行反复验证和鉴权。

分页的实现可能会在一个叫做`total_size`的`int32`类型的字段里面标明资源的总数。

## 列出子集合
某些时候，API需要让客户端对子集合进行`List/Search`操作。例如，图书馆（Library）API可能有一个书架（shelves）集合，每个书架里面有一个书本（books）集合，假如某个客户想要在所有的书架里面搜索一本书，在这种情形下，最推荐的做法是在子资源里面使用标准的`List`方法并为父级集合指定通配集合id`"-"`。对于这个API库的例子，我们可以使用以下的REST API请求：

```
GET https://library.googleapis.com/v1/shelves/-/books?filter=xxx
```

提示：选择`"-"`替代`"*"`的原因是为了避免URL escaping。

## 从子集合中获取唯一资源

某些时候，有些子集合中的资源具有包括其父集合在内也唯一的标识符。在不知道哪一个父集合包含它的时候，允许通过`Get`来获取这个资源可能是有用的。在这种情况下，建议在该资源上使用标准`Get`，并为所有父集合指定通配集合id`"-"`。举个例子，在图书馆API中，我们可以使用以下的REST API请求，如果这本书在所有的书架上的所有书中是唯一的：

```
GET https://library.googleapis.com/v1/shelves/-/books/{id}
```

在这个请求的响应中使用的资源名称一定要使用该资源的官方名称，为每一层父级集合使用实际的父级集合id而不是`"-"`。举个例子，以上请求返回的资源名称应该形如`shelves/shelf713/books/book8141`，而非`shelves/-/books.book8141`。

## 排列顺序
如果某一个API方法允许客户端指明列表结果的排序方式，请求消息应该包含以下字段：

```
string order_by = ...;
```

字符串的值应该遵循SQL语法：用逗号分隔的字段列表。例如：`"foo,bar"`。每个字段默认按照升序排列，如果要指明某一个字段值按照降序排列，应该给这个字段添加`" desc"`后缀。例如：`"foo desc,bar"`。

多余的空格可以忽略，`"foo,bar desc"`和`"  foo ,  bar  desc  "`是等价的。

## 请求校验
如果某个API方法有副作用，并且有必要在不产生副作用的情况下对请求进行校验，请求消息应该包含以下字段：

```
bool validate_only = ...;
```

当此字段设置为`true`时，服务端一定不能执行任何有副作用的操作，并且只执行与完整请求一致的针对实现的校验。

如果校验成功，必须返回`google.rpc.Code.OK`，并且使用相同请求信息的完整请求都不应该返回`google.rpc.Code.INVALID_ARGUMENT`。注意，此请求可能还是会因为其他错误（比如`google.rpc.Code.ALREADY_EXISTS`或竞态条件）而失败。

## 重复请求
对于网络API，幂等性是很重要的，因为当网络异常时它们能够安全地进行重试。然而一些API并不容易实现幂等性，例如需要避免不必要重复的创建资源操作。对于这类情况，请求信息应该包含一个唯一ID（例如UUID），这样服务端能够通过此ID来检测请求是否重复，保证请求只被处理一次。

```
// 用于检测冗余请求的唯一标识符
// 这个字段应该命名为 `request_id`
string request_id = ...;
```

如果服务端检测到重复的请求，它应该给客户端返回之前成功的响应，因为之前那个响应客户端很有可能没有接收到。

## 枚举默认值
每个枚举必须从`0`开始定义，它应该在枚举值没有被显式指明时使用。API必须在文档中指明应该如何处理默认值`0`。

如果有通用的默认行为，枚举值`0`就应该被使用，同时API文档也应该说明预期的行为。

如果没有通用的默认行为，枚举值`0`应该被命名为 `ENUM_TYPE_UNSPECIFIED`并且要和`INVALID_ARGUMENT`错误一起使用。

```
enum Isolation {
	// 没有指定
  ISOLATION_UNSPECIFIED = 0;
	// 从快照中读取。 如果所有读写操作都无法在逻辑上与并发事务串行化，则会发生冲突。
	SERIALIZABLE = 1;
	// 从快照中读取。并发事务向同一行写入时导致冲突。
	SNAPSHOT = 2;
}

// 未指定时，服务器将使用SNAPSHOT或更高的隔离级别。
Isolation level = 1;
```

`0`值可以使用一个惯用名称来命名。例如，在没有错误码的情况下，可以使用惯用名称`google.rpc.Code.OK`，在这种情况下，`OK`语义上等价于枚举类型上下文的`UNSPECIFIED`。

若存在本质上合理和安全的默认值，这个值就可以被用来作为`0`值。例如，在资源视图枚举中`BASIC`就是`0`值。

## 语法句法
在API设计中，有时候需要为某些特定的数据格式定义简单的语法，例如可接受的文本输入。为了在跨APIs中提供一致的开发体验和减少学习曲线，API设计者们必须使用如下Extended Backus-Naur Form (EBNF)句法变种来定义这些语法。

```
Production  = name "=" [ Expression ] ";" ;
Expression  = Alternative { "|" Alternative } ;
Alternative = Term { Term } ;
Term        = name | TOKEN | Group | Option | Repetition ;
Group       = "(" Expression ")" ;
Option      = "[" Expression "]" ;
Repetition  = "{" Expression "}" ;
```

注：`TOKEN`表示在语法之外定义的终端符号。

## 整数类型
在API设计中，不应该使用像`uint32`和`fixed32`这种无符号整型，这是因为一些重要的编程语言和系统（例如Java, JavaScript和OpenAPI）不能很好地支持它们，并且他们更容易导致溢出错误。另一个问题是，不同API很可能对同一个资源使用不同的数据类型（有符号和无符号整型）。

在大小和时间这种负数没有意义的类型中`可以`使用且仅可以使用`-1`来表示特定的意义，例如文件结尾（EOF）、无穷的时间、无资源限额或未知的年龄。当在这种情况下使用负数时，必须在文档中明确说明其意义以防止混淆。如果不够显而易见，API生产者也应该在文档中记录隐式默认值`0`表示的行为。

## 部分响应
客户端有时只需要响应信息中的特定子集。一些API平台提供了对部分响应的原生支持。Google API平台通过响应字段掩码来为其提供支持。对于任一REST API 调用，有一个隐式的系统query参数 `$fields`，它是 `google.protobuf.FieldMask`的JSON表示。在返回给客户端之前，响应消息会被`$fields`字段过滤。这个逻辑在API平台的所有API方法上都会被处理。

```
GET https://library.googleapis.com/v1/shelves?$fields=name
```

## 资源视图
为了减少网络流量，允许客户端对服务端的返回内容作出限制，让其只返回对客户端有用的字段而不是所有内容。API中的资源视图是通过向请求添加参数来实现的，该参数允许客户端指定要接收资源的哪个视图。

参数满足以下条件：

*   *应该*是枚举类型
*   *必须*命名为`view`

枚举中的每个值定义了资源的哪部分（字段）在响应中会被返回。文档中应该明确指定每个`view`值会返回什么。

```
package google.example.library.v1;

service Library {
  rpc ListBooks(ListBooksRequest) returns (ListBooksResponse) {
    option (google.api.http) = {
      get: "/v1/{name=shelves/*}/books"
    }
  };
}

enum BookView {
	// 什么都不指定，这等同于BASIC.
  BOOK_VIEW_UNSPECIFIED = 0;

  // 响应中只包含作者、标题、ISBN和唯一的图书ID。这是默认值。
  BASIC = 1;

  // 返回所有信息，包括书中的内容
  FULL = 2;
}

message ListBooksRequest {
  string name = 1;

  // 指定图书资源的哪些部分应该被返回
  BookView view = 2;
}
```

对应的 URL：
```
GET https://library.googleapis.com/v1/shelves/shelf1/books?view=BASIC
```

可以在[标准方法](Standard_Methods.md)一章中查看更多关于定义方法、请求和响应的内容。

## ETags
ETag是一个不透明的标识符，允许客户端进行条件性请求。为了支持 ETag，API应该在资源定义中包含一个字符串字段`etag`，它的语义必须与ETag的常用用法相匹配。通常，`etag`包含由服务器计算出的资源指纹。更多详细信息，请参阅 [维基百科](https://en.wikipedia.org/wiki/HTTP_ETag)和[RFC 7232](https://tools.ietf.org/html/rfc7232#section-2.3)。

ETags可以是强验证的或弱验证的，其中弱验证的ETag以`W/`为前缀。在这种情况下，强验证意味着具有相同ETag的两个资源具有每字节都相同的内容和相同的额外字段（例如，Content-Type），同时也意味着强验证的ETag允许对稍后组装的部分响应进行缓存。

相反，具有相同弱验证ETag值的资源意味着这些表示在语义上是等效的，但不一定每字节都相同，因此不适合于字节范围请求的响应缓存。

例如：

```
// 强验证的 ETag（包含引号）
"1a2f3e4d5b6c7c"
// 弱验证的 ETag（包含前缀和引号）
W/"1a2b3c4d5ef"
```

值得注意的是引号也是ETag值的一部分，而且为了遵循[RFC 7232](https://tools.ietf.org/html/rfc7232#section-2.3)它们一定要被表示出来。这就意味着ETags的JSON表示一定要对引号进行转义。例如ETags在JSON资源的表示是：

```
// 强验证
{ "etag": "\"1a2f3e4d5b6c7c\"", "name": "...", ... }
// 弱验证
{ "etag": "W/\"1a2b3c4d5ef\"", "name": "...", ... }
```

ETags中允许的字母：

*   可打印的ASCII码
*   RFC 2732中指定的非ASCII码，不过他们对开发者来说不是很友好
*   没有空格
*   除了上面的双引号，不能有别的双引号
*   RFC 7232中推荐不能使用反斜线以防止和转移符混淆

## 输出字段
API可能希望将由客户端提供的输入字段和只由服务端在特定资源上返回的输出字段进行区分。对于仅输出的字段，应该记录该字段的属性。

请注意，如果客户端在请求中设置了仅输出（output only）字段，或者客户端对仅输出字段指定了一个`google.protobuf.FieldMask`，则服务器必须接受该请求而不能报错。这意味着服务器必须忽略仅输出字段的存在及其任何指示。这个建议的原因是因为客户端通常会将服务器返回的资源重用为另一个请求的输入，例如一个获取到的`Book`将在`UPDATE`方法中被再次使用。如果要验证仅输出字段，客户端需要做清除输出字段的额外工作。

```
message Book {
  string name = 1;
  // 仅输出字段
  Timestamp create_time = 2;
}
```

## 单例资源
单例资源被使用于整个父级资源范围内（如果没有父级资源，则是整个API范围内）只有一个资源实例的情况下。

标准`Create`和`Delete`方法不能被用于单例资源上；单例会在其父资源创建或删除的时候被隐式创建或删除（或者在其没有父资源的前提下，会隐式存在）。资源必须可以通过标准的`Get`和`Update`方法被访问到。

例如，`User`资源的API应该对外暴露一个作用于单个用户`Setting`API：

```
rpc GetSettings(GetSettingsRequest) returns (Settings) {
  option (google.api.http) = {
    get: "/v1/{name=users/*/settings}"
  };
}

rpc UpdateSettings(UpdateSettingsRequest) returns (Settings) {
  option (google.api.http) = {
    patch: "/v1/{settings.name=users/*/settings}"
    body: "settings"
  };
}

[...]

message Settings {
  string name = 1;
  // 省略Setting字段
}

message GetSettingsRequest {
  string name = 1;
}

message UpdateSettingsRequest {
  Settings settings = 1;
  // 用于支持部分更新的字段掩码
  FieldMask update_mask = 2;
}
```

## 流的半关闭
对于任何双向API或客户端流式APIs，服务器都应该依赖由RPC系统提供的客户端启动的半关闭来完成客户端流。没有必要定义一个明确的完成消息。

客户需要在半关闭之前发送的任何信息都必须定义为请求消息的一部分。
