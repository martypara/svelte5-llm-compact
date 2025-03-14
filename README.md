# Svelte 5 Docs Minified for LLM Context

This tiny project compresses the Svelte 5 and SvelteKit documentation into a single dense text file optimized for LLM input, reducing token count to ~39,000 (~167,000 characters) from the official full LLM version’s ~222,630 tokens (~874,000 characters) (https://svelte.dev/llms-full.txt) and official small LLM version’s ~130,000 tokens (~528,500 characters) (https://svelte.dev/llms-small.txt). The goal is to provide a compact, LLM-friendly reference for Svelte 5, addressing issues like oversized docs with redundant JS/TS code variants, human-oriented fluff, and poor LLM knowledge of Svelte 5 (often mixing Svelte 3/4 or outdated beta data). The minified version aims to preserve most technical details, parameters, and edge cases but sacrifices full code examples, some nuances, readability and maybe even some edge cases. Excluded sections include SvelteKit’s Build and Deploy, Legacy APIs, and Errors/Warnings (see https://svelte.dev/docs/svelte/compiler-errors, https://svelte.dev/docs/svelte/compiler-warnings, https://svelte.dev/docs/svelte/runtime-errors, https://svelte.dev/docs/svelte/runtime-warnings).

This was made for my personal use but shared for anyone needing a lightweight Svelte 5 reference for LLMs.

## Stats Comparison
- **Minified Version**: ~39,000 tokens, ~167,000 characters
- **Official Small LLM Version**: ~130,000 tokens, ~528,500 characters
- **Official Full LLM Version**: ~222,630 tokens, ~874,000 characters

Loses some edge cases and nuances!

There is more potential to reduce and maybe more room to add important context back in.
