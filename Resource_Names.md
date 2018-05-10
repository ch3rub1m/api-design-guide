## 资源名称
在面向资源的APIs里，资源是命名实体，并且资源名称是他们的唯一标识。每一个资源必须有一个独一无二的资源名称。资源名称由资源本身的ID、它的各个父级的ID以及API服务名称组成。接下来我们会关注在资源ID和一个资源名称是如何被构建的。

gRPC APIs应当使用scheme-less URIs来描述资源名称。他们通常遵循REST URL的约定，并且表现的更像是一个网络上的文件路径。

集合是一种特殊的资源，它包含了一组相同类型的子资源。举个例子，一个文件夹是一组文件资源的集合。一个集合的资源ID被称为集合ID。

资源名称被有层级地按照集合IDs和资源IDs组织，通过斜杆分隔。如果一个资源包含一个子资源，那么子资源的名称将形如父级资源的名称尾随着一个子资源ID，他们同样是由斜杆来分隔。

例子1：一个存储服务有一组buckets的集合，而bucket又是一组objects的集合：
```
//storage.shopee.com/buckets/bucket-id/objects/object-id
```
例子2：一个邮件服务有一组用户的集合，每一个用户有一个settings子资源，并且settings子资源又有复数的子资源，比如customFrom：
```
//mail.shopee.com/users/name@shopee.com/settings/customFrom
```
一个API开发者可以为资源和集合IDs选择任意可接受的值，并且需要在资源层级内尽可能长来保证它们是唯一的。你可以在下文中找到更多关于如何选择合适的资源和集合IDs的指导方针。
通过分隔资源名称，比如`name.split(“/“)[n]`，只要各个分段都不包含任何的正斜杆，就可以分别获得集合IDs和资源IDs。

### 完整的资源名称

一个scheme-less URI由一个兼容DNS的API服务名称和一个资源路径组成。资源路径也叫相对资源名称。举个例子：
```
//library.googleapis.com/shelves/shelf1/books/book2
```
API服务名称是为了让客户端可以定位API服务的终端，它可以是一个内部服务专用的伪DNS名。如果API服务的名称在正下文中是明确的，那么相对资源名称也经常被使用。

### 相对资源名称

指的是一个不以“/”开头的URI路径。它标识了一个API服务内的资源。举个例子：
```
"shelves/shelf1/books/book2"
```

### 资源ID

一个不为空的URI分段，它标识了一个被包含在它的父资源内的资源，可以参照之前的例子。
在资源名称中，资源ID可以是多个URI的分段。举个例子：
```
"files/source/py/parser.py"
```
如果条件允许，API服务应该使用URL友好的资源IDs。资源IDs必须被清晰地标明它们是由客户端还是服务端来指定。举个例子，文件名通常是由客户端来指定的，而邮件的消息IDs则通常由服务端来指定。

### 集合ID

一个不为空的URI分段，它标识了被包含在它的父资源内集合资源，可以参照之前的例子。
由于集合IDs通常会出现在被生成的客户端库里，所以它们必须满足以下的需求：
 *  必须是合法的C/C++标识符
 *  必须是小写驼峰的复数形式。如果没有合适的复数形式，例如“evidence”或“weather”，那么应该使用单数形式。
 *  必须是清晰且简明的英文词汇
 *  过于概略的词汇应该避免使用，或者加以修饰。例如rowValues就比values更好。以下的词汇如果没有修饰应该被避免使用：
 *  elements
 *  entries
 *  instances
 *  items
 *  objects
 *  resources
 *  types
 *  values

### 资源名称 vs URL

虽然一个完整的资源名称看起来像是一个普通的URLs，但是它们其实并不相同。一个单独的资源可以被不同的方式暴露，例如不同的API版本，不同的API协议或者不同的网络终端。完整的资源名称并未指明这些信息，所以当它被使用时必须被映射到指定的API版本和API协议。

### 资源名称即字符串

资源名称必须是字符串，除非有无法处理的向后兼容问题。资源名称应该像普通的文件路径那样被处理，并且不支持URL encoding。

对于资源的定义，第一个字段应该是一个字符串，表示资源名称，并且它应该被命名为`name`。

> 提示：其他与name相关的字段应该加以修饰来消除歧义，例如display_name，first_name，last_name，full_name。

例如：
```
service LibraryService {
  rpc GetBook(GetBookRequest) returns (Book) {
    option (google.api.http) = {
      get: "/v1/{name=shelves/*/books/*}"
    };
  };
  rpc CreateBook(CreateBookRequest) returns (Book) {
    option (google.api.http) = {
      post: "/v1/{parent=shelves/*}/books"
      body: "book"
    };
  };
}

message Book {
  // Resource name of the book. It must have the format of "shelves/*/books/*".
  // For example: "shelves/shelf1/books/book2".
  string name = 1;

  // ... other properties
}

message GetBookRequest {
  // Resource name of a book. For example: "shelves/shelf1/books/book2".
  string name = 1;
}

message CreateBookRequest {
  // Resource name of the parent resource where to create the book.
  // For example: "shelves/shelf1".
  string parent = 1;
  // The Book resource to be created. Client must not set the `Book.name` field.
  Book book = 2;
}
```

> 为了资源名称的一致性，词首的正斜杆一定不能被任何URL模版变量捕获。例如，URL模版必须使用`/v1/{name=shelves/*/books/*}`而不是`"/v1{name=/shelves/*/books/*}"`

### 问题

Q：为什么不使用资源IDs来标识一个资源？

A：在一个大型系统里有种类繁多的资源。使用资源IDs去标识一个资源，我们事实上是在使用一个指定资源的元组去标识一个资源，例如(bucket, object)或者(user, album, photo)。它会导致一些主要的麻烦：

 *  开发者必须去理解并且记住这些匿名元组
 *  传递元组获取通常比传递字符串更难
 *  中心化的基础设施，例如日志和访问权限控制系统，并不能理解指定的元组
 *  特定的元组限制了API设计的灵活性，例如提供可重用的API接口。

Q：为什么要把特殊字段命名为name而不是id？

A：特殊字段是以资源“名称”的概念命名的。通常，我们发现`name`这个词的含义经常困扰开发者。例如，文件名真的只是名字，还是一个完整路径呢？通过预定这个标准字段`name`，开发者被强制选择一个更恰当的词汇，例如`display_name`或者`title`或者`full_name`。
