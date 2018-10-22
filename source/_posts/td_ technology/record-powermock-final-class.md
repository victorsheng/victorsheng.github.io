---
title: 使用powermock mock final修饰的类和方法
abbrlink: 66836954
date: 2018-10-12 15:34:15
tags:
categories:
---

# 背景
https://github.com/powermock/powermock/wiki/Mockito
官网说mockito 2.0以上版本已经支持final class和method的 mock,推荐使用,然后maven仓库上没有稳定版本
https://mvnrepository.com/artifact/org.mockito/mockito-all

改为使用PowerMock的方法
https://github.com/powermock/powermock/wiki/mockfinal


# 使用方法
mock final class和method

```
package com.demo.first.util.http.client.Impl;

import java.io.IOException;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;
import okhttp3.Response;
import okhttp3.ResponseBody;

@RunWith(PowerMockRunner.class)
@PrepareForTest({Response.class, ResponseBody.class})
public class OkHttpClientImplTest {

  @Test
  public void checkSuccessFul() throws IOException {
    String errorBody = "{\"errorCode\":4003,\"msg\":\"Target book doesn't apply to you\"}";
    int httpStatutsCode = 401;
    OkHttpClientImpl okHttpClient = new OkHttpClientImpl();
    Response response = PowerMockito.mock(Response.class);
    ResponseBody responseBody = PowerMockito.mock(ResponseBody.class);
    PowerMockito.when(responseBody.string()).thenReturn(errorBody);
    PowerMockito.when(response.body()).thenReturn(responseBody);
    PowerMockito.when(response.code()).thenReturn(httpStatutsCode);
    okHttpClient.checkSuccessFul(response);

  }
}

```

pom文件配置
```
 <!-- https://mvnrepository.com/artifact/org.mockito/mockito-all -->
    <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-all</artifactId>
      <version>1.10.19</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.powermock/powermock-module-junit4 -->
    <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-module-junit4</artifactId>
      <version>1.6.6</version>
      <scope>test</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.powermock/powermock-api-mockito -->
    <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-api-mockito</artifactId>
      <version>1.6.6</version>
      <scope>test</scope>
    </dependency>
```