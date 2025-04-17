# Compressing SQL Reference

Use Sonnect 3.7 Thinking model with the following prompt:

```
Your job is to read #file:sql-dialect.md file that contains DuckDB syntax, that is available on top of standard Posgress SQL flavor.

Read the text, understand it, find the key information that is needed for LLM to understand and use this syntax. Extract this key information and write it to the #file:sql-dialect-compressed.md file.

- a file must be as small as possible to save tokens
- a file must contain information about all features, do not drop imporant information
```