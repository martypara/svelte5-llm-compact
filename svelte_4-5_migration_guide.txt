# Svelte 4 → 5 Migration Summary

**Concise Summary (Svelte 4 → 5)**

- **New Runes API**  
  - Reactive variables use `let count = $state(0);`  
  - Derived values use `const double = $derived(count * 2);`  
  - Side effects use  
```
    $effect(() => {
      if (count > 5) alert('Count is too high!');
    });
```

- **Declaring Props**  
  - All props come from `let {...} = $props();`  
  - Example:
```
    <script>
      let { optional = 'unset', required } = $props();
    </script>
```
  - Renaming or forwarding props:
```
    <script>
      let { class: klass, ...rest } = $props();
    </script>
    <button class={klass} {...rest}>click me</button>
```

    Below is a concise, mostly bullet-pointed summary of the changes, focusing on code snippets:

---

### Event Changes

- **Svelte 4** used `on:` directives.
- **Svelte 5**: Use regular properties like `onclick` instead (no `:`).

**Example:**
```svelte
<script>
  let count = $state(0);
</script>

<button onclick={() => count++}>
  clicks: {count}
</button>
```

**Shorthand Example:**
```svelte
<script>
  let count = $state(0);

  function onclick() {
    count++;
  }
</script>

<button {onclick}>
  clicks: {count}
</button>
```

---

### Component Events

- **Svelte 4**: `createEventDispatcher` for emitting events.
- **Svelte 5**: Use callback props instead.

**Example (App.svelte):**
```svelte
<script lang="ts">
  import Pump from './Pump.svelte';

  let size = $state(15);
  let burst = $state(false);

  function reset() {
    size = 15;
    burst = false;
  }
</script>

<Pump
  inflate={(power) => {
    size += power;
    if (size > 75) burst = true;
  }}
  deflate={(power) => {
    if (size > 0) size -= power;
  }}
/>

{#if burst}
  <button onclick={reset}>new balloon</button>
  <span class="boom">💥</span>
{:else}
  <span class="balloon" style="scale: {0.01 * size}">
    🎈
  </span>
{/if}
```

**Example (Pump.svelte):**
```svelte
<script lang="ts">
  let { inflate, deflate } = $props();
  let power = $state(5);
</script>

<button onclick={() => inflate(power)}>inflate</button>
<button onclick={() => deflate(power)}>deflate</button>

<button onclick={() => power--}>-</button>
Pump power: {power}
<button onclick={() => power++}>+</button>
```

---

### Bubbling Events

- No more `<button on:click>` to forward. 
- Accept a callback prop like `onclick`:

```svelte
<script>
  let { onclick } = $props();
</script>

<button {onclick}>
  click me
</button>
```

- Can “spread” handlers with other props:

```svelte
<script>
  let props = $props();
</script>

<button {...props}>
  click me
</button>
```

---

### Event Modifiers

- **Svelte 4** example: `<button on:click|once|preventDefault={handler}>...`
- **Svelte 5**: No built-in modifiers. Handle `event.preventDefault()` in the function or wrap manually.

**Example Wrappers:**
```svelte
<script>
  function once(fn) {
    return function (event) {
      if (fn) fn.call(this, event);
      fn = null;
    };
  }

  function preventDefault(fn) {
    return function (event) {
      event.preventDefault();
      fn.call(this, event);
    };
  }
</script>

<button onclick={once(preventDefault(handler))}>...</button>
```

- For `capture`, do: `onclickcapture={...}`
- For `passive` or `nonpassive`, use an action to attach the event.

---

### Multiple Event Handlers

- **Svelte 4** allowed `<button on:click={one} on:click={two}>`.
- **Svelte 5**: Only one `onclick` property. Combine them:

```svelte
<button
  onclick={(e) => {
    one(e);
    two(e);
  }}
>
  ...
</button>
```

- When spreading props, place local handlers afterward to avoid overwriting:

```svelte
<button
  {...props}
  onclick={(e) => {
    doStuff(e);
    props.onclick?.(e);
  }}
>
  ...
</button>
```


---
---

### **Svelte 5 Changes - Condensed Bullet Points**

#### **Component Changes**
- **No More Class Components** → Components are now functions.
- **Instantiation** → Use `mount` or `hydrate` instead of `new Component()`.
- **Mount vs. Hydrate** → `hydrate` picks up server-rendered HTML, otherwise identical to `mount`.
- **No `$on`, `$set`, `$destroy`**
  - `$on` → Use `events` property instead (though callbacks are preferred).
  - `$set` → Use `$state` for reactivity.
  - `$destroy` → Use `unmount`.

#### **Code Examples**
```js
import { mount } from 'svelte';
import App from './App.svelte';

const app = mount(App, { 
  target: document.getElementById("app"),
  events: { event: callback } // Replacement for $on
});

const props = $state({ foo: 'bar' }); // Replacement for $set
props.foo = 'baz';

import { unmount } from 'svelte';
unmount(app); // Replacement for $destroy
```

#### **Legacy Compatibility**
- Use `createClassComponent` from `svelte/legacy` if needed.
- `compatibility.componentApi` option enables auto-backwards compatibility.

```js
export default {
  compilerOptions: {
    compatibility: {
      componentApi: 4
    }
  }
};
```

#### **Asynchronous Behavior**
- `mount` and `hydrate` are **not** synchronous.
- Use `flushSync()` if onMount must be guaranteed to run.

#### **Server-Side Rendering (SSR)**
- No `render()` method in components.
- Instead, use `render()` from `svelte/server`.

```js
import { render } from 'svelte/server';
import App from './App.svelte';

const { html, head } = render(App, { props: { message: 'hello' }});
```

- **CSS not included by default** → Use `css: 'injected'` to include styles.

#### **Typing Changes**
- `SvelteComponent` deprecated → Use `Component` type.
- `ComponentEvents` & `ComponentType` deprecated.

```ts
import type { Component } from 'svelte';
export declare const MyComponent: Component<{ foo: string }>;
```

#### **bind:this Changes**
- No longer returns `$set`, `$on`, or `$destroy` methods.
- Now only returns instance exports & property accessors (if enabled).

#### **Dynamic Components**
- `<svelte:component>` no longer needed.
- Components update dynamically when reassigned.

```svelte
<script>
  import A from './A.svelte';
  import B from './B.svelte';
  let Thing = $state();
</script>

<select bind:value={Thing}>
  <option value={A}>A</option>
  <option value={B}>B</option>
</select>

<!-- Both are now equivalent -->
<Thing />
<svelte:component this={Thing} />
```

#### **Other Changes**
- **Dot Notation** (`<foo.bar>`) now treated as a **component**, not an HTML tag.
- **Whitespace Handling Simplified** → Predictable behavior.
- **Reserved `children` Prop** → Cannot use `children` as a prop name; it's reserved.