# Go ENV

`go env` 是 Go 语言中的一个命令，用于查看和管理 Go 的环境变量。这些环境变量控制着 Go 工具链的行为，例如 Go 代码的存储位置、模块的管理方式、编译目标等。

以下是 `go env` 的一些主要作用：

### 1. 查看环境变量
通过运行 `go env` 命令，可以打印出当前 Go 环境中所有环境变量的值。这有助于开发者了解当前的开发环境配置。

```bash
go env
```

### 2. 设置环境变量
`go env` 命令还可以用来设置环境变量的值。通过指定 `-w` 参数，可以将环境变量的值写入到 Go 的环境配置文件中。

例如，设置 `GOPROXY` 环境变量的值：

```bash
go env -w GOPROXY=https://proxy.golang.org
```

### 3. 获取特定环境变量的值
如果只想获取某个特定环境变量的值，可以在 `go env` 命令后指定变量名。

例如，获取 `GOOS` 环境变量的值：

```bash
go env GOOS
```

### 4. 调试和问题排查
当遇到与 Go 环境相关的问题时，`go env` 可以帮助开发者快速查看当前的环境配置，从而进行调试和问题排查。

### 5. 环境变量的默认值
如果某些环境变量未被显式设置，`go env` 会根据系统的默认值来填充这些变量。这确保了 Go 工具链在大多数情况下能够正常工作。

### 6. 支持多平台开发
通过设置 `GOOS` 和 `GOARCH` 环境变量，开发者可以在不同的操作系统和架构之间进行交叉编译。这对于需要为多个平台构建应用程序的场景非常有用。

总之，`go env` 是 Go 开发中一个非常重要的工具，它帮助开发者管理和查看 Go 环境变量，从而更好地控制 Go 工具链的行为。

以下是 `go env` 中常用环境变量及其解释和默认值：

| 环境变量名 | 解释 | 默认值 |
| --- | --- | --- |
| `GO111MODULE` | 控制 Go Modules 的行为，可设置为 `on`、`off` 或 `auto`。`on` 表示启用 Go Modules 模式；`off` 表示禁用 Go Modules 模式；`auto` 表示根据当前目录下是否存在 `go.mod` 文件来判断是否启用 Go Modules 模式。 | `auto` |
| `GOARCH` | 目标机器的处理器架构，如 `amd64`、`386`、`arm` 等 | 当前系统架构 |
| `GOBIN` | `go install` 安装命令的目标目录 | `$GOPATH/bin` |
| `GOCACHE` | Go 命令存储缓存信息的目录 | `$HOME/.cache/go-build` |
| `GOENV` | Go 环境变量配置文件的位置 | 无 |
| `GOEXE` | 可执行文件的后缀名（在 Windows 上为 `.exe`，在其他系统上为空） | 系统默认 |
| `GOFLAGS` | 默认应用于 `go` 命令的标志列表 | 无 |
| `GOHOSTARCH` | Go 工具链二进制文件的架构（GOARCH） | 当前系统架构 |
| `GOHOSTOS` | Go 工具链二进制文件的操作系统（GOOS） | 当前操作系统 |
| `GOINSECURE` | 始终以不安全方式获取的模块路径前缀的逗号分隔列表 | 无 |
| `GOMODCACHE` | Go 命令存储下载模块的目录 | `$GOPATH/pkg/mod` |
| `GOOS` | 目标机器的操作系统，如 `linux`、`windows`、`darwin` 等 | 当前操作系统 |
| `GOPATH` | Go 项目的根目录，用于存放 Go 源码、包等 | 用户主目录下的 `go` 子目录，如 `$HOME/go` |
| `GOPRIVATE` | 直接获取的模块路径前缀的逗号分隔列表，这些模块不会通过代理获取 | 无 |
| `GONOPROXY` | 不能通过代理获取的模块路径前缀的逗号分隔列表 | 无 |
| `GONOSUMDB` | 不进行校验数据库对比的模块路径前缀的逗号分隔列表 | 无 |
| `GOPROXY` | Go Modules 代理的 URL，用于加速下载 Go 模块 | `https://proxy.golang.org` |
| `GOSUMDB` | 使用的校验数据库的名称，可以选择其公钥和 URL | `sum.golang.org` |
| `GOWORK` | 使用的工作区文件，如果为 `auto` 则在当前目录及其父目录中查找 | `auto` |
| `GOROOT` | Go 语言的安装目录 | 当前系统安装目录 |