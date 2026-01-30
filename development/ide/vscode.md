# Visual Studio Code

## Portable Configuration

Run VS Code with custom configuration directories to maintain separate environments:

```shell
D:\Software\DevelopTools\VSCode\CodeBin\Code.exe --extensions-dir "data/extensions" --user-data-dir "data/user-data"
```

## Common Launch Parameters

### Basic Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--extensions-dir <dir>` | Set the root path for extensions | `--extensions-dir "./vscode-extensions"` |
| `--user-data-dir <dir>` | Specifies the directory that user data is kept in | `--user-data-dir "./vscode-config"` |
| `--locale <locale>` | Set the display language | `--locale en` or `--locale zh-CN` |

### Workspace & File Operations

| Parameter | Description | Example |
|-----------|-------------|---------|
| `<file>` | Open a file | `Code.exe app.js` |
| `<folder>` | Open a folder | `Code.exe ./my-project` |
| `--goto <file:line[:character]>` | Open file at specific line and character | `Code.exe --goto app.js:10:5` |
| `--diff <file1> <file2>` | Compare two files | `Code.exe --diff file1.txt file2.txt` |
| `--add <folder>` | Add folder(s) to the last active window | `Code.exe --add ./new-folder` |
| `--wait` | Wait for the files to be closed before returning | `Code.exe --wait README.md` |

### Window Management

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--new-window` or `-n` | Force to open a new window | `Code.exe -n` |
| `--reuse-window` or `-r` | Force to open a file or folder in the last active window | `Code.exe -r ./project` |
| `--folder-uri <uri>` | Opens a window with given folder uri(s) | `Code.exe --folder-uri file:///path/to/folder` |
| `--file-uri <uri>` | Opens a window with given file uri(s) | `Code.exe --file-uri file:///path/to/file.js` |

### Development & Debugging

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--disable-extensions` | Disable all installed extensions | `Code.exe --disable-extensions` |
| `--disable-extension <ext-id>` | Disable specific extension | `Code.exe --disable-extension ms-python.python` |
| `--inspect-extensions <port>` | Allow debugging and profiling of extensions | `Code.exe --inspect-extensions 9223` |
| `--log <level>` | Log level to use (critical, error, warn, info, debug, trace) | `Code.exe --log debug` |
| `--verbose` | Print verbose output (implies --log trace) | `Code.exe --verbose` |

### Performance & System

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--max-memory <memory>` | Max memory size for a window (in Mbytes) | `Code.exe --max-memory 4096` |
| `--prof-startup` | Run CPU profiler during startup | `Code.exe --prof-startup` |
| `--disable-gpu` | Disable GPU hardware acceleration | `Code.exe --disable-gpu` |
| `--no-sandbox` | Disable the sandbox for all process types | `Code.exe --no-sandbox` |

## Practical Examples

### Create Portable VS Code Setup
```shell
# Create portable configuration
mkdir vscode-portable
cd vscode-portable

# Run with custom directories
Code.exe --extensions-dir "./extensions" --user-data-dir "./userdata" --locale en
```

### Compare Files
```shell
# Compare two configuration files
Code.exe --diff config-old.json config-new.json
```

### Open Specific Location
```shell
# Open file at specific line and column
Code.exe --goto src/main.js:25:10
```

### Debug Extensions
```shell
# Run with extension debugging enabled
Code.exe --inspect-extensions 9223 --extensionDevelopmentPath ./my-extension
```

### Performance Tuning
```shell
# Limit memory usage and disable GPU for older systems
Code.exe --max-memory 2048 --disable-gpu
```

## Notes

- Paths with spaces should be enclosed in quotes
- Relative paths are resolved relative to the current working directory
- Some parameters may require administrator privileges on Windows
- The `--verbose` flag is helpful for troubleshooting startup issues