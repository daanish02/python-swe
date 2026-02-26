# Development Environments

## Table of Contents

- [IDEs and Editors](#ides-and-editors)
- [Jupyter Notebooks](#jupyter-notebooks)
- [REPL and Interactive Python](#repl-and-interactive-python)
- [Editor Configuration](#editor-configuration)
- [Productivity Tips](#productivity-tips)
- [Summary](#summary)
- [Next Steps](#next-steps)

## IDEs and Editors

**IDE (Integrated Development Environment)**: A comprehensive application combining code editor, debugger, and tools in one package.

**Text editor**: A lightweight program for editing code, often extended with plugins for Python features.

### VS Code

```bash
# Install: https://code.visualstudio.com/

# Essential Python extensions:
# 1. Python (ms-python.python) - IntelliSense, linting, debugging
# 2. Pylance (ms-python.vscode-pylance) - Fast type checking
# 3. Jupyter (ms-toolsai.jupyter) - Notebook support

# Install via command palette (Ctrl+Shift+P):
# ext install ms-python.python
```

```json
// .vscode/settings.json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": false,
  "python.linting.ruffEnabled": true,
  "python.formatting.provider": "black",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  },
  "python.testing.pytestEnabled": true,
  "python.testing.unittestEnabled": false
}
```

**VS Code features:**

- IntelliSense (autocomplete)
- Integrated terminal
- Git integration
- Debugging with breakpoints
- Extensions marketplace
- Remote development (SSH, containers)

### PyCharm

```bash
# Community Edition (free): Python development
# Professional Edition (paid): Web frameworks, databases, scientific tools

# Download: https://www.jetbrains.com/pycharm/
```

**PyCharm features:**

- Intelligent code completion
- Built-in debugger and profiler
- Database tools (Professional)
- Django/Flask support (Professional)
- Scientific tools (Professional)
- Refactoring tools
- Code inspections

```python
# PyCharm-specific features:

# 1. Quick documentation: Ctrl+Q
# 2. Go to definition: Ctrl+B
# 3. Find usages: Alt+F7
# 4. Refactor rename: Shift+F6
# 5. Generate code: Alt+Insert
# 6. Reformat code: Ctrl+Alt+L
```

### Vim/Neovim

```bash
# Install Neovim
# macOS:
brew install neovim

# Linux:
sudo apt install neovim

# Windows:
winget install Neovim.Neovim
```

```vim
" Minimal Python setup with vim-plug
call plug#begin()
    Plug 'neovim/nvim-lspconfig'  " LSP support
    Plug 'nvim-treesitter/nvim-treesitter'  " Syntax highlighting
    Plug 'hrsh7th/nvim-cmp'  " Autocomplete
    Plug 'github/copilot.vim'  " AI completion
call plug#end()

" Python LSP configuration
lua << EOF
require'lspconfig'.pyright.setup{}
EOF
```

**Vim advantages:**

- Extremely fast
- Works over SSH
- Minimal resource usage
- Powerful text manipulation
- Customizable with plugins

### Sublime Text

```bash
# Download: https://www.sublimetext.com/

# Install Package Control
# View > Show Console
# Paste install script from packagecontrol.io
```

**Useful packages:**

- Anaconda - Python IDE features
- LSP - Language server support
- SublimeLinter - Linting framework
- GitGutter - Git integration

### Comparison

| Feature             | VS Code | PyCharm   | Vim/Neovim | Sublime |
| ------------------- | ------- | --------- | ---------- | ------- |
| **Free**            | ✓       | Community | ✓          | Trial   |
| **Easy to learn**   | ✓✓✓     | ✓✓✓       | ✓          | ✓✓✓     |
| **Python-specific** | ✓✓      | ✓✓✓       | ✓✓         | ✓✓      |
| **Performance**     | ✓✓      | ✓         | ✓✓✓        | ✓✓✓     |
| **Extensions**      | ✓✓✓     | ✓✓        | ✓✓✓        | ✓✓      |
| **Remote dev**      | ✓✓✓     | ✓✓        | ✓✓✓        | ✓       |

## Jupyter Notebooks

**Jupyter Notebook**: An interactive web-based environment for running Python code in cells, mixing code, visualizations, and markdown text.

### Installation

```bash
# Install Jupyter
pip install jupyter

# Or use JupyterLab (next-gen interface)
pip install jupyterlab

# Start Jupyter Notebook
jupyter notebook

# Start JupyterLab
jupyter lab
```

### Basic Usage

```python
# Cell 1: Import libraries
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Cell 2: Load data
data = pd.read_csv('data.csv')
data.head()

# Cell 3: Visualize
plt.figure(figsize=(10, 6))
plt.plot(data['x'], data['y'])
plt.show()

# Cell 4: Analysis
mean_value = data['y'].mean()
print(f"Mean: {mean_value}")
```

### Keyboard Shortcuts

```bash
# Command mode (press Esc):
# A - Insert cell above
# B - Insert cell below
# DD - Delete cell
# M - Convert to markdown
# Y - Convert to code
# Shift+Enter - Run cell and move to next

# Edit mode (press Enter):
# Ctrl+Enter - Run cell
# Shift+Enter - Run cell and insert below
# Tab - Autocomplete
# Shift+Tab - Show documentation
```

### Magic Commands

```python
# %timeit - Measure execution time
%timeit sum(range(1000))

# %time - Time single execution
%time result = expensive_function()

# %load - Load code from file
%load script.py

# %run - Run external script
%run preprocessing.py

# %matplotlib - Enable inline plots
%matplotlib inline

# %pdb - Enable debugger on error
%pdb on

# %pwd - Print working directory
%pwd

# %ls - List files
%ls

# %%writefile - Save cell to file
%%writefile utils.py
def helper():
    return 42
```

### VS Code Jupyter Support

```json
// Use Jupyter in VS Code
// 1. Install Jupyter extension
// 2. Create .ipynb file or use #%% in .py files

# Python file as notebook with #%%
#%%
import pandas as pd

#%%
data = pd.read_csv('data.csv')
data.head()

#%%
# Each #%% creates a new cell
data.describe()
```

### When to Use Notebooks

**Good for:**

- Data exploration and analysis
- Learning and experimentation
- Sharing results with visualizations
- Teaching and demonstrations
- Prototyping ideas

**Not ideal for:**

- Production code
- Version control (difficult to diff)
- Testing
- Large-scale projects
- Team collaboration (merge conflicts)

## REPL and Interactive Python

**REPL (Read-Eval-Print Loop)**: An interactive shell that reads your input, evaluates it, prints the result, and loops back for more input.

### Basic Python REPL

```bash
# Start Python REPL
python
# or
python3

# Interactive session
>>> 2 + 2
4
>>> name = "Alice"
>>> f"Hello, {name}"
'Hello, Alice'
>>> import math
>>> math.pi
3.141592653589793
>>> exit()  # or Ctrl+D (Unix) / Ctrl+Z (Windows)
```

### IPython

```bash
# Install IPython (enhanced REPL)
pip install ipython

# Start IPython
ipython
```

**IPython features:**

```python
# Tab completion
import os
os.pa<TAB>  # Shows: path, pathconf, ...

# Object introspection
len?  # Show docstring
len??  # Show source code

# Magic commands
%timeit sum(range(1000))
%history
%paste  # Paste multi-line code

# System commands
!ls  # Run shell command
!pip install requests

# Auto-reload modules
%load_ext autoreload
%autoreload 2  # Reload modules before execution
```

### bpython

```bash
# Install bpython (fancy REPL)
pip install bpython

# Start
bpython
```

**bpython features:**

- Syntax highlighting
- Autocomplete suggestions as you type
- Expected parameter list
- Rewind (undo) functionality
- Save to file

### ptpython

```bash
# Install ptpython
pip install ptpython

# Start
ptpython
```

**ptpython features:**

- Vi or Emacs keybindings
- Syntax highlighting
- Multi-line editing
- Autocomplete
- Fish-style autosuggestions

## Editor Configuration

### VS Code Python Setup

```json
// settings.json
{
  // Interpreter
  "python.defaultInterpreterPath": ".venv/bin/python",

  // Linting
  "python.linting.enabled": true,
  "python.linting.ruffEnabled": true,

  // Formatting
  "python.formatting.provider": "black",
  "editor.formatOnSave": true,
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnType": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": true
    }
  },

  // Testing
  "python.testing.pytestEnabled": true,
  "python.testing.pytestArgs": ["tests"],

  // Type checking
  "python.analysis.typeCheckingMode": "basic"
}
```

### PyCharm Configuration

```python
# File > Settings (Ctrl+Alt+S)

# Python Interpreter:
# - Settings > Project > Python Interpreter
# - Add interpreter (virtualenv, system, conda)

# Code Style:
# - Settings > Editor > Code Style > Python
# - Set from PEP 8 or Black

# Inspections:
# - Settings > Editor > Inspections > Python
# - Enable type checking, PEP 8, etc.

# External Tools:
# - Settings > Tools > External Tools
# - Add black, ruff, mypy
```

### .editorconfig

```ini
# .editorconfig - Consistent settings across editors
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.py]
indent_style = space
indent_size = 4
max_line_length = 88

[*.{json,yml,yaml}]
indent_size = 2
```

## Productivity Tips

### Code Snippets

```json
// VS Code snippets: .vscode/python.json
{
  "Main Guard": {
    "prefix": "main",
    "body": ["if __name__ == '__main__':", "    ${1:main()}"]
  },
  "Class": {
    "prefix": "class",
    "body": [
      "class ${1:ClassName}:",
      "    def __init__(self, ${2:args}):",
      "        ${3:pass}",
      "",
      "    def ${4:method}(self):",
      "        ${5:pass}"
    ]
  }
}
```

### Multi-Cursor Editing

```bash
# VS Code:
# Alt+Click - Add cursor
# Ctrl+Alt+Up/Down - Add cursor above/below
# Ctrl+D - Select next occurrence
# Ctrl+Shift+L - Select all occurrences

# Use case: Rename variable in multiple places
# 1. Select variable
# 2. Ctrl+D to select next
# 3. Type new name
```

### Keyboard Shortcuts to Learn

```bash
# VS Code essentials:
# Ctrl+P - Quick file open
# Ctrl+Shift+P - Command palette
# Ctrl+` - Toggle terminal
# Ctrl+B - Toggle sidebar
# F12 - Go to definition
# Shift+F12 - Find references
# F2 - Rename symbol
# Ctrl+/ - Toggle comment

# PyCharm essentials:
# Shift Shift - Search everywhere
# Ctrl+N - Find class
# Ctrl+Shift+F - Find in files
# Ctrl+Alt+L - Reformat code
# Ctrl+W - Extend selection
# Alt+Insert - Generate code
```

### Workspace Management

```bash
# Project structure
myproject/
├── .vscode/          # VS Code settings
│   ├── settings.json
│   └── launch.json
├── .venv/            # Virtual environment
├── src/              # Source code
├── tests/            # Tests
├── .gitignore
├── requirements.txt
└── README.md

# .gitignore for Python
__pycache__/
*.py[cod]
*.so
.Python
.venv/
.env
*.egg-info/
dist/
build/
.pytest_cache/
.mypy_cache/
```

## Summary

Development environments for Python:

**IDEs and Editors:**

- VS Code: Free, extensible, great Python support
- PyCharm: Powerful, Python-focused, professional features
- Vim/Neovim: Fast, remote-friendly, steep learning curve
- Sublime Text: Fast, lightweight, extensible

**Jupyter Notebooks:**

- Interactive cell-based development
- Great for data science and exploration
- Mix code, visualizations, and markdown
- Use JupyterLab for modern interface

**REPL options:**

- Python REPL: Basic, always available
- IPython: Enhanced with magic commands
- bpython: Fancy with autocomplete
- ptpython: Vi/Emacs bindings

**Configuration:**

- .editorconfig for consistency
- settings.json (VS Code) or IDE preferences
- Code snippets for common patterns
- Keyboard shortcuts for efficiency

**Productivity:**

- Learn keyboard shortcuts
- Use multi-cursor editing
- Configure formatters and linters
- Set up debugging configurations

Choose tools that match your workflow and learn them deeply.

## Next Steps

- Learn [Running Python Code](running-python.md)
- Set up [Virtual Environments](virtual-environments.md)
- Master [Package Management](package-management.md)
