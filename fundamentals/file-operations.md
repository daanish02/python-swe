# File Operations

## Table of Contents

- [Reading Files](#reading-files)
- [Writing Files](#writing-files)
- [File Modes](#file-modes)
- [Context Managers](#context-managers)
- [Working with Paths](#working-with-paths)
- [File Formats](#file-formats)
- [File System Operations](#file-system-operations)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Reading Files

### Basic File Reading

```python
# Open and read entire file
file = open('example.txt', 'r')
content = file.read()
print(content)
file.close()  # Always close files!
```

**Problem:** If an error occurs, file might not close properly.

### Reading Methods

```python
file = open('example.txt', 'r')

# Read entire file as string
content = file.read()

# Read one line
line = file.readline()

# Read all lines into list
lines = file.readlines()

file.close()
```

**Example:**

```python
# example.txt contains:
# Line 1
# Line 2
# Line 3

file = open('example.txt', 'r')

# read() - entire file as string
content = file.read()
print(repr(content))  # 'Line 1\nLine 2\nLine 3\n'

# Go back to start
file.seek(0)

# readline() - one line at a time
line1 = file.readline()  # 'Line 1\n'
line2 = file.readline()  # 'Line 2\n'

# Go back to start
file.seek(0)

# readlines() - list of lines
lines = file.readlines()  # ['Line 1\n', 'Line 2\n', 'Line 3\n']

file.close()
```

### Reading Line by Line

More memory efficient for large files:

```python
file = open('large_file.txt', 'r')
for line in file:
    print(line.strip())  # Remove \n
file.close()
```

### Reading with Size Limit

```python
file = open('example.txt', 'r')

# Read first 10 characters
chunk = file.read(10)

# Read next 20 characters
chunk = file.read(20)

file.close()
```

## Writing Files

### Basic File Writing

```python
# Write mode (overwrites existing file)
file = open('output.txt', 'w')
file.write('Hello, World!')
file.close()
```

### Writing Multiple Lines

```python
file = open('output.txt', 'w')

file.write('Line 1\n')
file.write('Line 2\n')
file.write('Line 3\n')

# Or use writelines() with a list
lines = ['Line 1\n', 'Line 2\n', 'Line 3\n']
file.writelines(lines)

file.close()
```

**Note:** `write()` doesn't add newlines automatically - you must include `\n`.

### Appending to Files

```python
# Append mode (adds to end of file)
file = open('log.txt', 'a')
file.write('New log entry\n')
file.close()
```

## File Modes

Common file modes:

| Mode   | Description      | Creates if Missing    | Truncates |
| ------ | ---------------- | --------------------- | --------- |
| `'r'`  | Read (default)   | No                    | No        |
| `'w'`  | Write            | Yes                   | Yes       |
| `'a'`  | Append           | Yes                   | No        |
| `'x'`  | Exclusive create | Yes (fails if exists) | No        |
| `'r+'` | Read and write   | No                    | No        |
| `'w+'` | Write and read   | Yes                   | Yes       |
| `'a+'` | Append and read  | Yes                   | No        |

**Binary modes:** Add `'b'` suffix:

```python
# Read binary file
file = open('image.png', 'rb')
data = file.read()
file.close()

# Write binary file
file = open('output.bin', 'wb')
file.write(b'\x00\x01\x02\x03')
file.close()
```

**Text vs Binary:**

- Text mode (`'r'`, `'w'`): Handles encoding, line endings automatically
- Binary mode (`'rb'`, `'wb'`): Raw bytes, no conversion

## Context Managers

### The `with` Statement

Automatically closes files, even if errors occur:

```python
# Recommended way to work with files
with open('example.txt', 'r') as file:
    content = file.read()
    print(content)
# File automatically closed here

# Can access content after block
print(content)
```

**Multiple files:**

```python
with open('input.txt', 'r') as infile, open('output.txt', 'w') as outfile:
    content = infile.read()
    outfile.write(content.upper())
```

### Why Use `with`?

```python
# Without with - must handle errors
file = open('example.txt', 'r')
try:
    content = file.read()
finally:
    file.close()  # Ensures file closes even with errors

# With with - automatic cleanup
with open('example.txt', 'r') as file:
    content = file.read()
# Automatically closes, even if exception occurs
```

### Common Patterns

**Read entire file:**

```python
with open('file.txt', 'r') as f:
    content = f.read()
```

**Read lines:**

```python
with open('file.txt', 'r') as f:
    for line in f:
        process(line.strip())
```

**Write lines:**

```python
with open('output.txt', 'w') as f:
    for item in items:
        f.write(f"{item}\n")
```

**Copy file:**

```python
with open('source.txt', 'r') as src, open('dest.txt', 'w') as dst:
    dst.write(src.read())
```

## Working with Paths

### pathlib Module (Recommended)

Modern, object-oriented path handling:

```python
from pathlib import Path

# Create path object
path = Path('data/file.txt')

# Check if exists
if path.exists():
    print("File exists")

# Check if file or directory
path.is_file()  # True if file
path.is_dir()   # True if directory

# Get file info
path.name       # 'file.txt'
path.stem       # 'file' (without extension)
path.suffix     # '.txt'
path.parent     # Path('data')

# Read/write with Path
content = path.read_text()
path.write_text('New content')

# Binary
data = path.read_bytes()
path.write_bytes(b'\x00\x01')
```

### Building Paths

```python
from pathlib import Path

# Join paths (works on all OS)
path = Path('data') / 'subdir' / 'file.txt'
# Windows: data\subdir\file.txt
# Unix: data/subdir/file.txt

# Current directory
cwd = Path.cwd()

# Home directory
home = Path.home()

# Absolute path
abs_path = path.resolve()

# Create parent directories
path.parent.mkdir(parents=True, exist_ok=True)
```

### Listing Directory Contents

```python
from pathlib import Path

directory = Path('.')

# List all items
for item in directory.iterdir():
    print(item)

# List files only
for file in directory.glob('*.txt'):
    print(file)

# Recursive search
for file in directory.rglob('*.py'):
    print(file)
```

### os.path (Legacy)

Still common in older code:

```python
import os

# Join paths
path = os.path.join('data', 'file.txt')

# Check existence
os.path.exists(path)
os.path.isfile(path)
os.path.isdir(path)

# Get parts
os.path.basename(path)  # 'file.txt'
os.path.dirname(path)   # 'data'
os.path.splitext(path)  # ('data/file', '.txt')

# Absolute path
os.path.abspath(path)
```

## File Formats

### Text Files

```python
# Simple text
with open('notes.txt', 'w') as f:
    f.write('This is a text file.\n')
    f.write('It contains plain text.\n')

with open('notes.txt', 'r') as f:
    content = f.read()
```

### CSV Files

```python
import csv

# Writing CSV
with open('data.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['Name', 'Age', 'City'])
    writer.writerow(['Alice', 30, 'NYC'])
    writer.writerow(['Bob', 25, 'LA'])

# Reading CSV
with open('data.csv', 'r') as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)  # ['Name', 'Age', 'City'], etc.

# Dictionary reader/writer
with open('data.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['Name'], row['Age'])

with open('output.csv', 'w', newline='') as f:
    fieldnames = ['Name', 'Age', 'City']
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerow({'Name': 'Alice', 'Age': 30, 'City': 'NYC'})
```

### JSON Files

```python
import json

# Writing JSON
data = {
    'name': 'Alice',
    'age': 30,
    'hobbies': ['reading', 'hiking']
}

with open('data.json', 'w') as f:
    json.dump(data, f, indent=2)

# Reading JSON
with open('data.json', 'r') as f:
    data = json.load(f)
    print(data['name'])  # Alice

# JSON to/from strings
json_string = json.dumps(data)
data = json.loads(json_string)
```

### Configuration Files

**INI format:**

```python
import configparser

# Writing
config = configparser.ConfigParser()
config['DEFAULT'] = {'ServerAlive': 'yes'}
config['Database'] = {'Host': 'localhost', 'Port': '5432'}

with open('config.ini', 'w') as f:
    config.write(f)

# Reading
config = configparser.ConfigParser()
config.read('config.ini')
print(config['Database']['Host'])  # localhost
```

## File System Operations

### Creating Directories

```python
from pathlib import Path

# Create directory
Path('new_dir').mkdir()

# Create with parents (like mkdir -p)
Path('parent/child/grandchild').mkdir(parents=True, exist_ok=True)

# Using os
import os
os.mkdir('new_dir')
os.makedirs('parent/child/grandchild', exist_ok=True)
```

### Deleting Files and Directories

```python
from pathlib import Path
import shutil

# Delete file
Path('file.txt').unlink()

# Delete empty directory
Path('empty_dir').rmdir()

# Delete directory and contents
shutil.rmtree('dir_with_contents')

# Using os
import os
os.remove('file.txt')
os.rmdir('empty_dir')
```

### Copying and Moving

```python
import shutil

# Copy file
shutil.copy('source.txt', 'dest.txt')
shutil.copy2('source.txt', 'dest.txt')  # Preserves metadata

# Copy directory
shutil.copytree('source_dir', 'dest_dir')

# Move (rename)
shutil.move('old_name.txt', 'new_name.txt')
shutil.move('file.txt', 'new_directory/')
```

### Renaming

```python
from pathlib import Path

# Using Path
Path('old.txt').rename('new.txt')

# Using os
import os
os.rename('old.txt', 'new.txt')
```

### Getting File Information

```python
from pathlib import Path
import os

path = Path('file.txt')

# Size in bytes
size = path.stat().st_size

# Modification time
mtime = path.stat().st_mtime

# Using os
size = os.path.getsize('file.txt')
mtime = os.path.getmtime('file.txt')

# Convert timestamp to datetime
from datetime import datetime
mod_time = datetime.fromtimestamp(mtime)
```

### Temporary Files

```python
import tempfile

# Temporary file
with tempfile.TemporaryFile(mode='w+') as f:
    f.write('temporary data')
    f.seek(0)
    print(f.read())
# File deleted when closed

# Named temporary file
with tempfile.NamedTemporaryFile(mode='w', delete=False) as f:
    f.write('data')
    temp_name = f.name

# File persists, but can be deleted manually
os.remove(temp_name)

# Temporary directory
with tempfile.TemporaryDirectory() as tmpdir:
    print(f"Created {tmpdir}")
    # Use tmpdir...
# Directory and contents deleted
```

## Summary

File operations in Python:

**Reading files:**

- `open('file.txt', 'r')` and `.read()`, `.readline()`, `.readlines()`
- Iterate over file object for line-by-line reading

**Writing files:**

- `open('file.txt', 'w')` for write, `'a'` for append
- `.write()` and `.writelines()`

**Best practices:**

- Always use `with` statement for automatic cleanup
- Use `pathlib.Path` for path operations
- Choose appropriate file mode

**File formats:**

- CSV with `csv` module
- JSON with `json` module
- Configuration with `configparser`

**File system:**

- `pathlib.Path` for creating, deleting, moving files/directories
- `shutil` for copying and moving
- `tempfile` for temporary files

Proper file handling is essential for data persistence, configuration, and program interaction with the file system.

## Next Steps

- Handle file errors with [Error Handling](error-handling.md)
- Organize code with [Modules and Imports](modules-and-imports.md)
- Explore advanced I/O in [Core Concepts](../core_concepts/)
