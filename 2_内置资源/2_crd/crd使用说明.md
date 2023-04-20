# 1.创建CRD
* 当创建新的CustomResourceDefinition（CRD）时，Kubernetes API 服务器会为你所指定的每一个版本生成一个 RESTful 的资源路径。
* CRD可以是命名空间作用域的，也可以是集群作用域的，取决于 CRD 的 scope 字段设置。 
* 与其他现有的内置对象一样，删除一个命名空间时，该名字空间下的所有 CRD 对象也会被删除。
* CustomResourceDefinition 本身是不受命名空间限制的，对所有命名空间可用。
> 例如，将下面的 CustomResourceDefinition 保存到 mycrd.yaml 文件，之后使用kubectl apply -f mycrd.yaml创建资源
>> 如果成功创建，一个新的受名字空间约束的RESTful API 端点会被创建在：/apis/stable.example.com/v1/namespaces/*/crontabs/...
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # 名字必需与下面的 spec 字段匹配，并且格式为 '<名称的复数形式>.<组名>'
  # 名称复数：crontabs，组名称stable.example.com
  name: crontabs.stable.example.com
spec:
  # 组名称，用于RESTAPI: /apis/<组>/<版本>
  # 组：stable.example.com
  # 版本：v1
  group: stable.example.com
  # 列举此 CustomResourceDefinition 所支持的版本
  versions:
    - name: v1
      # 每个版本都可以通过 served标志来独立启用或禁止
      served: true
      # 其中一个且只有一个版本必需被标记为存储版本
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          # 自定义属性
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  # 可以是 Namespaced 或 Cluster
  scope: Namespaced
  names:
    # 名称的复数形式，用于 URL：/apis/<组>/<版本>/<名称的复数形式>
    plural: crontabs
    # 名称的单数形式，作为命令行使用时和显示时的别名
    singular: crontab
    # kind 通常是单数形式的驼峰编码（CamelCased）形式。你的资源清单会使用这一形式。
    kind: CronTab
    # shortNames 允许你在命令行使用较短的字符串来匹配资源
    shortNames:
    - ct
```
# 2.创建自定义对象CR
> 在创建了 CustomResourceDefinition 对象之后，可以创建定制对象（Custom Objects）。
> 定制对象可以包含定制字段。这些字段可以包含任意的 JSON 数据。 
> 如下在类别CrontTab的定制对象中，设置了cronSpec、image、replicas字段值。
>> 创建cr：kubectl apply -f my-crontab.yaml
>>> 如果创建成功，使用：kubectl get crontab 或 kubectl get ct -o yaml可以查看资源
```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
  replicas: 3
```