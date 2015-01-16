Regex Tokenizer
===============

Nothing special.  A bare bones regex-based tokenizer.  You can slap it into anything.

It's very rudimentary, slicing off tokens from the input based on which rule matches first.  This may or may not be the behavior you want.  Choose wisely.



Usage
-----

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
t.addRule( /^test/i, 'test', false );
```

Tokenize input by calling the tokenize method:

```javascript
t.tokenize( someInputString );

// or, if this is not a stream, pass true as the second argument.
t.tokenize( someInputString, true );

// or-or, call .flush() afterwards.
t.tokenize( someInputString );
t.flush();
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



Method
------

At its heart, this takes a set of regex:typename pairs and outputs a stream of tokens, calling an onToken call back for every token it receives, in order of being found.  Pairs are tested in the order they were added, so use this to set up precedence rules.

Since this doesn't operate based off of an input stream that knows when it itself ends, you must either provide `true` as the second argument to `tokenize()`, or explicitly call finish() to guarantee no more output will be found.
