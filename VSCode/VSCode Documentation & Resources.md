## Official Resources
- [VSCode Documentation](https://code.visualstudio.com/docs)
- [VSCode Updates](https://code.visualstudio.com/updates)
- [VSCode API](https://code.visualstudio.com/api)
- [VSCode Tips and Tricks](https://code.visualstudio.com/docs/getstarted/tips-and-tricks)

## Essential Keyboard Shortcuts

### General
```plaintext
Ctrl/Cmd + P         # Quick file open
Ctrl/Cmd + Shift + P # Command palette
Ctrl/Cmd + ,         # Settings
Ctrl/Cmd + B         # Toggle sidebar
Ctrl/Cmd + J         # Toggle terminal
```
These shortcuts form the backbone of efficient VSCode navigation. Learn these first!

### Editing
```plaintext
Alt + ↑/↓           # Move line up/down
Shift + Alt + ↑/↓   # Copy line up/down
Ctrl/Cmd + /        # Toggle line comment
Ctrl/Cmd + D        # Select next occurrence
Ctrl/Cmd + L        # Select current line
```
These editing shortcuts can significantly speed up your coding process.

### Multi-Cursor
```plaintext
Alt + Click         # Add cursor
Ctrl/Cmd + Alt + ↑/↓ # Add cursor above/below
Ctrl/Cmd + Shift + L # Select all occurrences
```
Multi-cursor editing is one of VSCode's most powerful features for bulk editing.

## Essential Extensions

### General Development
```plaintext
# Must-Have Extensions
- ESLint                  # JavaScript linting
- Prettier               # Code formatting
- GitLens                # Git integration
- Live Server            # Local dev server
- Auto Rename Tag        # HTML/XML tag renaming
```

### Language Specific
```plaintext
# Python
- Python                 # Python language support
- Pylance               # Python language server

# JavaScript/TypeScript
- JavaScript (ES6)      # ES6+ support
- TypeScript            # TypeScript support

# React
- ES7+ React/Redux      # React snippets
- Simple React Snippets # React snippets
```

## Settings Configuration

### Basic Settings (settings.json)
```json
{
    "editor.formatOnSave": true,
    "editor.tabSize": 2,
    "editor.wordWrap": "on",
    "files.autoSave": "onFocusChange"
}
```
These settings provide a good starting point for most developers.

### Language-Specific Settings
```json
{
    "[python]": {
        "editor.tabSize": 4,
        "editor.formatOnSave": true
    },
    "[javascript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    }
}
```
Different languages often need different settings - this lets you customize per language.

## Workspace Organization

### Multi-Root Workspaces
```plaintext
workspace.code-workspace
├── frontend/          # React project
├── backend/           # Express API
└── docs/             # Documentation
```
Multi-root workspaces help organize related projects.

### Project-Specific Settings
```json
// .vscode/settings.json
{
    "python.pythonPath": "./venv/bin/python",
    "eslint.workingDirectories": ["./src"]
}
```
These settings apply only to the current project.

## Debugging

### Launch Configurations
```json
// .vscode/launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "program": "${workspaceFolder}/app.js"
        }
    ]
}
```
Custom debug configurations help streamline the debugging process.

## Git Integration

### Built-in Git Features
```plaintext
- Source Control panel (Ctrl/Cmd + Shift + G)
- Inline blame annotations
- Diff viewer
- Merge conflict resolution
```
VSCode's Git integration is robust enough for most developers' needs.

## Terminal Integration

### Terminal Customization
```json
{
    "terminal.integrated.fontSize": 14,
    "terminal.integrated.shell.windows": "C:\\Program Files\\Git\\bin\\bash.exe",
    "terminal.integrated.defaultProfile.osx": "zsh"
}
```
A well-configured terminal can make VSCode your primary development environment.

## Snippets

### Custom Snippets
```json
{
    "React Function Component": {
        "prefix": "rfc",
        "body": [
            "function ${1:ComponentName}() {",
            "    return (",
            "        <div>",
            "            $0",
            "        </div>",
            "    )",
            "}",
            "",
            "export default ${1:ComponentName}"
        ]
    }
}
```
Custom snippets can significantly speed up repetitive coding tasks.

## Remote Development

### Remote Extensions
```plaintext
- Remote - SSH
- Remote - Containers
- Remote - WSL
```
These extensions enable development in remote environments.

## Performance Tips

### Workspace Optimization
```plaintext
1. Exclude unnecessary folders
2. Limit extension count
3. Use workspace trust
4. Adjust auto-save settings
```
These optimizations can significantly improve VSCode's performance.

## Theme Customization

### Color Themes
```json
{
    "workbench.colorTheme": "One Dark Pro",
    "workbench.iconTheme": "material-icon-theme",
    "editor.tokenColorCustomizations": {
        "comments": "#229977"
    }
}
```
Visual customization can improve code readability and reduce eye strain.

## Learning Resources
- [VSCode Can Do That?](https://vscodecandothat.com/)
- [VSCode Recipes](https://github.com/Microsoft/vscode-recipes)
- [VSCode YouTube Channel](https://www.youtube.com/channel/UCs5Y5_7XK8HLDX0SLNwkd3w)

Remember:
- Keep extensions minimal
- Learn keyboard shortcuts progressively
- Customize for your workflow
- Update regularly
- Back up settings
- Use workspace settings
- Configure per language
- Leverage built-in features
