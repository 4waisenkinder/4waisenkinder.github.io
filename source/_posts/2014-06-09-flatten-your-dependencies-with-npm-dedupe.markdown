---
layout: post
title: "Flatten your npm dependencies with dedupe"
date: 2014-06-09 21:52
comments: true
author : stefanjudis
categories:
- npm
- node
---

I use npm for a while now and it is heavily integrated in my daily workflow. One thing that always bothers me, is that a fresh `npm install` on your project will probably download the same packages over and over again. Let us imagine your project has multiple dependencies that all have one package defined as another dependency.<!-- more -->

```
a
+-- b <-- depends on c@1.0.x
|   `-- c@1.0.10
`-- d <-- depends on c@~1.0.9
    `-- c@1.0.10
```

As you see above package `c` will be downloaded two times, although the dependencies `b` and `d` could use the same package. You might think now "So? Not a big deal...", but depending on dependencies of your dependencies you end up with a much slower `npm install` process, because packages got downloaded multiple times. And if you deal with a big project with a lot of dependencies I am not talking about a few seconds. ;)

Additionally the installation process can become less stable. In my case I included the following dependencies in my `package.json`.

```
  "dependencies": {
    "grunt": "~0.4.5",
    "grunt-photobox": "~0.8.0",
    "grunt-phantomas": "~0.7.1"
  }
```

The two grunt plugins either directly or one of their dependencies depend on `phantomjs`. This leads to the following output during the installation process.

```
> phantomjs@1.9.7-8 install /Users/stefan/Sites/blog-npm-dedupe/node_modules/grunt-contrib-jasmine/node_modules/grunt-lib-phantomjs/node_modules/phantomjs
> node install.js

PhantomJS detected, but wrong version 1.9.2 @ /opt/local/bin/phantomjs.
Download already available at /var/folders/03/cgg2bwfd6j92tfllylt5b91m0000gn/T/phantomjs/phantomjs-1.9.7-macosx.zip
Extracting zip contents
Copying extracted folder /var/folders/03/cgg2bwfd6j92tfllylt5b91m0000gn/T/phantomjs/phantomjs-1.9.7-macosx.zip-extract-1402344467586/phantomjs-1.9.7-macosx -> /Users/stefan/Sites/blog-npm-dedupe/node_modules/grunt-contrib-jasmine/node_modules/grunt-lib-phantomjs/node_modules/phantomjs/lib/phantom
Writing location.js file
Done. Phantomjs binary available at /Users/stefan/Sites/blog-npm-dedupe/node_modules/grunt-contrib-jasmine/node_modules/grunt-lib-phantomjs/node_modules/phantomjs/lib/phantom/bin/phantomjs

> phantomjs@1.9.7-8 install /Users/stefan/Sites/blog-npm-dedupe/node_modules/grunt-phantomas/node_modules/phantomas/node_modules/phantomjs
> node install.js

PhantomJS detected, but wrong version 1.9.2 @ /opt/local/bin/phantomjs.
Download already available at /var/folders/03/cgg2bwfd6j92tfllylt5b91m0000gn/T/phantomjs/phantomjs-1.9.7-macosx.zip
Extracting zip contents
Copying extracted folder /var/folders/03/cgg2bwfd6j92tfllylt5b91m0000gn/T/phantomjs/phantomjs-1.9.7-macosx.zip-extract-1402344475151/phantomjs-1.9.7-macosx -> /Users/stefan/Sites/blog-npm-dedupe/node_modules/grunt-phantomas/node_modules/phantomas/node_modules/phantomjs/lib/phantom
Writing location.js file
Done. Phantomjs binary available at /Users/stefan/Sites/blog-npm-dedupe/node_modules/grunt-phantomas/node_modules/phantomas/node_modules/phantomjs/lib/phantom/bin/phantomjs
```

Two times the same package is downloaded and unfortunately the `phantomjs` package downloads `phantomjs` via npm script functionality. It downloads and unzipps data itself. In my case I had to fight against an unstable network, which led to a bunch of failing intallation processes. Sometimes the first `phantomjs` download did not work. Sometimes the second. And all the time `npm install` failed. Especially `phantomjs` is a really popular dependency and I am sure a lot of projects download it much more often than only two times. ;)

After this experience I started digging around a bit and discovered `npm dedupe` - docs are [here](https://www.npmjs.org/doc/cli/npm-dedupe.html). It will check your installed dependencies and finally flatten the dependency structure by moving shared packages higher in the tree. Less duplicated packages - yay!!!

To show the difference - the old structure was:

```
grunt-phantomas
`-- phantomas
    `-- phantomjs

grunt-photobox
`-- phantomjs
```

And after using `dedupe` I ended up with:

```
phantomjs

// a lot of other shared
// dependencies also here :)

grunt-phantomas
`-- phantomas

grunt-photobox
```

Super nice!

To make sure all your colleagues receive the same flattened dependency structure use `npm shrinkwrap` to really lock all packages including versions and hierarchy. In case you have not used `shrinkwrap` before I also wrote a [post](http://4waisenkinder.de/blog/2013/07/10/why-you-should-use-npm-shrinkwrap-and-how-to-deal-with-a-dependency-fu-star-star-up/) about it.

To sum up, my workflow was the following:

```
$ npm install

$ npm dedupe

$ npm shrinkwrap
```

`dedupe` is currently marked as experimental and unfortunately I made the experience, that it is not 100% stable at the moment. I filed already an issue [here](https://github.com/npm/npm/issues/5448). But we should really start using it, because it will make the installation of npm packages just quicker and safer. I am really looking forward to this feature and I am sure, that it will become stable soon.

Thanks for reading and all comments on this topic are as usual more than welcome. ;)
