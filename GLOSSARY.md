## 网络API
通过计算机网络互相调用的应用程序接口。网络API通过包括HTTP在内的网络协议进行通信，而且它们常常是由不同的组织而不是接口调用方生成的。

## Google APIs
Google服务提供的网络API。其大多数托管在googleapis.com域内。Google APIs不包括诸如客户端库和SDK等其他类型的API。

## API接口
一个基于Protocol Buffers的服务定义。它一般类似于大多数编程语言中接口的概念。同一个API接口可以被任意数量的API服务实现。

## API版本
一个API接口的版本，或一组共同定义的接口的版本。API版本通常由一个字符串表示，例如`v1`，并且API版本会在API请求和Protocol Buffers包名称中显示。

## API方法
API接口里的单个操作。在Protocol Buffers中定义为`rpc`，通常对应了绝大多数编程语言的API接口。

## API请求
对API方法的单次调用。通常用来作为计费、日志、监控以及速率限制的单位。

## API服务
一个或若干个API接口的部署实现，可在一个或若干个网络端点上获取到该服务。每个API服务通过一个兼容[RFC 1035 DNS](https://www.ietf.org/rfc/rfc1035.txt)的服务名称来标识，例如`calendar.googleapis.com`。

## API端点
指API服务给实际API请求提供服务的网络地址。例如`pubsub.googleapis.com`、`content-pubsub.googleapis.com`。

## API产品
一个API服务加上其相关的组件，例如服务条款，文档，客户端库和服务支持等，一起作为一个产品呈现给用户。例如，Google Calender API。提示：人们说的API产品通常只是指API。

## API服务定义
指所有用于定义API服务的API接口定义（.proto文件）和API服务配置（.yaml文件）的集合。Google API服务定义的模式是[google.api.Service](https://github.com/googleapis/googleapis/blob/master/google/api/service.proto)。

## API消费者
使用API服务的实体。对于Google API来说，API消费者通常是带有客户端应用或服务资源的Google项目。

## API生产者
提供API服务的实体。对于Google API来说，API生产者就是带有API服务的Google项目。

## API后端
一群服务器以及实现API业务逻辑的相关基础设施。一个单独的的API后端服务器通常被称为一个API服务器。

## API前端
一群服务器以及为不同API服务提供通用功能的相关基础设施，例如负载均衡和鉴权等。一个单独的API前端服务器通常就被称为一个API代理。

提示：API前端和后端可能跑在两个距离很近的地方或者是两个距离很远的地方。在某些情况下，它们可以被编译成一个二进制程序并跑在同一个进程上。
