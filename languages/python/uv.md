# UV - Ultra Fast Python Package Installer

UV is an extremely fast Python package installer and resolver, written in Rust. It's designed to be a drop-in replacement for pip and pip-tools with significantly better performance.

## 🚀 Key Features

- **Ultra Fast**: 10-100x faster than pip
- **Dependency Resolution**: Advanced conflict resolution algorithm
- **Lock Files**: Deterministic builds with `uv.lock`
- **Virtual Environments**: Built-in venv management
- **Cross-Platform**: Works on Windows, macOS, and Linux
- **Rust-Powered**: Written in Rust for maximum performance

## 📦 Installation

### Using pip
```bash
pip install uv
```

### Using standalone installer (recommended)
```bash
# Linux/macOS
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Using package managers
```bash
# Homebrew (macOS/Linux)
brew install uv

# Scoop (Windows)
scoop install uv

# Chocolatey (Windows)
choco install uv
```

## 🛠️ Basic Usage

### Creating Virtual Environments
```bash
# Create a new virtual environment
uv venv

# Create with specific Python version
uv venv --python 3.11

# Create in custom directory
uv venv ./my-project-env
```

### Installing Packages
```bash
# Install a single package
uv pip install requests

# Install from requirements.txt
uv pip install -r requirements.txt

# Install with extras
uv pip install "requests[security]"

# Install in development mode
uv pip install -e .
```

### Managing Dependencies
```bash
# Add a dependency
uv add requests

# Add development dependency
uv add --dev pytest

# Remove a dependency
uv remove requests

# Sync with pyproject.toml
uv sync
```

## 📁 Project Management

### Initializing Projects
```bash
# Initialize a new Python project
uv init my-project

# Initialize with specific Python version
uv init my-project --python 3.11
```

### Working with pyproject.toml
```toml
[project]
name = "my-project"
version = "0.1.0"
description = "My awesome Python project"
dependencies = [
    "requests>=2.28.0",
    "click>=8.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "black>=22.0.0",
]
```

### Lock File Management
```bash
# Generate lock file
uv lock

# Update dependencies
uv lock --upgrade

# Update specific package
uv lock --upgrade-package requests

# Validate lock file
uv lock --locked
```

## ⚡ Performance Comparison

### Speed Benchmarks
```bash
# Typical performance improvements:
# - Dependency resolution: 10-50x faster
# - Package installation: 2-10x faster  
# - Environment creation: 3-15x faster
```

### Memory Usage
UV uses significantly less memory compared to traditional tools:
- No dependency resolution timeouts
- Efficient caching mechanisms
- Parallel downloads by default

## 🔧 Advanced Features

### Workspace Support
```bash
# Work with multiple packages
uv workspace add ../other-package

# Sync entire workspace
uv sync --workspace
```

### Custom Index URLs
```bash
# Use custom package index
uv pip install --index-url https://pypi.example.com/simple/ package-name

# Add extra index URLs
uv pip install --extra-index-url https://test.pypi.org/simple/ package-name
```

### Build Isolation
```bash
# Enable build isolation (default)
uv pip install --isolated package-name

# Disable build isolation
uv pip install --no-build-isolation package-name
```

## 🎯 Best Practices

### Project Structure
```
my-project/
├── src/
│   └── my_package/
├── tests/
├── pyproject.toml
├── uv.lock
└── README.md
```

### Development Workflow
```bash
# 1. Initialize project
uv init my-project
cd my-project

# 2. Add dependencies
uv add requests flask
uv add --dev pytest black

# 3. Install dependencies
uv sync

# 4. Activate environment
source .venv/bin/activate  # Linux/macOS
.venv\Scripts\activate     # Windows

# 5. Run tests
pytest

# 6. Format code
black .
```

## 🚨 Migration from pip

### Converting requirements.txt
```bash
# Generate requirements.txt from current environment
uv pip freeze > requirements.txt

# Install from existing requirements.txt
uv pip install -r requirements.txt
```

### Pip Compatibility
Most pip commands work directly with uv:
```bash
uv pip install package    # Same as pip install package
uv pip uninstall package  # Same as pip uninstall package
uv pip list              # Same as pip list
uv pip show package      # Same as pip show package
```

## 🔍 Troubleshooting

### Common Issues

**Package not found:**
```bash
# Check index URL
uv pip install --verbose package-name

# Try different index
uv pip install --index-url https://pypi.org/simple/ package-name
```

**Dependency conflicts:**
```bash
# View conflict details
uv pip install --dry-run package-name

# Force resolution (use carefully)
uv pip install --force-reinstall package-name
```

**Build failures:**
```bash
# Install build dependencies
uv pip install setuptools wheel

# Use pre-built wheels
uv pip install --only-binary=:all: package-name
```

## 📚 Resources

- [Official Documentation](https://docs.astral.sh/uv/)
- [GitHub Repository](https://github.com/astral-sh/uv)
- [Performance Benchmarks](https://docs.astral.sh/uv/guides/benchmarks/)
- [Migration Guide](https://docs.astral.sh/uv/guides/migration/)