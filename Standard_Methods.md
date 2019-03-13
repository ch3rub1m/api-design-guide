## 标准方法

这一章定义了标准方法的概念：`List`，`Get`，`Create`，`Update`和`Delete`。标准方法既减少了复杂度又增加了一致性。在Google APIs中超过70%的API方法是标准方法，这能使得它们更易学易用。

以下这张表格描述了如何将标准方法映射到HTTP方法：

Standard Method | HTTP Mapping                | HTTP Request Body | HTTP Response Body
----------------|-----------------------------|-------------------|------------------------
List            | `GET <collection URL>`        | N/A               | Resource* list
Get             | `GET <resource URL>`          | N/A               | Resource*
Create          | `POST <collection URL>`       | Resource          | Resource*
Update          | `PUT or PATCH <resource URL>` | Resource          | Resource*
Delete          | `DELETE <resource URL>`       | N/A               | `google.protobuf.Empty`**

*如果方法支持响应字段掩码——用于指定需要被返回的字段子集的话，那通过这些方法返回的资源中可能会包含局部数组。在某些场合，API平台天然支持所有方法的字段掩码。

**从并不立即删除资源的`Delete`方法返回的响应应该包含long running operation或者被修改的资源之一

一个标准方法也可能会对在一次API调用的时间跨度内未完成的请求返回long running operation。

