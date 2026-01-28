# Go Proxy Configuration

This guide covers configuring Go module proxies for optimal package download performance.

## Prerequisites
Ensure Go Modules are enabled:
```bash
go env -w GO111MODULE=on
```

## Available Proxies

### Qiniu Mirror (Recommended)
A stable and fast proxy suitable for most users:
```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

### Alibaba Cloud Mirror
Another reliable domestic mirror with extensive Go module resources:
```bash
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
```

### Official Mirror
May be slower than domestic mirrors but useful in specific cases:
```bash
go env -w GOPROXY=https://goproxy.io,direct
```

## Private Module Configuration
For private modules (e.g., internal company repositories), configure GOPRIVATE to bypass proxies:
```bash
# Set proxy
go env -w GOPROXY=https://goproxy.cn,direct

# Exclude specific domains from proxy
go env -w GOPRIVATE=*.corp.example.com
```

## Verify Configuration

### Check Proxy Setting
```bash
go env | grep GOPROXY
```

### Check Private Module Setting
```bash
go env | grep GOPRIVATE
```

## Test Configuration
Verify proxy configuration by downloading a sample module:
```bash
go get -v github.com/golang/example/hello
```

If the module downloads successfully, your configuration is working properly.