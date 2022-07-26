# gRPC 学习案例

## 安装
1. protoc 安装

- 进入 [protobuf release](https://github.com/protocolbuffers/protobuf/releases) 页面，选择适合自己操作系统的压缩包文件
- 解压 `protoc-x.x.x-osx-x86_64.zip` 并进入 `protoc-x.x.x-osx-x86_64`
```
$ cd protoc-x.x.x-osx-x86_64/bin
```
- 将启动的 `protoc` 二进制文件移动到被添加到环境变量的任意 path下，如 `$GOPATH/bin`，这里不建议直接将其和系统的以下path放在一起。
```
$ mv protoc $GOPATH/bin
```
>tip: `$GOPATH`为你本机的实际文件夹地址
- 验证安装结果
```
$ protoc --version
libprotoc x.x.x
```

2. protoc-gen-go/protoc-gen-go-grpc 安装
```
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

3. gRPC 资料
- 链接：[gRPC基于Go快速开始](https://grpc.io/docs/languages/go/quickstart/)
- 安装：`go get -u google.golang.org/grpc`

## protoc 的使用
`protoc` 生成 GO 代码：
```
$ protoc --proto_path=proto proto/*.proto --go_out=plugins=grpc:pb
```

4. Clang-Format proto 格式化工具
- 链接：[LLVM-14.0.6-win64.exe](https://github.com/llvm/llvm-project/releases/tag/llvmorg-14.0.6) 
- 介绍：可用于格式化（排版）多种不同语言的代码。其自带的排版格式主要有：LLVM, Google, Chromium, Mozilla, WebKit等
若 `-style=google` ，则表示应用Google的格式化风格
- 安装（window）：从链接处下载之后就可以进行安装了，安装时注意勾选环境变量
- 检查下是否安装成功：`clang-format -version`

5. Evans
- 链接：[Evans](https://github.com/ktr0731/evans)
- 介绍：`Evans` 是一款超酷的 `gRPC` 客户端，可让你构建并在交互式shell中将请求发送到 `gRPC` 服务器。
- 安装：
    1. 进入 [Evans release](https://github.com/ktr0731/evans/releases) 页面，选择适合自己操作系统的压缩包文件
    2. 解压 `evans_xx_xx.tar.gz` 并进入 `evans_xx_xx.tar`,
    ```
    $ cd evans_xx_xx.tar.gz
    ```
    3. 再解压 `evans_xx_xx.tar` 并进入 `evans_xx_xx`,
     ```
    $ cd evans_xx_xx
    ```
    4. 将启动的 `evans` 二进制文件移动到被添加到环境变量的任意 path下，如 `$GOPATH/bin`，这里不建议直接将其和系统的以下path放在一起。
    ```
    $ mv evans $GOPATH/bin
    ```
    >tip: `$GOPATH`为你本机的实际文件夹地址
    5. 验证安装结果
    ```
    $ evans --version
    evans 0.10.8
    ```
-  go install: `go install github.com/ktr0731/evans@latest`
-  使用：
    
    连接服务器：
    ```
    $ evans -r repl -p 8080
    ```
    查看服务上所有可用的软件包:
    ```
    $ show package
    ```
    选择特定的软件包:
    ```
    $ package proto
    ```
    显示该软件包所有的服务:
    ```
    $ show service
    ```
    显示该软件包所有的消息
    ```
    $ show message
    ```
    获取消息格式：
    ```
    $ desc XxxxRequest
    ```
    选择某个服务:
    ```
    $ service LaptopService
    ```
    待用该服务下的 API：
    ```
    $ call CreateLaptop
    ```
    调用API下，停止重复字段输入，使用 `Ctrl + D`
6. Nginx
- 下载链接：http://nginx.org/en/download.html
- 负载均衡：
```
worker_processes  1;

error_log  logs/error.log;

events {
    worker_connections  10;
}


http {
    access_log  logs/access.log;

    upstream pcbook_service {
        server 127.0.0.1:50051;
        server 127.0.0.1:50052;
    }

    server {
        listen       8080 http2;

        location / {
           grpc_pass grpc://pcbook_service;
        }
    }
}
```
- 负载均衡-SSL：
```
worker_processes  1;

error_log  logs/error.log;

events {
    worker_connections  10;
}


http {
    access_log  logs/access.log;

    upstream pcbook_service {
        server 127.0.0.1:50051;
        server 127.0.0.1:50052;
    }

    server {
        listen       8080 ssl http2;

        ssl_certificate ../cert/server-cert.pem;
        ssl_certificate_key ../cert/server-key.pem;

        ssl_client_certificate ../cert/ca-cert.pem;
        ssl_verify_client on;

        location / {
           grpc_pass grpc://pcbook_service;
        }
    }
}
```
- 负载均衡-SSL-服务器和Nginx之间授信(双向TLS)：
```
worker_processes  1;

error_log  logs/error.log;

events {
    worker_connections  10;
}


http {
    access_log  logs/access.log;

    upstream auth_service {
        server 127.0.0.1:50051;
        server 127.0.0.1:50052;
    }

    server {
        listen       8080 ssl http2;

        ssl_certificate ../cert/server-cert.pem;
        ssl_certificate_key ../cert/server-key.pem;

        ssl_client_certificate ../cert/ca-cert.pem;
        ssl_verify_client on;

        location / {
           grpc_pass grpcs://pcbook_service;

           grpc_ssl_certificate ../cert/server-cert.pem;
           grpc_ssl_certificate_key ../cert/server-key.pem;
        }
    }
}
```
- 负载均衡-SSL-服务器和Nginx之间授信(双向TLS)-指定rpc：
```
worker_processes  1;

error_log  logs/error.log;

events {
    worker_connections  10;
}


http {
    access_log  logs/access.log;

    upstream auth_services {
        server 127.0.0.1:50051;
    }

    upstream laptop_services {
        server 127.0.0.1:50052;
    }

    server {
        listen       8080 ssl http2;

        ssl_certificate ../cert/server-cert.pem;
        ssl_certificate_key ../cert/server-key.pem;

        ssl_client_certificate ../cert/ca-cert.pem;
        ssl_verify_client on;

        location /pcbook.AuthService {
           grpc_pass grpcs://auth_services;

           grpc_ssl_certificate ../cert/server-cert.pem;
           grpc_ssl_certificate_key ../cert/server-key.pem;
        }

        location /pcbook.LaptopService {
           grpc_pass grpcs://laptop_services;
        
           grpc_ssl_certificate ../cert/server-cert.pem;
           grpc_ssl_certificate_key ../cert/server-key.pem;
        }
    }
}
```
7. grpc-gateway
- 链接（使用 v2 版本）：https://github.com/grpc-ecosystem/grpc-gateway/tree/v2.11.0

8. Swagger
- 链接：

## 资源
- [jwt.io](https://jwt.io/) 解析 jwt token。

- [OpenSSL 安装](http://slproweb.com/products/Win32OpenSSL.html)

- [如何创建和签名SSL/TLS证书？](https://github.com/yellowStarts/go-pcbook-micro/blob/main/cert/post.md)
    
## 项目使用
清除 `ptotoc` 生成的 Go 代码:
```
$ make clean
```
生成代码:
```
$ make gen
```
运行服务器1:
```
$ make server1
```
运行服务器2:
```
$ make server2
```
运行SSL/TLS服务器1:
```
$ make server1-tls
```
运行SSL/TLS服务器2:
```
$ make server2-tls
```
运行服务器(grpc):
```
$ make server
```
运行服务器(rest API):
```
$ make server-rest
```
运行客户端:
```
$ make client
```
运行SSL/TLS客户端:
```
$ make client
```
测试：
```
$ make test
```
运行生成 `TLS` 证书脚本:
```
$ make cert
```

## 注意事项
- proto 文件 `import` 导包红线, 在 vscode 中 首选项>设置>搜索“protoc”, 做如下修改：
```
"protoc": {
        "path": "D:/Program Files/Go/bin/protoc",
        "options": [
            "--proto_path=proto",
        ]
    }
```
将 `path` 改成 `protoc` 命令实际存储的位置，将 `--proto_path` 修改成项目 proto 源文件存储的目录

- [missing mustEmbedUnimplemented*Server method； protobuf](https://blog.csdn.net/qq_38963393/article/details/122273996)
