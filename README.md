Regex Tokenizer
===============

Nothing special.  A bare bones regex-based tokenizer.  You can slap it into anything.

It's very rudimentary, slicing off tokens from the input based on which rule matches first.  This may or may not be the behavior you want.  Choose wisely.

This has no concerns for working asynchronously.  Add async support in a subclass or wrapper.



Method
------

At its heart, this takes a set of `[regex,typename]` pairs and outputs a stream of tokens, calling an onToken call back for every token it receives, in order of being found.  Pairs are tested in the order they were added, so use this to set up precedence rules.

Since this doesn't operate based off of an input stream that knows when it itself ends, you must either provide `true` as the second argument to `tokenize()`, or explicitly call finish() to guarantee no more output will be found.



API
---

### Tokenizer

```
Tokenizer( options )
```

Creates a new Tokenizer.

- `options`: Plain Object; An object with the following properties:
	- `rules`: Array; An array of rules, each rule itself specified as an array of 2 or 3 elements.  See *Tokenizer#addRule* below.
		- Default: `[]`
	- `onToken`: Function( `token` ); A function that gets called every time a token is sliced off the input stream.
		- Default: `function(){}`
		- Properties of `token`:
			- `type`: String; The type of token that was matched.
			- `value`: String; The slice of input that was matched.
	- `onFlush`: Function(); A function that gets called every time the input is flushed, usually when the input stream is finished and itself flushing.
		- Default: `function(){}`



### Tokenizer#tokenize

```
tokenizerInstance.tokenize( stringChunk, noBuffer = false )
```

Tokenizes a chunk of input, optionally treating it as the last or entire portion of the input.

- `stringChunk`: String; A chunk of the input stream.
- `noBuffer`: Boolean; Whether to allow storage of any remaining untokenized input into the internal buffer.  Pass true if the input is the entire contents of a file, or will have no input after this call.
	- Default: `false`



### Tokenizer#flush

```
tokenizerInstance.flush()
```

Flushes the input, tokenizing any remaining input then calling `onFlush()`.  Takes no arguments.



### Tokenizer#addRules

```
tokenizerInstance.addRules( rules )
```

Adds a list of rules to the Tokenizer instance.

- `rules`: Array; Rules to add.  Should contain sub arrays following this structure:
	- `[ ruleTest, tokenType, ignored = false ]`: passed as arguments to `Tokenizer#addRule()`.  See that method for details.



### Tokenizer#addRule

```
tokenizerInstance.addRule( ruleTest, tokenType, ignored = false )
```

Adds a single rule to this instance.

- `ruleTest`: RegExp | String | Function; The test which is applied to the input.
- `tokenType`: String; The type of the token.
- `ignored`: Boolean, optional; Whether this token type should be ignored rather than output through `onToken()`.
	- Default: `false`.

#### ruleTest

The parameter `ruleTest` has a couple different behaviors based on its type:
- RegExp: Has `#match` called against the input string chunk.  If it matches, `match[ 0 ]` is passed back as the slice.
- String: If this is found at the beginning of the input, it itself passed back as the slice.
- Function: `ruleTest` will be called with the whole current input, and should return a slice of the input if it matches, or null if it does not match.  Should only return a slice from the beginning of the input, or else nasal demons may result.



Example
-------

You can initialize everything from the get go, or assign things later.

```javascript
var Tokenizer = require( 'tokenizer' );

// pass rules in directly.
var t = new Tokenizer({ rules: rules, onToken: onToken, onFlush: onFlush });

// or add rules later:
t.addRules([
	[ 'string', 'tokenType' ],
	[ /^regex/, 'otherTokenType' ],
	[ /^\s+/, 'ignoredWhitespace', true ] // true as third item to ignore.
]);

// or add rules one at a time:
t.addRule( 'string', 'tokenType' );
t.addRule( /^regex/, 'otherTokenType' );
t.addRule( /^\s+/, 'ignoredWhitespace', true ); // true as third arg to ignore.
```

Tokenize input by calling the tokenize method:

```javascript
t.tokenize( someInputString );

// Then when the stream closes,
t.flush();

// or, if this is not a stream, pass true as the second argument to tokenize().
// This will cause the tokenizer to flush itself afterwards.
t.tokenize( someInputString, true );
```

Receive tokens one at a time through the onToken callback:

```javascript
function onToken( token ) {
	console.log( 'Got a token!', token.type, '=', token.value );
}
```

The input string is done when onFlush is called.

```javascript
function onFlush() {
	console.log( 'And it is done.' );
}
```
