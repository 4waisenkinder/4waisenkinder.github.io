---
layout: post
title: "Control your CSS animations with *animation-fill-mode*"
date: 2014-10-13 20:39
comments: true
author : stefanjudis
categories:
- css
- animation
---

Recently I had some time to play around with some CSS animations. Unfortunately I am not able to use them on a regular basis in my daily work and that is why I am really enjoying to do *#cssonly* stuff in my free time.

When using CSS animation there are two different (not technical) types of css animation.<!-- more --> There are the animations that has a starting set of set CSS properties which is exactly the same as the properties set at the end of the animation. You can check the example below.

### Same start and end CSS properties

{% codepen ALJef stefanjudis result 400 %}

The animation defines a movement of the red square by using `translate` and moves the square to the left, to the right and ends in the center where it actually started.

*If you are not familiar with CSS animations you might want to read something about it on [MDN](https://developer.mozilla.org/de/docs/Web/CSS/animation).*

### Different start and end CSS properties

But then there is the other type of animations that do not have the same properties as the applied styles when they finish. And these animations were always something I was struggling with.

So let us change the animation for the red square, so that it only moves to the right and has a different styling as applied before the animation kickes in.

{% codepen fElaA stefanjudis result 400 %}

As you see the animation finishes and the square jumps back to its position defined by the CSS that is not part of the animation.

```css
.square {
  // some more stuff here
  animation-name : move;
  animation-duration : 1s;
  animation-iteration-count :1;

  transform : translate( 0, 0 );
}

@keyframes move {
  0% {
    transform : translate( 0, 0 );
  }
  100% {
    transform : translate( 100%, 0 )
  }
}
```

But, that is not what I wanted, I wanted the square to keep the properties defined in the last frame of the animation. So I googled a bit and landed on [stackoverflow](http://stackoverflow.com/questions/4359627/stopping-a-css3-animation-on-last-frame). The first answer you will find there, is to set up the base styling of the animated element to the end state of the animation.

{% codepen GmCIf stefanjudis result 400 %}

```css
.square {
  // some more stuff here
  animation-name : move;
  animation-duration : 1s;
  animation-iteration-count :1;

  // set it initially to the state of the animation end
  transform : translate( 100%, 0 );
}

@keyframes move {
  0% {
    transform : translate( 0, 0 );
  }
  100% {
    transform : translate( 100%, 0 )
  }
}
```

This works kind of well, but I never liked this way and it felt like it is not the correct solution. And it turned out quickly, that it actually is not. When I was playing around with animations I used `animation-delay` a lot. So let us check if the last solution is working in combination with the usage of `transition-delay`.

{% codepen xHavI stefanjudis result 400 %}

```css
.square {
  // some more stuff here
  animation-name : move;
  animation-duration : 1s;
  animation-iteration-count :1;

  transform : translate( 100%, 0 );

  @for $i from 0 through 3 {
    &:nth-child(#{$i + 1}) {
      top : 1 + $i * 6rem;
      animation-delay : #{$i}s;
    }
  }
}

@keyframes move {
  0% {
    transform : translate( 0, 0 );
  }
  100% {
    transform : translate( 100%, 0 )
  }
}
```

What I did was I implemented four elements, which have all the same animation but with different set `transition-delay` property. And as you see, the squares are not jumping back when the animation finishes, which is good, but they are jumping when the animation starts, because the base styling is matching the animation end.

That shows clearly, that setting the animation end styles to the elements you want to animate is not an option at all.

### animation-fill-mode for the rescue

And basically there is a super easyIch komme solution to solve this problem (and I'm still surprised that it is not popping up, when googling this problem). There is the CSS property [animation-fill-mode](https://developer.mozilla.org/de/docs/Web/CSS/animation-fill-mode), which lets you define, if styles included in your defined animation should be applied to the depending element before and/or after the animation.

This means by applying `animation-fill-mode : forwards` to the animated squares, the squares can still have the base styling which should be applied before the animation starts and afterwards the styles defined in the animation at `100%` or `to` will be applied to the element when the animation finishes.

```css
.square {
  // some more stuff here
  animation-name : move;
  animation-duration : 1s;
  animation-iteration-count :1;
  animation-fill-mode : forwards;

  transform : translate( 0, 0 );

  @for $i from 0 through 3 {
    &:nth-child(#{$i + 1}) {
      top : 1 + $i * 6rem;
      animation-delay : #{$i}s;
    }
  }
}

@keyframes move {
  0% {
    transform : translate( 0, 0 );
  }
  100% {
    transform : translate( 100%, 0 )
  }
}
```

And here is the working result:

{% codepen CtoyH stefanjudis result 400 %}

**It can be as easy as that!**

And that is it for now - problem solved. :)

I thought it might be worth sharing and I hope you enjoyed it. And as always feedback is very welcome. :)
