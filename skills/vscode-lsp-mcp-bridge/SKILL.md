---
name: vscode-lsp-mcp-bridge
description: For source-code tasks only, use semantic VS Code language intelligence before text search.
applyTo: '**'
sources: https://github.com/AlwaysHC/vscode-lsp-mcp-bridge/tree/main
---

# VS Code LSP MCP Bridge

- This section applies only when the current user request requires understanding, navigating, diagnosing, reviewing, or editing source code. Ignore it for non-code tasks and do not call `vscode_lsp` tools in those tasks.
- For an applicable code task, call `semantic_navigation_guide` only when unsure which `vscode_lsp` tool applies; it is not a required first action.
- When a concrete declared symbol name or source position is known, use the matching `vscode_lsp` semantic tool before `rg`, `grep`, shell search, or raw file search for symbols, methods, definitions, declarations, implementations, references, callers, callees, hover, diagnostics, rename, code actions, and formatting.
- If a symbol name or method signature is provided, locate it with `workspace_symbols`, `document_symbols`, or a task-shaped named-symbol tool instead of text search.
- `workspace_symbols` searches declared symbol names, not concepts or source text. If only descriptive terms, literals, comments, markup, or serialized field names are known, use targeted text search directly.
- An empty `workspace_symbols` result is a query miss; it does not by itself show that a language provider failed or is not indexing the project.
- `document_symbols` results preserve provider hierarchies. Inspect every top-level result. Recursively inspect `children`; in C#, a namespace is commonly the root wrapper, and recovered same-file symbols may appear beside it.
- If a namespace root has no children, try `workspace_symbols` or a named-symbol tool before treating semantic lookup as failed.
- A workspace-routing ambiguity or a path resolved in another workspace is not a language-provider failure. Retry the semantic tool with an absolute path or a workspace-unique exact relative file before falling back or announcing a fallback.
- If a named-symbol tool reports an ambiguous resolution, retry with `file`, `containerName`, or `kind`; do not use its first candidate as an implicit selection.
- Fall back to text search only when the bridge is unavailable or the language provider fails, and state that fallback explicitly.
- After semantic lookup identifies a target, reading its source is normal when implementation text is required; do not describe that source inspection as an LSP fallback.
