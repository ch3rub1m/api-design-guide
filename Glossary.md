## 术语表

### 网络API
  * 应用程序在计算机网络上运行的接口。网络API使用网络协议（包括HTTP）进行通信，它会由不同的组织不断生成，而不只是由它的使用者生成。

### Google APIs
  * Google服务提供的网络API。其大多数托管在googleapis.com域内。Google APIs中不包含其他类型的API，例如客户端库和SDK。

### API接口
  * Protocol Buffers服务定义。通常会对应着绝大多数编程语言的接口。每个API接口可被任意数量的API服务调用。

### API版本
  * 一个API接口的版本，或一组共同定义的接口的版本。API版本通常由一个字符串表示，例如`v1`，并且API版本会在API请求和Protocol Buffers包名称中显示。

### API方法
  * API接口里的单个操作。在Protocol Buffers 中定义为`rpc`，通常对应了绝大多数编程语言的API接口。

### API请求
  * 对API方法的单个调用。通常可作为计费、日志记录、检测、速率限制的单位。

### API服务
  * 一个或若干个API接口的部署，可在一个或若干个网络端点上获取到。每个API服务可以通过其服务名称来标识，该名称需符合RFC 1035 DNS标准，例如calendar.googleapis.com。

### API端点
  * 指API服务给实际API请求提供服务的网络地址。例如pubsub.googleapis.com 、 content-pubsub.googleapis.com。

### API产品
  * 与API服务相关的组件（例如服务条款、文档、客户端库、服务支撑），它们会统一作为产品呈现给用户。

### API服务定义
  * 指所有用于定义API服务的API接口定义（.proto文件）和API服务配置（.yaml文件）的集合。Google API服务定义的模式是google.api.Service。

### API使用者
  * 使用API服务的实体。对于Google API来说，API使用者通常是带有客户端应用或服务资源的Google项目。

### API生产者
  * 生产API服务的实体。对于Google API来说，API生产者就是带有API服务的Google项目。

### API后端
  * 执行API服务业务逻辑的服务器及相关基础架构。通常称一个独立的API后端服务器为API服务器。

### API前端
  * 能提供跨API服务通用功能（如负载平衡和身份验证）的服务器及基础架构。通常称一个独立的API前端服务器为API代理。

  注意：API前端和API后端可在一起运行也可以在两地运行。一般它们可以编译成一个应用程序的二进制文件，并在单个进程中运行。
