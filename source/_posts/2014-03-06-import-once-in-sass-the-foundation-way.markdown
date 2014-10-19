---
layout: post
title: "Import once in Sass the 'Foundation way'"
date: 2014-03-06 22:52
comments: true
author : stefanjudis
categories:
- sass
- foundation
---

Currently I am working on a huge so called "redesign", which means to make a big e-commerce plattform responsive. But it is not only about fiddling a few media queries into some existant CSS. For me that is the perfect opportunity to clean up everything and build up a nice and modular CSS architecture. Each component separated into its own file. Margins, paddings, widths, colors, etc. controlled by a main config file. Maintainable and easy to work with. The tech stack is quite modern, so that I had the perfect basement to do fancy stuff.<!-- more -->

I decided to include the [Foundation front-end framework](http://foundation.zurb.com/) into our stack and started playing around with it. No matter, if you want to use all their styles or just a few components (the `block-grid` is awesome by the way) the source code is really worth a look. It is highly configurable and easily to integrate next to your own components.

Any complex system consists of tons of little components. When starting to model them in a CSS architecture multiple dependencies come up rather quickly. [Brad Frost](https://twitter.com/brad_frost) describes this principle really good in his article [Atomic Design](http://bradfrostweb.com/blog/post/atomic-web-design/). Basically it's really simple.

Let us say you build up a button, which is an in itself closed unit. Nice! After that you start building up widgets, that need the styles of that particular button. No problem - you can easily `import` it via the already included Sass function. But unfortunately it now becomes tricky, when you start developing your next wigdet, that also depends on your shiny button.

Here is some example code.

File : *components/buttons/_fancyButton.scss*
```css
.fancyButton {
  color : blue;
}

```

File : *components/_widgetA.scss*
```css
@import
  "components/buttons/fancyButton";
```

File : *components/_widgetB.scss*
```css
@import
  "components/buttons/fancyButton";
```

File : *styles.scss*
```css
@import
  "components/widgetA",
  "components/widgetB";
```

Generated file : *styles.css*
```css
.fancyButton {
  color: blue; }

.fancyButton {
  color: blue; }

```

Sass will now include the same file twice and you end up with a doubled set of CSS rules in your generated Stylesheet. The more components you have, the worse it gets. Talking about this problem my friend [Tomasz](https://google.com/+TomaszStryjewski) pointed me to the direction of a [Foundation mixin](https://github.com/zurb/foundation/blob/master/scss/foundation/_functions.scss#L8) solving exactly this use case.

```css
// IMPORT ONCE
// We use this to prevent styles from being loaded multiple times for compenents that rely on other components.
$modules: () !default;
@mixin exports($name) {
  // check if code with name is already stored inside of $modules
  @if (index($modules, $name) == false) {
    // if not push it into the list
    $modules: append($modules, $name);
    // include code that particular code
    @content;
  }
}
```

The logic included here is not magically, but it shows a few really nice features of Sass, which I for example do not use on a daily basis. The first thing to notice is the assignment of a so called `List` with the variable named `$modules`. [Lists in Sass](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#lists) behave kind of like an Array with String values and are defined as follows:

> Lists are just a series of other values, separated by either spaces or commas.

It is assigned with the `!default` flag to not overwrite an already existing `$modules` variable, which could easily happen, by importing the particular file twice.

**Side note** : *Variables with null values are treated as unassigned by !default* and will be overwritten.

After that a [mixin](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#defining_a_mixin) with the name `exports` is defined. If you do not know mixins, they are according to Sass documentation defined as follows:

>Mixins allow you to define styles that can be re-used throughout the stylesheet without needing to resort to non-semantic classes like .float-left. Mixins can also contain full CSS rules, and anything else allowed elsewhere in a Sass document. They can even take arguments which allows you to produce a wide variety of styles with very few mixins.

This mixin now accepts one argument which represents the actual name of the component. It is used to "register" new code.
First of all it checks if there was already a component (or whatever valid Sass code you like) with the same name included in `$modules`using the [index](http://sass-lang.com/documentation/Sass/Script/Functions.html#index-instance_method) function provided by Sass lists.

**Side note** : `index` either returns the index of the value or `false`.

If the name of the snippet is not included inside of the list it includes the particalar code and [appends](http://sass-lang.com/documentation/Sass/Script/Functions.html#append-instance_method) a new value to `$modules`.
If the name is already inside of the `$modules` list it just ignores it and prevents including the same code multiple times by just doing nothing and going on.

The rebuilt version using this mixin looks then as follows.

File : *helpers/_functions.scss*
```css
$modules: () !default;
@mixin exports($name) {
  @if (index($modules, $name) == false) {
    $modules: append($modules, $name) !global;
    @content;
  }
}
```
File : *components/buttons/_fancyButton.scss*
```css
@include exports("fancyButton") {
  .fancyButton {
    color : blue;
  }
}
```
File : *components/_widgetA.scss*
```css
@import
  "./buttons/fancyButton";
```
File : *components/_widgetB.scss*
```css
@import
  "./buttons/fancyButton";
```
File : *styles.scss*
```css
@import
  "helpers/functions",
  "components/widgetA",
  "components/widgetB";
```
Generated file : *styles.css*
```css
.fancyButton {
  color: blue;
}
```

Problem solved. :)
I generally do not use Sass for logical mixins or similar, but this shows perfectly what is possible. I definitely have to dig deeper into that. ;)

In my case of the "redesign" I wrapped all my components using this mixin and it works pretty well with ~30 components included in different stylesheets. Maybe that will help someone dealing with a lot of components. ;)

So that is it and thanks for reading.
