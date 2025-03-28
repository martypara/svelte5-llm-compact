## Svelte 5 for LLMs

Getting LLMs to write Svelte 5 is hard and it will stay that way until Svelte 5 code shows up in actual model training data. Until then, we’re stuck with partial understanding and workarounds.

Dumping full documentation doesn’t help. It’s too big, too noisy, and wastes tokens. Most models already understand Svelte 3/4 and since Svelte 5 doesn't change everything, only the new pieces need to be shown.

Even the official [`llms-small.txt`](https://svelte.dev/llms-small.txt) is over 130k tokens. That’s way beyond what models can reason over in a single prompt. The ideal context size is under 8k tokens. Pasting in too much usually makes things worse.

This repo is a minimal, example-based reference focused on what changed in Svelte 5. It's written to be copy-pasted into prompts. No extra explanation, just clean annotated code.

### What's in here

Each file is standalone and scoped:

- [svelte5_runes.txt](./svelte5_runes.txt) — `$state`, `$effect`, `$derived`, `$bindable`, `$props`, `$host`, `$inspect`
- [svelte5_events.txt](./svelte5_events.txt) — event handling, callback props, modifiers, bubbling
- [svelte5_snippets.txt](./svelte5_snippets.txt) — Svelte 5 snippets (`{#snippet}`, `{@render}`), slot replacement
- [svelte5_typescript.txt](./svelte5_typescript.txt) — typing props, components, DOM attributes
- [svelte_4-5_migration_guide.txt](./svelte_4-5_migration_guide.txt) — what changed from Svelte 4 to 5
- [svelte5_full_context.txt](./svelte5_full_context.txt) — everything above in one file (14k tokens, works decently with most SOTA models)

### Usage

Each file is meant to be loaded into context as-is. Stick to one or two files at a time to stay under the token limit. They're designed for fast understanding — short examples, no filler.
