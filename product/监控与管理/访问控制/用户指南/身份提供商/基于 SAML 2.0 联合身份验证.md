腾讯云支持基于 SAML 2.0（安全断言标记语言 2.0）的联合身份验证，SAML 2.0 是许多身份验证提供商（Identity Provider, IdP）使用的一种开放标准。使用身份提供商可实现联合单点登录（Federated Single Sign-on, SSO），用户可以授权经过联合身份验证通过的用户登录腾讯云管理控制台或调用腾讯云 API 操作，而不必为企业或组织中的每一个成员都创建一个 CAM 子用户。同时 SAML 2.0 为通用开放协议，您不必编写自定义身份代理代码，可以直接通过使用 SAML 来简化在腾讯云的联合身份验证的过程。

## SAML 身份提供商

身份提供商是访问管理（CAM）中的一个实体，可以认为是外部授信账号集合。基于 SAML 2.0 联合身份验证的身份提供商（IdP）描述了支持 SAML 2.0（安全断言标记语言 2.0）标准的身份提供商（IdP）服务。如果您希望建立 SAML 2.0 协议兼容的身份提供商（例如 Microsoft Active Directory 联合身份验证服务）与腾讯云之间的信任，以便企业或组织内成员能够访问腾讯云资源，则需要创建 SAML 身份提供商。关于创建 SAML 身份提供商，请参阅 [创建身份提供商](https://cloud.tencent.com/document/product/598/30290)。

## 身份提供商角色

创建 SAML 提供商后，您必须创建一个或多个以 SAML 身份提供商作为角色载体的身份提供商角色。角色是拥有一组权限的虚拟身份，进行资源访问时使用的是临时安全证书。在 SAML 2.0 断言上下文中，角色可分配给经身份提供商 (IdP) 验证身份的联合身份用户使用。此角色允许身份提供商请求临时安全证书进行腾讯云资源访问。此角色关联的策略决定了联合身份用户可在腾讯云资源的访问范围。关于创建基于 SAML 2.0 联合身份验证的身份提供商的角色，请参阅 [创建角色](https://cloud.tencent.com/document/product/598/19381)。
![](https://main.qcloudimg.com/raw/86f82050ccb875c96864b11561acea9a.png)

## 使用基于 SAML 2.0 联合身份验证访问腾讯云 API

![1](https://main.qcloudimg.com/raw/65eb02712b75d7bfcbba509b8f10be7c.png)
1.	企业或组织用户使用客户端请求企业的身份提供商进行身份验证。
2.	身份提供商根据企业的身份认证系统进行验证。
3.	返回用户验证的结果。
4.	身份提供商根据验证结果，生成一个标准的 SAML 2.0 断言文档，并返回到客户端。
5.	客户端根据 SAML 2.0 断言、身份提供商的资源描述和使用的身份提供商角色的资源描述向 sts:AssumeRoleWithSAML 请求申请临时安全密钥。
6.	STS 校验 SAML 2.0 断言信息。
7.	返回校验结果。
8.	根据返回结果申请临时证书，并返回到客户端。

## 使用基于 SAML 2.0 联合身份验证实现联合单点登录（SSO）
![](https://main.qcloudimg.com/raw/10d90eb5e5f91927e873cec8dc0e5823.png)
1.	企业或组织用户使用浏览器访问腾讯云服务。
2.	腾讯云服务返回认证请求到浏览器。
3.	浏览器重定向到企业组织的身份提供商（IdP）。
4.	企业验证用户身份。
5.	企业用户身份验证成功，返回用户信息到身份提供商（IdP）。
6.	身份提供商（IdP）生成标准的 SAML2.0 断言，返回到浏览器。
7.	浏览器将 SAML 2.0 断言重定向到腾讯云。
8.	开始进行腾讯云 SSO 登录服务，请求 cAuth 并验证用户身份。
9.	返回腾讯云验证结果。
10.	验证成功，返回登录态。
11.	重定向到腾讯云控制台服务。


