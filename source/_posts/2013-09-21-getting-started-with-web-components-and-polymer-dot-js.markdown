---
layout: post
title: "Getting started with web components and polymer.js"
date: 2013-09-21 18:35
comments: true
author: stefanjudis
categories:
- web components
- polymer
- future stuff
---
I spent a lot of time on [codepen](http://codepen.io) the last days to train my CSS skills and to relax a bit (it is really awesome to just play around there while watching a movie). One thing I created was a styled checkbox. And while doing that, the idea came to me, that this checkbox is a perfect use case to start playing around with this magic thing called "web components", that is heavily around since a few month and is probably the "next big thing" in web development.

<!-- more -->

Web components are basically small encapsuled units inside of a website or web application(like for example the HTML5 video element). They include their own styling (CSS) and their own behaviour (JS). And this is the biggest advantage of them. Imagine little pieces inside of your web app, that are not influenced by the global stylesheet and whose children are not affected by any written JavaScript function. This way you end up with a box of bricks and the only thing to do is putting them together easily. Additionally you can use them whereever you want, because they include everything they need to look awesome and to behave awesome. Sounds really like a dream to me. :)

If you are interested in that topic, recommended ressources are the following:

- [slides about web components created by Google](http://html5-demos.appspot.com/static/webcomponents/index.html)
- [talk about web components at CSSconf.eu ](http://www.youtube.com/watch?v=U45e-zq4bTs&feature=youtu.be)
- [W3C working draft about web components](http://www.w3.org/TR/2013/WD-components-intro-20130606/)

I started looking around and decided to use [polymer.js](http://www.polymer-project.org) (written by the Google guys) to play around with the principle of web components. Unfortunately they are rarely supported these days and this library gives me the opportunity to use this technique already today by providing needed polyfills. 

So, here is my checkbox! Let's make a "real" new fancy web component out of it.

{% codepen ojDif stefanjudis result 400 %}

I started by downloading the basic stuff from [HTML5BOILERPLATE](http://html5boilerplate.com) and the polymer component library. The first things to start using it are implementing the polymer script inside of the ```head``` of the document and loading your first web component file. Web components need to include a hyphen inside of their name defined by the specs to avoid conflicts with existing elements. 

```
<!-- @start Polymer stuff -->
<script src="js/vendor/polymer.min.js"></script>

<link rel="import" href="components/sj-checkbox.html">
<!-- @end Polymer stuff -->
```

Next step is to create this component html file, that will define the new web component. I created a new folder for my web components and created the file ```sj-checkbox.html```. Inside of that file I started to implement the new component regarding the polymer documentation.

```
<polymer-element name="sj-checkbox">
  <template>
    hello world
  </template>
  <script>
    Polymer( 'sj-checkbox', {} );
  </script>
</polymer-element>
```

That is all you need. Wrap everything in the ```polymer-element``` tag, set a name as attribute, use the new HTML5 template element to define your markup, call the Polymer constructor inside of a ```script``` tag and you are done. After that, you can use a new custom element inside of your ``ìndex.html```.

```
<ul id="switchesComponents">
	<li><sj-checkbox></sj-checkbox>
	<li><sj-checkbox></sj-checkbox>
	<li><sj-checkbox></sj-checkbox>
</ul>
```

When you inspect this in a current version of Chrome Canary and you enabled the needed flags (look [here](http://html5-demos.appspot.com/static/webcomponents/index.html#69)) you will see the power of this technique.

{% img left /images/blog/stefanjudis/simpleComponent.jpg 244 239 'First simple web component' 'First simple web component' %}

Our little 'hello world' message is visible on our screen and it is encapsulated inside of the [shadow DOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/). I mean, how awesome is that?

After this success I started implementing the actual new checkbox markup and the needed styles.

```
<polymer-element name="sj-checkbox">
  <template>
    <style>
      /* all my styles here */
    </style>
    <div class="switch">
      <div class="switch__circle">
          <input id="switcher" class="switch__input" type="checkbox">
          <div class="switch__innerCircles"></div>
          <label class="switch__label" for="switcher">Switch me!!!</label>
        </div>
      </div>
    </div>
  </template>
  <script>
    Polymer( 'sj-checkbox', {});
  </script>
</polymer-element>

```

That worked actually on first try and I saw the new checkbox and its "hidden" markup inside of the shadow DOM tree. I prepared the checkbox to be configurable easily so that it is possible to get it in three different sizes by just adding the class "small", "medium" or "big" to the ```switch```container. To achieve that, you can easily add custom attributes to your new created web component.

```
<ul id="switchesComponents">
  <li><sj-checkbox size="big"></sj-checkbox>
  <li><sj-checkbox size="medium"></sj-checkbox>
  <li><sj-checkbox size="small"></sj-checkbox>
</ul>
```

Additionally you have to define inside of your component html file (remember ```components/sj-checkbox.html```) that this value is readable. This is done by setting a new attribute ```attributes``` to the ```polymer-element``` element defining the given values. It is also possible to set default value for the size attribute then. You can then access the value by calling the attribute name wrapped into two curly braces inside of your component html.


{% codeblock %}{% raw %}
<!-- define attributes here -->
<polymer-element name="sj-checkbox" attributes="size">
  <template>
    <style>
	  /* all my styles here */
    </style>
    <!-- backslashes are not needed - but octopress forces me to do it like that :( -->
    <div class="switch {{size}}">
      <div class="switch__circle">
          <input id="switcher" class="switch__input" type="checkbox" on-change="changeHandler"checked="checked">
          <div class="switch__innerCircles"></div>
          <label class="switch__label" for="switcher">Switch me!!!</label>
        </div>
      </div>
    </div>
  </template>
  <script>
  	// define default values here
    Polymer( 'sj-checkbox', {
      size: 'medium'
    } );
  </script>
</polymer-element>

{% endraw %}{% endcodeblock %}

So far so good. The checkbox appears with its full markup and depending styles.

{% img left /images/blog/stefanjudis/fullComponent.jpg 545 232 'Full checkbox web component' 'Full checkbox web component' %}

After that the checkboxes are implemented in different sizes and it is time to add some functionality to them. Usually the way is to bind an event listener to the change event of the input element and then update whatever needs to be updated. Unfortunately this elements do not trigger any change events anymore, because they are hidden inside of the shadow DOM. Polymer provides an easy way to implement custom events to solve this problem.

You only have to implement a ```on-change``` attribute on the input field hidden inside of the shadow DOM and define a callback function, that should be executed when the event is triggered. Inside of this function you are able to trigger a custom event using ```this.fire()```.

{% codeblock %}{% raw %}
<polymer-element name="sj-checkbox" attributes="size">
  <template>
    <style>
	  /* all my styles here */
    </style>
    <div class="switch {{size}}">
      <div class="switch__circle">
          <input id="switcher" class="switch__input" type="checkbox" on-change="changeHandler" checked="checked">
          <div class="switch__innerCircles"></div>
          <label class="switch__label" for="switcher">Switch me!!!</label>
        </div>
      </div>
    </div>
  </template>
  <script>
    Polymer( 'sj-checkbox', {
      size: 'medium',
      changeHandler: function( event ) {
        this.fire(
          'change',
          {
            id: event.target.id
          }
        );
      }
    } );
  </script>
</polymer-element>
{% endraw %}{% endcodeblock %}

Afterwards you can easily listen to this event and react to it.

```
<ul id="switchesComponents">
	<li><sj-checkbox size="big"></sj-checkbox>
	<li><sj-checkbox size="medium"></sj-checkbox>
	<li><sj-checkbox size="small"></sj-checkbox>
</ul>
<div id="switchesComponentsNotifications" class="notification"></div>
<script>
	( function( document ) {
		var switchList    = document.getElementById( 'switchesComponents' );

		switchList.addEventListener( 'change', changeHandler, false );

		function changeHandler( event ) {
			var notification = document.getElementById( 'switchesComponentsNotifications' );

			notification.innerHTML = '!!! ' + event.detail.id +' switched !!!';
		}
	} )( document );
</script>
```

About that is to say, that when you fire events this way inside of your component you can pass as second argument an object which will be accessable inside of the event handler under the ```detail``` key. In my case I am just handing over the id of the input element.

### Stuff to be careful with

The two following issues with polymer are already reported bug tickets. Before continue reading you can check out the state of both here:

- [prevent event bubbling in fallback](https://github.com/Polymer/polymer/issues/296)
- [prevent dublicate ID's in fallback](https://github.com/Polymer/polymer/issues/295)


When checking browser compability of this I noticed that using id's inside of your web components can be really tricky. There is no problem in case the browser supports web components (id's are inside of the shadow DOM and totally encapsulated), but if not polymer does some magic and your component will appear in the form of normal html. Then you've got the same id multiple times in your application, which leads to invalid markup and maybe unexpected behaviour. My solution for that was adding kind of ```fallbackId``` to the component.

```
<ul id="switchesComponents">
	<li><sj-checkbox size="big" fallbackId="checkboxBigComponent"></sj-checkbox>
```

{% codeblock %}{% raw %}
<div class="switch {{size}}">
  <div class="switch__circle">
      <input id="{{fallbackId}}" class="switch__input" type="checkbox" on-change="changeHandler"checked="checked">
      <div class="switch__innerCircles"></div>
      <label class="switch__label" for="{{fallbackId}}">Switch me!!!</label>
    </div>
  </div>
</div>
{% endraw %}{% endcodeblock %}

Another thing I noticed is, that you should always namespace your events triggered by your components. In my first implementation (which you see a few lines above) I defined the component to trigger a ```change``` event. That worked fine in Canary, but unfortunately in other browsers there are two events fired. One by the web component itself and on by the input field that is not hidden inside of the shadow DOM. I ended up with triggering an event called ```componentChange``` and that worked fine.

Jap and that is it. You can check out the result [here](http://stefanjudis.github.io/webComponents-tutorial/) - check it out in Canaray, it's just awesome.

I will continue playing around with this. My next goal is to achieve a new input element type like ```<input type="myFancyCheckbox">``` that I actually can use inside of my forms, but I have got no idea how this works yet. Addiontally implementing custom pseudo elements to make the components accessible and styleable from the outside seams to be a reasonable next step.

So, thanks for reading. Feedback is as always really welcome. ;)

A follow up post about making the checkbox customizable can be found [here](http://4waisenkinder.de/blog/2013/10/05/getting-started-with-web-components-and-polymer-dot-js-ii/).