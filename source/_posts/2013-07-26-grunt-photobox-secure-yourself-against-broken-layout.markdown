---
layout: post
title: "grunt-photobox - secure yourself against broken layout"
date: 2013-07-26 00:32
comments: true
author: stefanjudis
categories:
- grunt
- workflow
- phantomjs
---

Recently at work we ran into the issue, that we had a broken layout in production. Everybody knows that and sure everybody produced that already. Most of the time it is not a big deal, but it is an uncomfortable situation to get the message like "Hey, the product detail page is broken.". We implemented [grunt](http://gruntjs.com) a few weeks before and got the idea to make the check for broken layout a bit easier for us and the QA-Team than clicking through the whole site in different screen sizes. 

<!-- more -->

And this is the result of it:

**grunt-photobox**

{% img left /images/blog/stefanjudis/photoBox.png 420 300 'grunt-photobox logo' 'grunt-photobox logo' %}

Woooohooooo!!! Let me explain how it works:

- set up url's, that are important for you
- set up screensizes, that are important for you - think of responsive web design
- set up a path to render the screenshots and an index.html

So let's set it up:

I downloaded a nice "mobile-first" site from [html5boilerplate](http://html5boilerplate.com) and implemented grunt. When you're ready to go, you have to install ```grunt-photobox```.

```bash 
$ npm install --save-dev grunt-photobox
```

Your ```package.json``` should include inside of the ```devDepencies``` photobox afterwards:

```js
{
  "name": "grunt-photobox-tutorial",
  "version": "0.1.0",
  "dependencies": {},
  "devDependencies": {
    "grunt": "~0.4.1",
    "grunt-photobox": "~0.1.2"
  }
}
```

After that let's start to configure grunt-photobox. We have to set the url's - in this case it is just ```http://localhost/grunt-photobox-tutorial/```, because it is a dummy site. But at least we can set different screensizes to check if something broke and if the mobile site really works in a nice responsive way.

Your ```Gruntfile.js``` should look like that:

```js
module.exports = function(grunt) {
  grunt.initConfig( {
    pkg: grunt.file.readJSON('package.json'),

    photobox: {
      options: {
      	// set needed url's
        urls: [ 'http://localhost/grunt-photobox-tutorial/' ],
        // set needed screensizes
        screenSizes: [ '400x800', '600x800', '1200x800' ]
      }
    }
  } );

  // Load photobox plugin
  grunt.loadNpmTasks( 'grunt-photobox' );

  // Default task(s).
  grunt.registerTask( 'default', [ 'photobox' ] );
};

```

**And that's it.**

Now you can run ```$ grunt```( we set photobox to default - if it's not default, run ```$ grunt photobox```)

{% img left /images/blog/stefanjudis/photoBoxOutput.png 596 180 'output of photobox grunt command' 'output of photobox grunt command' %}

It will tell you, that a new ```index.html``` was created. Per default it is ```photobox/```, but you can change that inside of the options, if you want to. When you run photobox for the first time, there is nothing to compare -  keep that in mind, when calling the photobox index for the first time. ;)

So let's break something and check if we can detect a broken layout with it. I commented out some css lines and ran photobox again. And here is the result:


{% img left /images/blog/stefanjudis/photoBoxBroken.png 1120 750 'broken layout' 'broken layout' %}

**YEEEEESSSSS**. It is obvious, that something went wrong here with the size of 1200x800 and that we were able to notice that without click and checking by ourselves. ;)

Inside of the generated index file you've got the possibility, to check the old and new version in seperate images or to overlay both pictures. What you see here, is the 'overlay' mode.

If you want to check out the output ```index.html``` of this example and play around with it - [here it is](http://stefanjudis.github.io/grunt-photobox-tutorial/photobox).

We will implement that at work soon. If you have any feedback or feature requests, please let me know and write me an [email](mailto:stefanjudis@gmail.com) or ping on [Twitter](https://twitter.com/stefanjudis). It is still in early stage, though. I would really like to have some feedback on that, because I think, it can speed up the qa-process heavily.

THX.