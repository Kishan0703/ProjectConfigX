
# ConfigX Learning Resources

Essential external resources for understanding the technologies powering ConfigX.

### 1. Lark (Parsing Engine)
ConfigX uses **Lark** to parse the `ConfigXQL` query language.
*   **Documentation**: [Lark ReadTheDocs](https://lark-parser.readthedocs.io/en/latest/)
*   **Playground**: [Lark Grammar Playground](https://www.lark-parser.org/ide/) (Great for testing grammar changes!)
*   **Cheatsheet**: [Lark Cheatsheet](https://github.com/lark-parser/lark/blob/master/docs/json_tutorial.md)
*   **Our Grammar File**: [`configx/qlang/configxql.lark`](../configx/qlang/configxql.lark)

### 2. Python Struct (Binary Persistence)
`SnapshotStore` uses Python's built-in `struct` module for packing binary data.
*   **Docs**: [Python struct module](https://docs.python.org/3/library/struct.html)
*   **Format Strings**: We use Big-Endian (`>`) and standard type codes (`I` for int, `d` for double, etc.).

---

