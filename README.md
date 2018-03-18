# REPL Goal for JS

For all examples, a single input passed to the REPL will be represented by lines prefixed by `> ` and output from the REPL will be represented by lines prefixed by `< `.

* [V8 Issue](https://bugs.chromium.org/p/v8/issues/detail?id=6903)
* [Related Node PR](https://github.com/nodejs/node/pull/17285)

## Grammar Intentions

These are a list of intended usages with grammar examples, production mechanics will be done at a later time.

* Top Level Await

```js
> Date.now() - await {then(f) {setTimeout(() => f(Date.now), 1000;)}}
< 1000
```

* Reused environment record that removes conflicting bindings between evluations

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

* Sloppy by default

```js
> with ({x: 1}) x
< 1
> let x = 2; x
< 2
> eval('let x =3; x')
< 3
> x
< 3
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

* Strict mode DirectivePrologue

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

### Productions

```
REPL Input :
  [lookahead = {] ObjectLiteral[+Await]
  [lookahead != {] Script[+Await, +Import]

ScriptBody:
  ScriptItemList[?Await, ?Import]

ScriptItemList:
  ScriptItem[?Await, ?Import]
  ScriptItemList ScriptItem[?Await, ?Import]

ScriptItem:
  if (+import) ImportDeclaration
  StatementListItem[~Yield, ?Await, ?Import, ~Return]
```

## Semantic Intentions

These are a list of intended spec changes by adding various things with new semantics.

1. Allow the repeated running of lines from scrollback history.

Allow repeated running of the same input even if the input would collide with existing bindings on the REPL Environment Record.

2. Abililty to remove bindings that have permanently been stuck in the Temporal Dead Zone.

Allow cleaning up of bindings that have become stuck in a TDZ.

3. Separation from the global scope.

Allow a separation from the Global Environment in order to preserve logical consistency of how bindings work and giving a level of isolation to prevent leaving accidental bindings on the global when using tools such as a debugger.

### REPL Lexical Environment

A Lexical Environment with an outer environment reference to a Global Lexical Environment. A REPL environment's Environment Record may be shared amongst multiple REPL Input Records.

### REPL Input Record

A REPL Input Record encapsulates information about a repl input being evaluated. Each REPL Input Record contains the fields.

#### REPL Input Record Fields

Field Name | Value Type | Meaning
---- | ---- | ----
[[Realm]]	Realm Record | undefined | The realm within which this REPL Input Record was created. undefined if not yet assigned.
[[Environment]]	Lexical Environment | a REPL Lexical Environment | The Lexical Environment containing the top level bindings for this REPL Input Record. This field is set when the REPL Input Record is instantiated.
[[ECMAScriptCode]] | a Parse Node | The result of parsing the source text of this module using REPL as the goal symbol.
[[HostDefined]] | Any, default value is undefined. | Field reserved for use by host environments that need to associate additional information with a REPL Input Record.

### REPL Environment Record

A REPL Environment Record is a declarative Environment Record that is used to represent the top-level scope of a REPL. It may be reused amongst multiple REPL Input Records.

### ParseREPLInput ( sourceText, realm, REPLEnvironment, hostDefined )

Parses the sourceText and initializes all the lexical bindings declared in `sourceText` on a REPL Environment, removing them before hand if needed to avoid errors from bindings already being defined. All lexical bindings declared in `sourceText` may be deleted.

Returns a REPL Input Record.

### REPL Input . Evaluate

Evaluates a REPL Input Record.

Returns a Promise to the Completion Value. 
