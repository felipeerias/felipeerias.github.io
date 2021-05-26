---
layout: post
title: Towards richer colors in Blink
date: 2021-06-01
---

## Introduction

The study of color joins together concepts from physics (how light works), biology (how our eyes see), computing, and more. There is a long and rich history following the desire to be able to manipulate richer materials and colors when creating visual art, and the same is true of the Web today.

This blog post is based on my talk at BlinkOn 14 (May 2021). You can watch the recording here:

{% include youtubePlayer.html id="eHZVuHKWdd8" %}

> [Towards richer colors in Blink (BlinkOn 14)](https://www.youtube.com/watch?v=eHZVuHKWdd8)

And the slides are available here: [Towards richer colors in Blink - slides)](https://docs.google.com/presentation/d/1u_pPs6uq3nQUvBEPmBz_cJsFepfZi07Y3RbY62_i-FU/edit?usp=sharing).

This post will talk about the ongoing efforts to specify richer colors on the Web platform, with a focus on how Blink/Chromium paints colors and some ideas about what it would take to widen the range of colors that it is able to paint.


## Color on the Web

A color space is a way to describe and organize colors so they can be identified and reproduced with accuracy. Some color spaces are more or less arbitrary (e.g. the Pantone collection) but the ones that we will focus on are based on mathematical descriptions.

These color spaces consist of a mathematical color model that specifies how colors are described (i.e. tuples of numbers) and a precise description of how those components are to be interpreted.

Until recently, the Web has been built on top of the sRGB color space (1996) which describes colors with a RGB color model (red, green and blue) plus a non-linear transfer function to link the numerical value for each component with the intensity of the corresponding primary.

Traditionally, colors in the Web are specified in the sRGB color space with a value between 0 and 255 for each component plus an opacity value. CSS includes plenty of functions and shortcuts to define a color in that sRGB space.


![web srgb](/assets/img/web_srgb.png "sRGB in the Web")

> [<color> CSS data type](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value)

There are many other color spaces. The one of the left is called CIE XYZ and was specifically designed to cover all colors the average human can see.

From that large map of colors within human perception, the graph on the right identifies those that fall within the sRGB color space.

As you can see, there are many colors that we can perceive but fall outside of sRGB. They can not be described within sRGB.


![home tab](/assets/img/ciexyz_srgb.png "CIE XYZ and sRGB")


> [Color: From Hexcodes to Eyeballs](http://jamie-wong.com/post/color/) (Jamie Wong)

> [CIE 1931 color space](https://en.wikipedia.org/wiki/CIE_1931_color_space) (Wikipedia)

> [sRGB](https://en.wikipedia.org/wiki/SRGB) (Wikipedia)

The range of colors that a hardware display is able to show is called its gamut. The sRGB color space gained popularity early on because it was well suited to be displayed by the CRT devices monitors that were common at the time. As technology has improved over time, many displays nowadays are able to display colors that go beyond the sRGB color space.

On the Web platform in particular there is increasing interest for adding support for wider color gamuts to different elements. These articles are a good introduction:

* [Unlocking Colors](https://bkardell.com/blog/Unlocking-Colors.html) (Brian Kardell)
* [LCH colors in CSS: what, why, and how?](https://lea.verou.me/2020/04/lch-colors-in-css-what-why-and-how/) (Lea Verou)

Several JavaScript libraries already provide a lot of functionality for manipulating colors (but are limited by the limits of what can be displayed by the browser):

* [Color JS](https://colorjs.io/)
* [D3 d3-interpolate](https://github.com/d3/d3-interpolate#color-spaces)
* [chroma JS](https://gka.github.io/chroma.js/)

The major Web browsers offer different levels of support for color management and access to wider gamuts.


### Color standards in the Web


This post will focus specifically on adding support on [Blink](https://www.chromium.org/blink) and [Chromium](https://www.chromium.org/Home) for richer colors in elements defined in HTML and CSS. The reference specification for this is the CSS Color Module elaborated by the CSS Working Group:

* current: [CSS Color Module level 4](https://drafts.csswg.org/css-color)
* next iteration: [CSS Color Module level 5](http://drafts.csswg.org/css-color-5)

There is as well a [Color on the Web](https://www.w3.org/community/colorweb/) community group at the W3C that among other things organises a [workshop on wide color gamut for the Web](https://www.w3.org/Graphics/Color/Workshop/overview.html).

There was also [a very interesting discussion](https://github.com/w3ctag/design-reviews/issues/488) at the W3C's Technical Architecture Group about how having colors outside of the sRGB gamut opened up questions about interoperability between the different elements of the platform.

(This list does not pretend to be exhaustive and it intentionally leaves aside the many groups working on standards beyond CSS and beyond the Web in general.)

###  CSS Color

The CSS Color spec, among other things:

* extends the color() function to let the author explicitly indicate the desired colorspace of a color, including color spaces with a wide gamut.
* defines the lab() and lch() functions to specify colors in [the CIE L*A*B colorspace](https://en.wikipedia.org/wiki/CIELAB_color_space).
* provides detailed control over how interpolation happens, as well as many other features.
* contains a reference implementation for the operations described in it.

So why is this a big deal?


### Display more colors


First, using only sRGB limits the range of colors that can be displayed. Many modern monitors have a wider gamut than sRGB, often close to another standard called Display-P3. Here you can see both of those spaces over the same graph that we saw before:


![srgb p3](/assets/img/dpcip3-20190103-5.jpg "Display-P3 and sRGB")

> [Why DCI-P3 is the New Standard of Color Gamut?](https://www.msi.com/blog/why-dci-p3-is-the-new-standard-of-color-gamut)
> [Wide-gamut color on the web](https://www.reddit.com/r/webdev/comments/ctiixa/widegamut_color_on_the_web_the_status_in_august)

The Display-P3 space is about one third larger than sRGB. This means that from CSS we have no access to roughly one third of the colors that modern monitors can display.

This is another way of visualizing the same thing, where the white line in each case represents the boundary between what can be described within sRGB and what is inside Display-P3.


![srgb p3 outline](/assets/img/sRGB_P3_outline.png "Display-P3 and sRGB")


> [Wide Gamut Color in CSS with Display-P3](https://webkit.org/blog/10042/wide-gamut-color-in-css-with-display-p3/) (WebKit)

As you can see, the colors that fall within the Display-P3 space but outside of sRGB are the most intense and vivid.

And that is not all, there are color spaces that are even larger than Display-P3 which are for now only used for professional equipment but which, at some point in the future, will probably become popular in their turn.

Adding wider color spaces to the Web is as much about supporting what hardware can do today as it is about setting us in the path to support what it will do in the future.


### Define colors in a consistent and predictable way


Secondly, another limitation of sRGB in the Web is that it is not perceptually uniform: the same numeric amount of change in a value does not cause similar changes in the colors that we perceive.

We can see this clearly with HLS, which is an alternate way to express the same sRGB colors in terms of hue, lightness and saturation.


![hsl problems](/assets/img/hsl_problems.png "HSL problems")


> [Color spaces for human beings](https://www.boronine.com/2012/03/26/Color-Spaces-for-Human-Beings/)

In the first example, a change of 20 degres in hue does not always cause a similar change in the color that we perceive. In the second, changing the lightness also changes the saturation that we perceive even if its numberical value has stayed the same. And in the third, colors with the same lightness value can have very different perceived lightness.

This means that sRGB (and HSL) can not be used to accurately predictable adjust lightness, saturation or hue, find complementary colors, calculate the perceived contrast between two colors, etc.

One of the new functionalities in the CSS Color spec is to be able to use color spaces where the same numerical changes in one of the values brings similar perceived changes, like the LCH color space (Lightness, Chroma, Hue).


![lch examples](/assets/img/lch_examples.png "LCH examples")

> [LCH colour picker](https://css.land/lch)

> [CIE LAB color space](https://en.wikipedia.org/wiki/CIELAB_color_space)

> [Perceptually uniform color spaces](https://programmingdesignsystems.com/color/perceptually-uniform-color-spaces)

In the LCH color space, the same changes in lightness (first row), chroma ("amount of color", second row), and hue bring about similar and predictable changes in the colors that we perceive.


### Interpolation, etc.


Supporting additional color spaces will let you specify different ways to interpolate between colors in gradients and transitions.

This happens because the path to get from one color to another is not the same on different color spaces. Here are some examples:


![interpolation examples 1](/assets/img/interpolationexamples1.png "Interpolation examples 1")


![interpolation examples 2](/assets/img/interpolationexamples2.png "Interpolation examples 2")


> [Color JS - interpolation](https://colorjs.io/docs/interpolation.html)

Interpolation is just one example where adding richer color capabilities to the Web dramatically broadens the range of tools available to authors when creating their sites.


## Color in Chromium


Now let's talk about [Chromium](https://www.chromium.org/Home). As you know, it is the Free Software portion of the Chrome and Edge Web browsers.

The Web engine inside of it is called [Blink](https://www.chromium.org/blink) and it implements the Web Platform standards that describe how to turn Web content into pixels on the screen.

Blink itself is a fork of WebKit, which is the Web engine used by Safari and others.


### Render pipeline

Blink basically creates a rendering pipeline that takes Web sources as input (pages, stylesheets, and so on).

It parses them, applies styles, defines geometry, arranges the content into layers and tiles, paints those and sends them over to be displayed.

This job of actually painting those pixels is carried out by a multiplatform graphics library called [Skia](https://skia.org/).


![Blink pipeline](/assets/img/blinkpipeline.png "Blink pipeline")

> [Life of a Pixel](http://bit.ly/lifeofapixel)


### Richer colors

In Chromium, there is already some support for color management, @media queries (color-gamut), color profiles (tags) in images, and so on. There is now also an intent to experiment with additional color spaces for canvas, WebGL and WebGPU.

> [Color managing canvas contents](https://github.com/WICG/canvas-color-space/blob/main/CanvasColorSpaceProposal.md)

However, there isn't yet support for using richer color spaces with individual Web elements like we have seen in the previous section.

Within Blink, CSS colors are parsed and stored into a small structure, just 32 bits, that is 8 bits per RGB color channel plus alpha.

> [third_party/blink/renderer/platform/graphics/color.h](https://source.chromium.org/chromium/chromium/src/+/master:third_party/blink/renderer/platform/graphics/color.h)

These colors are eventually handed over to the Skia library to carry out the actual drawing. Skia then uses its own similar 32-bit format.

> SkColor in [third_party/skia/include/core/SkColor.h](https://source.chromium.org/chromium/chromium/src/+/master:third_party/skia/include/core/SkColor.h)

We can show this on the previous diagram. A Web page specifies a color in sRGB which is stored in a 32-bit format and passed across the rendering pipeline until it reaches Skia, where is is converted to a similar format, rastered, and displayed.


![Blink pipeline colors](/assets/img/blinkpipelineoverlay.png "Blink pipeline colors")



This means that thoughout Blink's rendering pipeline colors are represented using only 32 bits, and this limits the precision and the richness of the colors that can be used and displayed in websites by Chromium.


### Some ideas from WebKit

Blink started as a fork of WebKit in 2013 and although they have evolved in different ways, we can still look at WebKit to get some inspiration for storing and manipulating high-precision colors.

Without getting into too much detail, WebKit supports a high precision representation of colors that stores four float values plus a colorspace. LAB is one of those spaces that may be used to define colors in WebKit.

> [Improving Color on the Web](https://webkit.org/blog/6682/improving-color-on-the-web/)

> [Wide Gamut Color in CSS with Display-P3](https://webkit.org/blog/10042/wide-gamut-color-in-css-with-display-p3/)

> [WebCore/platform/graphics/Color.h](https://trac.webkit.org/browser/webkit/trunk/Source/WebCore/platform/graphics/Color.h)

> [WebCore/platform/graphics/ColorComponents.h](https://trac.webkit.org/browser/webkit/trunk/Source/WebCore/platform/graphics/ColorComponents.h)

> [WebCore/platform/graphics/ColorSpace.h](https://trac.webkit.org/browser/webkit/trunk/Source/WebCore/platform/graphics/ColorSpace.h)

Having this support for higher precision colors has already made it possible to implement several color features in WebKit, for example:

* lab(), lch() and color(lab ...); details at [Safari technology preview 120](https://webkit.org/blog/11548/release-notes-for-safari-technology-preview-120/)
* color(a98-rgb ...), color(prophoto-rgb ...), color(rec2020 ...), color(xyz ...), hwb(); details at [Safari technology preview 121](https://webkit.org/blog/11555/release-notes-for-safari-technology-preview-121/)
* color-contrast() and color-mix() from CSS Color 5; details at [Safari technology preview 121](https://webkit.org/blog/11577/release-notes-for-safari-technology-preview-122/)

An importabnt difference is that WebKit uses the platform's graphic libraries directly (e.g. CoreGraphics on Mac) whereas Chromium uses Skia across different platforms. Support for displaying colors beyond the sRGB gamut may not be available in all platforms.


### High precision colors in Skia

Interestingly, Skia does not have the same limits in color precision and range as Blink does.

Internally, it has a format for high-precision colors that holds four float values, and it is also able to take color spaces into account.

> SkRGBA4f and SkColor4f in [third_party/skia/include/core/SkColor.h](https://source.chromium.org/chromium/chromium/src/+/master:third_party/skia/include/core/SkColor.h)

Much of the Skia API is already able to take as input a colorspace and one or more high precision colors defined in it. Skia is also able to convert between source and destination color spaces, so colors can be manipulated with flexibility before being adapted to be displayed on concrete hardware.

> In Skia, a color space is defined by a transfer function and a gamut.
> Transfer function: SRGB, 2Dot2, Linear, Rec2020, PQ, HLG.
> Gamut: SRGB, AdobeRGB, DisplayP3, Rec2020, XYZ.
> [third_party/skia/include/core/SkColorSpace.h](https://source.chromium.org/chromium/chromium/src/+/master:third_party/skia/include/core/SkColorSpace.h)

So, Skia is able to paint richer colors on hardware that supports them.

This means that, if we managed to get that rich color information defined in the Web sources at the beginning of the pipeline all the way to Skia at the end of the pipeline, we would be able to paint those colors correctly on the screen :)

There are a couple things worth mentioning here.

First, Skia's representation of high-precision colors still uses the RGBA structure, so out of the box Skia does not support other formats like LAB or LCH.

Secondly, as we have seen, the CSS Color spec provides ways to specify the interpolation colorspace for gradients, transitions, etc.

Blink relies on Skia for this interpolation, but Skia does not provide fine-grained control: Skia will always use the colorspace where the source colors have been defined, and does not support interpolating in a different space.


### Summary

As a very broad summary, the first step to support wider, richer color gamuts in Blink is to parse the CSS code using those new features.

Those wide gamut colors and their colorspaces need to be stored in a high-precision format that can be used throughout Blink's rendering pipeline.

At the end of the pipeline, it needs to be translated so Skia can paint those colors correctly in the desired hardware. For this, we will also need more fine-grained control over interpolation and probably other changes.

This work is not straightforward because it would touch a lot of different components, and it might also have an impact on memory, on performance, on how paint information is recorded and used, etc.

For Web authors it is important that these features are available at the same time, so they can rely on the new functionality provided by the CSS Color spec.


## In Closing

I hope that with this you got a better understanding of the value of adding richer colors to the Web and the scope of the work that would be needed to do so in Chromium.

These are some steps in the long road to increase the expressivity of the web platform and to widen the range of tools that are available to authors when creating the Web.

Thank you very much for reading.


