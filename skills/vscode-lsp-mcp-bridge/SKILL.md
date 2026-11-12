---
name: vscode-lsp-mcp-bridge
description: >-
  Use when understanding, navigating, diagnosing, reviewing, or editing source
  code with VS Code LSP MCP Bridge (vscode_lsp), including symbol lookup,
  definitions, references, callers, callees, types, diagnostics, rename,
  code actions, and formatting. Do not use for non-code tasks such as ordinary
  document editing.
metadata:
  source: https://github.com/AlwaysHC/vscode-lsp-mcp-bridge/tree/main
---

# VS Code LSP MCP Bridge

Use `vscode_lsp` for semantic evidence from the language providers in the open VS Code workspace. Feature availability depends on the installed language extension and project state.

## Scope and search choice

- Apply this skill only when the current request requires understanding, navigating, diagnosing, reviewing, or editing source code. Do not call any `vscode_lsp` tool for non-code tasks.
- For a concrete declared symbol, method signature, or known source position, use the matching semantic tool before `rg`, `grep`, shell search, or raw file search for that symbol or its relationships.
- For descriptive concepts, literals, comments, markup, or serialized field names, use targeted text search directly. `workspace_symbols` searches declared names, not arbitrary source text.
- Call `semantic_navigation_guide` only when unsure which semantic tool fits. It is not a mandatory first action.

## Tool selection

Names below omit the MCP server prefix. Follow the schemas available in the current session.

| Task or known input                                                       | Preferred tool                                                                           |
| ------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Who calls a named symbol?                                                 | `find_callers_for_symbol`                                                                |
| What does a named symbol call?                                            | `find_callees_for_symbol`                                                                |
| Definition by symbol name                                                 | `find_definition_for_symbol`                                                             |
| References by symbol name                                                 | `find_references_for_symbol`                                                             |
| Combined context by symbol name                                           | `symbol_context_for_symbol`                                                              |
| Declared-name lookup or candidate discovery                               | `workspace_symbols`                                                                      |
| Structure of a known file                                                 | `document_symbols`                                                                       |
| Definition, declaration, implementation, or type definition at a position | `go_to_definition`, `go_to_declaration`, `go_to_implementation`, `go_to_type_definition` |
| References, type/documentation, or combined context at a position         | `find_references`, `hover`, `symbol_context`                                             |
| Call graph or type hierarchy by name                                      | `call_hierarchy_for_symbol`, `type_hierarchy_for_symbol`                                 |
| Call graph or type hierarchy at a position                                | `call_hierarchy`, `type_hierarchy`                                                       |
| Language errors and warnings                                              | `diagnostics`                                                                            |
| Rename eligibility and preview                                            | `prepare_rename`, `preview_rename`                                                       |
| Code actions or source actions                                            | `code_actions`, `organize_imports`, `fix_all`                                            |
| Formatting                                                                | `format_document`, `format_range`, `format_on_type`                                      |

When only the symbol name is known, prefer a task-shaped named-symbol tool. Use `workspace_symbols` or `document_symbols` to locate or disambiguate declarations when needed. For a known position, pass `file`, `line`, and `column`; editor coordinates are **1-based**.

## Caveats and ambiguous results

- An empty `workspace_symbols` result is a query miss, not evidence of provider failure or missing indexing. Refine the declared name, inspect `document_symbols` for a known file, or try an appropriate named-symbol tool.
- Inspect **every top-level result** from `document_symbols` and recursively inspect `children`. In C#, a namespace commonly wraps types and members; recovered same-file symbols may appear beside that root.
- If a namespace has no children, inspect sibling roots and try `workspace_symbols` or a named-symbol tool before treating semantic lookup as failed.
- If a named-symbol tool returns `resolutionStatus: ambiguous`, retry with `file`, `containerName`, or `kind`. Do not implicitly select its first candidate.
- A routing ambiguity or a path resolved in another workspace is not a language-provider failure. Retry the semantic tool with an absolute path or an exact relative path unique among open workspaces before announcing or using a fallback.
- If refined semantic queries remain empty, report what those queries did not find. Do not repeat them without new information or claim that the symbol cannot exist elsewhere in the project.

## Source inspection and fallback

After semantic lookup identifies the target, read the relevant source when implementation text is needed. This is normal source inspection, not an LSP fallback.

For a semantic question, fall back to text search only when the bridge is unavailable or the language provider fails or explicitly does not support the requested operation. Evidence includes an unavailable MCP tool, a connection error, or an explicit provider error or unsupported-operation response. State the fallback and its reason. An empty query result, an invalid request, or an ambiguous workspace path does not establish provider failure.

Successful semantic navigation does not need a redundant `rg` cross-check. If the user separately requests textual occurrences, perform that as a text-search task.

## Changes and result handling

- Use `prepare_rename` and `preview_rename` to inspect a rename. `rename_symbol` refuses `apply: true`; perform the full semantic rename in VS Code. Do not apply only the visible subset of a generic `WorkspaceEdit`, which can omit hidden operations.
- Applying supported formatting or completions requires `vscodeLspMcpBridge.enableWriteTools` and per-operation VS Code approval. Executing provider commands through code actions requires explicit `executeCommand: true` and a separate approval. Respect those bridge requirements when carrying out an authorized edit.
- Report concrete paths as `path:line`. Distinguish definitions, references, and call sites. Disclose truncated results or graph depth limits; a partial graph does not prove that other relationships are absent.
- After edits, run the project checks appropriate to the change.

## Example

Request: “Who calls `SaveAsync`?” Only the method name is known.

1. Call `find_callers_for_symbol` with:

   ```json
   { "query": "SaveAsync" }
   ```

2. If resolution is ambiguous, retry with an appropriate `file`, `containerName`, or `kind`.
3. Inspect the returned call sites and read their source if context is needed. Report file/line locations and any result limits.

## Sources

- Local policy basis: [draft AGENTS.md section](SKILL.draft.md).
- Tool contracts and limitations: [upstream tool catalog](https://github.com/AlwaysHC/vscode-lsp-mcp-bridge/blob/main/docs/TOOLS.md).
- Workspace and language-provider requirements: [upstream README](https://github.com/AlwaysHC/vscode-lsp-mcp-bridge).
