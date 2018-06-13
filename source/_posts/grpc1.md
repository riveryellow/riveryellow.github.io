---
title: grpc实践(一)
cdn: header-off
date: 2018-06-08 16:49:59
header-img: "/img/grpc.png"
tags: 
	- rpc
---

# 实践背景

   目前在着手的一个音乐app服务端项目中，有些诸如用户服务、谷歌api查询等可以公用的功能杂糅在服务端中，其他项目也同样杂糅着类似的功能，因此想要把这些共用性比较高的功能服务都抽离出来，降低耦合。

# Why GRPC?

- __Simple service definition 简单的服务定义__

   Define your service using [Protocol Buffers](https://developers.google.com/protocol-buffers/), a powerful binary serialization toolset and language
   ![Concept Diagram](/img/grpc-landing-1.svg)

- __Works across languages and platforms 跨语言、跨平台__

   Automatically generate idiomatic client and server stubs for your service in a variety of languages and platforms
   通过protobuf定义的服务，可以根据需要生成出各种语言和平台的统一的服务代码。
   ![Concept Diagram](/img/grpc-landing-2.svg)

- __Start quickly and scale__

   Install runtime and dev environments with a single line and also scale to millions of RPCs per second with the framework
   ![Concept Diagram](/img/grpc-landing-3.svg)

# Quick Start

## 环境条件

+ JDK: version 7 or higher

## 定义gRPC服务

   gRPC服务使用[Protocol Buffers](https://developers.google.com/protocol-buffers/)在.proto文件中定义。

   官方demo：

   根据如下的定义可以看出，服务端和客户端的Greeter服务的stub上都有一个 SayHello RPC方法和 SayHelloAgain RPC方法，而这两个方法都需要从客户端得到入参 HelloRequest，然后从服务端获得返回 HelloReply。

```protobuf
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  // Sends another greeting
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}

```

   根据当前的业务需求，将用户服务的定义抽离出来:
```protobuf
/* generate code: protoc --proto_path=src --java_out=src/main/java src/main/proto/auth.proto*/
syntax = "proto3";

option java_package = "com.gomo.grpc.authorize.gen";
option java_outer_classname = "AbstractAuthorizeService";
// 生成rpc service的java代码
option java_generic_services = true;

package gen;

service Authorization {
    rpc AuthorizeByAccessToken (AuthorizeRequest) returns (AccessTokenDTO) {}
}

message AccessTokenDTO {
    string accessToken = 1;
    int64 expireAt = 2;
    bool bind = 3;
}

message AuthorizeRequest {
    string accountId = 1;
    string accessToken = 2;
    int64 expiredAt = 3;
}
```

## 生成服务的java代码

   根据官方文档，通过.proto文件生成java代码的方式有两种：
   1. 使用 [Protocol Buffers](https://developers.google.com/protocol-buffers/) 的编译器 protoc(__protoc的版本一定要和pom文件中的protobuf-java版本保持一致__)；
   
   _在执行protoc命令之前，要对maven工程进行一次package，生成protobuf-java-2.5.0.jar，否则，生成出来的java类会出现各种"Can not resolve Symbol XXX"的问题_

   ```
protoc --proto_path=IMPORT_PATH --java_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
   ```
   具体的生成细节参考：[Protocol Buffers - Java Generated Code](https://developers.google.com/protocol-buffers/docs/reference/java-generated)

   运行命令之后，可以在指定路径下看到生成好的java代码
   
   2. 当工程中使用Maven或Gradle时，可以使用proto的构建插件：
   ```xml
<build>
  <extensions>
    <extension>
      <groupId>kr.motd.maven</groupId>
      <artifactId>os-maven-plugin</artifactId>
      <version>1.5.0.Final</version>
    </extension>
  </extensions>
  <plugins>
    <plugin>
      <groupId>org.xolstice.maven.plugins</groupId>
      <artifactId>protobuf-maven-plugin</artifactId>
      <version>0.5.1</version>
      <configuration>
        <protocArtifact>com.google.protobuf:protoc:3.2.0:exe:${os.detected.classifier}</protocArtifact>
        <pluginId>grpc-java</pluginId>
        <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.2.0:exe:${os.detected.classifier}</pluginArtifact>
        <outputDirectory>${basedir}/src/main/java</outputDirectory>
        <clearOutputDirectory>false</clearOutputDirectory>
      </configuration>
      <executions>
        <execution>
          <goals>
            <goal>compile</goal>
            <goal>compile-custom</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
   ```
   compile / package 之后，可以在target和src路径下看到由protobuf-maven-plugin生成的AbstractAuthorizeService.java和AuthorizationGrpc.java

## 创建服务端

1. 覆盖(override)生成的service代码

   * 创建AuthorizeServer类作为服务端监听类；
   * 在AuthorizeServer中创建内部类覆盖、实现具体的验证方法；
   ```java
package com.gomo.grpc.authorize.service;

public class AuthorizeService extends AuthorizationGrpc.AuthorizationImplBase {

   /**
     * 验证token服务
     * @param request
     * @param responseObserver server-side callback
     */
    @Override
    public void authorizeByAccessToken(AbstractAuthorizeService.AuthorizeRequest request, StreamObserver<AbstractAuthorizeService.AccessTokenDTO> responseObserver) {
        responseObserver.onNext(authorize(request));
        responseObserver.onCompleted();
    }

    private AbstractAuthorizeService.AccessTokenDTO authorize(AbstractAuthorizeService.AuthorizeRequest request){
        TokenAuthAccount tokenAuthAccount = tokenAuthAccountService.readAuthAccount(request.getAccessToken(), true);
        Account account = saveUser(tokenAuthAccount);
        AccessToken accessToken = new AccessToken(account.getAccountId(), Constants.ACCOUNT_TOKEN_EXPIRES_IN_DAYS, TimeUnit.DAYS);
        accessTokenRedisTemplate.opsForValue().set(generateAccessTokenRedisKey(accessToken.getAccessToken()), accessToken, accessToken.getExpiresIn(), TimeUnit.SECONDS);
        return AbstractAuthorizeService.AccessTokenDTO.newBuilder().setBind(account.isBind()).setAccessToken(accessToken.getAccessToken()).build();
    }
}

   ```

2. 运行gRPC服务端
   官方的运行方式如下：
	```java
	public static void main(String[] args) {
        Server server = ServerBuilder
          .forPort(8080)
          .addService(new HelloServiceImpl()).build();
 
        server.start();
        server.awaitTermination();
    }
	```

   由于要与spring boot进行整合，故使用开源工具[LogNet](http://www.lognet-travel.com/)出品的[grpc-spring-boot-starter](https://github.com/LogNet/grpc-spring-boot-starter)对grpc与spring boot进行整合。

   根据官方文档说法，starter能够兼容spring boot 1.5.X 和 2.X.X.

   * 首先引入maven依赖
   ``` xml
   <dependency>
      <groupId>org.lognet</groupId>
      <artifactId>grpc-spring-boot-starter</artifactId>
      <version>2.3.2</version>
   </dependency>
   ```

   * 配置application.yml
   ``` yml
   grpc:
     port: 8080 #default 6565
   ```

   * 给具体的实现service加上grpc注解@org.lognet.springboot.grpc.GRpcService


## 创建客户端

   按照官方文档的操作，创建连接的Channel做数据交互即可。
```java
import com.gomo.grpc.authorize.gen.AbstractAuthorizeService;
import com.gomo.grpc.authorize.gen.AuthorizationGrpc;
import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;
import io.grpc.StatusRuntimeException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @description: grpc认证服务
 * @create: 2018-06-12
 */
public class AuthorizeClient {

    private static final Logger LOGGER = LoggerFactory.getLogger(AuthorizeClient.class);

    private final ManagedChannel channel;
    private final AuthorizationGrpc.AuthorizationBlockingStub blockingStub;
    private final AuthorizationGrpc.AuthorizationStub asyncStub;

    public AuthorizeClient(String host, int port) {
		// Channels are secure by default (via SSL/TLS). For the example we disable TLS to avoid needing certificates.
        this(ManagedChannelBuilder.forAddress(host, port).usePlaintext(true));
    }

    /**
     * Construct client for accessing RouteGuide server using the existing channel.
     */
    public AuthorizeClient(ManagedChannelBuilder<?> channelBuilder) {
        channel = channelBuilder.build();
        blockingStub = AuthorizationGrpc.newBlockingStub(channel);
        asyncStub = AuthorizationGrpc.newStub(channel);
    }

    public AbstractAuthorizeService.AccessTokenDTO authorize(String accessToken) {
        try {
            AbstractAuthorizeService.AuthorizeRequest authorizeRequest = AbstractAuthorizeService.AuthorizeRequest.newBuilder().setAccessToken(accessToken).build();
            return blockingStub.authorizeByAccessToken(authorizeRequest);
        } catch (StatusRuntimeException e) {
            LOGGER.error("RPC failed: {}", e.getStatus(), e);
        }
        return null;
    }
    
}
```