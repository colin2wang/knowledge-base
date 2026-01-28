# Go Environment Management

The `go env` command manages Go environment variables that control the behavior of the Go toolchain, including code storage, module management, and compilation targets.

## Key Functions

### 1. View All Environment Variables
Print values of all Go environment variables to understand current configuration:
```bash
go env
```

### 2. Set Environment Variables
Use the `-w` flag to write environment variables to Go's configuration file:
```bash
go env -w GOPROXY=https://proxy.golang.org
```

### 3. Get Specific Variable Value
Retrieve only a specific environment variable:
```bash
go env GOOS
```

### 4. Debugging & Troubleshooting
Quickly check environment configuration when encountering Go-related issues.

### 5. Multi-Platform Development
Cross-compile for different operating systems and architectures by setting `GOOS` and `GOARCH`.

## Common Environment Variables

| Variable | Description | Default |
| --- | --- | --- |
| `GO111MODULE` | Controls Go Modules behavior: `on` (enabled), `off` (disabled), or `auto` (enabled if go.mod exists) | `auto` |
| `GOARCH` | Target machine architecture (amd64, 386, arm, etc.) | Current system architecture |
| `GOBIN` | Directory where `go install` places executables | `$GOPATH/bin` |
| `GOCACHE` | Directory for build cache | `$HOME/.cache/go-build` |
| `GOEXE` | Executable file suffix (`.exe` on Windows, empty otherwise) | System default |
| `GOFLAGS` | Default flags for `go` commands | None |
| `GOHOSTARCH` | Go toolchain binary architecture | Current system architecture |
| `GOHOSTOS` | Go toolchain binary operating system | Current operating system |
| `GOINSECURE` | Comma-separated list of module path prefixes to always fetch insecurely | None |
| `GOMODCACHE` | Directory for downloaded modules | `$GOPATH/pkg/mod` |
| `GOOS` | Target machine operating system (linux, windows, darwin, etc.) | Current operating system |
| `GOPATH` | Root directory for Go source code and packages | `$HOME/go` |
| `GOPRIVATE` | Comma-separated list of module path prefixes to fetch directly (bypassing proxy) | None |
| `GONOPROXY` | Comma-separated list of module path prefixes that cannot use proxy | None |
| `GONOSUMDB` | Comma-separated list of module path prefixes to skip checksum verification | None |
| `GOPROXY` | URL of Go Modules proxy for accelerated downloads | `https://proxy.golang.org` |
| `GOSUMDB` | Checksum database name | `sum.golang.org` |
| `GOWORK` | Workspace file to use (`auto` searches current and parent directories) | `auto` |
| `GOROOT` | Go installation directory | Current system installation path |