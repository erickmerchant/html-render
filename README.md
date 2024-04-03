# html-render

A tiny front-end framework using a fluent interface for constructing UI. Also has shallow reactivity. Only 1.3 kB minified and compressed. Also it's fully tree-shakeable if you use a bundler like rollup. Currently it is hosted on JSR. Install with `deno add @erickmerchant/html-render`. See the examples directory for usage.

## API

### `watch` and `effect`

The reactivity API. `watch` will make an object reactive so that any time a property changes any effects that read that prop are rerun. `effect` is for effects that are not part of the DOM. For instance setting localStorage or something like that.

If Signals land in browsers in the near future, they will be used as the underlying mechanism of the reactive API.

```javascript
let state = watch({
	hasError: false,
	name: "World"
	list: watch([
		"a",
		"b",
		"c"
	])
})

effect(() => {
	localStorage.setItem("my-state", JSON.stringify(state));
})
```

### `html`, `svg`, and `math`

These three are proxies for getting functions to call to construct DOM elements. There are three seperate proxies, because they each have a specific namespace that must be used when creating elements.

For example:

```javascript
import {html} from "@erickmerchant/html-render";

let {div} = html;

div(); // Node {children: [], props: [], name: 'div', namespace: 'http://www.w3.org/1999/xhtml'}
```

### `mixin`

In order to be fully tree-shakeable you must declare what methods you'll use from the fluent interface.

For instance this will throw an error, because we haven't said we need `classes`.

```javascript
import {html} from "@erickmerchant/html-render";

let {div} = html;

div().classes("my-div");
```

But this will work:

```javascript
import {html, classes, mixin} from "@erickmerchant/html-render";

let {div} = html;

mixin(classes);

div().classes("my-div");
```

In this way you're forced to import nearly everything you'll use, but if you use for instance rollup then you won't end up with unused code.

### `node.attr`, `node.prop`, `node.on`, `node.classes`, `node.styles`, and `node.data`

These are part of the fluent interface, and are all ways of defining props, attributes, and events.

```javascript
form()
	.classes("my-form", {
		error: () => state.hasError,
	})
	.attr("action", () => state.endpoint)
	.attr("method", "POST")
	.data({
		formId: () => state.formId,
	})
	.on("submit", async (e) => {
		e.preventDefault();

		let formData = new FormData(e.currentTarget);

		try {
			await fetch(state.endpoint, {
				method: "POST",
				body: formData,
			});
		} catch (error) {
			state.hasError = true;
		}
	})
	.append(
		button()
			.prop("type", "button")
			.prop("disabled", () => state.disabled)
			.styles({color: () => (state.hasError ? "red" : "green")})
			.on("click", () => {
				console.log("submiting the form");

				state.disabled = true;
			})
			.append("Submit")
	);
```

With the exception of `node.on`, because it already takes a closure as an argument, many of these methods take a function in places where they could also take a literal value. In these cases these become effects that will run again when state changes. It's important to note though that you don't have to use a closure to use state. It's just if you want the attribute or prop to update when state changes.

`on` can also take an array as its first argument to attach a handler to multiple events, and it accepts an optional third argument just like addEventListener.

### `node.append`

`append` is the primary way to add children to a node.

For instance in this example you construct a paragraph with two children. The first the literal string "hello", and the second a closure that provides a value from state. When `state.name` updates just that text node's `nodeValue` will update.

```javascript
p().append("hello", () => state.name);
```

### `node.map`

This is the second way to add children. It's used to add a chunk of ui for every item in a watched list. You pass it a watched array, and a function for producing either `null` (skip this element), or either function that produces nodes, or nodes. It's most efficient to provide a function though, so that each item isn't rerendered every single time the array changes. `ctx` is an object with two properties, `item` and `index`, where `item` is an item from the array, and `index` is its position. It will get automatically passed to the function returned each iteration.

```javascript
ol().map(state.list, (ctx) => {
	if (ctx.item.show) return liView;

	return null;
});

function liView(ctx) {
	return li().text(ctx.item);
}
```

### `attr`, `prop`, `on`, `classes`, `styles`, `data`, `append`, and `map`

The above mentioned methods can also be used outside the fluent interface. In those forms they take a DOM element as their first argument. For example:

```javascript
import {classes, watch} from "@erickmerchant/html-render";

let state = watch({
	a: true,
	b: false,
	c: false,
});
let element = document.querySelector("#my-element");

classes(element, {
	a: () => state.a,
	b: () => state.b,
	c: () => state.c,
});
```

And if that trivial example is tree-shaken you should just end up with the code for classes, and the reactive API, which will be far less than 1.3 kB.

Use `append` in this form once you have constructed everything, to put it into your document.

```javascript
append(target, div().text("I'm a div"));
```

## Prior Art

- [jQuery](https://github.com/jquery/jquery)
- [Ender](https://github.com/ender-js/Ender)
- [HyperScript](https://github.com/hyperhype/hyperscript)
- [@vue/reactivity](https://github.com/vuejs/core/tree/main/packages/reactivity)
- [Solid](https://www.solidjs.com/)
- [html](https://github.com/yoshuawuyts/html)
