## 自定义方法

这一个章节将会讨论如何在API设计中使用自定义方法。

自定义方法指的是除了五种标准方法以外的API方法。他们只会用于那些无法轻易通过标准方法来表达的功能上。一般来说，API设计者应该在条件允许的情况下尽可能地使用标准方法。标准方法有着更简单且定义明确的语义，更为大部分开发人员所熟知，所以他们更容易被使用且更不容易出错。标准方法的另一个优势是API平台对标准方法有着更好的支持，例如错误处理、日志、监控等。

一个自定义方法可以与资源、集合或服务相关联。它可能携带任意请求、返回任意响应，也可能支持流式请求和流式响应。

自定义方法名必须遵守[方法命名约定](https://cloud.google.com/apis/design/naming_convention#method_names)。

### HTTP映射
对于自定义方法，它们应该使用如下通用HTTP映射：
```
https://service.name/v1/some/resource/name:customVerb
```
使用`:`代替'/'来将自定义动词从资源名称中分离是为了支持任意的路径。举个例子，撤销删除一个文件可以映射到`POST /files/a/long/file/name:undelete`

选择HTTP映射时应遵循以下准则：

*   自定义方法应该使用HTTP`POST`动词，因为它具有最灵活的语义，例外是用作get或list备选的方法可能会使用`GET`。（有关详细信息，请参阅第三个准则。）
*   自定义方法不应该使用HTTP`PATCH`，但可以使用其它HTTP动词。在这种情况下，这些方法必须遵循该动词的标准[HTTP语义](https://tools.ietf.org/html/rfc2616#section-9)。
*   值得注意的是，使用HTTP`GET`的自定义方法必须是幂等的，并且没有副作用。例如，在资源上实现特殊视图的自定义方法应该使用HTTP`GET`。
*   请求消息中接收与自定义方法相关联的资源或集合的资源名称的字段应该映射到URL路径。
*   URL路径必须以由冒号后跟自定义动词组成的后缀结尾。
*   如果用于自定义方法的HTTP动词允许HTTP请求体（POST，PUT，PATCH或自定义HTTP动词），则此类自定义方法的HTTP配置必须使用`body: "*"`子句，所有其余的请求消息字段应映射到HTTP请求体。
*   如果用于自定义方法的HTTP动词不接受HTTP请求体（GET，DELETE），则此类方法的HTTP配置一定不能使用body子句，并且所有剩余的请求消息字段都应映射到URL query parameters。

**警告**：如果服务实现多个API，则API生产者必须小心创建服务配置以避免API之间的自定义动词冲突。

```
// 这是一个服务级别的自定义方法.
rpc Watch(WatchRequest) returns (WatchResponse) {
  // 自定义方法映射到HTTP POST。所有请求参数被放入body。
  option (google.api.http) = {
    post: "/v1:watch"
    body: "*"
  };
}

// 这是一个集合级别的自定义方法。
rpc ClearEvents(ClearEventsRequest) returns (ClearEventsResponse) {
  option (google.api.http) = {
    post: "/v3/events:clear"
    body: "*"
  };
}

// 这是一个资源级别的自定义方法。
rpc CancelEvent(CancelEventRequest) returns (CancelEventResponse) {
  option (google.api.http) = {
    post: "/v3/{name=events/*}:cancel"
    body: "*"
  };
}

// 这是一个批量获取自定义方法
rpc BatchGetEvents(BatchGetEventsRequest) returns (BatchGetEventsResponse) {
  // 批量获取方法映射到HTTP GET动词。
  option (google.api.http) = {
    get: "/v3/events:batchGet"
  };
}
```

### 使用场合

在以下场景中使用自定义方法可能是个正确的选择：

*   **重启一台虚拟机**该设计的替代方案可以是“创建一个在重启集合中的重启资源”，但感觉这样过于复杂，或者“虚拟机有一个可变状态，而客户端可以将其从RUNNING状态变为RESTARTING”，而这会带来更多的问题，例如哪些状态之间是可以互相转移的。此外，重启是一个众所周知的概念，它可以很方便地被转化成一个满足开发者期望的自定义方法。
*   **发送邮件**创建一个邮件消息不一定需要发送（草稿）。跟替代方案（移动一个消息到“Outbox”集合）对比起来，自定义方法的优势在于API的使用者更容易发现并更直接地建立起概念。
*   **提拔一个雇员**。如果这个功能被实现为一个标准的update，那客户端就必须重复实现企业政策中与升职流程相关的部分，来确保升职是作用在正确的等级，并且在相同的职业梯队内。
*   **批量处理方法**对于性能关键的方法，提供自定义批处理方法以减少每个请求的开销可能很有用。

一些使用标准方法比自定义方法更合适的例子：
*   使用不同的查询参数来查询资源（使用标准的`list`方法与标准列表过滤）。
*   简单的资源字段更改（使用标准`update`方法与字段掩码）。
*   关闭通知（使用标准`delete`方法）。

### 通用自定义方法

如下所示是一些常用且有用的自定义方法合集。API设计人员在引入他们自己的设计前应该优先考虑使用这些名称以促进跨API的一致性

Method Name            | Custom verb | HTTP verb | Note
-----------------------|-------------|-----------|-----------------------------------------------
Cancel                 | :cancel     | POST      | 取消一个长时操作(构建、计算等等)
BatchGet\<plural noun> | :batchGet   | GET       | 批量获取多个资源。(在List的描述中获取更多细节)
Move                   | :move       | POST      | 将一个资源从一个父级移动到另一个。
Search                 | :search     | GET       | 获取不符合列表语义的数据列表的替代方案。
Undelete               | :undelete   | POST      | 还原一个先前删除的资源。推荐的保留时期为30天。
