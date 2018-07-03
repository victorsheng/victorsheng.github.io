title: access-token的生成策略
author: Victor
date: 2018-06-21 17:29:42
tags:
categories:
---
# oauth2.0规范中的access token

https://tools.ietf.org/html/rfc6749

oauth2.0规范并没有定义如何access-token的格式及含义,对此业界对access_token的处理也截然不同

## 策略1:共享db的方式
https://ruby-china.org/topics/15396

问题:

Authorization Server (AS)这边是如何生成access token (AT) 的， 还有就是 Resource Server (RS) 这边如何去验证 请求里面的AT是正确的？(前提是验证的时候 RS 和 AS 没有交互~~~) 谢谢 ！！！

答案:

用 SecureRandom.hex 之类的东西产生就行了，重点是 client 如何取得这个 Token 。另外，验证 token 的时候， Resource Server 和 Authorization Server 之间本来就没有交互，只是存取同一个数据库而已。
## 策略2: jwt自包含信息,rs需要通过公钥校验jwt


## 策略3:rs通过rest接口访问op的 tokeninspection接口,获取代表的信息
https://tools.ietf.org/html/rfc7662

### Google way
Google Oauth2 Token Validation
https://developers.google.com/accounts/docs/OAuth2UserAgent#validatetoken

Request:
```
https://www.googleapis.com/oauth2/v1/tokeninfo?access_token=1/fFBGRNJru1FQd44AzqT3Zg
```
Respond:
```
{
  "audience":"8819981768.apps.googleusercontent.com",
  "user_id":"123456789",
  "scope":"https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email",
  "expires_in":436
}
```

### Github way
Respond:


```
{
  "id": 1,
  "url": "https://api.github.com/authorizations/1",
  "scopes": [
    "public_repo"
  ],
  "token": "abc123",
  "app": {
    "url": "http://my-github-app.com",
    "name": "my github app",
    "client_id": "abcde12345fghij67890"
  },
  "note": "optional note",
  "note_url": "http://optional/note/url",
  "updated_at": "2011-09-06T20:39:23Z",
  "created_at": "2011-09-06T17:26:27Z",
  "user": {
    "login": "octocat",
    "id": 1,
    "avatar_url": "https://github.com/images/error/octocat_happy.gif",
    "gravatar_id": "somehexcode",
    "url": "https://api.github.com/users/octocat"
  }
}
```


### Amazon way
Request :
```
https://api.amazon.com/auth/O2/tokeninfo?access_token=Atza|IQEBLjAsAhRmHjNgHpi0U-Dme37rR6CuUpSR...
```
Response :
```
HTTP/l.l 200 OK
Date: Fri, 3l May 20l3 23:22:l0 GMT 
x-amzn-RequestId: eb5be423-ca48-lle2-84ad-5775f45l4b09 
Content-Type: application/json 
Content-Length: 247 

{ 
  "iss":"https://www.amazon.com", 
  "user_id": "amznl.account.K2LI23KL2LK2", 
  "aud": "amznl.oa2-client.ASFWDFBRN", 
  "app_id": "amznl.application.436457DFHDH", 
  "exp": 3597, 
  "iat": l3ll280970
}
```

https://stackoverflow.com/questions/12296017/how-to-validate-an-oauth-2-0-access-token-for-a-resource-server?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa