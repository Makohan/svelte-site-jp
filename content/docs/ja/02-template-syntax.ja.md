---
title: Template syntax
---


### Tags

---

`<div>` のような小文字のタグは、通常の HTML 要素を表します。大文字のタグ、例えば `<Widget>` や `<Namespace.Widget>` は *コンポーネント* を表します。

```sv
<script>
	import Widget from './Widget.svelte';
</script>

<div>
	<Widget/>
</div>
```


### Attributes and props

---

デフォルトでは、属性はHTMLと全く同じように動作します。

```sv
<div class="foo">
	<button disabled>can't touch this</button>
</div>
```

---

HTMLのように、値は引用符で囲まれていない場合があります。

```sv
<input type=checkbox>
```

---

属性値には JavaScript の式を含めることができます。

```sv
<a href="page/{p}">page {p}</a>
```

---

あるいは、JavaScript の式にすることもできます。

```sv
<button disabled={!clickable}>...</button>
```

---

Boolean な属性は、その値が [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) であれば要素に含まれ、[falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) であれば除外されます。

それ以外の属性は、その値が [nullish](https://developer.mozilla.org/en-US/docs/Glossary/Nullish) (`null` または `undefined`) でない限り含まれます。

```html
<input required={false} placeholder="This input field is not required">
<div title={null}>This div has no title attribute</div>
```

---

式には、通常の HTML ではシンタックスハイライトに失敗するような文字が含まれている可能性があるので、値を引用符で囲むことが許可されています。引用符は値の解析方法には影響しません。

```sv
<button disabled="{number !== 42}">...</button>
```

---

属性名と値が一致する場合(`name={name}`)は、`{name}`で置き換えることができます。

```sv
<!-- These are equivalent -->
<button disabled={disabled}>...</button>
<button {disabled}>...</button>
```

---

慣習として、コンポーネントに渡される値は DOM の機能である *属性* ではなく、*プロパティ* または *props* と呼ばれます。

要素の場合と同様に、`name={name}` は `{name}` の短縮形に置き換えることができます。

```sv
<Widget foo={bar} answer={42} text="hello"/>
```

---

*スプレッド属性* は、多くの属性やプロパティを一度に要素やコンポーネントに渡すことを可能にします。

要素またはコンポーネントは、通常のものとは別に、複数のスプレッド属性を持つことができます。

```sv
<Widget {...things}/>
```

---

*`$$props`* は、`export` で宣言されていないものも含めて、コンポーネントに渡されるすべての props を参照します。これは Svelte の最適化が難しいので、一般的には推奨されません。しかし、コンパイル時にどのような props がコンポーネントに渡されるかわからない場合など、稀なケースでは便利です。

```sv
<Widget {...$$props}/>
```

---

*`$$restProps`* には、`export` で宣言されて *いない* props のみが含まれます。これは他の未知の属性をコンポーネントの要素に渡すために使用できます。 *`$$props`* と同じ最適化の問題を共有しており、同様に推奨されません。

```html
<input {...$$restProps}>
```


> `input` 要素やその子要素である `option` 要素の `value` 属性は、`bind:group` や `bind:checked` を使用している場合、スプレッド属性で設定してはいけません。Svelte はこのような場合、要素の `value` をマークアップで直接見ることができるようにして、バインドされた変数にリンクさせる必要があります。

---

### Text expressions

```sv
{expression}
```

---

テキストにはJavaScriptの式を含めることもできます。

```sv
<h1>Hello {name}!</h1>
<p>{a} + {b} = {a + b}.</p>
```


### Comments

---

コンポーネント内でHTMLコメントを使用することができます。

```sv
<!-- this is a comment! -->
<h1>Hello world</h1>
```

---

`svelte-ignore` で始まるコメントは、マークアップの次のブロックに対する警告を無効にします。通常、これらはアクセシビリティの警告です。正当な理由で警告を無効にしていることを確認してください。

```sv
<!-- svelte-ignore a11y-autofocus -->
<input bind:value={name} autofocus>
```


### {#if ...}

```sv
{#if 式}...{/if}
```
```sv
{#if 式}...{:else if 式}...{/if}
```
```sv
{#if 式}...{:else}...{/if}
```

---

条件付きでレンダリングされるコンテンツは、if ブロックでラップできます。

```sv
{#if answer === 42}
	<p>what was the question?</p>
{/if}
```

---

追加の条件は `{:else if 式}` で付け足すことができ、`{:else}` 句で終わらせることもできます。

```sv
{#if porridge.temperature > 100}
	<p>too hot!</p>
{:else if 80 > porridge.temperature}
	<p>too cold!</p>
{:else}
	<p>just right!</p>
{/if}
```


### {#each ...}

```sv
{#each 式 as name}...{/each}
```
```sv
{#each 式 as name, index}...{/each}
```
```sv
{#each 式 as name (key)}...{/each}
```
```sv
{#each 式 as name, index (key)}...{/each}
```
```sv
{#each 式 as name}...{:else}...{/each}
```

---

each ブロックで値のリストの反復処理ができます。

```sv
<h1>Shopping list</h1>
<ul>
	{#each items as item}
		<li>{item.name} x {item.qty}</li>
	{/each}
</ul>
```

配列や配列のような値、つまり `length` プロパティを持つオブジェクトを反復処理するのに each ブロックを使用できます。

---

each ブロックは `array.map(...)` のコールバックの第 2 引数に相当する*インデックス*を指定することもできます。

```sv
{#each items as item, i}
	<li>{i + 1}: {item.name} x {item.qty}</li>
{/each}
```

---

*key* の式（各リストアイテムを一意に識別できる必要があります）が与えられた場合、データが変更されたときに Svelte は末尾にアイテムを追加したり削除するのではなくそれを使用して差分を取ります。key はどんなものでもよいですが、オブジェクト自体が変更されたときに同一性を維持できるため、文字列か数値をお勧めします。

```sv
{#each items as item (item.id)}
	<li>{item.name} x {item.qty}</li>
{/each}

<!-- もしくはインデックスを追加 -->
{#each items as item, i (item.id)}
	<li>{i + 1}: {item.name} x {item.qty}</li>
{/each}
```

---

each ブロックでは分割代入や残余構文のパターンを自由に使えます。

```sv
{#each items as { id, name, qty }, i (id)}
	<li>{i + 1}: {name} x {qty}</li>
{/each}

{#each objects as { id, ...rest }}
	<li><span>{id}</span><MyComponent {...rest}/></li>
{/each}

{#each items as [id, ...rest]}
	<li><span>{id}</span><MyComponent values={rest}/></li>
{/each}
```

---

each ブロックには `{:else}` 句を入れることもできます。これはリストが空の場合にレンダリングされます。

```sv
{#each todos as todo}
	<p>{todo.text}</p>
{:else}
	<p>No tasks today!</p>
{/each}
```


### {#await ...}

```sv
{#await 式}...{:then name}...{:catch name}...{/await}
```
```sv
{#await 式}...{:then name}...{/await}
```
```sv
{#await 式 then name}...{/await}
```
```sv
{#await 式 catch name}...{/await}
```

---

await ブロックを使用すると、Promise が取りうる 3 つの状態（保留中、成功、失敗）に分岐できます。

```sv
{#await promise}
	<!-- promise が保留中 -->
	<p>waiting for the promise to resolve...</p>
{:then value}
	<!-- promise が成功した -->
	<p>The value is {value}</p>
{:catch error}
	<!-- promise が失敗した -->
	<p>Something went wrong: {error.message}</p>
{/await}
```

---

promise が失敗した時に何もレンダリングする必要がない場合（もしくはエラーが発生しない場合）は `catch` ブロックを省略できます。

```sv
{#await promise}
	<!-- promise が保留中 -->
	<p>waiting for the promise to resolve...</p>
{:then value}
	<!-- promise が成功した -->
	<p>The value is {value}</p>
{/await}
```

---

保留中の状態を気にしない場合は、最初のブロックを省略することもできます。

```sv
{#await promise then value}
	<p>The value is {value}</p>
{/await}
```

---

逆にエラー状態のみを表示したい場合は `then` ブロックを省略できます。

```sv
{#await promise catch error}
	<p>The error is {error}</p>
{/await}
```

### {#key ...}

```sv
{#key 式}...{/key}
```

key ブロックは式の値が変更されたときに、その中身を破棄して再作成します。

---

これは、値が変更されるたびに要素のトランジションを再生したい場合に便利です。

```sv
{#key value}
	<div transition:fade>{value}</div>
{/key}
```

---

コンポーネントを囲んで使用した場合、コンポーネントの再インスタンス化と再初期化をもたらします。

```sv
{#key value}
	<Component />
{/key}
```

### {@html ...}

```sv
{@html expression}
```

---

In a text expression, characters like `<` and `>` are escaped; however, with HTML expressions, they're not.

The expression should be valid standalone HTML — `{@html "<div>"}content{@html "</div>"}` will *not* work, because `</div>` is not valid HTML.

> Svelte does not sanitize expressions before injecting HTML. If the data comes from an untrusted source, you must sanitize it, or you are exposing your users to an XSS vulnerability.

```sv
<div class="blog-post">
	<h1>{post.title}</h1>
	{@html post.content}
</div>
```


### {@debug ...}

```sv
{@debug}
```
```sv
{@debug var1, var2, ..., varN}
```

---

The `{@debug ...}` tag offers an alternative to `console.log(...)`. It logs the values of specific variables whenever they change, and pauses code execution if you have devtools open.

```sv
<script>
	let user = {
		firstname: 'Ada',
		lastname: 'Lovelace'
	};
</script>

{@debug user}

<h1>Hello {user.firstname}!</h1>
```

---

`{@debug ...}` accepts a comma-separated list of variable names (not arbitrary expressions).

```sv
<!-- Compiles -->
{@debug user}
{@debug user1, user2, user3}

<!-- WON'T compile -->
{@debug user.firstname}
{@debug myArray[0]}
{@debug !isReady}
{@debug typeof user === 'object'}
```

The `{@debug}` tag without any arguments will insert a `debugger` statement that gets triggered when *any* state changes, as opposed to the specified variables.



### Element directives

As well as attributes, elements can have *directives*, which control the element's behaviour in some way.


#### [on:*eventname*](on_element_event)

```sv
on:eventname={handler}
```
```sv
on:eventname|modifiers={handler}
```

---

Use the `on:` directive to listen to DOM events.

```sv
<script>
	let count = 0;

	function handleClick(event) {
		count += 1;
	}
</script>

<button on:click={handleClick}>
	count: {count}
</button>
```

---

Handlers can be declared inline with no performance penalty. As with attributes, directive values may be quoted for the sake of syntax highlighters.

```sv
<button on:click="{() => count += 1}">
	count: {count}
</button>
```

---

Add *modifiers* to DOM events with the `|` character.

```sv
<form on:submit|preventDefault={handleSubmit}>
	<!-- the `submit` event's default is prevented,
	     so the page won't reload -->
</form>
```

The following modifiers are available:

* `preventDefault` — calls `event.preventDefault()` before running the handler
* `stopPropagation` — calls `event.stopPropagation()`, preventing the event reaching the next element
* `passive` — improves scrolling performance on touch/wheel events (Svelte will add it automatically where it's safe to do so)
* `nonpassive` — explicitly set `passive: false`
* `capture` — fires the handler during the *capture* phase instead of the *bubbling* phase
* `once` — remove the handler after the first time it runs
* `self` — only trigger handler if event.target is the element itself

Modifiers can be chained together, e.g. `on:click|once|capture={...}`.

---

If the `on:` directive is used without a value, the component will *forward* the event, meaning that a consumer of the component can listen for it.

```sv
<button on:click>
	The component itself will emit the click event
</button>
```

---

It's possible to have multiple event listeners for the same event:

```sv
<script>
	let counter = 0;
	function increment() {
		counter = counter + 1;
	}

	function track(event) {
		trackEvent(event)
	}
</script>

<button on:click={increment} on:click={track}>Click me!</button>
```

#### [bind:*property*](bind_element_property)

```sv
bind:property={variable}
```

---

Data ordinarily flows down, from parent to child. The `bind:` directive allows data to flow the other way, from child to parent. Most bindings are specific to particular elements.

The simplest bindings reflect the value of a property, such as `input.value`.

```sv
<input bind:value={name}>
<textarea bind:value={text}></textarea>

<input type="checkbox" bind:checked={yes}>
```

---

If the name matches the value, you can use a shorthand.

```sv
<!-- These are equivalent -->
<input bind:value={value}>
<input bind:value>
```

---

Numeric input values are coerced; even though `input.value` is a string as far as the DOM is concerned, Svelte will treat it as a number. If the input is empty or invalid (in the case of `type="number"`), the value is `undefined`.

```sv
<input type="number" bind:value={num}>
<input type="range" bind:value={num}>
```

---

On `<input>` elements with `type="file"`, you can use `bind:files` to get the [`FileList` of selected files](https://developer.mozilla.org/en-US/docs/Web/API/FileList).

```sv
<label for="avatar">Upload a picture:</label>
<input
	accept="image/png, image/jpeg"
	bind:files
	id="avatar"
	name="avatar"
	type="file"
/>
```


##### Binding `<select>` value

---

A `<select>` value binding corresponds to the `value` property on the selected `<option>`, which can be any value (not just strings, as is normally the case in the DOM).

```sv
<select bind:value={selected}>
	<option value={a}>a</option>
	<option value={b}>b</option>
	<option value={c}>c</option>
</select>
```

---

A `<select multiple>` element behaves similarly to a checkbox group.

```sv
<select multiple bind:value={fillings}>
	<option value="Rice">Rice</option>
	<option value="Beans">Beans</option>
	<option value="Cheese">Cheese</option>
	<option value="Guac (extra)">Guac (extra)</option>
</select>
```

---

When the value of an `<option>` matches its text content, the attribute can be omitted.

```sv
<select multiple bind:value={fillings}>
	<option>Rice</option>
	<option>Beans</option>
	<option>Cheese</option>
	<option>Guac (extra)</option>
</select>
```

---

Elements with the `contenteditable` attribute support `innerHTML` and `textContent` bindings.

```sv
<div contenteditable="true" bind:innerHTML={html}></div>
```

##### Media element bindings

---

Media elements (`<audio>` and `<video>`) have their own set of bindings — six *readonly* ones...

* `duration` (readonly) — the total duration of the video, in seconds
* `buffered` (readonly) — an array of `{start, end}` objects
* `played` (readonly) — ditto
* `seekable` (readonly) — ditto
* `seeking` (readonly) — boolean
* `ended` (readonly) — boolean

...and five *two-way* bindings:

* `currentTime` — the current playback time in the video, in seconds
* `playbackRate` — how fast or slow to play the video, where 1 is 'normal'
* `paused` — this one should be self-explanatory
* `volume` — a value between 0 and 1
* `muted` — a boolean value where `true` is muted

Videos additionally have readonly `videoWidth` and `videoHeight` bindings.

```sv
<video
	src={clip}
	bind:duration
	bind:buffered
	bind:played
	bind:seekable
	bind:seeking
	bind:ended
	bind:currentTime
	bind:playbackRate
	bind:paused
	bind:volume
	bind:muted
	bind:videoWidth
	bind:videoHeight
></video>
```

##### Block-level element bindings

---

Block-level elements have 4 readonly bindings, measured using a technique similar to [this one](http://www.backalleycoder.com/2013/03/18/cross-browser-event-based-element-resize-detection/):

* `clientWidth`
* `clientHeight`
* `offsetWidth`
* `offsetHeight`

```sv
<div
	bind:offsetWidth={width}
	bind:offsetHeight={height}
>
	<Chart {width} {height}/>
</div>
```

#### bind:group

```sv
bind:group={variable}
```

---

Inputs that work together can use `bind:group`.

```sv
<script>
	let tortilla = 'Plain';
	let fillings = [];
</script>

<!-- grouped radio inputs are mutually exclusive -->
<input type="radio" bind:group={tortilla} value="Plain">
<input type="radio" bind:group={tortilla} value="Whole wheat">
<input type="radio" bind:group={tortilla} value="Spinach">

<!-- grouped checkbox inputs populate an array -->
<input type="checkbox" bind:group={fillings} value="Rice">
<input type="checkbox" bind:group={fillings} value="Beans">
<input type="checkbox" bind:group={fillings} value="Cheese">
<input type="checkbox" bind:group={fillings} value="Guac (extra)">
```

#### [bind:this](bind_element)

```sv
bind:this={dom_node}
```

---

To get a reference to a DOM node, use `bind:this`.

```sv
<script>
	import { onMount } from 'svelte';

	let canvasElement;

	onMount(() => {
		const ctx = canvasElement.getContext('2d');
		drawStuff(ctx);
	});
</script>

<canvas bind:this={canvasElement}></canvas>
```


#### class:*name*

```sv
class:name={value}
```
```sv
class:name
```

---

A `class:` directive provides a shorter way of toggling a class on an element.

```sv
<!-- These are equivalent -->
<div class="{active ? 'active' : ''}">...</div>
<div class:active={active}>...</div>

<!-- Shorthand, for when name and value match -->
<div class:active>...</div>

<!-- Multiple class toggles can be included -->
<div class:active class:inactive={!active} class:isAdmin>...</div>
```


#### use:*action*

```sv
use:action
```
```sv
use:action={parameters}
```

```js
action = (node: HTMLElement, parameters: any) => {
	update?: (parameters: any) => void,
	destroy?: () => void
}
```

---

Actions are functions that are called when an element is created. They can return an object with a `destroy` method that is called after the element is unmounted:

```sv
<script>
	function foo(node) {
		// the node has been mounted in the DOM

		return {
			destroy() {
				// the node has been removed from the DOM
			}
		};
	}
</script>

<div use:foo></div>
```

---

An action can have a parameter. If the returned value has an `update` method, it will be called whenever that parameter changes, immediately after Svelte has applied updates to the markup.

> Don't worry about the fact that we're redeclaring the `foo` function for every component instance — Svelte will hoist any functions that don't depend on local state out of the component definition.

```sv
<script>
	export let bar;

	function foo(node, bar) {
		// the node has been mounted in the DOM

		return {
			update(bar) {
				// the value of `bar` has changed
			},

			destroy() {
				// the node has been removed from the DOM
			}
		};
	}
</script>

<div use:foo={bar}></div>
```


#### transition:*fn*

```sv
transition:fn
```
```sv
transition:fn={params}
```
```sv
transition:fn|local
```
```sv
transition:fn|local={params}
```


```js
transition = (node: HTMLElement, params: any) => {
	delay?: number,
	duration?: number,
	easing?: (t: number) => number,
	css?: (t: number, u: number) => string,
	tick?: (t: number, u: number) => void
}
```

---

A transition is triggered by an element entering or leaving the DOM as a result of a state change.

When a block is transitioning out, all elements inside the block, including those that do not have their own transitions, are kept in the DOM until every transition in the block has completed.

The `transition:` directive indicates a *bidirectional* transition, which means it can be smoothly reversed while the transition is in progress.

```sv
{#if visible}
	<div transition:fade>
		fades in and out
	</div>
{/if}
```

> By default intro transitions will not play on first render. You can modify this behaviour by setting `intro: true` when you [create a component](docs#Client-side_component_API).

##### Transition parameters

---

Like actions, transitions can have parameters.

(The double `{{curlies}}` aren't a special syntax; this is an object literal inside an expression tag.)

```sv
{#if visible}
	<div transition:fade="{{ duration: 2000 }}">
		flies in, fades out over two seconds
	</div>
{/if}
```

##### Custom transition functions

---

Transitions can use custom functions. If the returned object has a `css` function, Svelte will create a CSS animation that plays on the element.

The `t` argument passed to `css` is a value between `0` and `1` after the `easing` function has been applied. *In* transitions run from `0` to `1`, *out* transitions run from `1` to `0` — in other words `1` is the element's natural state, as though no transition had been applied. The `u` argument is equal to `1 - t`.

The function is called repeatedly *before* the transition begins, with different `t` and `u` arguments.

```sv
<script>
	import { elasticOut } from 'svelte/easing';

	export let visible;

	function whoosh(node, params) {
		const existingTransform = getComputedStyle(node).transform.replace('none', '');

		return {
			delay: params.delay || 0,
			duration: params.duration || 400,
			easing: params.easing || elasticOut,
			css: (t, u) => `transform: ${existingTransform} scale(${t})`
		};
	}
</script>

{#if visible}
	<div in:whoosh>
		whooshes in
	</div>
{/if}
```

---

A custom transition function can also return a `tick` function, which is called *during* the transition with the same `t` and `u` arguments.

> If it's possible to use `css` instead of `tick`, do so — CSS animations can run off the main thread, preventing jank on slower devices.

```sv
<script>
	export let visible = false;

	function typewriter(node, { speed = 50 }) {
		const valid = (
			node.childNodes.length === 1 &&
			node.childNodes[0].nodeType === Node.TEXT_NODE
		);

		if (!valid) return {};

		const text = node.textContent;
		const duration = text.length * speed;

		return {
			duration,
			tick: (t, u) => {
				const i = ~~(text.length * t);
				node.textContent = text.slice(0, i);
			}
		};
	}
</script>

{#if visible}
	<p in:typewriter="{{ speed: 20 }}">
		The quick brown fox jumps over the lazy dog
	</p>
{/if}
```

If a transition returns a function instead of a transition object, the function will be called in the next microtask. This allows multiple transitions to coordinate, making [crossfade effects](tutorial/deferred-transitions) possible.


##### Transition events

---

An element with transitions will dispatch the following events in addition to any standard DOM events:

* `introstart`
* `introend`
* `outrostart`
* `outroend`

```sv
{#if visible}
	<p
		transition:fly="{{ y: 200, duration: 2000 }}"
		on:introstart="{() => status = 'intro started'}"
		on:outrostart="{() => status = 'outro started'}"
		on:introend="{() => status = 'intro ended'}"
		on:outroend="{() => status = 'outro ended'}"
	>
		Flies in and out
	</p>
{/if}
```

---

Local transitions only play when the block they belong to is created or destroyed, *not* when parent blocks are created or destroyed.

```sv
{#if x}
	{#if y}
		<p transition:fade>
			fades in and out when x or y change
		</p>

		<p transition:fade|local>
			fades in and out only when y changes
		</p>
	{/if}
{/if}
```


#### in:*fn*/out:*fn*

```sv
in:fn
```
```sv
in:fn={params}
```
```sv
in:fn|local
```
```sv
in:fn|local={params}
```

```sv
out:fn
```
```sv
out:fn={params}
```
```sv
out:fn|local
```
```sv
out:fn|local={params}
```

---

Similar to `transition:`, but only applies to elements entering (`in:`) or leaving (`out:`) the DOM.

Unlike with `transition:`, transitions applied with `in:` and `out:` are not bidirectional — an in transition will continue to 'play' alongside the out transition, rather than reversing, if the block is outroed while the transition is in progress. If an out transition is aborted, transitions will restart from scratch.

```sv
{#if visible}
	<div in:fly out:fade>
		flies in, fades out
	</div>
{/if}
```



#### animate:*fn*

```sv
animate:name
```

```sv
animate:name={params}
```

```js
animation = (node: HTMLElement, { from: DOMRect, to: DOMRect } , params: any) => {
	delay?: number,
	duration?: number,
	easing?: (t: number) => number,
	css?: (t: number, u: number) => string,
	tick?: (t: number, u: number) => void
}
```

```js
DOMRect {
	bottom: number,
	height: number,
	​​left: number,
	right: number,
	​top: number,
	width: number,
	x: number,
	y: number
}
```

---

An animation is triggered when the contents of a [keyed each block](docs#each) are re-ordered. Animations do not run when an element is removed, only when the each block's data is reordered. Animate directives must be on an element that is an *immediate* child of a keyed each block.

Animations can be used with Svelte's [built-in animation functions](docs#svelte_animate) or [custom animation functions](docs#Custom_animation_functions).

```sv
<!-- When `list` is reordered the animation will run-->
{#each list as item, index (item)}
	<li animate:flip>{item}</li>
{/each}
```

##### Animation Parameters

---

As with actions and transitions, animations can have parameters.

(The double `{{curlies}}` aren't a special syntax; this is an object literal inside an expression tag.)

```sv
{#each list as item, index (item)}
	<li animate:flip="{{ delay: 500 }}">{item}</li>
{/each}
```

##### Custom animation functions

---

Animations can use custom functions that provide the `node`, an `animation` object and any `paramaters` as arguments. The `animation` parameter is an object containing `from` and `to` properties each containing a [DOMRect](https://developer.mozilla.org/en-US/docs/Web/API/DOMRect#Properties) describing the geometry of the element in its `start` and `end` positions. The `from` property is the DOMRect of the element in its starting position, the `to` property is the DOMRect of the element in its final position after the list has been reordered and the DOM updated.

If the returned object has a `css` method, Svelte will create a CSS animation that plays on the element.

The `t` argument passed to `css` is a value that goes from `0` and `1` after the `easing` function has been applied. The `u` argument is equal to `1 - t`.

The function is called repeatedly *before* the animation begins, with different `t` and `u` arguments.


```sv
<script>
	import { cubicOut } from 'svelte/easing';

	function whizz(node, { from, to }, params) {

		const dx = from.left - to.left;
		const dy = from.top - to.top;

		const d = Math.sqrt(dx * dx + dy * dy);

		return {
			delay: 0,
			duration: Math.sqrt(d) * 120,
			easing: cubicOut,
			css: (t, u) =>
				`transform: translate(${u * dx}px, ${u * dy}px) rotate(${t*360}deg);`
		};
	}
</script>

{#each list as item, index (item)}
	<div animate:whizz>{item}</div>
{/each}
```

---


A custom animation function can also return a `tick` function, which is called *during* the animation with the same `t` and `u` arguments.

> If it's possible to use `css` instead of `tick`, do so — CSS animations can run off the main thread, preventing jank on slower devices.

```sv
<script>
	import { cubicOut } from 'svelte/easing';

	function whizz(node, { from, to }, params) {

		const dx = from.left - to.left;
		const dy = from.top - to.top;

		const d = Math.sqrt(dx * dx + dy * dy);

		return {
		delay: 0,
		duration: Math.sqrt(d) * 120,
		easing: cubicOut,
		tick: (t, u) =>
			Object.assign(node.style, {
				color: t > 0.5 ? 'Pink' : 'Blue'
			});
	};
	}
</script>

{#each list as item, index (item)}
	<div animate:whizz>{item}</div>
{/each}
```

### Component directives

#### [on:*eventname*](on_component_event)

```sv
on:eventname={handler}
```

---

Components can emit events using [createEventDispatcher](docs#createEventDispatcher), or by forwarding DOM events. Listening for component events looks the same as listening for DOM events:

```sv
<SomeComponent on:whatever={handler}/>
```

---

As with DOM events, if the `on:` directive is used without a value, the component will *forward* the event, meaning that a consumer of the component can listen for it.

```sv
<SomeComponent on:whatever/>
```


#### [bind:*property*](bind_component_property)

```sv
bind:property={variable}
```

---

You can bind to component props using the same syntax as for elements.

```sv
<Keypad bind:value={pin}/>
```

#### [bind:this](bind_component)

```sv
bind:this={component_instance}
```

---

Components also support `bind:this`, allowing you to interact with component instances programmatically.

> Note that we can't do `{cart.empty}` since `cart` is `undefined` when the button is first rendered and throws an error.

```sv
<ShoppingCart bind:this={cart}/>

<button on:click={() => cart.empty()}>
	Empty shopping cart
</button>
```



### `<slot>`

```sv
<slot><!-- optional fallback --></slot>
```
```sv
<slot name="x"><!-- optional fallback --></slot>
```
```sv
<slot prop={value}></slot>
```

---

Components can have child content, in the same way that elements can.

The content is exposed in the child component using the `<slot>` element, which can contain fallback content that is rendered if no children are provided.

```sv
<!-- Widget.svelte -->
<div>
	<slot>
		this fallback content will be rendered when no content is provided, like in the first example
	</slot>
</div>

<!-- App.svelte -->
<Widget></Widget> <!-- this component will render the default content -->

<Widget>
	<p>this is some child content that will overwrite the default slot content</p>
</Widget>
```

#### [`<slot name="`*name*`">`](slot_name)

---

Named slots allow consumers to target specific areas. They can also have fallback content.

```sv
<!-- Widget.svelte -->
<div>
	<slot name="header">No header was provided</slot>
	<p>Some content between header and footer</p>
	<slot name="footer"></slot>
</div>

<!-- App.svelte -->
<Widget>
	<h1 slot="header">Hello</h1>
	<p slot="footer">Copyright (c) 2019 Svelte Industries</p>
</Widget>
```

#### [`$$slots`](slots_object)

---

`$$slots` is an object whose keys are the names of the slots passed into the component by the parent. If the parent does not pass in a slot with a particular name, that name will not be a present in `$$slots`. This allows components to render a slot (and other elements, like wrappers for styling) only if the parent provides it.

Note that explicitly passing in an empty named slot will add that slot's name to `$$slots`. For example, if a parent passes `<div slot="title" />` to a child component, `$$slots.title` will be truthy within the child.

```sv
<!-- Card.svelte -->
<div>
	<slot name="title"></slot>
	{#if $$slots.description}
		<!-- This <hr> and slot will render only if a slot named "description" is provided. -->
		<hr>
		<slot name="description"></slot>
	{/if}
</div>

<!-- App.svelte -->
<Card>
	<h1 slot="title">Blog Post Title</h1>
	<!-- No slot named "description" was provided so the optional slot will not be rendered. -->
</Card>
```

#### [`<slot let:`*name*`={`*value*`}>`](slot_let)

---

Slots can be rendered zero or more times, and can pass values *back* to the parent using props. The parent exposes the values to the slot template using the `let:` directive.

The usual shorthand rules apply — `let:item` is equivalent to `let:item={item}`, and `<slot {item}>` is equivalent to `<slot item={item}>`.

```sv
<!-- FancyList.svelte -->
<ul>
	{#each items as item}
		<li class="fancy">
			<slot prop={item}></slot>
		</li>
	{/each}
</ul>

<!-- App.svelte -->
<FancyList {items} let:prop={thing}>
	<div>{thing.text}</div>
</FancyList>
```

---

Named slots can also expose values. The `let:` directive goes on the element with the `slot` attribute.

```sv
<!-- FancyList.svelte -->
<ul>
	{#each items as item}
		<li class="fancy">
			<slot name="item" {item}></slot>
		</li>
	{/each}
</ul>

<slot name="footer"></slot>

<!-- App.svelte -->
<FancyList {items}>
	<div slot="item" let:item>{item.text}</div>
	<p slot="footer">Copyright (c) 2019 Svelte Industries</p>
</FancyList>
```


### `<svelte:self>`

---

`<svelte:self>` 要素を使用すると、コンポーネントにそれ自体を再帰的に含めることができます。

マークアップのトップレベルに表示することはできません。また、無限ループを防ぐために、 `if` または `each` ブロック内にある必要があります。

```sv
<script>
	export let count;
</script>

{#if count > 0}
	<p>counting down... {count}</p>
	<svelte:self count="{count - 1}"/>
{:else}
	<p>lift-off!</p>
{/if}
```

### `<svelte:component>`

```sv
<svelte:component this={expression}/>
```

---

`<svelte:component>` 要素は、 `this` プロパティで指定されたコンポーネントのコンストラクタを用いて、コンポーネントを動的にレンダリングします。プロパティが変更されると、コンポーネントは破棄されて再生成されます。

`this` の値が falsy である場合、コンポーネントはレンダリングされません。

```sv
<svelte:component this={currentSelection.component} foo={bar}/>
```


### `<svelte:window>`

```sv
<svelte:window on:event={handler}/>
```
```sv
<svelte:window bind:prop={value}/>
```

---

The `<svelte:window>` element allows you to add event listeners to the `window` object without worrying about removing them when the component is destroyed, or checking for the existence of `window` when server-side rendering.

```sv
<script>
	function handleKeydown(event) {
		alert(`pressed the ${event.key} key`);
	}
</script>

<svelte:window on:keydown={handleKeydown}/>
```

---

You can also bind to the following properties:

* `innerWidth`
* `innerHeight`
* `outerWidth`
* `outerHeight`
* `scrollX`
* `scrollY`
* `online` — an alias for window.navigator.onLine

All except `scrollX` and `scrollY` are readonly.

```sv
<svelte:window bind:scrollY={y}/>
```


### `<svelte:body>`

```sv
<svelte:body on:event={handler}/>
```

---

As with `<svelte:window>`, this element allows you to add listeners to events on `document.body`, such as `mouseenter` and `mouseleave` which don't fire on `window`.

```sv
<svelte:body
	on:mouseenter={handleMouseenter}
	on:mouseleave={handleMouseleave}
/>
```


### `<svelte:head>`

```sv
<svelte:head>...</svelte:head>
```

---

This element makes it possible to insert elements into `document.head`. During server-side rendering, `head` content is exposed separately to the main `html` content.

```sv
<svelte:head>
	<link rel="stylesheet" href="tutorial/dark-theme.css">
</svelte:head>
```


### `<svelte:options>`

```sv
<svelte:options option={value}/>
```

---

The `<svelte:options>` element provides a place to specify per-component compiler options, which are detailed in the [compiler section](docs#svelte_compile). The possible options are:

* `immutable={true}` — you never use mutable data, so the compiler can do simple referential equality checks to determine if values have changed
* `immutable={false}` — the default. Svelte will be more conservative about whether or not mutable objects have changed
* `accessors={true}` — adds getters and setters for the component's props
* `accessors={false}` — the default
* `namespace="..."` — the namespace where this component will be used, most commonly "svg"; use the "foreign" namespace to opt out of case-insensitive attribute names and HTML-specific warnings
* `tag="..."` — the name to use when compiling this component as a custom element

```sv
<svelte:options tag="my-custom-element"/>
```
