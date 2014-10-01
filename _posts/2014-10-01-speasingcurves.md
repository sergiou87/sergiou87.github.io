---
layout: post
title:  "SPEasingCurves"
date:   2014-10-01 20:57:04
categories: programming ios pods
---

For those times you need something more than *UIViewAnimationCurveEase[In|Out|InOut]*, now you can use [SPEasingCurves](https://github.com/sergiou87/SPEasingCurves) to create more realistic animations.

<!--more-->

It's based on the easing curves from [easings.net](http://easings.net) and it works with **CoreAnimation** and [**Pop**](http://github.com/facebook/pop) animations.

![SPEasingCurves example](https://raw.github.com/sergiou87/SPEasingCurves/master/SPEasingCurvesExample/example.gif)

If you want to use it with **Pop**, you'll probably need at least **1.0.7** version, which includes [a fix I submitted](https://github.com/facebook/pop/pull/141) for timing functions that exceed 1.0 that made the animation to stop prematurely.
