## 目录结构
  API服务通常用`.proto`文件来定义API接口，用`.yaml`文件来配置API服务。

  每个API在API仓库中必须有一个API目录，该目录包括它的定义文件和构建脚本。

  API目录应该遵循以下标准的层级结构：
  * API目录
    * 仓库依赖
      * `BUILD` - 构建文件
      * `METADATA` - 构建元数据文件
      * `OWNERS` - API目录所有者
    * 配置文件
      * `{service}.yaml` - 基准服务配置文件，它是`google.api.Service`的proto消息的YAML表示。
      * `prod.yaml` - 产品服务配置文件。
      * `staging.yaml` - 线上测试服务配置文件。
      * `test.yaml` - 测试服务配置文件。
      * `local.yaml` - 本地服务配置文件。
    * 文档文件
      * `README.md` - 主要的指导文件。包括产品概述、技术描述等。
      * `doc/*` - 技术文档文件。应该使用Markdown格式。
    * 接口定义
      * `v[0-9]*/*` - 每个目录包含了API主版本号、主要的proto文件和构建脚本。
      * `{subapi}/v[0-9]*/* `- 每个{subapi}目录都包含了子API的接口定义。每个子API都有独立的主版本号。
      * `type/*` - proto文件包含了不同API共享的类型，同一个API不同版本共享的类型，或是API和服务共享的类型。`type/*`下定义的类型一旦发布了就不能有重大更改。
