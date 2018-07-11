title: openid-connect
date: 2018-06-06 16:37:06
tags:
    - 安全
    - 认证
    - 授权
categories:
    - 安全
---
# 概念
OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0 protocol. It enables Clients to verify the identity of the End-User based on the authentication performed by an Authorization Server, as well as to obtain basic profile information about the End-User in an interoperable and REST-like manner.

This specification defines the core OpenID Connect functionality: authentication built on top of OAuth 2.0 and the use of Claims to communicate information about the End-User. It also describes the security and privacy considerations for using OpenID Connect.


# Id Token的好处
- 无状态会话  :放入浏览器cookie中，ID令牌可以实现轻量级无状态会话。 这消除了将会话存储在服务器端（存储器或磁盘上）的需要，这可能是管理和扩展的负担。 通过验证ID令牌来检查会话。 当令牌达到超过其发布时间戳（iat）的某个预定义年龄时，应用程序可以简单地通过静默向op发出请求
- 将身份传递给第三方  :当需要知道用户身份时，可以将ID令牌传递给其他应用程序组件或后端服务，例如记录审计跟踪。
- 令牌交换  :




# 标准规范
http://openid.net/specs/openid-connect-core-1_0.html

#$ 实际流程:
https://connect2id.com/learn/openid-connect


# 标准中的接口


![upload successful](/images/pasted-173.png)

| Endpoint | Purpose |
| :--- | :--- |
| [Server discovery](https://connect2id.com/products/server/docs/api/discovery) | Discover the OAuth 2.0 / OpenID Connect endpoints, capabilities, supported cryptographic algoritms and features. |
| [Server JWK set](https://connect2id.com/products/server/docs/api/jwk-set) | Retrieve the public server JSON Web Key \(JWK\) required to verify the authenticity of issued ID and access tokens. |
| [Client registration](https://connect2id.com/products/server/docs/api/client-registration) | Create, access, update and delete client registrations with the server. |
| [Authorisation](https://connect2id.com/products/server/docs/api/authorization) | The client sends the end-user’s browser to this endpoint to request their authentication and consent. This endpoint is used in the code and implicit OAuth 2.0 flows which require end-user interaction. |
| [Token](https://connect2id.com/products/server/docs/api/token) | Post an OAuth 2.0 grant \(code, refresh token, resource owner password credentials, client credentials\) to obtain an ID and / or access token. |
| [Token introspection](https://connect2id.com/products/server/docs/api/token-introspection) | Validate an access token and retrieve its underlying authorisation \(for resource servers\). |
| [Token revocation](https://connect2id.com/products/server/docs/api/token-revocation) | Revoke an obtained access or refresh token. |
| [UserInfo](https://connect2id.com/products/server/docs/api/userinfo) | Retrieve profile information and other attributes for a logged-in end-user. |
| [Check session iframe](https://connect2id.com/products/server/docs/api/check-session) | Poll the OpenID provider for changes of end-user authentication status. |
| [Logout \(end-session\)](https://connect2id.com/products/server/docs/api/logout) | Sign out an end-user. |