以下各节详细介绍了每种标准方法。这些示例显示了在.proto文件中通过特殊注释定义HTTP映射的方法。你可以在[Google APIs](https://github.com/googleapis/googleapis)仓库中找到许多使用标准方法的示例。

### List
`List`方法携带一个集合名称和若干个参数作为输入，并且返回一组匹配输入的资源列表。

`List`通常用于搜索资源。 `List`适合于来自单个集合的数据，该集合的大小有限且未被缓存。对于更广泛的情况，应该使用[自定义方法](Custom_Methods.md)`Search`。

批量获取（该方法可以携带多个资源ID并且返回对应的每一个对象）应该被实现为自定义方法`BatchGet`，而不是使用`List`。但是，如果你的`List`方法已经提供了类似的功能，那么你可能可以复用`List`来实现这个意图。如果你正在使用自定义方法`BatchGet`，它应该被映射到`HTTP GET`。

适合的通用模式：[分页](./Design_Patterns.md#列表分页)，[结果排序](./Design_Patterns.md#排列顺序)。
适合的命名约定：[过滤字段](./Naming_Conventions.md#列表过滤器字段)，[结果字段](./Naming_Conventions.md#列表响应)。

HTTP映射：
*   `List`方法必须使用HTTP`GET`动词
*   请求消息中用于接受被获取资源的集合名称的字段应该映射到URL路径中。如果集合名称映射到URL路径，则URL模板的最后一部分（集合ID）必须是文字。
*   所有剩余的请求消息字段应该映射到URL query parameters。
*   没有请求体；API设置中禁止声明`body`子句。
*   响应体应该包含一个资源列表以及可选的元数据。

例子：
```
// Lists books in a shelf.
rpc ListBooks(ListBooksRequest) returns (ListBooksResponse) {
  // List method maps to HTTP GET.
  option (google.api.http) = {
    // The `parent` captures the parent resource name, such as "shelves/shelf1".
    get: "/v1/{parent=shelves/*}/books"
  };
}

message ListBooksRequest {
  // The parent resource name, for example, "shelves/shelf1".
  string parent = 1;

  // The maximum number of items to return.
  int32 page_size = 2;

  // The next_page_token value returned from a previous List request, if any.
  string page_token = 3;
}

message ListBooksResponse {
  // The field name should match the noun "books" in the method name.  There
  // will be a maximum number of items returned based on the page_size field
  // in the request.
  repeated Book books = 1;

  // Token to retrieve the next page of results, or empty if there are no
  // more results in the list.
  string next_page_token = 2;
}
```

### Get
`Get`方法携带一个资源名称和若干个参数，返回指定的资源。

HTTP映射：
*   `Get`方法必须使用HTTP`GET`动词
*   请求消息中用于接受被获取资源的集合名称的字段应该映射到URL路径中。
*   所有剩余的请求消息字段应该映射到URL query parameters。
*   没有请求体；API设置中禁止声明`body`子句。
*   返回的资源应该映射到整个响应体。

例子：
```
// Gets a book.
rpc GetBook(GetBookRequest) returns (Book) {
  // Get maps to HTTP GET. Resource name is mapped to the URL. No body.
  option (google.api.http) = {
    // Note the URL template variable which captures the multi-segment resource
    // name of the requested book, such as "shelves/shelf1/books/book2"
    get: "/v1/{name=shelves/*/books/*}"
  };
}

message GetBookRequest {
  // The field will contain name of the resource requested, for example:
  // "shelves/shelf1/books/book2"
  string name = 1;
}
```

### Create

`Create`方法携带一个父资源名称，一个资源名称及若干个参数。它在指定的父级下创建一个新的资源，并返回该新创建的资源。

如果一个API支持创建资源，它应该对所有允许创建的资源提供一个`Create`方法。

HTTP映射：
*   `Create`方法必须使用HTTP`POST`动词
*   请求消息中应该有一个字段`parent`用于指定将被创建资源的父级资源名称。
*   所有剩余的请求消息字段应该映射到URL query parameters。
*   请求可能会包含一个叫做`<resource>_id`的字段用于允许调用者选择一个客户端自分配id。这个字段必须映射到URL query parameters。
*   包含资源的请求消息字段应该映射到请求体。如果在`Create`方法中使用了`body`的HTTP配置子句，必须依照`body: "<resource_field>"`的格式。
*   返回的资源应该映射到整个响应体。


如果`Create`方法支持客户端指定资源名称并且该资源已经存在，则该请求应该失败并返回错误码`ALREADY_EXISTS`，也可以由服务端指定一个不同的资源名称，不过必须在文档中清晰地标示出：被创建的资源名称有可能和输入的不同。

`Create`方法必须以资源作为输入，因为这样如果资源的结构被更改了，也没有必要同时修改请求的结构和资源的结构。对于无法在客户端修改的资源字段，必须在文档里标注为“Output only”。

举个例子：
```
// Creates a book in a shelf.
rpc CreateBook(CreateBookRequest) returns (Book) {
  // Create maps to HTTP POST. URL path as the collection name.
  // HTTP request body contains the resource.
  option (google.api.http) = {
    // The `parent` captures the parent resource name, such as "shelves/1".
    post: "/v1/{parent=shelves/*}/books"
    body: "book"
  };
}

message CreateBookRequest {
  // The parent resource name where the book to be created.
  string parent = 1;

  // The book id to use for this book.
  string book_id = 3;

  // The book resource to create.
  // The field name should match the Noun in the method name.
  Book book = 2;
}

rpc CreateShelf(CreateShelfRequest) returns (Shelf) {
  option (google.api.http) = {
    post: "/v1/shelves"
    body: "shelf"
  };
}

message CreateShelfRequest {
  Shelf shelf = 1;
}
```

### Update

`Update`方法发起一个包含了资源和若干个参数的请求。它可以用于更新置顶的资源和它的属性，并返回更新后的资源。

可变的资源属性应该通过`Update`方法来更改，除非这个属性包含了资源名称或者父级信息。任何的重命名或移动一个资源的动作都不应该在`Update`里发生，应该由自定义方法来处理。

HTTP映射：
*   `Update`方法应该支持部分资源更新，使用HTTP`PATCH`动词以及一个叫做`update_mask`的`FieldMask`。
*   需要更高阶的修补语义的`Update`方法，比如添加数据到一个重复字段，应该通过[自定义方法](Custom_Methods.md)来实现。
*   如果`Update`方法只支持全量资源更新，它必须使用HTTP动词`PUT`。但是全量更新是非常令人沮丧的，因为它在增加新的资源字段的时候有向后兼容问题。
*   请求消息中用于接受被获取资源的集合名称的字段应该映射到URL路径中。这个字段也可能在资源消息本身里。
*   包含资源的请求消息字段必须映射到请求体。
*   所有剩余的请求消息字段必须映射到URL query parameters。
*   响应消息必须是被更新后的资源本身。

如果API允许客户端指定资源名称，服务器可能会允许客户端来指定一个不存在的资源名称并自动创建一个新的资源。而如果服务器不提供这个能力，那么当资源名称不存在时，`Update`方法应该失败。如果仅有这一个错误，那么应该使用错误码`NOT_FOUND`。

提供`Update`方法并支持资源创建的API，也应该同时提供`Create`方法。根本原因在于如果`Update`方法是唯一的方式，那人们可能不会很清楚地知道该如何创建资源。

例子：
```
// Updates a book.
rpc UpdateBook(UpdateBookRequest) returns (Book) {
  // Update maps to HTTP PATCH. Resource name is mapped to a URL path.
  // Resource is contained in the HTTP request body.
  option (google.api.http) = {
    // Note the URL template variable which captures the resource name of the
    // book to update.
    patch: "/v1/{book.name=shelves/*/books/*}"
    body: "book"
  };
}

message UpdateBookRequest {
  // The book resource which replaces the resource on the server.
  Book book = 1;

  // The update mask applies to the resource. For the `FieldMask` definition,
  // see https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#fieldmask
  FieldMask update_mask = 2;
}
```

### Delete

`Delete`方法携带一个资源名称及若干个参数，来立刻删除或在某个特定时间删除指定的资源。`Delete`方法应该返回`google.protobuf.Empty`。

API不应该依赖于任何由`Delete`方法返回的信息，因为它不能被重复调用。

HTTP映射：
*   `Delete`方法必须使用HTTP`DELETE`动词
*   请求消息中用于接受被获取资源的集合名称的字段应该映射到URL路径中。
*   所有剩余的请求消息字段应该映射到URL query parameters。
*   没有请求体；API设置中禁止声明`body`子句。
*   如果`Delete`方法立刻移除了资源，它应该返回一个空响应。
*   如果`Delete`方法开始了一个long-running operation，它应该返回一个long-running operation。
*   如果`Delete`方法只是将资源标记为已删除，它应该返回更新后的资源。

调用`Delete`方法应该是幂等的，但不需要产生相同的响应，任何数量的删除请求都应该导致一个资源（最终）被删除，但只有第一个请求应该成功。随后的请求应该产生一个`google.rpc.Code.NOT_FOUND`。

例子：
```
// Deletes a book.
rpc DeleteBook(DeleteBookRequest) returns (google.protobuf.Empty) {
  // Delete maps to HTTP DELETE. Resource name maps to the URL path.
  // There is no request body.
  option (google.api.http) = {
    // Note the URL template variable capturing the multi-segment name of the
    // book resource to be deleted, such as "shelves/shelf1/books/book2"
    delete: "/v1/{name=shelves/*/books/*}"
  };
}

message DeleteBookRequest {
  // The resource name of the book to be deleted, for example:
  // "shelves/shelf1/books/book2"
  string name = 1;
}
```
