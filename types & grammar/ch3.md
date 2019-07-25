#  Types & Grammar
# Chapter 3: Natives

Commonly used natives:

* `String()`
* `Number()`
* `Boolean()`
* `Array()`
* `Object()`
* `Function()`
* `RegExp()`
* `Date()`
* `Error()`
* `Symbol()` -- added in ES6!

These natives are actually **built-in functions**.

|Natives                |Use Constructor Form                        |is new Optional                         |
|----------------|-------------------------------|-----------------------------|
|String()	 |No          			 |            		       |   			      |
|Number()        |No  				             |   
|Boolean()       |No, `if(Boolean(false)) // is  true`   considered as object'						 |				|
|Array()		 |No,bcz with single parameter it creates empty slots, with more params it considers as elements.					 |	Yes			|
|Object()        |No							|yes
|Function()		 |Mostly No , only If params and body are dynamic ||
|RegExp()		 |Some times when dynamic expression	||
|Date()			 |yes, only way, defaults to current date, ES 5-> Date.now(), ||
|Error()		 |yes, only way|Yes|
|Symbol()		 |yes, only way|Yes

|                |                        						|
|----------------|-------------------------------                                       |
|Special Types 	 |Natives are Special Types of Object *(BUILT IN)*                      |
|instanceOf      |To Check Special type use "instanceof"  intead "typeof"               |   
|Object.prototype.toString()		 |To Find Special type => `Object.prototype.toString([1,2])`				 |	
|Unboxing       |`.valueOf()`						                |
|Prototypes	|apply,call and bind =>Function.prototype                               |

```js
var a = new String( "abc" );

typeof a; // "object" ... not "String"

a instanceof String; // true

Object.prototype.toString.call( a ); // "[object String]"
```

The result of the constructor(`new String("abc")`) is an object wrapper around the primitive (`"abc"`) value.

 `typeof` shows  not their own special *types*.

## Internal `[[Class]]` 

Helps to find the subtype of `"object"` type

`Object.prototype.toString(..)` method called against the value passed. For example:

```js
Object.prototype.toString.call( [1,2,3] );			// "[object Array]"

Object.prototype.toString.call( /regex-literal/i );	// "[object RegExp]"
```

So, for the array in this example, the internal `[[Class]]` value is `"Array"`, and for the regular expression, it's `"RegExp"`. In most cases, this internal `[[Class]]` value corresponds to the built-in native constructor (see below) that's related to the value, but that's not always the case.

What about primitive values? First, `null` and `undefined`:

```js
Object.prototype.toString.call( null );			// "[object Null]"
Object.prototype.toString.call( undefined );	// "[object Undefined]"
```

But for the other simple primitives like `string`, `number`, and `boolean`, another behavior actually kicks in, which is usually called "boxing" (see "Boxing Wrappers" section next):

```js
Object.prototype.toString.call( "abc" );	// "[object String]"
Object.prototype.toString.call( 42 );		// "[object Number]"
Object.prototype.toString.call( true );		// "[object Boolean]"
```

## Boxing Wrappers

Primitive values don't have properties or methods, so to access `.length` or `.toString()` you need an object wrapper around the value.  JS will automatically *box* (aka wrap) the primitive value to fulfill such accesses.

```js
var a = "abc";

a.length; // 3
a.toUpperCase(); // "ABC"
```

Never do things like `new String("abc")`, `new Number(42)`, etc -- always prefer using the literal primitive values `"abc"` and `42`.

### Object Wrapper Gotchas

Consider `Boolean` wrapped values:

```js
var a = new Boolean( false );

if (!a) {
	console.log( "Oops" ); // never runs now a is an object
}
```


If you want to manually box a primitive value, you can use the `Object(..)` function (no `new` keyword):

```js
var a = "abc";
var b = new String( a );
var c = Object( a );

typeof a; // "string"
typeof b; // "object"
typeof c; // "object"

b instanceof String; // true
c instanceof String; // true

Object.prototype.toString.call( b ); // "[object String]"
Object.prototype.toString.call( c ); // "[object String]"
```

