# JetBrains IDE Configuration

## Hosts Configuration

Block JetBrains crack sites to prevent unauthorized activation attempts:

```bash
# Block jetbrains crack sites
127.0.0.1  idechajian.com
127.0.0.1  repo.idechajian.com
127.0.0.1  ww25.repo.idechajian.com
```

To apply these changes:
1. Open `C:\Windows\System32\drivers\etc\hosts` with administrator privileges
2. Add the above entries to the file
3. Save and close the file
4. Flush DNS cache: `ipconfig /flushdns`

## IDE Installation and Setup

### Download Locations
- Official Website: https://www.jetbrains.com/products/
- Toolbox App: https://www.jetbrains.com/toolbox-app/

### Common JetBrains Products
- IntelliJ IDEA (Java/Kotlin development)
- PyCharm (Python development)
- WebStorm (JavaScript/TypeScript development)
- PhpStorm (PHP development)
- GoLand (Go development)
- Rider (.NET development)

## License Management

### Activation Methods
1. **Official License**: Purchase from JetBrains website
2. **Student License**: Free for students with educational email
3. **Open Source License**: Free for active open source projects
4. **Evaluation License**: 30-day trial period

### License Server Configuration
For enterprise environments using license servers:
```
Help → Register → License server
Enter server URL: http://your-license-server:port
```

## Performance Optimization

### JVM Options Configuration
Create or modify `idea64.vmoptions` file:

```bash
# Memory settings
-Xms2048m
-Xmx4096m
-XX:ReservedCodeCacheSize=1024m

# Garbage collection
-XX:+UseConcMarkSweepGC
-XX:SoftRefLRUPolicyMSPerMB=50

# Other optimizations
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
```

Location varies by installation:
- Windows: `%USERPROFILE%\AppData\Roaming\JetBrains\<Product><Version>`
- macOS: `~/Library/Application Support/JetBrains/<Product><Version>`
- Linux: `~/.config/JetBrains/<Product><Version>`

### Plugin Management
Recommended essential plugins:
- **String Manipulation**: Advanced text transformation
- **Rainbow Brackets**: Color-coded bracket matching
- **Key Promoter X**: Learn shortcuts interactively
- **GitToolBox**: Enhanced Git integration
- **SonarLint**: Real-time code quality analysis

## Customization

### Keymap Settings
Popular keymap schemes:
- **Default**: Standard JetBrains keymap
- **Visual Studio**: For Visual Studio users
- **Eclipse**: For Eclipse users
- **Sublime Text**: For Sublime Text users
- **Emacs**: For Emacs users
- **Vim**: For Vim users (with Ideavim plugin)

### Code Style Configuration
Import code style templates:
1. File → Settings → Editor → Code Style
2. Select language (Java, Python, etc.)
3. Import scheme from file or URL
4. Apply to project or globally

### Live Templates
Create custom live templates for frequently used code patterns:
1. File → Settings → Editor → Live Templates
2. Select appropriate template group
3. Click "+" to add new template
4. Configure abbreviation, description, and template text

## Troubleshooting

### Common Issues and Solutions

**IDE Startup Problems**:
```bash
# Clear IDE caches
File → Invalidate Caches and Restart

# Reset IDE settings
Delete configuration directory and restart
```

**Slow Performance**:
- Increase heap size in vmoptions
- Disable unused plugins
- Exclude large folders from indexing
- Use power-saving mode when battery is low

**Plugin Conflicts**:
- Start IDE with `idea.bat --safe-mode` (Windows)
- Disable plugins one by one to identify conflicts
- Check plugin compatibility with IDE version

### Log Files Location
- Windows: `%USERPROFILE%\AppData\Local\JetBrains\<Product><Version>\log`
- macOS: `~/Library/Logs/JetBrains/<Product><Version>`
- Linux: `~/.cache/JetBrains/<Product><Version>/log`

## Useful Shortcuts

### Navigation
- `Ctrl+Shift+N`: Find file by name
- `Ctrl+Shift+Alt+N`: Find symbol
- `Ctrl+E`: Recent files
- `Ctrl+Shift+E`: Recent locations
- `Ctrl+B`: Go to declaration
- `Ctrl+Alt+B`: Go to implementation

### Editing
- `Ctrl+D`: Duplicate line/selection
- `Ctrl+Y`: Delete line
- `Ctrl+Shift+J`: Join lines
- `Ctrl+Alt+L`: Reformat code
- `Ctrl+Alt+O`: Optimize imports

### Refactoring
- `Shift+F6`: Rename
- `Ctrl+Alt+M`: Extract method
- `Ctrl+Alt+V`: Extract variable
- `Ctrl+Alt+F`: Extract field
- `Ctrl+Alt+C`: Extract constant

## Backup and Migration

### Configuration Backup
Backup important directories:
```bash
# Windows backup script
xcopy "%USERPROFILE%\AppData\Roaming\JetBrains" "D:\Backup\JetBrains_Config" /E /I
xcopy "%USERPROFILE%\AppData\Local\JetBrains" "D:\Backup\JetBrains_Local" /E /I
```

### Settings Export/Import
1. File → Manage IDE Settings → Export Settings
2. Select components to export
3. Save as .jar file
4. Import on another machine: File → Manage IDE Settings → Import Settings

## Enterprise Deployment

### Silent Installation
Windows silent install:
```batch
ideaIU-2023.2.3.win.exe /S /CONFIG=path\to\config.xml
```

### Shared Configuration
For team environments:
1. Set up shared configuration repository
2. Use Settings Repository plugin
3. Configure company-wide code styles
4. Share inspection profiles and templates