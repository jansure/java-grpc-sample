# java-grpc-sample

#简介：
Grpc的优势这里我就不说了, 第一它是基于Http2.0协议的, 可以保持客户端与服务器端长连接, 基于二进制流,也就是字节流, 不是文本流.
​	同时他提供了跨平台(语言)的开发 , 但是他也有成本就是需要定义他的接口规范, 遵循他的一套规范, 也就是说就算你是Java语言, 你不能随意定义一个接口, 我就能帮你实现RPC , 只是他可以帮助你生成一个符合他规范的接口.
# 开始
##  引入maven依赖

```xml
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-all</artifactId>
    <version>1.12.0</version>
</dependency>

<build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.4.1.Final</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.5.0</version>
                <configuration>
                    <pluginId>grpc-java</pluginId>
                   <protoSourceRoot>${project.basedir}/src/main/resources/protobuf</protoSourceRoot>
                    <clearOutputDirectory>false</clearOutputDirectory> <protocArtifact>com.google.protobuf:protoc:3.0.2:exe:${os.detected.classifier}</protocArtifact>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.2.0:exe:${os.detected.classifier}</pluginArtifact>
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

## 定义proto内容
```
syntax = "proto3"; // 协议版本

// 选项配置
option java_package = "com.example.grpc.api";
option java_outer_classname = "RPCDateServiceApi";
option java_multiple_files = true;

// 定义包名
package com.example.grpc.api;

// 服务接口.定义请求参数和相应结果	
service RPCDateService {
    rpc getDate (RPCDateRequest) returns (RPCDateResponse) {
    }
}

// 定义请求体
message RPCDateRequest {
    string userName = 1;
}

// 定义相应内容
message RPCDateResponse {
    string serverDate = 1;
}

```

## 生成grpc接口
```shell
mvn protobuf:compile && mvc protobuf:compile-custom
```


## 实现接口
```java
public class RPCDateServiceImpl extends RPCDateServiceGrpc.RPCDateServiceImplBase {
    @Override
    public void getDate(RPCDateRequest request, StreamObserver<RPCDateResponse> responseObserver) {
        // 请求结果,我们定义的
        RPCDateResponse rpcDateResponse = null;
        String userName = request.getUserName();
        String response = String.format("你好: %s, 今天是%s.", userName, LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd")));
        try {
            // 定义响应,是一个builder构造器. 
            rpcDateResponse = RPCDateResponse
                    .newBuilder()
                    .setServerDate(response)
                    .build();
        } catch (Exception e) {
            responseObserver.onError(e);
        } finally {
            // 这种写法是observer,异步写法,老外喜欢用这个框架.
            responseObserver.onNext(rpcDateResponse);
        }
        responseObserver.onCompleted();
    }
}
```

## 定义服务端

```java
public class GRPCServer {
    private static final int port = 9999;

    public static void main(String[] args) throws Exception {
        // 设置service接口.
        Server server = ServerBuilder.
                forPort(port)
                .addService(new RPCDateServiceImpl())
                .build().start();
        System.out.println(String.format("GRpc服务端启动成功, 端口号: %d.", port));
        server.awaitTermination();
    }
}

```


