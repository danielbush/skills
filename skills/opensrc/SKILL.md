---
name: opensrc
description: "Fetch and reference dependency source code using opensrc. Use when you need to understand how a package works internally, read its implementation, or fetch source for npm, PyPI, crates.io packages or GitHub repos. Trigger on phrases like 'how does X work internally', 'fetch the source for X', 'add X using opensrc', or when deeper understanding of a dependency is needed beyond types/interfaces."
---

# opensrc

Fetch and reference dependency source code for deeper understanding of implementation details.

## Usage

Source code is stored in `opensrc/` with an index at `opensrc/sources.json`.

```bash
npx opensrc <package>           # npm package (e.g., npx opensrc zod)
npx opensrc pypi:<package>      # Python package (e.g., npx opensrc pypi:requests)
npx opensrc crates:<package>    # Rust crate (e.g., npx opensrc crates:serde)
npx opensrc <owner>/<repo>      # GitHub repo (e.g., npx opensrc vercel/ai)
```

## When to use

- When you need to understand how a package works internally, not just its types/interface
- When debugging behaviour that isn't explained by documentation
- When the user asks to fetch or add source for a dependency

## Checking available sources

Read `opensrc/sources.json` to see what's already fetched before running `npx opensrc` again.
