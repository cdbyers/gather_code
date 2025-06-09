# gather_code

A minimal Python utility to assemble source files into a single organized document.

## Features

-  **File tree visualization** - Shows your project structure
-  **Code collection** - Gathers all code files into one document
-  **Filtering** - Ignores common build/cache directories and sensitive files
-  **Fast & lightweight** - Single Python file using only standard library

## Included File Types
- **Scripts & Code:** Python, R, JavaScript/TypeScript, Shell scripts, Swift, Kotlin, Java, Dart, C#, C/C++
- **Web & Frontend:** HTML, CSS/SCSS/SASS, Vue, Svelte, React (JSX/TSX)
- **Mobile & Desktop:** Swift, Kotlin, Java, Dart, C#, C/C++
- **Configuration:** JSON, YAML, TOML, XML, INI, environment files
- **Database:** SQL, GraphQL queries
- **Documentation:** Markdown, reStructuredText, LaTeX
- **Build & Project:** Dockerfiles, Makefiles, Gradle, Xcode project files
- **Other Languages:** Go, Rust, Ruby, PHP, Lua, R, Scala, Clojure, Haskell

*Customize inclusions and exclusions by editing the sets at the top of `gather_code.py`*

## Usage

Run in your project's directory:

```bash
python gather_code.py
```

**To use from anywhere:** Run these commands in this directory to create an alias:

```bash
echo "alias gather_code='python3 $(pwd)/gather_code.py'" >> ~/.bashrc
source ~/.bashrc
```
After setting up the alias, you can run gather_code in any project directory.

## Output

Creates `project_[date].txt` containing your complete project structure and all source code files, like this:

```
================================================================================
PROJECT: gather_code
================================================================================

└── .
    ├── code_docs.txt
    ├── gather_code.py
    ├── LICENSE
    └── README.md

================================================================================
FILES
================================================================================


################################################################################
# README.md
################################################################################

# gather_code

[recursion!]


################################################################################
# gather_code.py
################################################################################

import os

IGNORE = {
    # Version control & build
    ".git", ".svn", ".hg", "__pycache__", "node_modules", "venv", "env", ".venv",
    "dist", "build", ".next", ".nuxt", "target", "bin", "obj",
    
    # IDE & editors
    ".vscode", ".idea", "xcuserdata", ".vs", "*.swp", "*.swo", ".DS_Store",
    
    # Secrets & credentials
    ".env", ".envrc", ".secrets", ".aws", ".ssh", "id_rsa", "id_ed25519",
    "*.key", "*.pem", "*.p12", "*.pfx", "*.jks", "*.keystore", 
    "credentials.json", "secrets.json", "config.json", ".netrc",
    "api_keys.txt", "passwords.txt", ".password", "auth.json",
    
    # Certificates
    "*.crt", "*.cer", "*.ca-bundle", "*.p7b", "*.p7c", "*.der",
    
    # Database files
    "*.db", "*.sqlite", "*.sqlite3", "*.mdb",
    
    # Logs & temp
    "*.log", "logs", "tmp", "temp", ".tmp", "*.pid", "*.lock"
}
EXTENSIONS = {
    # Scripts & Code
    ".py", ".r", ".js", ".ts", ".jsx", ".tsx", ".sh", ".bash", ".zsh", ".fish",
    # Web
    ".html", ".css", ".scss", ".sass", ".less", ".vue", ".svelte",
    # Mobile & Desktop
    ".swift", ".kt", ".java", ".dart", ".cs", ".cpp", ".c", ".h", ".hpp",
    # Config & Data
    ".json", ".yaml", ".yml", ".toml", ".xml", ".ini", ".cfg", ".conf",
    # Database & Query
    ".sql", ".graphql", ".gql",
    # Documentation & Markup
    ".md", ".rst", ".tex",
    # Build & Project
    ".dockerfile", ".makefile", ".cmake", ".gradle", ".pbxproj", ".entitlements",
    # Functional & Other
    ".go", ".rs", ".rb", ".php", ".lua", ".r", ".scala", ".clj", ".hs"
}
SPECIAL = {"pyproject.toml"}


def should_include_file(file_path):
    """Check if file should be included."""
    filename = os.path.basename(file_path)
    _, ext = os.path.splitext(filename)
    path_parts = file_path.split(os.sep)
    
    return (ext in EXTENSIONS or filename in SPECIAL) and \
           not any(part in IGNORE for part in path_parts)


def read_file_safe(file_path):
    """Safely read file content."""
    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            return f.read()
    except Exception:
        return f"Error reading {file_path}"


def gather_files(root="."):
    """Collect all relevant files and their contents."""
    files = {}
    for root_dir, dirs, filenames in os.walk(root):
        # Remove ignored directories to prevent traversal
        dirs[:] = [d for d in dirs if d not in IGNORE]
        
        for filename in filenames:
            file_path = os.path.join(root_dir, filename)
            if should_include_file(file_path):
                rel_path = os.path.relpath(file_path, root)
                files[rel_path] = read_file_safe(file_path)
    return files


def format_tree(path=".", prefix="", is_last=True):
    """Generate file tree structure."""
    if os.path.basename(path) in IGNORE:
        return ""
    
    connector = "└── " if is_last else "├── "
    result = f"{prefix}{connector}{os.path.basename(path)}\n"
    
    if os.path.isdir(path):
        try:
            entries = [os.path.join(path, name) for name in os.listdir(path)]
            entries = [e for e in entries if os.path.basename(e) not in IGNORE]
            entries.sort(key=lambda x: (not os.path.isdir(x), os.path.basename(x).lower()))
            
            for i, entry in enumerate(entries):
                next_prefix = prefix + ("    " if is_last else "│   ")
                result += format_tree(entry, next_prefix, i == len(entries) - 1)
        except PermissionError:
            pass
    
    return result


def main():
    """Generate assembled code documentation."""
    project_name = os.path.basename(os.getcwd())
    files = gather_files()
    
    # Build content sections
    header = ["=" * 80, f"PROJECT: {project_name}", "=" * 80, ""]
    tree_section = [format_tree().strip(), ""]
    files_header = ["=" * 80, "FILES", "=" * 80, ""]
    
    content = header + tree_section + files_header
    
    # Add file contents
    for file_path in sorted(files.keys()):
        content.extend([
            f"\n{'#' * 80}",
            f"# {file_path}",
            f"{'#' * 80}\n",
            files[file_path]
        ])
    
    # Write output
    with open("assembled_code.txt", "w", encoding="utf-8") as f:
        f.write("\n".join(content))
    
    print("Documentation written to assembled_code.txt")


if __name__ == "__main__":
    main()
```


