Helm学习笔记
---

Helm是什么？
---
- **“Helm帮助你管理k8s应用——即使是最复杂的k8s应用，Helm Chart也能帮助你轻松定义，安装和升级它们”
—— Helm官网**

我认为Helm从更高的层面完整定义了一个k8s应用。在k8s中，我们能找到许多不同的和应用有关的资源，例如deployment,
service,ingress等，它们都从不同的角度描述了k8s应用的某个方面。而Helm通过Chart概念，将这些不同的资源收敛到
一起，定义了最接近我们理解的“应用”的概念。

- **“Chart可以轻松的被创建，划分版本，分享和发布”——Helm官网**

将一个k8s应用抽象成为一个Chart之后，就可以对这个原子性的应用实体进行一系列需要的操作。比如实际应用中的创建，更
新，删除，回滚。Chart同时具备很好的移植性，因此可以像“包”一样发布和托管。

Helm能做什么
---
- 从零构建Chart
- 将Chart打包成Chart archive文件（tgz）
- 和Chart仓库交互
- 向已有的k8s内安装，卸载Chart
- 管理已经安装的Chart的发布周期

重要概念
---
- Chart：创建一个k8s应用必备的一揽子信息
- Config：创建一个k8s应用的配置信息
- Release：一个运行的Chart+Config的实例，每次install Chart会创建一个新的release
- Repository：用来存放和共享Chart

Helm架构
---
- Helm3
    Helm3采用了纯客户端架构，客户端直接与k8s api server交互。Helm3的另一个组件是Helm库，提供了Helm的可编程接口。
    - Helm Client
        - 本地Chart开发
        - 管理仓库
        - 管理release
        - 和Helm library交互
    - Helm library
        - 结合Chart和Config构建release
        - 向k8s安装Charts，并提供release object
        - 升级和卸载Chart
        
Chart文件结构
---
```
wordpress/
  Chart.yaml          # A YAML file containing information about the chart
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  values.yaml         # The default configuration values for this chart
  values.schema.json  # OPTIONAL: A JSON Schema for imposing a structure on the values.yaml file
  charts/             # A directory containing any charts upon which this chart depends.
  crds/               # Custom Resource Definitions
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```
