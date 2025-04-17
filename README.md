# Reusable prompts with syntax documentation

## About this repository

This repository contains a demo of how VS Code reusable prompts with LLM-friendly documentation can be used to "teach" copilot a new SQL dialect.

The basic behavior of copilot is to generate Standard SQL queries, because DuckDB SQL dialect is not well known to the models that I tested.

As seen below.

![No Docs](./imgs/no-docs.png)

## Getting documentation

[llmstxt.site](https://llmstxt.site/) contains a collection of links to LLM-friendly documentation. 

I took the DuckDB SQL dialect documentation from [DuckDB Docks](https://duckdb.org/duckdb-docs.md).
However the full reference is too long to be used as a prompt. It's over 50k lines of text.

It needs to be cleaned up, I copied only the SQL reference, that is stored in [docs-processing/sql-dialect.md](docs-processing/sql-dialect.md).


# Compressing SQL Reference

SQL reference is still very verbose, contains a lot of examples and unnecessary text. Most of the models fail to understand and use it.

I used Sonnect 3.7 Thinking model with the following prompt:

```
Your job is to read #file:sql-dialect.md file that contains DuckDB syntax, that is available on top of standard PostgreSQL flavor.

Read the text, understand it, find the key information that is needed for LLM to understand and use this syntax. Extract this key information and write it to the #file:sql-dialect-compressed.md file.

- a file must be as small as possible to save tokens
- a file must contain information about all features, do not drop important information
```

The model was able to compress the SQL reference and keep it under 750 lines of text, that is much more manageable.

## Reusable prompts

This repository contains a reusable prompt that can be attached to any DuckDB SQL related request.

Reusable prompts are experimental feature of VS Code. [As documented here](https://code.visualstudio.com/docs/copilot/copilot-customization#_reusable-prompt-files-experimental).


a neat feature of reusable prompts is that they reference external files. In our case it's a SQL reference doc.

```
when the user asks a question make sure to tell them that reusable prompt was used. Add this info in the beginning of the answer.
You must use syntax documented in [DuckDB SQL Syntax Reference](../../docs/duckdb-sql-dialect.md). If you cannot find these instructions let the user know.
```

I added notes and the warning for debugging purposes, they are not needed for the prompt to work, but provide a clear context for the user.

Prompts can be attached to any conversation as a context

![Reusable prompt](./imgs/attaching-prompt.png)

## Results

Even GPT-4o, Basic model can now generate DuckDB SQL queries.

![DuckDB SQL](./imgs/result.png)