Use Boxed object wrapper sparingly with caution

## Unboxing

to get the underlying primitive value out, you can use the `valueOf()` method:

```js
var a = new String( "abc" );
var b = new Number( 42 );
var c = new Boolean( true );

a.valueOf(); // "abc"
b.valueOf(); // 42
c.valueOf(); // true
```

## Natives as Constructors

For `array`, `object`, `function`, and regular-expression values,  use the literal form , to create  same sort of object as constructor. 

Avoid unless you really know you need them, they introduce exceptions and gotchas.

### `Array(..)`

```js
var a = new Array( 1, 2, 3 );
a; // [1, 2, 3]

var b = [1, 2, 3];
b; // [1, 2, 3]
```

**Note:** For `Array(..)` constructor `new` keyword is optional. `Array(1,2,3)` is the same outcome as `new Array(1,2,3)`.

If only one `number` argument is passed, instead of providing that value as *contents* , it's taken as a length to "presize the array".

You're creating an  empty array, but setting the `length` property of the array to the numeric value specified.

**Note:** An array with at least one "empty slot" in it is often called a "sparse array."

To visualize the difference, try this:

```js
var a = new Array( 3 );
var b = [ undefined, undefined, undefined ];
var c = [];
c.length = 3; // this line introduces empty slot

a;     // `[ undefined x 3 ]`
b;    //  `[ undefined, undefined, undefined ]`
c;   //   `[ undefined x 3 ]`
```
The above code snippet actually behave the same in some cases **but differently in others**:

```js
a.join( "-" ); // "--"  it just uses index
b.join( "-" ); // "--"

a.map(function(v,i){ return i; }); // [ undefined x 3 ]  it uses the element which is empty slot now
b.map(function(v,i){ return i; }); // [ 0, 1, 2 ]
```

Bottom line: **never ever, under any circumstances**, should you intentionally create and use these exotic empty-slot arrays. Just don't do it. They're nuts.

### `Object(..)`, `Function(..)`, and `RegExp(..)`

The `Object(..)`/`Function(..)`/`RegExp(..)` constructors are optional avoid it unless a special case.

There's practically no reason to ever use the `new Object()` 
```js
var c = new Object();
c.foo = "bar";
c; // { foo: "bar" }
```

The `Function` constructor is helpful only in the rarest of cases, where you need to dynamically define a function's parameters and/or its function body. **Do not just treat `Function(..)` as an alternate form of `eval(..)`.** You will almost never need to dynamically define a function in this way.
```js
var e = new Function( "a", "return a * 2;" );
var f = function(a) { return a * 2; };
function g(a) { return a * 2; }
```

Regular expressions defined in the literal form (`/^a*b+/g`) for performance reasons -- the JS engine precompiles and caches them before code execution.  `RegExp(..)` has some reasonable utility: to dynamically define the pattern for a regular expression.

```js
var name = "Mano";
var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );

var matches = someText.match( namePattern );
```

### `Date(..)` and `Error(..)`

The `Date(..)` and `Error(..)` native constructors are much more useful than the other natives, because there is no literal form for either.

To create a date object value, you must use `new Date()`. The `Date(..)` constructor accepts optional arguments to specify the date/time , if omitted, current date/time is assumed.

Common reason you construct a date object is to get the current timestamp value  `getTime()` in ES5 `Date.now()`.

But an even easier way is to just call the static helper function defined as of ES5: `Date.now()`. And to polyfill that for pre-ES5 is pretty easy:

```js
if (!Date.now) {
	Date.now = function(){
		return (new Date()).getTime();
	};
}
```

**Note:** If you call `Date()` without `new`, you'll get back a **string** representation of the date/time at that moment. `"Fri Jul 18 2014 00:31:02 GMT-0500 (CDT)"`.

The `Error(..)` constructor `new` is optional.

