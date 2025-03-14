# Go 配置源

以下是一些常用的 Go 国内源及其配置方法：

### 七牛镜像源
七牛镜像源是一个稳定且快速的国内镜像源，适合大多数用户使用。
- **配置命令**：
    ```bash
    go env -w GO111MODULE=on
    go env -w GOPROXY=https://goproxy.cn,direct
    ```
- **验证配置**：
    ```bash
    go env | grep GOPROXY
    ```

### 阿里云镜像源
阿里云镜像源也是一个常用的国内镜像源，提供了丰富的 Go 模块资源。
- **配置命令**：
    ```bash
    go env -w GO111MODULE=on
    go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
    ```
- **验证配置**：
    ```bash
    go env | grep GOPROXY
    ```

### 官方镜像源
虽然官方镜像源的速度可能不如国内镜像源快，但在某些情况下也可以使用。
- **配置命令**：
    ```bash
    go env -w GO111MODULE=on
    go env -w GOPROXY=https://goproxy.io,direct
    ```
- **验证配置**：
    ```bash
    go env | grep GOPROXY
    ```

### 配置私有模块
如果你的项目中使用了私有模块（如公司内部仓库），可以通过设置 `GOPRIVATE` 环境变量来指定哪些模块不通过代理获取，直接从本地获取。
- **配置命令**：
    ```bash
    go env -w GOPROXY=https://goproxy.cn,direct
    go env -w GOPRIVATE=*.corp.example.com
    ```
- **验证配置**：
    ```bash
    go env | grep GOPRIVATE
    ```

### 测试配置
配置完成后，可以通过以下命令测试是否生效：
```bash
go get -v github.com/golang/example/hello
```

如果能够成功下载并安装该示例模块，说明配置已经生效。