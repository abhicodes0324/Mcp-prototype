# MCP Prototype — Multi-Language Parser for MetaCall

A cross-platform C/C++ tool and library for parsing multi-language projects using [Tree Sitter](https://tree-sitter.github.io/tree-sitter/). Extracts functions, classes, and builds dependency trees without runtime execution.

**GSoC Project:** Implement Multi-Language Parser

## Features

- **Static analysis** — Parse source files without executing code
- **Multi-language** — Python and JavaScript/TypeScript (extensible)
- **Symbol extraction** — Functions, classes, and methods
- **Import/require detection** — Build dependency graphs
- **C API** — Embeddable in other projects
- **CLI** — Command-line interface for quick analysis

## Build

### Prerequisites

- CMake 3.14+
- C11 compiler (GCC, Clang, MSVC)
- Git (for fetching dependencies)

### Build Steps

```bash
# 1. Fetch dependencies (requires curl, run once)
./scripts/fetch-deps.sh

# 2. Build (from project root)
mkdir build && cd build
cmake ..
cmake --build .
```

### Install

Run from the project root:

```bash
# Option A: Install to /usr/local (requires sudo)
sudo cmake --build build --target install

# Option B: Install to user directory (no sudo)
cmake -B build -DCMAKE_INSTALL_PREFIX=$HOME/.local
cmake --build build --target install
# Add to PATH if needed: export PATH="$HOME/.local/bin:$PATH"
```

After install, the `mcp` binary is available in your PATH. To run without installing, use `./build/mcp` from the project root (paths are relative to your current directory).

## Usage

### CLI

```bash
# From project root (after install, or use ./build/mcp without install)
mcp parse tests/sample.py

# Parse and show human-readable output
mcp parse tests/sample.py --format text

# List functions and classes
mcp list-functions tests/sample.js

# Build dependency graph for a directory
mcp deps tests/
```

When running from the `build/` directory, use relative paths like `../tests/sample.py`.

### C API Example

```c
#include "mcp_parser.h"

int main(void) {
    mcp_parser *parser = mcp_parser_create();
    mcp_result *result = mcp_parser_parse_file(parser, "script.py", NULL, 0);

    if (result) {
        const mcp_file_result *fr = mcp_result_get_file(result);
        for (size_t i = 0; i < fr->symbol_count; i++) {
            printf("%s: %s (line %u)\n",
                   fr->symbols[i].type == MCP_SYMBOL_FUNCTION ? "function" : "class",
                   fr->symbols[i].name, fr->symbols[i].line);
        }
        mcp_result_free(result);
    }

    mcp_parser_destroy(parser);
    return 0;
}
```

## Output Format

### Parse Result (JSON)

```json
{
  "file": "sample.py",
  "language": "python",
  "functions": [
    {"name": "greet", "line": 6},
    {"name": "add", "line": 10}
  ],
  "classes": [
    {"name": "Calculator", "line": 13},
    {"name": "DataProcessor", "line": 20}
  ],
  "imports": [
    {"module": "os"},
    {"module": "sys"},
    {"module": "utils"}
  ]
}
```

### Dependency Graph (JSON)

```json
{
  "nodes": [
    {"id": "/path/to/main.py", "path": "...", "symbols": [...]},
    {"id": "/path/to/utils.py", "path": "...", "symbols": [...]}
  ],
  "edges": [
    {"from": "/path/to/main.py", "to": "/path/to/utils.py"}
  ]
}
```

## Project Structure

```
mcp-prototype/
├── include/
│   └── mcp_parser.h      # Public C API
├── src/
│   ├── parser.c          # Core parsing logic
│   ├── api.c             # JSON export helpers
│   ├── dependency_builder.c
│   └── extractors/
│       ├── python_extractor.c
│       └── js_extractor.c
├── cli/
│   └── main.c
├── tests/
│   ├── sample.py
│   ├── sample.js
│   ├── utils.py
│   └── utils.js
└── CMakeLists.txt
```

## Extending to New Languages

1. Add Tree Sitter grammar as FetchContent dependency
2. Create `src/extractors/<lang>_extractor.c` implementing extraction logic
3. Register in `mcp_lang_from_path()` and `parser.c`
4. Add query/traversal for function, class, and import nodes

## Use Cases

- **Intellisense** — Feed parsed symbols to IDE plugins (e.g., VS Code)
- **Function Mesh** — Break multi-language projects into distributable subparts
- **Documentation** — Generate API docs from static analysis
- **Refactoring** — Understand cross-file dependencies

## License

Apache 2.0

## Resources

- [Tree Sitter Documentation](https://tree-sitter.github.io/tree-sitter/)
- [Tree Sitter C API](https://github.com/tree-sitter/tree-sitter/blob/master/lib/include/tree_sitter/api.h)
- [Tree Sitter Tutorial](https://dev.to/shrsv/making-sense-of-tree-sitters-c-api-2318)
