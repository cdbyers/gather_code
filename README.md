# gather_code

A minimal Python utility to assemble all source files into a single organized document.

## Features

- 🌳 **File tree visualization** - Shows your project structure
-  **Source code collection** - Gathers all code files into one document
-  **Smart filtering** - Ignores common build/cache directories and sensitive files
-  **Fast & lightweight** - Pure Python with minimal dependencies

## Default File Types

- Python (`.py`)
- Web files (`.html`, `.css`, `.js`)
- Shell scripts (`.sh`)
- Swift (`.swift`)
- Xcode project files (`.pbxproj`, `.entitlements`)
- Configuration files (`pyproject.toml`)

You can add or change these by altering the sets at the top of gather_code.py

## Usage

```bash
# Generate documentation for current directory
python gather_code.py
```

## Output

Creates a `code_docs.txt` like this: 

```
================================================================================
FILE TREE STRUCTURE FOR: gather_code
================================================================================

└── 
    ├── LICENSE
    ├── README.md
    └── gather_code.py

================================================================================
FILE CONTENTS
================================================================================

################################################################################
# FILE: gather_code.py
################################################################################

import os
from pathlib import Path

IGNORE_DIRS = {".git", "__pycache__", "venv", "node_modules", "xcuserdata"}
CODE_EXTENSIONS = {".py", ".html", ".sh", ".css", ".js", ".swift", ".pbxproj", ".entitlements"}
SPECIAL_FILES = {"pyproject.toml"}


def should_include_file(file_path):
    """Check if file should be included based on extension or name."""
    return file_path.suffix in CODE_EXTENSIONS or file_path.name in SPECIAL_FILES


def read_file_safe(file_path):
    """Safely read file content."""
    try:
        return file_path.read_text(encoding="utf-8")
    except Exception as e:
        return f"Error reading file: {e}"


def gather_files(root_path="."):
    """Gather all code files and their contents."""
    root = Path(root_path)
    files = {}
    
    for path in root.rglob("*"):
        if (path.is_file() and 
            should_include_file(path) and 
            not any(part in IGNORE_DIRS for part in path.parts)):
            files[str(path.relative_to(root))] = read_file_safe(path)
    
    return files


def format_tree(path=Path("."), prefix="", is_last=True):
    """Generate tree structure as string."""
    if any(part in IGNORE_DIRS for part in path.parts):
        return ""
    
    connector = "└── " if is_last else "├── "
    tree = f"{prefix}{connector}{path.name}\n"
    
    if path.is_dir():
        try:
            children = sorted([p for p in path.iterdir() 
                             if not any(part in IGNORE_DIRS for part in p.parts)])
            for i, child in enumerate(children):
                extension = "    " if is_last else "│   "
                tree += format_tree(child, prefix + extension, i == len(children) - 1)
        except PermissionError:
            pass
    
    return tree


def main():
    """Generate documentation file."""
    project_name = Path.cwd().name
    files = gather_files()
    
    content = [
        "=" * 80,
        f"FILE TREE STRUCTURE FOR: {project_name}",
        "=" * 80,
        "",
        format_tree().strip(),
        "",
        "=" * 80,
        "FILE CONTENTS", 
        "=" * 80,
        ""
    ]
    
    for file_path in sorted(files.keys()):
        content.extend([
            "#" * 80,
            f"# FILE: {file_path}",
            "#" * 80,
            "",
            files[file_path],
            ""
        ])
    
    Path("code_docs.txt").write_text("\n".join(content), encoding="utf-8")
    print("Documentation written to code_docs.txt")


if __name__ == "__main__":
    main()
```


