---
layout: post
title: "Why you shouldn't use Backbone's built-in Array functions"
date: 2013-08-13 23:15
author: stefanjudis
comments: true
categories: 
- backbone
- performance
---

Today I ran into an issue that let me dive into the Backbone.js library. Very often developers just use the stuff that is in there because it just works fine, but when it does not do what it is supposed to do the best way is to have a look into the used library and figure out what is going on.

My case was the following:

I had some lines for getting the index of a particular model stored inside of a Backbone collection.

<!-- more -->

```js
var index = collection.indexOf( model );
```

Not very complicated and it worked immediately. I did not really think about it. A colleague came around and pointed me into the direction, that the native Array function *indexOf* is only supported in IE9 and higher. I changed it to use the particular LoDash/Underscore utility function and it did not work anymore.

```js
var index = _.indexOf( collection, model );
```

So, what was going on? After thinking about it, the idea that a Backbone collection uses the native Array function *indexOf* does not even make sense. So where does it come from? I checked the code and noticed that Backbone collections implement the Array functions of LoDash/Underscore. I created a [JSBin](http://jsbin.com/apowiz/25/edit) to try this principle myself. 

```js
var ExampleObject = function( array ) {
   this.array = array; 
};

// the methods we want to implement
// in our object
var utilityMethods = [ 'indexOf', 'filter' ],
    slice          = [].slice;

// iterate over the wished methods
_.each( utilityMethods, function( method ) {
  ExampleObject.prototype[ method ] = function() {
    var args = slice.call( arguments );
    
    args.unshift( this.array );
    
    return _[ method ].apply( _, args );
  };
});

// create the new object
var example = new ExampleObject( [ 5, 4, 3, 2, 1 ] );

// proof that indexOf works
console.log( example.indexOf( 2 ) ); // output: 3

// proof that filter workds
console.log(
  example.filter( function( value ) {
    return value % 2;
  } )
); // output: [5, 3, 1]
```

Let us dive into the stuff that goes on inside of the each loop.

```js
ExampleObject.prototype[ method ] = function() {};
```

That is a basic pattern to extent the prototype of JavaScript object. In our case the prototype is *ExampleObject* and this way it has a method that is shared by every instance of *ExampleObject*.

```js
var args = slice.call( arguments );
```

This line was my first learning of digging into the Backbone codebase. Calling the Array function *slice* and passing in *arguments* is a common way to receive an Array out of the arguments object, which is available in every JavaScript function ( more information about that [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/arguments) ). My usual way to do that is *Array.prototype.slice( arguments )*. Backbone caches the slice function before using it for better performance ( which is proved [here](http://jsperf.com/array-prototype-slice-call-vs-slice-call/7)). It is not huge difference, but when we can be faster, we should be faster. ;)

```js
args.unshift( this.array );
```

This line puts the actual array ( -> *this.array* ) to this start of this new generated "arguments array" to make it fit into the way LoDash/Underscore functions expect the handed in arguments to be. 

```js
return _[ method ].apply( _, args );
```

And this is the final result. It is an actual call to the origin LoDash/Underscore function in the context of Underscore/LoDash. The context needs to be set to LoDash/Underscore itself, because we don't know, if the library calls other methods contained in itself. To sum up this line for example of *indexOf* is similar to:

```js
example.indexOf( 2 );
// calls under the hood
_.indexOf( example.array, 2 );
```

No magic. but a nice way to make the usability of your objects more convenient. 

But there is also a bad part about that. I checked the performance of the origin utility methods and the integrated ones and there is really a huge difference ( check jsperf [here](http://jsperf.com/integrated-utility-function-vs-utility-function-2) ). On my machine the new integrated functions are around 75% (!) percent slower ( tested in Firefox, Chrome and Opera ). 75% is really way too much for convenience.

{% img /images/blog/stefanjudis/utilityFunctions.png 970 387 'result of jsperf' 'result of jsperf' %}

The reason for that difference is probably the needed context binding for the integrated utility functions to provide the LoDash/Underscore context inside of the function itself. But that is only guessing. ;)

No matter what, for me is clear that I will use the original utility functions instead of the integrated ones.

Thx for reading. Any ideas on that are really welcome. :)