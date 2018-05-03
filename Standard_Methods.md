## 标准方法

这一章定义了标准方法的概念：`List`，`Get`，`Create`，`Update`和`Delete`。标准方法既减少了复杂度又增加了一致性。在Google APIs中超过70%的API方法是标准方法，这能使得它们更易学易用。

*如果方法支持响应字段掩码——用于指定需要被返回的字段子集的话，那通过这些方法返回的资源中可能会包含局部数组。在某些场合，API平台天然支持所有方法的字段掩码。

**从并不立即删除资源的`Delete`方法返回的响应应该包含long running operation或者被修改的资源之一

一个标准方法也可能会对在一次API调用的时间跨度内未完成的请求返回long running operation。

### List

`List`方法携带一个集合名称和若干个参数作为输入，并且返回一组匹配输入的资源列表。

`List`通常用来搜索资源。`List`适合用来从一个单一大小有限且未被缓存的集合中获取数据。在更多的场合，应该使用自定义方法`Search`。

批量获取（该方法可以携带多个资源ID并且返回对应的每一个对象）应该被实现为自定义方法`BatchGet`，而不是使用`List`。但是，如果你的`List`方法已经提供了类似的功能，那么你可能可以复用`List`来实现这个意图。

### Get

`Get`方法携带一个资源名称和若干个参数，返回指定的资源。

### Create

`Create`方法携带一个父资源名称，一个资源名称及若干个参数。它在指定的父级下创建一个新的资源，并返回该新创建的资源。

如果一个API支持创建资源，它应该对所有允许创建的资源提供一个`Create`方法。

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

如果API允许客户端指定资源名称，服务器可能会允许客户端来指定一个不存在的资源名称并自动创建一个新的资源。而如果服务器不提供这个能力，那么当资源名称不存在时，`Update`方法应该失败。如果仅有这一个错误，那么应该使用错误码`NOT_FOUND`。

提供`Update`方法并支持资源创建的API，也应该同时提供`Create`方法。根本原因在于如果`Update`方法是唯一的方式，那人们可能不会很清楚地知道该如何创建资源。

### DELETE

`Delete`方法携带一个资源名称及若干个参数，来立刻删除或在某个特定时间删除指定的资源。`Delete`方法应该返回`google.protobuf.Empty`。

API不应该依赖于任何由`Delete`方法返回的信息，因为它不能被重复调用。

调用`Delete`方法应该是幂等的，但不需要产生相同的响应，任何数量的删除请求都应该导致一个资源（最终）被删除，但只有第一个请求应该成功。随后的请求应该产生一个`google.rpc.Code.NOT_FOUND`。

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
