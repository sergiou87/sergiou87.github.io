---
layout: post
title:  "Drawing circles"
date:   2014-09-03 15:24:04
categories: programming ios
---

During the development of **Fever** and **Plex** apps, in several occasions I needed to draw a circle to create a circular progress view (just like [DACircularProgress](https://github.com/danielamitay/DACircularProgress)) with beautiful animations when progress changes.

However, I've always encountered a disturbing bug(?) drawing those circles that was driving me crazy.

<!--more-->

In this case, my first approach was simply creating a `CAShapeLayer`, then create a circle path and animate the `strokeEnd` property to simulate the progress. Really easy.

The result:

![Circle with CAShapeLayer](/public/drawing-circles/CAShapeLayer.png)

I don't know if you can see it because it's very subtle, but the circle isn't perfectly round. That's what drives me crazy.

Fortunately, the `UIBezierPath` class has a property [flatness](https://developer.apple.com/library/ios/documentation/uikit/reference/UIBezierPath_class/Reference/Reference.html#//apple_ref/occ/instp/UIBezierPath/flatness) that allows us to change that. So I set it to a low value and this is the result I get:

![Circle with CAShapeLayer… again](/public/drawing-circles/CAShapeLayer.png)

Yep, exactly the same. I didn't even bother to use another image… it just doesn't work!

So I give a try to manually draw the circle using **CoreGraphics** in a custom layer:

![Circle with CoreGraphics](/public/drawing-circles/CoreGraphics.png)

Perfect! Let's try changing the flatness of the `UIBezierPath`:

![Flat circle with CoreGraphics](/public/drawing-circles/CoreGraphics-flat.png)

Working like a charm!

So unless I'm doing something wrong (if so, please let me know [on Twitter](https://twitter.com/sergiou87)) I'd say there is a bug in `CAShapeLayer` that ignores the `flatness` of the path.

That's why I decided to report the bug in [Open Radar](http://openradar.io) and also create a small sample application to show the problem with the flatness in `CAShapeLayer`s:

![Drawing Circles demo app](/public/drawing-circles/flatness-demo-app.gif)

**Demo app:** https://github.com/sergiou87/DrawingCircles
**Open Radar bug report:** http://openradar.io/23784682
