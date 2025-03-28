# Svelte 5 Runes Overview

---
## $State
---

<!-- ================================== -->
<!-- Example 1: Basic Usage of $state -->
<!-- ================================== -->

<!-- Demonstrates how to create and update a reactive state variable using $state. -->

---

<!-- Filename: Counter.svelte -->
<script>
	let count = $state(0); // count is reactive
</script>

<button onclick={() => count++}>
	clicks: {count} <!-- UI updates automatically when count changes -->
</button>

---

<!-- Notes: -->
<!-- - $state makes `count` reactive. -->
<!-- - You can modify `count` like a normal variable, no need for a special setter. -->

<!-- ================================== -->
<!-- Example 2: Deep Reactivity with Arrays -->
<!-- ================================== -->

<!-- Demonstrates how $state makes arrays deeply reactive. -->

---

<!-- Filename: TodoList.svelte -->
<script>
	let todos = $state([
		{ done: false, text: 'Learn Svelte' },
		{ done: false, text: 'Build an app' }
	]);

	function toggle(index) {
		todos[index].done = !todos[index].done; // Reactivity applies to individual items
	}
</script>

<ul>
	{#each todos as todo, index}
		<li>
			<input type="checkbox" bind:checked={todo.done} onclick={() => toggle(index)} />
			{todo.text}
		</li>
	{/each}
</ul>

<button onclick={() => todos.push({ done: false, text: 'New task' })}>
	Add Todo
</button>

---

<!-- Notes: -->
<!-- - `todos` is a reactive proxy, meaning updates to properties trigger UI updates. -->
<!-- - `todos.push()` automatically makes new objects reactive. -->

<!-- ================================== -->
<!-- Example 3: Reactivity in Classes -->
<!-- ================================== -->

<!-- Demonstrates using $state inside a class to create reactive properties. -->

---

<!-- Filename: TodoClassExample.svelte -->
<script>
	class Todo {
		done = $state(false);
		text = $state('');

		constructor(text) {
			this.text = text;
		}

		reset = () => {
			this.text = '';
			this.done = false;
		}
	}

	let todo = new Todo('Buy groceries');
</script>

<p>{todo.text} - {todo.done ? 'Done' : 'Not done'}</p>
<button onclick={() => todo.done = !todo.done}>Toggle</button>
<button onclick={() => todo.reset()}>Reset</button>

---

<!-- Notes: -->
<!-- - Using an arrow function for `reset` ensures `this` remains correct. -->
<!-- - Class properties become reactive with $state. -->

<!-- ================================== -->
<!-- Example 4: Using $state.raw -->
<!-- ================================== -->

<!-- Demonstrates when to use $state.raw to avoid deep reactivity. -->

---

<!-- Filename: RawStateExample.svelte -->
<script>
	let person = $state.raw({
		name: 'Alice',
		age: 30
	});

	function updatePerson() {
		// This won't work because `person` is raw (not reactive)
		person.age += 1; 

		// Instead, we must reassign the whole object
		person = { ...person, age: person.age + 1 };
	}
</script>

<p>{person.name} is {person.age} years old.</p>
<button onclick={updatePerson}>Increase Age</button>

---

<!-- Notes: -->
<!-- - `person.age += 1` does nothing because $state.raw objects are immutable. -->
<!-- - To update, replace the entire object. -->

<!-- ================================== -->
<!-- Example 5: Taking a Snapshot of Reactive State -->
<!-- ================================== -->

<!-- Demonstrates how to extract a non-proxied snapshot of a reactive object. -->

---

<!-- Filename: SnapshotExample.svelte -->
<script>
	let counter = $state({ count: 0 });

	function logSnapshot() {
		console.log($state.snapshot(counter)); // Logs a plain object, not a Proxy
	}
</script>

<p>Count: {counter.count}</p>
<button onclick={() => counter.count++}>Increment</button>
<button onclick={logSnapshot}>Log Snapshot</button>

---

<!-- Notes: -->
<!-- - `snapshot` removes reactivity, making it safe to pass to external libraries. -->

<!-- ================================== -->
<!-- Example 6: Passing Reactive State to Functions -->
<!-- ================================== -->

<!-- Demonstrates passing a reactive state into a function and ensuring it remains reactive. -->

---

<!-- Filename: FunctionStateExample.svelte -->
<script>
	function add(getA, getB) {
		return () => getA() + getB();
	}

	let a = $state(1);
	let b = $state(2);
	let total = add(() => a, () => b); // Pass getters to maintain reactivity
</script>

<p>{a} + {b} = {total()}</p>
<button onclick={() => a++}>Increment A</button>
<button onclick={() => b++}>Increment B</button>

---

<!-- Notes: -->
<!-- - Directly passing `a` and `b` would not be reactive. -->
<!-- - Using getter functions ensures the function always retrieves the latest values. -->


<!-- ================================== -->
<!-- Extra: Optional Getter/Setter Binding -->
<!-- ================================== -->

<!-- Shows how to bind an input to a reactive value with a custom getter/setter. -->

<script>
	let name = $state('Alice');
</script>

<input bind:value={
	() => name,
	(v) => name = v.trimStart()
} />

<p>Hello, {name}!</p>

<!-- Notes:
- This uses the new bind: syntax (Svelte 5.9+) with a getter and setter.
- Useful for applying validation or transformation automatically.
-->


---
## $Derived
---

<!-- ================================== -->
<!-- Example 1: Basic Derived State -->
<!-- ================================== -->

<!-- Demonstrates how `$derived` updates a value based on `$state`. -->

---

<!-- Filename: App.svelte -->
<script lang="ts">
	let count = $state(0);
	let doubled = $derived(count * 2); // `doubled` automatically updates when `count` changes
</script>

<button onclick={() => count++}>
	Increment
</button>

<p>{count} doubled is {doubled}</p>

---

<!-- Notes: -->
- `$derived(count * 2)` ensures `doubled` always reflects the latest `count`.
- You **cannot** modify state inside `$derived`, e.g., `count++` inside the expression.

<!-- ================================== -->
<!-- Example 2: Using $derived.by for Complex Computation -->
<!-- ================================== -->

<!-- Demonstrates how `$derived.by` allows more complex calculations. -->

---

<!-- Filename: App.svelte -->
<script lang="ts">
	let numbers = $state([1, 2, 3]);

	// `total` is automatically recalculated when `numbers` change
	let total = $derived.by(() => {
		let sum = 0;
		for (const n of numbers) {
			sum += n;
		}
		return sum;
	});
</script>

<button onclick={() => numbers.push(numbers.length + 1)}>
	Add Number
</button>

<p>{numbers.join(' + ')} = {total}</p>

---

<!-- Notes: -->
- `$derived.by(() => { ... })` is useful when the derived state requires loops or multiple steps.
- The derived state (`total`) automatically updates when `numbers` change.

<!-- ================================== -->
<!-- Example 3: Understanding Dependencies -->
<!-- ================================== -->

<!-- Shows how derived state tracks dependencies and updates accordingly. -->

---

<!-- Filename: App.svelte -->
<script lang="ts">
	let count = $state(0);
	let isEven = $derived(count % 2 === 0);
</script>

<button onclick={() => count++}>
	Increment
</button>

<p>{count} is {isEven ? 'Even' : 'Odd'}</p>

---

<!-- Notes: -->
- `$derived` tracks all **synchronously accessed** state inside its expression.
- If `count` changes, `isEven` will automatically update.

<!-- ================================== -->
<!-- Example 4: Avoiding Unnecessary Updates -->
<!-- ================================== -->

<!-- Demonstrates how Svelte skips updates when derived values remain the same. -->

---

<!-- Filename: App.svelte -->
<script lang="ts">
	let count = $state(0);
	let large = $derived(count > 10);
</script>

<button onclick={() => count++}>
	Increment
</button>

<p>Is count large? {large}</p>

---

<!-- Notes: -->
- Svelte **only updates** when `large` changes (`false` → `true` or vice versa).
- If `count` changes **within the same threshold** (e.g., `5 → 6`), no update occurs.

---


---
## $Effect
---

<!-- ================================== -->
<!-- Example 1: Basic Usage of $effect -->
<!-- ================================== -->

<!-- Demonstrates how to use `$effect` to reactively update a canvas when state changes. -->

---

<!-- Filename: Canvas.svelte -->
<script>
	let size = $state(50);
	let color = $state('#ff3e00');
	let canvas;

	$effect(() => {
		const context = canvas.getContext('2d');
		context.clearRect(0, 0, canvas.width, canvas.height);

		// This effect runs whenever `color` or `size` changes
		context.fillStyle = color;
		context.fillRect(0, 0, size, size);
	});
</script>

<canvas bind:this={canvas} width="100" height="100" />

<input type="color" bind:value={color} />
<input type="range" bind:value={size} min="10" max="100" />

---

<!-- Notes:
- `$effect` automatically tracks dependencies (`size`, `color`).
- Runs only in the browser, not during SSR.
- Avoid modifying state inside an effect to prevent infinite loops.
-->

<!-- ================================== -->
<!-- Example 2: Using Teardown in $effect -->
<!-- ================================== -->

<!-- Demonstrates how to return a cleanup function inside `$effect` to clear intervals. -->

---

<!-- Filename: Counter.svelte -->
<script>
	let count = $state(0);
	let milliseconds = $state(1000);

	$effect(() => {
		const interval = setInterval(() => {
			count += 1;
		}, milliseconds);

		// Cleanup function runs before effect re-runs or on component destroy
		return () => {
			clearInterval(interval);
		};
	});
</script>

<h1>{count}</h1>

<button onclick={() => (milliseconds *= 2)}>Slower</button>
<button onclick={() => (milliseconds /= 2)}>Faster</button>

---

<!-- Notes:
- The interval is cleared before the next effect run or when the component is destroyed.
- If `milliseconds` changes, the old interval is removed before a new one is created.
-->

<!-- ================================== -->
<!-- Example 3: Dependency Tracking in $effect -->
<!-- ================================== -->

<!-- Demonstrates how `$effect` only tracks dependencies accessed synchronously. -->

---

<!-- Filename: AsyncCanvas.svelte -->
<script>
	let size = $state(50);
	let color = $state('#ff3e00');
	let canvas;

	$effect(() => {
		const context = canvas.getContext('2d');
		context.clearRect(0, 0, canvas.width, canvas.height);

		// This effect re-runs when `color` changes but not when `size` changes
		context.fillStyle = color;

		setTimeout(() => {
			context.fillRect(0, 0, size, size);
		}, 0);
	});
</script>

<canvas bind:this={canvas} width="100" height="100" />

<input type="color" bind:value={color} />
<input type="range" bind:value={size} min="10" max="100" />

---

<!-- Notes:
- `size` is accessed inside `setTimeout`, so it is NOT a tracked dependency.
- The effect runs when `color` changes, but `size` updates do not trigger a re-run.
-->

<!-- ================================== -->
<!-- Example 4: $effect.pre for Pre-DOM Updates -->
<!-- ================================== -->

<!-- Demonstrates how `$effect.pre` can be used to run logic before the DOM updates. -->

---

<!-- Filename: AutoScroll.svelte -->
<script>
	import { tick } from 'svelte';

	let div = $state();
	let messages = $state([]);

	$effect.pre(() => {
		if (!div) return;

		// Tracks `messages.length` so the effect re-runs when messages change
		messages.length;

		// Auto-scroll logic before the DOM updates
		if (div.offsetHeight + div.scrollTop > div.scrollHeight - 20) {
			tick().then(() => {
				div.scrollTo(0, div.scrollHeight);
			});
		}
	});
</script>

<div bind:this={div}>
	{#each messages as message}
		<p>{message}</p>
	{/each}
</div>

<button onclick={() => messages = [...messages, 'New message']}>Add Message</button>

---

<!-- Notes:
- `$effect.pre` runs before the DOM updates.
- Useful for cases like auto-scrolling where timing is important.
-->

<!-- ================================== -->
<!-- Example 5: Avoiding Infinite Loops in $effect -->
<!-- ================================== -->

<!-- Demonstrates the correct way to derive state instead of modifying it within `$effect`. -->

---

<!-- Filename: DerivedState.svelte -->
<script>
	let count = $state(0);
	let doubled = $derived(count * 2); // Correct approach

	// Incorrect approach (causes infinite loops):
	// $effect(() => {
	//   doubled = count * 2;
	// });
</script>

<h1>{count} doubled is {doubled}</h1>

<button onclick={() => count += 1}>Increment</button>

---

<!-- Notes:
- Use `$derived` instead of modifying state inside `$effect`.
- Prevents unnecessary re-renders and infinite loops.
-->

---


---
## $Bindable
---

<!-- ================================== -->
<!-- Example 1: Basic Two-Way Binding with $bindable -->
<!-- ================================== -->

<!-- Demonstrates how to use `$bindable` to create a prop that can be bound to a parent component. -->

---

<!-- Filename: FancyInput.svelte -->
<script lang="ts">
	let { value = $bindable(), ...props } = $props(); // `value` is bindable, allowing two-way data flow
</script>

<input bind:value={value} {...props} />

---

<!-- Filename: App.svelte -->
<script lang="ts">
	import FancyInput from './FancyInput.svelte';

	let message = $state('hello'); // Using `$state` to create a reactive state variable
</script>

<FancyInput bind:value={message} />
<p>{message}</p> <!-- This will update as the user types in the input -->

---

<!-- Notes: -->
<!-- - The `bind:value` directive ensures that `message` in the parent updates when the input changes. -->
<!-- - This is useful for creating reusable, controlled input components. -->
<!-- - The parent is not required to bind to `value`; it can simply pass a normal prop. -->

<!-- ================================== -->
<!-- Example 2: Default Fallback for Bindable Props -->
<!-- ================================== -->

<!-- Shows how to provide a default value when the parent does not pass a bindable prop. -->

---

<!-- Filename: FancyInput.svelte -->
<script lang="ts">
	let { value = $bindable('fallback'), ...props } = $props(); // Default value set to 'fallback'
</script>

<input bind:value={value} {...props} />

---

<!-- Filename: App.svelte -->
<script lang="ts">
	import FancyInput from './FancyInput.svelte';
</script>

<FancyInput /> <!-- No `bind:value`, so it defaults to 'fallback' -->

---

<!-- Notes: -->
<!-- - If `value` is not provided by the parent, it starts with 'fallback'. -->
<!-- - This is useful for ensuring a component always has a meaningful initial state. -->

<!-- ================================== -->
<!-- Example 3: Using $bindable with Multiple Props -->
<!-- ================================== -->

<!-- Demonstrates how to use multiple `$bindable` props in a single component. -->

---

<!-- Filename: FancyInput.svelte -->
<script lang="ts">
	let { value = $bindable('default text'), disabled = $bindable(false), ...props } = $props();
</script>

<input bind:value={value} bind:disabled={disabled} {...props} />

---

<!-- Filename: App.svelte -->
<script lang="ts">
	import FancyInput from './FancyInput.svelte';

	let message = $state('Editable text');
	let isDisabled = $state(false);
</script>

<FancyInput bind:value={message} bind:disabled={isDisabled} />
<button onclick={() => isDisabled = !isDisabled}>
	Toggle Disabled
</button>

---

<!-- Notes: -->
- The input's `value` and `disabled` states can both be bound to the parent.
- The button toggles the `disabled` state dynamically.
- Useful for forms and UI elements requiring controlled states.


---
## $Props
---

<!-- ================================== -->
<!-- Example 1: Basic Prop Usage with $props -->
<!-- ================================== -->

<!-- Demonstrates how to receive props in a Svelte component using $props -->

---

<!-- Filename: App.svelte -->
<script lang="ts">
	import MyComponent from './MyComponent.svelte';
</script>

<MyComponent adjective="cool" />

---

<!-- Filename: MyComponent.svelte -->
<script lang="ts">
	let props = $props();
</script>

<p>This component is {props.adjective}</p>

---

<!-- Explanation:
- `MyComponent` receives a prop `adjective`.
- The `$props` rune allows access to the props object inside `MyComponent`.
- `props.adjective` displays the value passed from `App.svelte`.
-->

<!-- ================================== -->
<!-- Example 2: Destructuring Props -->
<!-- ================================== -->

<!-- Demonstrates how to destructure props for cleaner code -->

---

<!-- Filename: MyComponent.svelte -->
<script lang="ts">
	let { adjective } = $props();
</script>

<p>This component is {adjective}</p>

---

<!-- Explanation:
- Instead of accessing `props.adjective`, we destructure `adjective` directly from `$props()`.
- This makes the code more concise and readable.
-->

<!-- ================================== -->
<!-- Example 3: Fallback Values for Props -->
<!-- ================================== -->

<!-- Demonstrates how to provide default values when a prop is not passed -->

---

<!-- Filename: MyComponent.svelte -->
<script lang="ts">
	let { adjective = 'happy' } = $props();
</script>

<p>This component is {adjective}</p>

---

<!-- Explanation:
- If `adjective` is not provided by the parent, it defaults to `'happy'`.
- The fallback value does **not** become reactive.
-->

<!-- ================================== -->
<!-- Example 4: Renaming Props -->
<!-- ================================== -->

<!-- Demonstrates how to rename props when destructuring -->

---

<!-- Filename: MyComponent.svelte -->
<script lang="ts">
	let { super: trouper = 'lights are gonna find me' } = $props();
</script>

<p>Song lyric: {trouper}</p>

---

<!-- Explanation:
- `super` is a reserved keyword in JavaScript.
- We rename `super` to `trouper` while destructuring.
- It also includes a default fallback value.
-->

<!-- ================================== -->
<!-- Example 5: Using Rest Props -->
<!-- ================================== -->

<!-- Demonstrates how to collect all remaining props using the rest syntax -->

---

<!-- Filename: MyComponent.svelte -->
<script lang="ts">
	let { a, b, c, ...others } = $props();
</script>

<p>a: {a}, b: {b}, c: {c}</p>
<p>Other props: {JSON.stringify(others)}</p>

---

<!-- Explanation:
- `a`, `b`, and `c` are extracted explicitly.
- The rest of the props are collected into the `others` object.
- This can be useful for passing arbitrary props to a child component.
-->

<!-- ================================== -->
<!-- Example 6: Updating Props (Ephemeral State) -->
<!-- ================================== -->

<!-- Demonstrates how prop values update when the parent changes, but can be overridden temporarily -->

---

<!-- Filename: App.svelte -->
<script lang="ts">
	import Child from './Child.svelte';

	let count = $state(0);
</script>

<button onclick={() => count += 1}>
	clicks (parent): {count}
</button>

<Child {count} />

---

<!-- Filename: Child.svelte -->
<script lang="ts">
	let { count } = $props();
</script>

<button onclick={() => count += 1}>
	clicks (child): {count}
</button>

---

<!-- Explanation:
- The `count` prop updates when changed in `App.svelte`.
- However, inside `Child.svelte`, `count` can be modified temporarily.
- This is useful for unsaved, ephemeral state.
-->

<!-- ================================== -->
<!-- Example 7: Avoiding Object Prop Mutation -->
<!-- ================================== -->

<!-- Demonstrates how object mutations do not work if props are plain objects -->

---

<!-- Filename: App.svelte -->
<script lang="ts">
	import Child from './Child.svelte';
</script>

<Child object={{ count: 0 }} />

---

<!-- Filename: Child.svelte -->
<script lang="ts">
	let { object } = $props();
</script>

<button onclick={() => object.count += 1}>
	clicks: {object.count}
</button>

---

<!-- Explanation:
- The object prop `{ count: 0 }` is passed to `Child.svelte`.
- Mutating `object.count` **does not update the parent**.
- This is because plain objects are not reactive.
-->

<!-- ================================== -->
<!-- Example 8: Mutating a Reactive Prop (Warning) -->
<!-- ================================== -->

<!-- Demonstrates the warning when mutating a reactive state prop -->

---

<!-- Filename: App.svelte -->
<script lang="ts">
	import Child from './Child.svelte';

	let object = $state({ count: 0 });
</script>

<Child {object} />

---

<!-- Filename: Child.svelte -->
<script lang="ts">
	let { object } = $props();
</script>

<button onclick={() => object.count += 1}>
	clicks: {object.count}
</button>

---

<!-- Explanation:
- `object` is a `$state` reactive proxy.
- Mutating `object.count` inside `Child.svelte` **does work**, but shows a warning.
- This warns against mutating state that does not "belong" to the child.
-->

<!-- ================================== -->
<!-- Example 9: Using $props.id() for Unique IDs -->
<!-- ================================== -->

<!-- Demonstrates how to generate unique component-scoped IDs -->

---

<!-- Filename: MyComponent.svelte -->
<script lang="ts">
	const uid = $props.id();
</script>

<form>
	<label for="{uid}-firstname">First Name: </label>
	<input id="{uid}-firstname" type="text" />

	<label for="{uid}-lastname">Last Name: </label>
	<input id="{uid}-lastname" type="text" />
</form>

---

<!-- Explanation:
- `$props.id()` generates a unique ID for this component instance.
- This is useful for accessibility (`for` and `aria-labelledby` attributes).
- Ensures consistency between server and client during hydration.
-->


---
## $Host
---

<!-- ================================== -->
<!-- Example 1: Using $host to Dispatch Events -->
<!-- ================================== -->

<!-- Demonstrates how to use $host inside a custom element to dispatch events. -->

---

<!-- Filename: Stepper.svelte -->
<svelte:options customElement="my-stepper" />

<script lang="ts">
	function dispatch(type) {
		$host().dispatchEvent(new CustomEvent(type)); 
		// $host() gives access to the custom element itself.
		// Dispatches 'increment' or 'decrement' events.
	}
</script>

<button onclick={() => dispatch('decrement')}>decrement</button>
<button onclick={() => dispatch('increment')}>increment</button>

---

<!-- Filename: App.svelte -->
<script lang="ts">
	import './Stepper.svelte';

	let count = $state(0); // State variable to track count
</script>

<my-stepper
	ondecrement={() => count -= 1} 
	onincrement={() => count += 1} 
></my-stepper>

<p>count: {count}</p> <!-- Displays updated count -->

---

<!-- Notes -->
- `$host()` only works in components compiled as **custom elements** (`<svelte:options customElement="..."/>`).
- This approach allows dispatching custom events from a **shadow DOM encapsulated component**.
- Ensure the parent component listens to the dispatched events (`ondecrement`, `onincrement`).


---
## $Inspect
---

<!-- ================================== -->
<!-- Example 1: Basic $inspect Usage -->
<!-- ================================== -->

<!-- Demonstrates how `$inspect` logs changes to reactive state in development mode. -->

---

<!-- Filename: App.svelte -->
<script>
	let count = $state(0);
	let message = $state('hello');

	$inspect(count, message); // Logs `count` and `message` whenever they change
</script>

<button onclick={() => count++}>Increment</button>
<input bind:value={message} />

---

<!-- Notes:
- `$inspect` will log changes to `count` and `message` automatically.
- It only works during development and is ignored in production.
- Useful for debugging reactive state.
-->

---

<!-- ================================== -->
<!-- Example 2: Custom Logging with .with -->
<!-- ================================== -->

<!-- Demonstrates how `$inspect(...).with` replaces default logging with a custom function. -->

---

<!-- Filename: App.svelte -->
<script>
	let count = $state(0);

	$inspect(count).with((type, count) => {
		if (type === 'update') {
			console.log(`Count updated to: ${count}`);
		}
	});
</script>

<button onclick={() => count++}>Increment</button>

---

<!-- Notes:
- Instead of logging automatically, `$inspect(count).with(...)` calls a custom function.
- The callback receives `type` (`"init"` or `"update"`) and the updated values.
- Useful for debugging or conditional logging.
-->

---

<!-- ================================== -->
<!-- Example 3: Tracing Changes with console.trace -->
<!-- ================================== -->

<!-- Demonstrates how to trace reactive state changes using `$inspect(...).with(console.trace)`. -->

---

<!-- Filename: App.svelte -->
<script>
	let message = $state('Hello');

	$inspect(message).with(console.trace);
</script>

<input bind:value={message} />

---

<!-- Notes:
- Using `console.trace` with `$inspect.with()` helps track where state changes originate.
- Best used when debugging complex reactive flows.
-->

---

<!-- ================================== -->
<!-- Example 4: Tracing Effects with $inspect.trace -->
<!-- ================================== -->

<!-- Demonstrates how `$inspect.trace()` tracks function re-execution due to reactive state changes. -->

---

<!-- Filename: App.svelte -->
<script>
	import { doSomeWork } from './utils';

	$effect(() => {
		$inspect.trace(); // Logs which state changes caused this effect to run
		doSomeWork();
	});
</script>

---

<!-- Notes:
- `$inspect.trace()` is useful inside `$effect()` to track reactivity.
- When the effect re-runs, it logs which state triggered the change.
- Added in Svelte 5.14.
-->


# Svelte 5 Event Handling

<!-- ================================== -->
<!-- Example 1: Basic Event Handling -->
<!-- ================================== -->

**Svelte 5 replaces `on:` directives with direct property bindings for event handlers.**

---

<!-- Filename: App.svelte -->
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

**Key changes**:
- Removed `on:click`, replacing it with `{onclick}` as a property.
- Uses a named function (`onclick`) instead of inline arrow functions.

---

<!-- ================================== -->
<!-- Example 2: Component Event Callbacks -->
<!-- ================================== -->

**In Svelte 5, components no longer use `createEventDispatcher()`. Instead, they accept callback props.**

---

<!-- Filename: Pump.svelte -->
```svelte
<script lang="ts">
	let { inflate, deflate } = $props(); // Props passed from parent
	let power = $state(5);
</script>

<button onclick={() => inflate(power)}>inflate</button>
<button onclick={() => deflate(power)}>deflate</button>
<button onclick={() => power--}>-</button>
Pump power: {power}
<button onclick={() => power++}>+</button>
```

---

<!-- Filename: App.svelte -->
```svelte
<script lang="ts">
	import Pump from './Pump.svelte';

	let size = $state(15);
	let burst = $state(false);

	function inflateHandler(power) {
		size += power;
		if (size > 75) burst = true;
	}

	function deflateHandler(power) {
		if (size > 0) size -= power;
	}

	function reset() {
		size = 15;
		burst = false;
	}
</script>

<Pump inflate={inflateHandler} deflate={deflateHandler} />

{#if burst}
	<button onclick={reset}>new balloon</button>
	<span class="boom">💥</span>
{:else}
	<span class="balloon" style="scale: {0.01 * size}">
		🎈
	</span>
{/if}
```

**Key changes**:
- Removed `createEventDispatcher()`, instead using callback props (`inflate`, `deflate`).
- Events are now just function calls inside `Pump.svelte`.
- **Parent (App.svelte)** passes handlers like `inflateHandler` to `Pump`.

---

<!-- ================================== -->
<!-- Example 3: Bubbling Events with Callback Props -->
<!-- ================================== -->

**To forward an event from an element to a component, use callback props instead of `on:`.**

---

<!-- Filename: Clickable.svelte -->
```svelte
<script>
	let { onclick } = $props(); // Accepts an `onclick` function from parent
</script>

<button {onclick}>
	click me
</button>
```

**Key changes**:
- No need to manually forward events like `on:click` → `{onclick}` does the job.

---

<!-- ================================== -->
<!-- Example 4: Spreading Props Including Event Handlers -->
<!-- ================================== -->

**Instead of manually listing every event, just spread props.**

---

<!-- Filename: Button.svelte -->
```svelte
<script>
	let props = $props();
</script>

<button {...props}>
	click me
</button>
```

**Key changes**:
- `{...props}` automatically applies all attributes, including event handlers like `onclick`.

---

<!-- ================================== -->
<!-- Example 5: Multiple Event Handlers on the Same Element -->
<!-- ================================== -->

**Instead of duplicate event bindings, combine handlers manually.**

---

<!-- Filename: App.svelte -->
```svelte
<script>
	function one(event) {
		console.log('First handler', event);
	}

	function two(event) {
		console.log('Second handler', event);
	}
</script>

<button
	onclick={(e) => {
		one(e);
		two(e);
	}}
>
	...
</button>
```

**Key changes**:
- Instead of multiple `on:click={one} on:click={two}`, handlers are manually merged.

---

<!-- ================================== -->
<!-- Example 6: Preventing Local Handlers from Being Overwritten -->
<!-- ================================== -->

**Ensure that when spreading props, local event handlers still execute.**

---

<!-- Filename: App.svelte -->
```svelte
<script>
	let props = $props();

	function doStuff(event) {
		console.log('Handled locally', event);
	}
</script>

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

**Key changes**:
- The local `onclick` executes **before** the `props.onclick`, preventing overwrites.

---

<!-- ================================== -->
<!-- Example 7: Event Modifiers as Functions -->
<!-- ================================== -->

**Modifiers like `once` and `preventDefault` should now be implemented manually.**

---

<!-- Filename: App.svelte -->
```svelte
<script>
	function once(fn) {
		return function (event) {
			if (fn) fn.call(this, event);
			fn = null; // Ensures it only runs once
		};
	}

	function preventDefault(fn) {
		return function (event) {
			event.preventDefault();
			fn.call(this, event);
		};
	}

	function handler(event) {
		console.log('Button clicked');
	}
</script>

<button onclick={once(preventDefault(handler))}>
	...
</button>
```

**Key changes**:
- Used **higher-order functions** to replace `on:click|once|preventDefault={handler}`.

---
---

// svelte/events - `on` function
// `on` attaches an event handler and returns a cleanup function.
// It ensures correct handler order compared to `addEventListener`.

import { on } from 'svelte/events';

// Attach an event to `window`
function on<Type extends keyof WindowEventMap>(
    window: Window,
    type: Type,
    handler: (this: Window, event: WindowEventMap[Type]) => any,
    options?: AddEventListenerOptions
): () => void;

// Attach an event to `document`
function on<Type extends keyof DocumentEventMap>(
    document: Document,
    type: Type,
    handler: (this: Document, event: DocumentEventMap[Type]) => any,
    options?: AddEventListenerOptions
): () => void;

// Attach an event to an `HTMLElement`
function on<Element extends HTMLElement, Type extends keyof HTMLElementEventMap>(
    element: Element,
    type: Type,
    handler: (this: Element, event: HTMLElementEventMap[Type]) => any,
    options?: AddEventListenerOptions
): () => void;

// Attach an event to a `MediaQueryList`
function on<Element extends MediaQueryList, Type extends keyof MediaQueryListEventMap>(
    element: Element,
    type: Type,
    handler: (this: Element, event: MediaQueryListEventMap[Type]) => any,
    options?: AddEventListenerOptions
): () => void;

// Generic event attachment
function on(
    element: EventTarget,
    type: string,
    handler: EventListener,
    options?: AddEventListenerOptions
): () => void;

---
---

Use `on:eventname={handler}` to attach handlers (e.g., `onclick={() => ...}`).
- Handlers can mutate `$state` directly; UI updates automatically.
- Shorthand `{eventname}` works if the handler is a variable or function.
- Events like `click`, `input`, etc., are delegated; assume bubbling unless stopped.
- Events fire after bindings (e.g., `oninput` after `bind:value`).
- Case-sensitive: `onclick` ≠ `onClick` (custom events may use uppercase).

<script>
  let count = $state(0);         // Reactive state
  let text = $state('');         // Input value
  const logClick = () => console.log('Clicked!'); // Named handler
  
  // $effect to react to state changes
  $effect(() => console.log(`Count is now ${count}`));
</script>

<!-- Inline handler with $state -->
<button onclick={() => count++}>
  Increment: {count}
</button>

<!-- Shorthand with named handler -->
<button {logClick}>
  Log Click
</button>

<!-- Input event with binding -->
<input
  oninput={(e) => text = e.target.value}
  bind:value={text}
  placeholder="Type here"
/>
<p>You typed: {text}</p>

<!-- Keydown event with condition -->
<input
  onkeydown={(e) => e.key === 'Enter' && count++}
  placeholder="Press Enter to increment"
/>

Note: For other events (e.g., `onmouseover`, `onfocus`), follow the same pattern:
`on:eventname={() => ...}` or `{eventname}`. Adjust the handler logic based on the event’s purpose.

# Svelte 5 Snippets

<!-- ================================== -->
<!-- Example 1: Basic Snippet Usage -->
<!-- ================================== -->

<!-- Demonstrates how to create and use a snippet to avoid repetitive markup -->

---

<!-- Filename: ImageList.svelte -->
<script>
	let images = [
		{ src: "image1.jpg", caption: "Image 1", href: "https://example.com" },
		{ src: "image2.jpg", caption: "Image 2" }
	];
</script>

{#snippet figure(image)}
	<figure>
		<img src={image.src} alt={image.caption} />
		<figcaption>{image.caption}</figcaption>
	</figure>
{/snippet}

{#each images as image}
	{#if image.href}
		<a href={image.href}>
			{@render figure(image)}
		</a>
	{:else}
		{@render figure(image)}
	{/if}
{/each}

---

<!-- This example shows how snippets reduce duplication in templates -->

<!-- ================================== -->
<!-- Example 2: Snippet Scope and Visibility -->
<!-- ================================== -->

<!-- Demonstrates snippet scoping rules and visibility -->

---

<!-- Filename: ScopedSnippets.svelte -->
<div>
	{#snippet x()}
		{#snippet y()}
			<p>Inside y</p>
		{/snippet}

		<!-- Rendering `y` inside `x` is allowed -->
		{@render y()}
	{/snippet}

	<!-- This will cause an error because `y` is out of scope -->
	{@render y()}
</div>

<!-- This will also cause an error because `x` is out of scope -->
{@render x()}

---

<!-- Notes: Snippets are only accessible within their lexical scope -->

<!-- ================================== -->
<!-- Example 3: Recursive Snippets -->
<!-- ================================== -->

<!-- Demonstrates how a snippet can reference itself -->

---

<!-- Filename: Countdown.svelte -->
{#snippet blastoff()}
	<span>🚀</span>
{/snippet}

{#snippet countdown(n)}
	{#if n > 0}
		<span>{n}...</span>
		{@render countdown(n - 1)}
	{:else}
		{@render blastoff()}
	{/if}
{/snippet}

{@render countdown(5)}

---

<!-- Recursive snippets are useful for countdowns, trees, and nested structures -->

<!-- ================================== -->
<!-- Example 4: Passing Snippets to Components -->
<!-- ================================== -->

<!-- Demonstrates passing snippets as props to a component -->

---

<!-- Filename: Table.svelte -->
<script>
	let { data, header, row } = $props();
</script>

<table>
	<thead>
		<tr>{@render header()}</tr>
	</thead>
	<tbody>
		{#each data as item}
			<tr>{@render row(item)}</tr>
		{/each}
	</tbody>
</table>

---

<!-- Filename: App.svelte -->
<script>
	import Table from "./Table.svelte";

	const fruits = [
		{ name: "Apples", qty: 5, price: 2 },
		{ name: "Bananas", qty: 10, price: 1 },
		{ name: "Cherries", qty: 20, price: 0.5 }
	];
</script>

{#snippet header()}
	<th>Fruit</th>
	<th>Qty</th>
	<th>Price</th>
	<th>Total</th>
{/snippet}

{#snippet row(d)}
	<td>{d.name}</td>
	<td>{d.qty}</td>
	<td>{d.price}</td>
	<td>{d.qty * d.price}</td>
{/snippet}

<Table data={fruits} {header} {row} />

---

<!-- Snippets allow passing reusable markup structures into components -->

<!-- ================================== -->
<!-- Example 5: Default Slot with Snippets -->
<!-- ================================== -->

<!-- Demonstrates how snippets replace slots in Svelte 5 -->

---

<!-- Filename: Button.svelte -->
<script>
	let { children } = $props();
</script>

<button>{@render children()}</button>

---

<!-- Filename: App.svelte -->
<Button>Click Me</Button>

---

<!-- Any non-snippet content passed to a component becomes the `children` snippet -->

<!-- ================================== -->
<!-- Example 6: Optional Snippet Props -->
<!-- ================================== -->

<!-- Demonstrates handling optional snippet props -->

---

<!-- Filename: OptionalSnippet.svelte -->
<script>
	let { children } = $props();
</script>

{#if children}
	{@render children()}
{:else}
	<p>Fallback content</p>
{/if}

---

<!-- Filename: App.svelte -->
<OptionalSnippet />
<OptionalSnippet>
	{#snippet children()}
		<p>Custom content</p>
	{/snippet}
</OptionalSnippet>

---

<!-- Use `optional chaining` or `#if` blocks to handle missing snippets -->

<!-- ================================== -->
<!-- Example 7: Exporting Snippets -->
<!-- ================================== -->

<!-- Demonstrates exporting a snippet for use in other components -->

---

<!-- Filename: math.svelte -->
<script module>
	export { add };
</script>

{#snippet add(a, b)}
	{a} + {b} = {a + b}
{/snippet}

---

<!-- Filename: App.svelte -->
<script>
	import { add } from "./math.svelte";
</script>

{@render add(2, 3)}

---

<!-- Note: You can pass snippets to components that use slots, but not vice versa -->

<!-- ================================== -->
<!-- Example 8: Multiple Content Placeholders -->
<!-- ================================== -->

<!-- Demonstrates using multiple snippets instead of named slots -->

---

<!-- Filename: Layout.svelte -->
<script>
	let { header, main, footer } = $props();
</script>

<header>
	{@render header?.()}
</header>

<main>
	{@render main?.()}
</main>

<footer>
	{@render footer?.()}
</footer>

---

<!-- Filename: App.svelte -->
<script>
	import Layout from "./Layout.svelte";
</script>

<Layout>
	{#snippet header()}
		<h1>My Website</h1>
	{/snippet}
	
	{#snippet main()}
		<p>Welcome to my website!</p>
	{/snippet}
	
	{#snippet footer()}
		<p>© 2023 My Website</p>
	{/snippet}
</Layout>

---


<!-- ================================== -->
<!-- Example 9: Passing Data to Parent with Snippets -->
<!-- ================================== -->

<!-- Demonstrates how snippets allow passing data from child to parent -->

---

<!-- Filename: List.svelte -->
<script>
	let { items, item, empty } = $props();
</script>

{#if items.length}
	<ul>
		{#each items as entry}
			<li>
				{@render item(entry)}
			</li>
		{/each}
	</ul>
{:else}
	{@render empty?.()}
{/if}

---

<!-- Filename: App.svelte -->
<script>
	import List from "./List.svelte";
	
	const fruits = ['Apple', 'Banana', 'Cherry'];
</script>

<List items={fruits}>
	{#snippet item(fruit)}
		<span>{fruit}</span>
	{/snippet}
	
	{#snippet empty()}
		<span>No fruits available</span>
	{/snippet}
</List>

---

<!-- 
The child component passes data as parameters to the snippet function,
which the parent can then access and use within the snippet definition.
-->

# Svelte 5 + TypeScript

<!-- ================================== -->
<!-- Example 1: Using TypeScript in Svelte Components -->
<!-- ================================== -->

<!-- Demonstrates basic TypeScript setup inside a Svelte component. -->

---

<!-- Filename: App.svelte -->
<script lang="ts">
	let name: string = 'world';

	function greet(name: string) {
		alert(`Hello, ${name}!`);
	}
</script>

<button onclick={() => greet(name)}>Greet</button>

---

### Notes:
- Adding `lang="ts"` enables TypeScript.
- Type annotations (`string`) help with static checking.
- No runtime overhead since TypeScript removes type annotations at compile time.

---

<!-- ================================== -->
<!-- Example 2: Typing Component Props -->
<!-- ================================== -->

<!-- Shows how to type component props in Svelte. -->

---

<!-- Filename: Greeting.svelte -->
<script lang="ts">
	interface Props {
		name: string;
	}

	let { name }: Props = $props();
</script>

<p>Hello, {name}!</p>

---

<!-- Filename: App.svelte -->
<script lang="ts">
	import Greeting from './Greeting.svelte';
</script>

<Greeting name="Alice" />

---

### Notes:
- `Props` interface ensures `name` is always a string.
- `$props()` extracts the component’s props with proper typing.

---

<!-- ================================== -->
<!-- Example 3: Generic Component Props -->
<!-- ================================== -->

<!-- Demonstrates generic props for flexible typing. -->

---

<!-- Filename: List.svelte -->
<script lang="ts" generics="Item">
	interface Props {
		items: Item[];
		select: (item: Item) => void;
	}

	let { items, select }: Props = $props();
</script>

{#each items as item}
	<button onclick={() => select(item)}>{item}</button>
{/each}

---

<!-- Filename: App.svelte -->
<script lang="ts">
	import List from './List.svelte';

	const names = ['Alice', 'Bob', 'Charlie'];

	function handleSelect(name: string) {
		console.log('Selected:', name);
	}
</script>

<List items={names} select={handleSelect} />

---

### Notes:
- `generics="Item"` allows the component to accept any item type.
- Ensures `items` and `select` function operate on the same type.

---

<!-- ================================== -->
<!-- Example 4: Typing $state -->
<!-- ================================== -->

<!-- Shows how to type state variables in Svelte. -->

---

<!-- Filename: Counter.svelte -->
<script lang="ts">
	let count: number = $state(0);

	function increment() {
		count += 1;
	}
</script>

<button onclick={increment}>Count: {count}</button>

---

### Notes:
- `$state(0)` initializes a reactive variable with a type.
- Without an initial value, TypeScript infers `number | undefined`.

---

<!-- ================================== -->
<!-- Example 5: Typing Wrapper Components -->
<!-- ================================== -->

<!-- Demonstrates how to type wrapper components that extend native HTML elements. -->

---

<!-- Filename: Button.svelte -->
<script lang="ts">
	import type { HTMLButtonAttributes } from 'svelte/elements';

	let { children, ...rest }: HTMLButtonAttributes = $props();
</script>

<button {...rest}>{@render children?.()}</button>

---

<!-- Filename: App.svelte -->
<script lang="ts">
	import Button from './Button.svelte';
</script>

<Button onclick={() => alert('Clicked!')}>Click Me</Button>

---

### Notes:
- `HTMLButtonAttributes` ensures `Button.svelte` supports all standard button attributes.
- `...rest` spreads remaining props onto the `<button>`.

---

<!-- ================================== -->
<!-- Example 6: Typing Custom Events and Attributes -->
<!-- ================================== -->

<!-- Demonstrates how to extend built-in DOM types for custom attributes and events. -->

---

<!-- Filename: additional-svelte-typings.d.ts -->
declare namespace svelteHTML {
	interface IntrinsicElements {
		'custom-element': { customProp: string; 'on:customEvent': (e: CustomEvent<any>) => void };
	}
}

---

<!-- Filename: App.svelte -->
<script lang="ts">
	function handleEvent(e: CustomEvent) {
		console.log('Custom event:', e.detail);
	}
</script>

<custom-element customProp="Hello" on:customEvent={handleEvent}></custom-element>

---

### Notes:
- Extending `IntrinsicElements` allows TypeScript to recognize `<custom-element>` and its props.
- Avoids TypeScript errors when using experimental or third-party web components.

---

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
