
# ConfigX: The Complete Guide

This document is the definitive reference for ConfigX, combining architecture, rules, and syntax.

---

## 1. Architecture Overview

ConfigX is a **strict hierarchical data engine**. It manages parameters using a tree of Nodes.

### Project Structure
```text
configx/
│
├── core/                 # Core Data Model & Logic
│   ├── __init__.py
│   ├── node.py           # Node class (Atomic unit)
│   ├── tree.py           # ConfigTree (CRUD engine)
│   └── errors.py         # Custom error definitions
│
├── storage/              # Persistence Layer (Implemented)
│   ├── __init__.py
│   ├── snapshot.py       # Binary Snapshot Store (.cx)
│   ├── wal.py            # Write-Ahead Log (.wal)
│   └── runtime.py        # Storage Runtime controller
│
├── qlang/                # ConfigXQL Language Engine (Implemented)
│   ├── __init__.py
│   ├── configxql.lark    # Grammar definition file
│   ├── parser.py         # Lark Paser & Transformer
│   └── interpreter.py    # AST Interpreter
│
├── runtime/              # Public API Facade
│   ├── __init__.py
│   └── configx.py        # Main ConfigX class
│
├── tests/                # Test Suite
│   ├── test_tree.py
│   └── ...
│
├── configx.toml          # Project config
├── pyproject.toml        # Dependencies & Build
├── README.md
└── __init__.py
```

### Testing Guide

We use **pytest** for our test suite.
*   **Run all tests**: `pytest tests/`
*   **Run specific test**: `pytest tests/test_runtime_persistence.py`


### Future / Planned Modules
*   `configx/cli/`: Command Line Interface (Phase 4)
*   `configx/server/`: HTTP API Server (Phase 5)
*   `configx/ai/`: AI Interpreter & Agents (Phase 6)

### Core Components
*   **Node**: The atomic unit. Can be a **Leaf** (value) or **Interior** (children).
*   **ConfigTree**: The manager. Handles path navigation (`app.ui.theme`), CRUD operations, and rules.
*   **Storage**: Handles persistence via **Snapshots** (.cx) and **Write-Ahead Logs** (WAL).
*   **QLang**: The query language (`ConfigXQL`) for interacting with the tree.

---

## 2. Core Rules (Strict Tree Model)

### Rule 1: Leaf vs Interior
A node is **either** a leaf (value) or interior (children). **Never both.**
*   *Yes*: `app.theme = "dark"`
*   *Yes*: `app.ui.color = "red"`
*   *Illegal*: `app.theme = "dark"` AND `app.theme.opacity = 0.5` (Conflict!)

### Rule 2: Validation
*   Setting a value on an existing interior node raises `ConfigNodeStructureError`.
*   Strict Mode: Setting a deep path without existing parents raises `ConfigStrictModeError`.

---

## 3. ConfigXQL Syntax Guide

The syntax for `resolve(query)` is expressive and powerful.

### Basic Operations (CRUD)
| Operation | Syntax Example | Description |
| :--- | :--- | :--- |
| **Branch Creation** | `appSettings` | Creates `appSettings` dict/branch |
| **Nested Branch** | `appSettings.uisettings` | Creates nested structure |
| **Set String** | `app.theme=dark` | Sets string value |
| **Set Typed** | `app.port:INT=8080` | Explicitly parsers as int |
| **Set Boolean** | `app.debug:BOOL=true` | Sets boolean value |
| **Safe Get** | `app.theme!` | Returns value or None |
| **Unsafe Get** | `app.theme` | Returns value or Error |
| **Delete Leaf** | `app.theme-` | Deletes the node |
| **Delete Branch** | `app.ui-` | Deletes entire subtree |



### Advanced Features 
**Note** : These features are not yet implemented & the syntax is subject to change. `WIP`

#### Batch Functions
```python
# Create multiple values at once
confx.resolve('ui.*[theme="dark":STR, keys="ctrl+w":STR]')

# Delete multiple values
confx.resolve('ui.*[theme, keys]-')
```

#### Wildcards
```python
# Update all direct children
confx.resolve('[.].userId=0001')

# Update all nested descendants (recursive)
confx.resolve('[..].userId=0001')

# Pattern match directories
confx.resolve('[*].userId=[000*]')
```

#### Inbuilt Functions
```python
# Count items in a branch
confx.resolve('products!count')

# Aggregates
confx.resolve('products!sum="price"')
confx.resolve('products!max("price")')

# Search
confx.resolve('products!search="theme==dark"')
```

#### Logic & Conditions
```python
# If theme is light, set it to dark
confx.resolve('ui.theme=="light":theme="dark"')

# Chain operations with semicolon
confx.resolve('ui.theme=="light":theme="dark";ui.reload')
```

---

## 4. API Reference

### Python API
*   `set(path, value)`: Set value.
*   `get(path)`: Get value.
*   `delete(path)`: Remove node.
*   `to_dict()` / `load_dict()`: Import/Export JSON-compatible dicts.

### Return Types

| Operation | Method | Return Value | Exception on Failure |
| :--- | :--- | :--- | :--- |
| **SET** | `set()` | The assigned value (primitive) | `ConfigInvalidPathError` (if path invalid)<br>`ConfigNodeStructureError` (if resolving to interior node) |
| **GET** | `get()` | The value (primitive) or Dict (if interior) | `ConfigPathNotFoundError` |
| **DELETE** | `delete()` | `True` (if deleted), `False` (if not found) | `ConfigNodeStructureError` (if try to delete root) |
| **RESOLVE** | `resolve()` | Depends on operation (see above) | Depends on operation |

---

### Persistence
*   **WAL**: Operations are logged sequentially.
*   **Snapshot**: Full state dump on close.

### Persistence Rules

*   **WAL (Write-Ahead Log)**: Every `SET` and `DELETE` operation is appended to `wal.cx` *before* it is applied to memory (if persistence is enabled).
*   **Snapshot**: On `close()` or `checkpoint()`, the entire tree is dumped to `snapshot.cx` and the WAL is cleared.
*   **Recovery**: On startup, ConfigX loads `snapshot.cx` first, then replays `wal.cx` to restore the exact state.

