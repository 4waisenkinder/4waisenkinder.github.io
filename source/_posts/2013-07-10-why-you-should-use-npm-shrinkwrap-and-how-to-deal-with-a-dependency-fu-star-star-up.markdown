---
layout: post
title: "\"shrinkwrap\" and how to deal with a dependency fu** up"
date: 2013-07-10 19:13
comments: true
author : stefanjudis
categories:
- npm
- deployment
---
A few people may have noticed, that ```npm install grunt``` was failing today. According to the pull requests made, it was failing for 10 ( ! ) hours. When you think right know, that this is not a big deal, you probably do not use it in production. Because then it becomes a big problem very quickly.

<!-- more -->

The reason for failing was a first renamed and later removed dependency of grunt itself. Grunt defines its dependencies ( defined inside of the ```package.json```, if you are not familiar with npm ) with  a ```~``` in front of each version number. That is quite common and there is nothing special about it. The [tilde](https://npmjs.org/doc/json.html#Tilde-Version-Ranges) defines, that every version greater than the specifed one and less than the next major release is fine to install.

What happened exactly, was that npm was not able to find a version of a particular dependency that fits this conditions ( because of renaming and removing ). And that is it. Nothing special about it, but an important insight.

**Whenever we publish something on npm, we have to let it there - no renaming or removing. There can always be people using it and it can screw up their build!**

However, we had to figure out a workaround today quickly, and that is what we did until it was fixed.

We forked grunt at Github. Fixed the dependency problem and set the "new" grunt as our project dependency. Npm allows you to set dependencies that are not published at npm easily. That can be quiet handy from time to time. ;)

```
"devDependencies": {
  "grunt": "git://github.com/you/your-grunt-fork.git"
}
```

Later on there were a few discussions about npm dependency handling on Twitter and [@asciidisco](https://twitter.com/asciidisco) pointed me into the direction of ```npm shrinkwrap```.

This command locks all your installed dependencies and writes a new file called **npm-shrinkwrap.json**. 

This file is similar to php's **composer.lock** or ruby's **Gemfile.lock**. When you call ```npm install``` later on it installs the locked versions written in this file. The version numbers are locked recursively, so that you are safe on production level without any bad surprises.

There can always be a version jump in a dependency, which is quite welcome, but I don't want check out if it works on production side first...

So the solution is:

**Stay with your package.json and define dependencies with '>=' or '~' to receive the newest stuff locally, but lock all dependencies for production.**

As always opinions and solutions are very welcome about this topic. ;)