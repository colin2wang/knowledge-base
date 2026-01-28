# Go Modules

Go Modules is Go's dependency management system that enables: 
- Working in any directory (not just `GOPATH`)
- Installing specific versions of dependencies
- Supporting multiple versions of the same dependency

## Basic Usage

### 1. Initialize a Module
Create a new module with a `go.mod` file containing module name and Go version:
```bash
go mod init example.com/myproject
```

### 2. Add Dependencies
Install specific versions of dependencies:
```bash
# Add specific version
go get github.com/labstack/echo@v3.3.10

# Add latest version
go get github.com/labstack/echo
```

### 3. Update Dependencies
```bash
# Update to latest versions
go get -u github.com/labstack/echo

# Update to latest patch versions
go get -u=patch github.com/labstack/echo
```

### 4. Tidy Dependencies
Automatically add missing dependencies and remove unused ones:
```bash
go mod tidy
```

### 5. Download Dependencies
Download all dependencies to local cache (`$GOPATH/pkg/mod`):
```bash
go mod download
```

### 6. Vendor Dependencies
Copy dependencies to project's `vendor` directory (useful for offline builds):
```bash
go mod vendor
```

### 7. Verify Dependencies
Ensure dependencies are complete and unmodified:
```bash
go mod verify
```

### 8. View Dependency Graph
```bash
go mod graph
```

### 9. Replace Dependencies
Substitute dependencies with local paths or other modules (useful for development):
```bash
go mod replace github.com/example/foo v1.2.3 /path/to/local/foo
```

## Summary
Go Modules provides a flexible, controlled dependency management solution for Go projects, allowing easy initialization, management, and maintenance of project dependencies.