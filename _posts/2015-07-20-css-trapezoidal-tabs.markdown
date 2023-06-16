---
layout: post
title:  "3D Transformations: Perspective & Rotate"
date:   2015-07-20 22:33:25
categories: Tech
author: Zuri Pabón
---

This time, I'd like to introduce those awesome Perpective and Rotate (Ratata?) CSS transformations and to do it, I'll stand it by using a common UI element so we may find a good use for it: Tabs.

Currently tabs UI elements are somehow overladen, you can find plenty of plugins ready to be shiped into your application as it's used quite often in many applications, but when we need to deal with richer user interfaces requiring advanced tabs customizations, it becomes a bit more tricky to find it, so we probably will end up making it by ourselves. Here I just want to share some of the ideas which may take you to the point you can eventually make your own custom tabs transformations. I'll just go with the perspective and rotate CSS transformation on this article

Let's start defining some HTML mark up. Just a simple <p> element:

{% highlight html %}
<p class="tab">Trapezoid tab</p>
{% endhighlight %}

and let's add now some CSS transformation to it:

{% highlight css %}
.tab {
	display: inline-block;
	padding: 1em;
	margin: 1em;
  	color:white;
  	position: relative;
}

.tab::before {
	content: '';
	z-index: -1;
	background: black;
	transform: perspective(10px) rotateX(3deg);
  	border-radius: 10px 10px 0 0;
  	position: absolute;
	top: 0; right: 0; bottom: 0; left: 0;
}
{% endhighlight %}

Check it out at [codepen](http://codepen.io/Tsur/pen/rVrEde)

All the magic takes place with the transform rule. It basically says the <p> to use a 3D space by using the perspective element. It takes the distance to it as the argument, so that the greater the value, the further the distance the user is looking at it. Imagine yourself looking at a cube far far away, This will eventually looks as a single 2D point, so this is as the CSS perspective works. By default the vanishing point for the 3D perspective is the center, as looking at it face to face following a strict straight line, so using it does nothing else than actually activating a 3D perspective meaning you'll have to use it with some other transformation. You can also modify the vanishing point by the property perspective-origin. It is like moving the user/viewer position from left to right(x-position), and from top to bottom(y-position)., but remember it will take no effect until you actually apply some transformation. Thats our next stop: rotateX.

We already have our <p> element converted into a 3D perspective so let's apply some 3D transformation, otherwise it would be like studying the lesons all the semester and finally not taking the exam. The rotateX is pretty straight forward, it just rotates the element around its X-axis at a given degree, like a pig or potatoes spinning over a fire.

