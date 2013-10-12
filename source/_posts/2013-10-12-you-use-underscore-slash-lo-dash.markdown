---
layout: post
title: "Using Underscore/Lo-Dash and stopping reinvent the wheel"
date: 2013-10-12 12:00
comments: true
author: stefanjudis
categories:
- underscore
- lo-dash
- utilities
---
Many projects include utility libraries like [underscore](http://underscorejs.org/)/[lo-dash](http://lodash.com/) these days. The reasons for that can be different. Either a MV* framework like [Backbone.js](http://backbonejs.org/) is on bord and has it as dependency or the developers just discovered how much value these utility libraries provide. They are able to improve your daily workflow heavily by giving you a bunch of useful functions and helpers that will work across all the common browsers. I am using underscore for quite a long time now and last week I had to discover, that I am still not using all the features and that I reinvent the wheel in some cases for stuff that could (and should) be done by these libraries. <!-- more -->It is not a big deal, if it works at the end, but if you load underscore/lo-dash you should get out all of it. Additionally there is to say, that these libraries are built to give you the best solution performance wise. So the chance is quite high, that your solution will not perform as well (and as quick) as the underscore/lo-dash implementation does. Performance is important, so let us go the "quicker way"!

I see this blog as a kind of ressource for myself to store and discover stuff, that I want to remember and not to forget. To get a bit better at using these libraries I decided to have a more detailed look at the functions I do not use and to figure out use cases to make my code better and my workflow quicker in the future.

So let us dive into that and imagine my friend and colleague Bob comes to my table and wants to solve a problem he has. ;)

*Note: If you only have to support modern browsers, you are able to use native Array functions like `map`, `filter` and `reduce`. Then there is probably no need to use underscore/lo-dash for these. They are supported in IE9 and higher.* 

### _.map

> Hey Stefan, I have got a list of persons. How can I **make them all** 5cm **smaller**?

```javascript
var persons = [
  {
    name   : 'Bob',
    height : 195
  },
  {
    name   : 'Stefan',
    height : 183
  }
]

persons = _.map( persons, function( person ) {
  person.height = person.height - 5;
  
  return person;
} );

// => [ { height: 190, name: "Bob" }, { height: 178, name: "Stefan" } ]    
```

### _.pluck

> Hey Stefan, I have got a list of persons. How can I get **all their heights**?

```javascript
var persons = [
  {
    name   : 'Bob',
    height : 195
  },
  {
    name   : 'Stefan',
    height : 183
  }
]

personsHeights = _.pluck( persons, 'height' );

// => [ 195, 183 ]
```

### _.reduce

> Hey Stefan, I have got a list of persons. How can I get the **sum of their heights**?

```javascript
var persons = [
  {
    name   : 'Bob',
    height : 195
  },
  {
    name   : 'Stefan',
    height : 183
  }
]

var heightSum = _.reduce( persons, function( memo, person ) {
  return memo + person.height;
}, 0 );

// => 378
```


> Hey Stefan, I have got a list of persons. How can I get **the biggest height**?

```javascript
var persons = [
  {
    name   : 'Bob',
    height : 195
  },
  {
    name   : 'Stefan',
    height : 183
  }
]

var biggestHeight = _.reduce( persons, function( memo, person ) {
  if ( memo > person.height ) {
    return memo;
  } else {
    return person.height;
  }
}, 0 );

// => 195
```

### _.where

> Hey Stefan, I have got a list of persons. How can I get all entries **where the name is "Bob"**?

```javascript
var persons = [
  {
    name   : 'Bob',
    height : 195
  },
  {
    name   : 'Stefan',
    height : 183
  }
]

var personsNamedBob = _.where( persons, { name : 'Bob' } );

// => [ { name : "Bob", height : 195 } ]
```

### _.every

> Hey Stefan, I have got a list of persons. How can I get to know if they are **all taller than 1.90m**?

```javascript

var persons = [
  {
    name   : 'Bob',
    height : 195
  },
  {
    name   : 'Stefan',
    height : 183
  }
]

var allPersonsTallerThan190 = _.every( persons, function( person ) {
  return ( person.height > 190 );
} );

// => false
```

### _.countBy

> Hey Stefan, I have got a list of persons. How can I **group and count** them **depending on their height**?

```javascript

var persons = [
  {
    name   : 'Bob',
    height : 195
  },
  {
    name   : 'Stefan',
    height : 183
  }
]

var groupedAndCountedPersons = _.countBy( persons, function( person ) {
  if ( person.height >= 180 && person.height < 190 ) {
    return '180-189';
  }
  
  if ( person.height >= 190 && person.height < 200 ) {
  	return '190-199';
  }
  
  return 'outOfRange';
} );

// => { 190-199: 1, 180-189: 1 }
```

### _.size

> Hey Stefan, how can I check **how many attributes** I **set** for a person?

```javascript
var personA = {};

_.size( personA ); // => 0

var personB = {
  height : 200
}

_.size( personB ); // => 1

```

### More complex use case

> Hey Stefan, I have got a list of persons including their incomes. How can I get **the sum of all freelance incomes**?

```javascript
var persons = [
  {
    name   : 'Bob',
    incomes : [ {
    	type  : 'Daily work',
    	value : 2000
      },
      {
        type  : 'Freelance',
        value : 1000
      },
      {
        type  : 'Freelance',
        value : 500
      }      
    ]
  },
  {
    name   : 'Stefan',
    incomes : [
      {
        type  : 'Daily Work',
        value : 750
      },
      {
        type  : 'Freelance',
        value : 2000
      }
    ]
  }
];

var freelanceIncomes = _.reduce( persons, function( memo, person ) {
  return memo + _.reduce( _.where( person.incomes, { type : 'Freelance' } ), function( memo, income ) {
    return memo + income.value;
  }, 0 )
}, 0 );

// => 3500
```

### Conclusion

Especially the last example shows the power of utility libraries (or Array functions). There are no foreach loops and no if-statements needed - no needed extra code to write. A kind of complex operation is done by 4 lines of underscore/lo-dash code. There are always multiple ways to solve a given problem, but doing it with the help of underscore/lo-dash (when it is available) is never a bad choice.

For me that means, especially when I have to deal with Arrays, I will always have a first look inside of the documentation of my utility library and check if the author provides me a way to solve my problem. Solving problems that are already solved should not be my job and **I really should not reinvent the wheel**. ;)

And that is it for today. Thx for reading and ideas/improvement on that are really welcome.