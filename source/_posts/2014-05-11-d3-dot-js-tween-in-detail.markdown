---
layout: post
title: "Using basic and tween transitions in d3.js"
date: 2014-05-11 13:25
comments: true
author : stefanjudis
categories:
- d3
- javascript
- svg
---

Playing around at [CodePen](http://codepen.io) is one of my favourite activities, when doing 'nothing' lately. Especially the combination of [dribbble](http://dribbble.com) and CodePen is super nice. Browsing for beautiful designs and ideas on dribbble and putting them into code became a nice activity for me. There are a lot of great people hanging around at dribbble and you can get a lot of inspiration over there.

Over the last few month I started being interested in dashboards and how to analyze/visualize data. Unfortunately I am not a designer and this is why I really enjoy to just build dashboards according to a dribbble design.

<!-- more -->

### Dashboard examples

Two example of these tryouts are the following. They include some basic [d3.js](d3js.org) charting and make usage of a few different kinds of animation to make them look pretty and have some fun on my side.

#### Example 1 - referenced as "bright dashboard" later on

{% img center /images/blog/stefanjudis/d3TweenExample1.jpg 350 401 'Example 1 of d3 dashboard tryout at CodePen' 'Example 1 of d3 dashboard tryout at CodePen' %}

You can check the coded result [here](http://codepen.io/stefanjudis/pen/gkHwJ) and the used design [here](http://dribbble.com/shots/1252153-Charts-Inforgraphic-Templates/attachments/170613).


#### Example 2 - referenced as "dark dashboard" later on

{% img center /images/blog/stefanjudis/d3TweenExample2.jpg 500 341 'Example 2 of d3 dashboard tryout at CodePen' 'Example 2 of d3 dashboard tryout at CodePen' %}

You can check the coded result [here](http://codepen.io/stefanjudis/pen/jawGn) and the used design [here](http://dribbble.com/shots/1244676-Cloud-Storage/attachments/168917).

Last week I got a Tweet by [@jessicard](https://twitter.com/jessicard) asking, if I may want to explain how the `tween` function in d3.js works, because it is used several times.

And here we are - let us dive into basic and `tween` transitions in d3. *I'm really sorry for the delay by the way*...

### Basic animations in d3.js

To achieve basic transition there is no big effort in d3.

```javascript
;( function( d3 ) {
  // grab svg container
  var d3container = d3.select( '.container' );
  // append svg
  var svg = d3container.append( 'svg' );
  // append group element to include some circles
  var dots = svg.append( 'g' );

  // create loop to append some circle
  for ( var i = 1; i <= 5; ++i ) {
    // append circle
    dots.append( 'circle' )
        // set radius to '0' because we want
        // animate it later on
        .attr( 'r', 0 )
        // kind of center it horizontally - I know it is not exactly in the middle ;)
        .attr( 'cx',  150 )
        // give different y position
        .attr( 'cy', i * 25 )
        // add event handler
        .on( 'mouseenter', function() {
          // select element in current context
          d3.select( this )
            // add transition
            .transition()
            // change attribute
            .attr( 'r', 10 );
        } )
        // add event handler
        .on( 'mouseleave', function() {
          // select element in current context
          d3.select( this )
            // add transition
            .transition()
            // change attribute
            .attr( 'r', 6 );
        } )
        // add transition for kick off
        .transition()
        // add delay so that it looks nice
        .delay( 200 * i )
        // set radius to wished size
        .attr( 'r', 6 );
  }
} )( d3 );
```

There is not much magic going on there. `d3` does pretty much all the work for us. For more information you can visit the [transition page](https://github.com/mbostock/d3/wiki/Transitions) of the `d3` [wiki](https://github.com/mbostock/d3/wiki). It is basically just setting an attribute to an element, creating a transition and changing the given attribute later on. It will be transitioned *magically*.

You can check the result of these lines of code below (you may have to press *replay* to see the initial animation in action).

{% codepen dafDm stefanjudis result 400 %}

That is already pretty nice, but let us dig a bit deeper. ;)

### Tween animations in d3.js

There are some cases, where basic transitions will just not work. For example you can have a quick look at the following code, where I tried to animate `text` elements showing numbers from zero to 10.

{% codepen xrGLy stefanjudis result 400 %}

It is unfortunately not working with the usage of a basic transition. This is the use case for so called `tweens` in `d3`. The general documentation for `tweens` can be found [here](https://github.com/mbostock/d3/wiki/Transitions#tween).

The important facts about `transition.tween( name, factory )` are described as follows:

> Registers a custom tween for the specified name. When the transition starts, the specified factory function will be invoked for each selected element in the transition, being passed that element's data (d) and index (i) as arguments, with the element as the context (this). The factory should return the tween function to be called over the course of the transition. The tween function is then called repeatedly, being passed the current normalized time t in [0, 1]. If the factory returns null, then the tween is not run on the selected element.

So, but what does that mean? Let us modifiy the not working text transition example and have a deeper look at the reworked example.

```javascript
;( function( d3 ) {
  var d3container = d3.select( '.container' );
  var svg = d3container.append( 'svg' );
  var dots = svg.append( 'g' );

  for ( var i = 1; i <= 5; ++i ) {
    dots.append( 'text' )
        .text( 0 )
        .attr( 'text-anchor', 'middle' )
        .attr( 'x',  150 )
        .attr( 'y', i * 25 )
        .transition()
        .delay( 300 * i )
        .tween( 'text', function() {
          // get current value as starting point for tween animation
          var currentValue = +this.textContent;
          // create interpolator and do not show nasty floating numbers
          var interpolator = d3.interpolateRound( currentValue, 10 );

          // this returned function will be called a couple
          // of times to animate anything you want inside
          // of your custom tween
          return function( t ) {
            // set new value to current text element
            this.textContent = interpolator( t );
          };
        } );
  }
} )( d3 );
```

The first argument of the `tween` function represents the name of your new custom `tween`. The second argument is a function that should behave like a factory and return another function. The goal of this factory function is to create (and return) a function that is able to interpolate data and to calculate values depending on the `current normalized time`. The `current normalize time` are floating values from zero to one. Zero represents the starting point and one represents the end point of your tween animation.

This means that this return function must be able to do something depending on the `current normalized time`(see line 23).
`d3` already offers a lot of functions to solve exactly this problem - the so called `interpolators`. In this example I made usage of `interpolatorRound`. Let us check what that stands for and see what is actually doing by playing just a bit around in the JavaScript console.

```
// creating new interpolator
var i = d3.interpolateRound( 0, 100 );
> undefined
// using 0.25 as example for 'current normalized time' -> animation start
i( 0 )
> 0
// using 0.25 as example for 'current normalized time'
i( 0.25 )
> 25
// using 0.5 as example for 'current normalized time'
i( 0.5 )
> 50
// using 0.75 as example for 'current normalized time'
i( 0.75 )
> 75
// using 1 as example for 'current normalized time' -> animation end
i( 1 )
> 100
```

`d3` offers a lot of different interpolators. In our case I used `interpolatorRound`, because it returns the nearest integer value and I do not want to see crazy floating values in our custom `tween`.

Available interpolators are:

- `d3.interpolatNumber`
- `d3.interpolatRound`
- `d3.interpolatString`
- `d3.interpolatRgb`
- `d3.interpolatHsl`
- `d3.interpolatLab`
- `d3.interpolatHcl`
- `d3.interpolatArray`
- `d3.interpolatObject`
- `d3.interpolatTransform`
- `d3.interpolatZoom`

As you see there are quite a few and they will matched your needs in most of the cases. ;)

This works fine. The next step is to create a interpolator with the current text value as starting point and with our desired value as end point. This is easily achievable, because the factory is executed in the context of the depending element. That means, that we can easily get the current value by using `this.textContent`(see line 16).

But still the question remains, what is going on to animate this particalur text node.
To stick everything together `d3` will execute the `factory` once and is expecting a function as return value. This function will be executed with a ton of values from zero to one (argument `t` in the example) later during the `tween`. These values represent the current progress of the transition. This returned function will additionally be executed in the context of the particalur element(in our case the text node) and exactly this becomes really handy. So we can also set the value easily via `this.textContent`(see line 25).

To make our circles a bit more fun let us add a transition to the `mouseenter` and `mouseleave` events. For that we could duplicate the code which kicks off the transition at the beginning and set a different value to `d3.interpolateRound`, but code duplication is never a good idea. So let us implement a nice little helper function.

```javascript
;( function( d3 ) {
  var d3container = d3.select( '.container' );
  var svg = d3container.append( 'svg' );
  var dots = svg.append( 'g' );

  for ( var i = 1; i <= 5; ++i ) {
    dots.append( 'text' )
        .text( 0 )
        .attr( 'text-anchor', 'middle' )
        .attr( 'x',  150 )
        .attr( 'y', i * 25 )
        .on( 'mouseenter', function() {
          d3.select( this )
            .transition()
            .tween( 'text', tweenText( 50 ) );
        } )
        .on( 'mouseleave', function() {
          d3.select( this )
            .transition()
            .tween( 'text', tweenText( 10 ) );
        } )
        .transition()
        .delay( 300 * i )
        .tween( 'text', tweenText( 10 ) );
  }

  /**
   * Tween functions
   */
  function tweenText( newValue ) {
    return function() {
      // get current value as starting point for tween animation
      var currentValue = +this.textContent;
      // create interpolator and do not show nasty floating numbers
      var i = d3.interpolateRound( currentValue, newValue );

      return function(t) {
        this.textContent = i(t);
      };
    }
  }
} )( d3 );
```

This way it is relatively clean and we have no code duplication in our code. You can check the working result below.

{% codepen HpFiz stefanjudis result 400 %}


So far so good. Let us have a look at a bit more complicated looking `tween`. In both charts there are animated d3 areas, which I animated using `tween`, too.

### Animating areas with `attrTween`

{% codepen nuagm stefanjudis result 400 %}

So let us check the code for that. It basically works the same. ;)

```javascript
;( function( d3 ) {
  var data = [
    {
      date  : '2006-02-22',
      value : 950
    },
    // ...
    // more data
    // ...
    {
      date  : '2006-08-22',
      value : 1000
    }
  ];

  //
  // lot of setup code here
  //

  // set up value to start animation
  var startData = data.map( function( datum ) {
                    return {
                      date  : datum.date,
                      value : 0
                    };
                  } );

  //
  // a bit more stuff here :)
  //

  // Add the area path.
  svg.append( 'path' )
      // set data
      .datum( startData )
      // apply area depending on data
      .attr( 'd', area )
      // create transition
      .transition()
      // set duration of transition
      .duration( 500 )
      // define tween for attribute 'd'
      .attrTween( 'd', function() {
        // create interpolator which will
        // be able to handle `current normalized time`
        var interpolator = d3.interpolateArray( startData, data );

        // function called several times
        // with values from 0.0 to 1.0
        return function( t ) {
          // calculate needed values to
          // represent 'area' path with interpolated Array
          //
          // return it to set it directly to attribute 'd'
          return area( interpolator( t ) );
        }
      } );
  }
} )( d3 );
```

Let us assume you already have set up an area and it is already shown in your SVG - I will not go into detail about this topic here, because it could fill its own blog post. This area is based on an Array representing the data. The solution for animating this I found so far is to duplicate the data via `map` and set all the values to zero(see line 21). It does not have to be zero though - these changed values will basically be the starting point of your animation.

In case of the example code you see `startData`, which represents the animation start and `data` which represents the animation end.

You have to define the transition and set a duration of 500ms. That is all to be ready to go and add our `tween`.

To animate everythings the `d3` method `attrTween` is used. `attrTween` works mostly the same as `tween` with one difference. The wished animated attribute is modified directly when using `attrTween`.

`attrTween` expects the attribute, which should be animated, and a factory as arguments. This factory has to return a function that is able to deal with the `current normalized time`(values from `0.0` to `1.0`).

So how does this work with two Arrays?

`d3` has already an interpolator for that integrated - `d3.interpolateArray`.

```
var i = d3.interpolateArray( [ 0, 0, 0 ], [ 100, 100, 100 ] );
> undefined
i ( 0 )
> [0, 0, 0]
i( 0.25 )
> [25, 25, 25]
i( 0.33 )
> [33, 33, 33]
```

This returned function then has to return a value that represents the depending attribute(`d` in this case - see line 55). The returned value will be set to the attribute directly. This way there is no need to use `this` or to modify the element by yourself. You return the desired value and that is it. ;)

### Sum up

Once you understood how the concept of `tweening` works, it not that hard. But the beginning is tough. I know that, in my case it took several runs to get it. ;)

Important things to remember are:

- use `tween` or `attrTween` to modify your elements
- set up a `factory` correctly and pass it to the `tween` function
- make sure the returned function works properly with passed in `current normalized time` values

And that is it for today. Maybe this helps someone. This concept is really hard to describe, so I hope it is kind of understandable. You can play around with every example on CodePen and hopefully you will enjoy `tweening` as much as I do later on. Thanks for reading. :)
