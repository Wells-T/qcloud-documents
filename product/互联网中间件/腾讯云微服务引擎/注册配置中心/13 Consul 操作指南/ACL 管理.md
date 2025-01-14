## 操作场景
Consul 引擎支持使用 Consul ACLs 功能进行访问控制。您可以根据业务场景配置策略和角色，并生成对应的 Token 进行安全控制访问。

## ACL参数配置
1. 登录 TSE 控制台 。
2. 在左侧导航栏单击注册中心，单击目标实例的“ID/名称”，在系统参数 页，选择 ACLs 功能项，可查看当前ACLs配置参数。
参数说明如下：
|参数	|说明|
|-----|-----|
|enabled	|是否开启 ACL 功能|
|default_policy	|ACL 策略，支持 allow 和 deny。allow 表示全部允许，此时策略配置为黑名单；deny 表示全部拒绝，此时策略配置为白名单|
|tokens.master	|管理员 Token，具备访问所有资源权限，TSE Consul 实例创建时默认生成，不可更改|
|tokens.agent	|Consul Agent Token，用于 Consul Agent Client 启动时配置 Agent 所需资源，TSE Consul 实例创建时默认生成，不可更改|
说明：对于新建的 TSE Consul 实例， ACLs 默认使用 allow 策略，无需配置 ACL 即可使用 Consul SDK/HTTP API 访问集群资源。

## 启用ACL
如果您的Consul Agent Client 已加入 Consul 集群，线上已有服务注册到 Agent Client 中，为了避免启用 ACL 后，Client 无法与 Server 进行通信，推荐使用该方式进行平滑启用 ACL 功能。
### 操作步骤
1、 将 token.master 添加到业务代码中。
- 如果您使用Spring Cloud，可参考如下示例。
```
spring:
  cloud:
    consul:
      discovery:
        acl-token: "94c45ad9-b344-87dd-05d8-56ad71031a92"
        health-check-path: /health #需要业务自行实现一个接口，供consul server来请求
        health-check-interval: 15s #consul server端每隔15s请求一次健康检测接口
        register-health-check: true #开启健康检查
```
注意：Spring Cloud Consul 默认通过心跳上报的方式维护服务健康状态，由于心跳请求不支持 token 参数，因此 ACL 开启后，您需要自行实现一个健康检查的接口，并配置在 health-check-path 中。

- 如果您使用 Consul HTTP 请求，可参考如下示例。
```
//Consul Go SDK
consulClient, err := consul_api.NewClient(&consul_api.Config{
		Address: "127.0.0.1:8500",
		HttpClient: &http.Client{
			Timeout: 3 * time.Second,
		},
		Scheme: "http",
		Token:  "94c45ad9-b344-87dd-05d8-56ad71031a92",
	}
)
```

- 如果您使用 Consul SDK，可参考如下示例。
```
// Query参数
curl http://127.0.0.1:8500/v1/agent/members?token="94c45ad9-b344-87dd-05d8-56ad71031a92"
// Header头
curl -H 'X-Consul-Token:94c45ad9-b344-87dd-05d8-56ad71031a92}' http://127.0.0.1:8500/v1/agent/members
```

2、 启用 Consul Server ACL
- (1) 登录 TSE 控制台。
- (2) 在左侧导航栏单击注册中心，单击目标实例的“ID/名称”，在系统参数 页，选择 ACL 功能项，点击修改参数，编辑 JSON 内容，将 default_policy 配置项设置为 deny，点击保存。
```
{
    "enabled": true,
    "default_policy": "deny",
    "tokens": {
        "master": "94c45ad9-b344-87dd-05d8-56ad71031a92",
        "agent": "a92fa275-3bfe-f668-f88c-bf79d17af9e0"
    }
}
```

3、 启用 Consul Agent Client ACL
- (1) 修改 Consul Agent Client 配置文件 config.json，添加集群 ACL 配置和 tokens.agent。
- (2) 重启 Consul Agent Client，等待重启完毕后，验证 Consul Agent Client 是否与集群建立正常通信。
```
{
    "acl":{
        "enabled":true,
        "default_policy":"deny",
        "tokens":{
            "agent":"a92fa275-3bfe-f668-f88c-bf79d17af9e0"
        }
    }
}
```

## ACL操作
Consul ACL 包括策略、角色、令牌三种数据模型。令牌可关联多个策略和角色，角色可管理多个策略。令牌具有的权限为直接或间接关联的全部策略的总和。
开启 ACL 后，可通过配置策略、角色和令牌进行访问控制。

### 策略操作
1. 登录 TSE 控制台 。
2. 在左侧导航栏单击注册中心，单击目标实例的“ID/名称”，在 ACL 页，可查看当前实例的策略、角色和令牌。
3. 新建策略：在策略栏，点击新建，新建策略，填写策略名称、规则和描述，生成一个自定义策略。
4. 查看策略详情：策略详情包括基本信息、已绑定角色和已绑定令牌。
5. 编辑策略：点击编辑，支持编辑策略名称、规则和描述。
6. 删除策略：点击删除，展示当前策略已绑定的资源，确认后点击保存，进行删除。
7. 绑定/解绑角色：点击绑定角色，选择该策略需要绑定/解绑的角色，支持多选，选择完成后点击保存。
8. 绑定/解绑令牌：点击绑定令牌，选择该角色需要绑定/解绑的令牌，支持多选，选择完成后点击保存。

### 角色操作
1. 登录 TSE 控制台 。
2. 在左侧导航栏单击注册中心，单击目标实例的“ID/名称”，在 ACL 页，可查看当前实例的策略、角色和令牌。
3. 新建角色：在角色栏，点击新建，填写角色名称和描述，点击下一步，您可以直接为该角色绑定策略。
4. 查看角色详情：角色详情包括角色基本信息和已绑定的策略。
5. 编辑角色：点击编辑，支持编辑角色名称和描述。
6. 删除角色：点击删除，确认要删除的角色名称后点击保存，进行删除。
7. 绑定/解绑策略：点击绑定策略，选择该角色需要绑定/解绑的策略，支持多选，选择完成后点击保存。

### 令牌操作
1. 登录 TSE 控制台 。
2. 在左侧导航栏单击注册中心，单击目标实例的“ID/名称”，在 ACL 页，可查看当前实例的策略、角色和令牌。
3. 新建令牌：在角色栏，点击新建，填写令牌名称和描述，点击下一步，您可以直接为该令牌绑定策略，以及绑定角色。
4. 查看令牌详情：角色详情包括令牌基本信息、已绑定的角色和已绑定的策略。
5. 编辑令牌：点击编辑，支持编辑令牌名称和描述。
6. 删除令牌：点击删除，确认要删除的令牌名称后点击保存，进行删除。
7. 绑定/解绑策略：点击绑定策略，选择该角色需要绑定/解绑的策略，支持多选，选择完成后点击保存。
8. 绑定/解绑角色：点击绑定角色，选择该角色需要绑定/解绑的策略，支持多选，选择完成后点击保存。
