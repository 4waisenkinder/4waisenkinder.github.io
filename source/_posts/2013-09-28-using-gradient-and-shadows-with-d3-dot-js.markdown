---
layout: post
title: "Using SVG gradients and filters with d3.js"
date: 2013-09-28 21:53
comments: true
author: stefanjudis
categories:
- d3
- svg
---

I play around with the [d3.js](http://d3js.org/) library for a while now and I always thought that stuff like gradients and shadows are not so easy to create with SVG and that it is simply not made for that. The reason for that is, that when I'm browsing through the d3 [example gallery](https://github.com/mbostock/d3/wiki/Gallery) there is so much awesome data visulation, but shadows and gradients are not really present. I know, we are inside of the "flat design era" right now, so who should care about that, but I really like shadows, when they are at the correct places. Gradients on the other hand make very often the tiny tiny difference of a good design and a great design.

<!-- more -->

I started working on a nice design I found at [dribbble](http://dribbble.com/shots/1244676-Cloud-Storage/attachments/168917) (I know it's not pixel perfect) and came to the point to implement some nice d3 animations rather quickly. And here is the [final result](http://codepen.io/stefanjudis/full/jawGn).

{% img left /images/blog/stefanjudis/cloudStorageUi.jpg 740 473 'Cloud storage user interface at codepen' 'Cloud storage user interface at codepen' %}

The timeline diagram and the three circles are made in SVG. Both diagram types are supposed by the design to include multiple shadows and gradients. So let's have a look how that works.

Shadows and gradients are so called graphical objects in terms of SVG. Depending on their type they won't be rendered. For gradients and filters (shadows are filters basically) is that the case. If you are interested and want to dive more into that, here are the general specs: 

- [spec for gradients](http://www.w3.org/TR/SVG/intro.html#TermGradientElement)
- [spec for filters](http://www.w3.org/TR/SVG/filters.html#FilterElement)

The way this works is, that you have to define these elements first and then reference them via id inside of the element you want to use them. Pretty straight forward. The spec recommends that all elements that will be referenced should be defined inside of a ```defs``` element.

So let's start with that and see some d3 code! :)

```js
// append new svg
// and center it
var svg = d3container.append( 'svg' )
                     .attr( 'width', width )
                     .attr( 'height', height )
                     .append( 'g' )
                     .attr(
                       'transform',
                       'translate(' + width / 2 + ',' + height / 2 + ')'
                     );
    
// filter stuff
/* For the shadow filter... */
// everything that will be referenced
// should be defined inside of a <defs> element ;)
var defs = svg.append( 'defs' );

// append filter element
var filter = defs.append( 'filter' )
                 .attr( 'id', 'dropshadow' ) /// !!! important - define id to reference it later

// append gaussian blur to filter
filter.append( 'feGaussianBlur' )
      .attr( 'in', 'SourceAlpha' )
      .attr( 'stdDeviation', 3 ) // !!! important parameter - blur
      .attr( 'result', 'blur' );

// append offset filter to result of gaussion blur filter
filter.append( 'feOffset' )
      .attr( 'in', 'blur' )
      .attr( 'dx', 2 ) // !!! important parameter - x-offset
      .attr( 'dy', 3 ) // !!! important parameter - y-offset
      .attr( 'result', 'offsetBlur' );

// merge result with original image
var feMerge = filter.append( 'feMerge' );

// first layer result of blur and offset
feMerge.append( 'feMergeNode' )
       .attr( 'in", "offsetBlur' )

// original image on top
feMerge.append( 'feMergeNode' )
       .attr( 'in', 'SourceGraphic' );
// end filter stuff
```

Applying a shadow to an element in the world of SVG needs a bit more than doing it CSS, but I have to admit, that the documentation for SVG filters is really good. The W3C provides a lot of example including one for [shadows](http://www.w3.org/TR/SVG/filters.html#AnExample). Developers that are used to work with CSS mostly care only about three parameters - x-offset, y-offset and blur. I marked them inside of the code example.

The fourth parameter that is configurable in CSS is unfortunately not so easy to set. As long we are setting filters to a given element, it seems to me relatively hard to achieve a completely different color as a shadow color - but if anyone knows a way to do that, I would really like to know.

To sum up, what basically happens here are three steps:

1. create gaussion blur on alpha channel - ```feGaussianBlur``` | [spec](http://www.w3.org/TR/SVG/filters.html#feGaussianBlurElement)
2. move result of gaussian blur to wished position - ```feOffset``` | [spec](http://www.w3.org/TR/SVG/filters.html#feOffsetElement)
3. define output result by merge (original source graphic over result of offset operation)- ```feMerge``` | [spec](http://www.w3.org/TR/SVG/filters.html#feMergeElement)

And that is the result of the d3 operations inside of the SVG:

```xml
<defs>
  <filter id="dropshadow">
    <feGaussianBlur in="SourceAlpha" stdDeviation="2" result="blur"></feGaussianBlur>
    <feOffset in="blur" dx="2" dy="3" result="offsetBlur"></feOffset>
    <feMerge>
      <feMergeNode></feMergeNode>
      <feMergeNode in="SourceGraphic"></feMergeNode>
    </feMerge>
  </filter>
</defs>
```

And that is it for defining a shadow filter in SVG. After that we only need to reference it inside of our wished elements.

```js
var meter = svg.append( 'g' )
               .attr( 'class', 'progress-meter' );
    
meter.append( 'title' )
     .text( 'Progress meter showing amount of space used for ' + type );
    
meter.data( [ { value : .0,  index : .5 } ] )
     .append( 'path' )
     .attr( 'class', 'ui__downloadList__backgroundCircle' )
     .attr( 'd', arcBackground )
     .attr( 'filter', 'url(#dropshadow)' ) // !!! important - set id of predefined filter
     .transition()
     .duration( duration )
     .attrTween( 'd', tweenArcBackground( { value : 1 } ) );
```

And here is the result with applied shadow filter to two ```text``` elements and two ```path``` elements:

{% img left /images/blog/stefanjudis/shadowResult.png 160 166 'Result of SVG shadow filter' 'Result of SVG shadow filter' %}

Gradients are quite similar to achieve. Let us check the needed d3 code to define a gradient element:

```js
var gradientForegroundPurple = defs.append( 'linearGradient' )
                                   .attr( 'id', 'gradientForegroundPurple' )
                                   .attr( 'x1', '0' )
                                   .attr( 'x2', '0' )
                                   .attr( 'y1', '0' )
                                   .attr( 'y2', '1' );
    
gradientForegroundPurple.append( 'stop' )
                        .attr( 'class', 'purpleForegroundStop1' )
                        .attr( 'offset', '0%' );
    
gradientForegroundPurple.append( 'stop' )
                        .attr( 'class', 'purpleForegroundStop2' )
                        .attr( 'offset', '100%' ); 
```

Defining of the ```gradient``` element is not as complicated as the ```filter``` element. I decided to use a ```linearGradient``` (spec [here](http://www.w3.org/TR/SVG/pservers.html#LinearGradientElement)). All we have to do is give it an id (we need to reference it later). Per default the linear gradient is a horizontal one. I wanted to have a vertical one and that is why there are coordinates set inside of the code example. The coordinates describe a vector which represents the gradient.

By appending two ```stop``` elements we are able to define which color should be set where. This gradient is a really basic one defining colours from start to end point. Much more complicated things are possible and Mozilla has a few really good examples at [MDN](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Gradients).

And that is the result of the d3 operations inside of the SVG:

```xml
<defs>
  <linearGradient id="gradientForegroundPurple" x1="0" x2="0" y1="0" y2="1">
    <stop class="purpleForegroundStop1" offset="0%"></stop>
    <stop class="purpleForegroundStop2" offset="100%"></stop>
  </linearGradient>
</defs>
```

The nice thing about gradients is, that it is possible to define the colors inside of a stylesheet. I really like that, because everything related to colours belongs exactly there. That makes the whole thing much better maintainable.

The ```stop``` points for the gradient include the class elements, which I can reference inside of my stylesheet (I prefer using [Compass](http://compass-style.org/) by the way).

```css
// $c-cyanBright = predefined colour variable
// adjust-lightness = helper function included in compass
.cyanBackgroundStop1 { stop-color: adjust-lightness( $c-cyanBright, 7 ); }
.cyanBackgroundStop2 { stop-color: $c-cyanBright; }
.cyanForegroundStop1 { stop-color: adjust-lightness( $c-cyanBright, 29 ); }
.cyanForegroundStop2 { stop-color: adjust-lightness( $c-cyanBright, 15 ); }
```

The last step is to set the gradient to the wished element. This is done by referencing it inside of the stylesheet by its id.

```css
// yup, I'm trying out this fancy BEM way ;)
.ui__downloadList__item.cyan .ui__downloadList__backgroundCircle {
  fill: url(#gradientBackgroundCyan);
}
```

And here is the result:

{% img left /images/blog/stefanjudis/gradientResult.png 158 166 'Result of SVG shadow filter' 'Result of SVG shadow filter' %}

And that's it. If you want to play around with it check out the [pen](http://codepen.io/stefanjudis/pen/jawGn) ad Codepen. Feedback is as always welcome and I hope you have as much fun playin around with d3 as I have. :)