Error object current execution stack context into the object. This stack context includes the function call-stack and the line-number where the error object was created, which makes debugging that error much easier.

Use such an error object with the `throw` operator:

```js
function foo(x) {
	if (!x) {
		throw new Error( "x wasn't provided" );
	}
	// ..
}
```

Error object instances generally have at least a `message` property, and sometimes other properties like `type`. `toString()` on the error object to get a friendly-formatted error message.

**Tip:** Technically, in addition to the general `Error(..)` native, there are several other specific-error-type natives: `EvalError(..)`, `RangeError(..)`, `ReferenceError(..)`, `SyntaxError(..)`, `TypeError(..)`, and `URIError(..)`. But it's very rare to manually use these specific error natives. They are automatically used if your program actually suffers from a real exception.

### `Symbol(..)`

New as of ES6,primitive value type "Symbol". Symbols are special "unique" (not strictly guaranteed!) values that can be used as properties on objects with little fear of any collision. They're primarily designed for special built-in behaviors of ES6 constructs, but you can also define your own symbols.

There are several predefined symbols in ES6, accessed as static properties of the `Symbol` function object, like `Symbol.create`, `Symbol.iterator`, etc. To use them, do something like:

```js
obj[Symbol.iterator] = function(){ /*..*/ };
```

To define your own custom symbols, use the `Symbol(..)` native. The `Symbol(..)` native "constructor" is unique in that you're not allowed to use `new` with it, as doing so will throw an error.

```js
var mysym = Symbol( "my own symbol" );
mysym;				// Symbol(my own symbol)
mysym.toString();	// "Symbol(my own symbol)"
typeof mysym; 		// "symbol"

var a = { };
a[mysym] = "foobar";

Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]
```

**Note:** `Symbol`s are *not* `object`s, they are simple scalar primitives.

### Native Prototypes

Each of the built-in native constructors has its own `.prototype` object -- `Array.prototype`, `String.prototype`, etc.

These objects contain behavior unique to their particular object subtype.

For example, all string objects, and by extension (via boxing) `string` primitives, have access to default behavior as methods defined on the `String.prototype` object.

**Note:** By documentation convention, `String.prototype.XYZ` 

By virtue of prototype delegation (see the *this & Object Prototypes* title in this series), any string value can access these methods:

```js
var a = " abc ";

a.indexOf( "c" ); // 3
a.toUpperCase(); // " ABC "
a.trim(); // "abc"
```

The other constructor prototypes contain behaviors appropriate to their types. All functions have access to `apply(..)`, `call(..)`, and `bind(..)` because `Function.prototype` defines them.

But, some of the native prototypes aren't *just* plain objects:

```js
typeof Function.prototype;			// "function"
Function.prototype();				// it's an empty function!

RegExp.prototype.toString();		// "/(?:)/" -- empty regex
"abc".match( RegExp.prototype );	// [""]
```

As you can see, `Function.prototype` is a function, `RegExp.prototype` is a regular expression, and `Array.prototype` is an array. Interesting and cool, huh?

#### Prototypes As Defaults

`Function.prototype` being an empty function, `RegExp.prototype` being an "empty" (e.g., non-matching) regex, and `Array.prototype` being an empty array, make them all nice "default" values to assign to variables if those variables wouldn't already have had a value of the proper type.


**Note:** While we're pointing out these native prototypes and some usefulness, be cautious of relying on them and even more wary of modifying them in any way. See Appendix A "Native Prototypes" for more discussion.

## Review

JavaScript provides object wrappers around primitive values, known as natives (`String`, `Number`, `Boolean`, etc). These object wrappers give the values access to behaviors appropriate for each object subtype (`String#trim()` and `Array#concat(..)`).

If you have a simple scalar primitive value like `"abc"` and you access its `length` property or some `String.prototype` method, JS automatically "boxes" the value (wraps it in its respective object wrapper) so that the property/method accesses can be fulfilled.
