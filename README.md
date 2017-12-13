# REPL Goal for JS

For all examples, a single input passed to the REPL will be represented by lines prefixed by `> ` and output from the REPL will be represented by lines prefixed by `< `.

## Grammar Intentions

These are a list of intended usages with grammar examples, production mechanics will be done at a later time.

* Top Level Await

```js
> Date.now() - await {then(f) {setTimeout(() => f(Date.now), 1000;)}}
< 1000
```

* Reused environment record

```js
> let x = 1; let y = 'a'; x
< 1
> let x = 2; x
< 2
> x
< 2
> y
< 'a'
```

* `this` is the global

```js
> typeof this
< "object"
> let x = 1; this.x
< undefined
```

* Sloppy Script by default

```js
> with ({x: 1}) x
< 1
```

* Top level static import

```js
> import {x} from './y';
< undefined
```

* Object literal at start

```js
> {x: 1}
< {"x": 1}
```

* Object literal at start

```js
> {x: 1}
< {"x": 1}
```

* Strict mode DirectiveProlog

```js
> "use strict"; with ({}) {}
< SyntaxError: Strict mode code may not include a with statement
```

* REPL Environment Record is not Global Environment Record

```js
> geval = eval; geval('let x = 1; x')
< 1
> let x = 2; x
< 2
> geval('x')
< 1
```

## Semantic Intentions

These are a list of intended spec changes by adding various things with new semantics.

### REPL Environment Record

An Environment Record that can remove lexical bindings as long as it does not have an associated execution context running.

### ParseREPLInput ( sourceText, realm, REPLEnvironment, hostDefined )

Parses the sourceText and initializes all the lexical bindings declared in `sourceText` on REPLEnvironment, removing them before hand if needed to avoid errors from bindings already being defined.

Returns a REPL Source Text Record.

### REPL Source Text . Evaluate

Evaluates a REPL Source Text Record. Prevents the REPL Environment Record it corresponds to from removing lexical bindings until it finishes, unless they are marked as deletable.

Returns a Promise to the Completion Value. 
