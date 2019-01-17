# Forced Dark Mode for the Web

**Authors:**

* Dominic Mazzoni, Google, dmazzoni@google.com

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Introduction](#introduction)
- [Use cases](#use-cases)
- [History](#history)
  - [Approach 1: Overriding CSS colors and background images](#approach-1-overriding-css-colors-and-background-images)
  - [Approach 2: Inverting Colors for the Whole Screen](#approach-2-inverting-colors-for-the-whole-screen)
  - [Approach 3: Rewriting CSS](#approach-3-rewriting-css)
  - [Approach 4: Smart inversion](#approach-4-smart-inversion)
- [Scope](#scope)
- [Existing prefers-color-scheme media query](#existing-prefers-color-scheme-media-query)
- [Meta element for color scheme support](#meta-element-for-color-scheme-support)
- [CSS color-scheme property](#css-color-scheme-property)
- [CSS invert property](#css-invert-property)
  - [How is invert different than the CSS filter?](#how-is-invert-different-than-the-css-filter)
  - [Why both color-scheme and invert?](#why-both-color-scheme-and-invert)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Introduction

Some operating systems now offer a Dark Mode, where most UI is presented with
light text on a dark background. Advantages can include lower battery life on
some types of displays, accessibility, or just user preferences. Ideally each
individual app provides its own dark theme, however when one is not provided,
some systems may allow the user to choose to force apps into a dark mode anyway.

This document proposes new web standards to enable web browsers to implement
a forced dark mode while still giving web developers control over the appearance
of their own sites.

## Use cases

* Battery: on OLED displays, dark pixels consume less power than light ones, so
  a dark theme can use
  [as much as 60% less power](https://www.phonearena.com/news/Google-promotes-the-use-of-Dark-Mode_id110831).
* Accessibility: many people have an easier time reading light text on a
  dark background.
* User preference: Some people find a dark theme more aesthetically pleasing.

## History

Dark Mode has actually been around for decades under a different name.
Until recently it was typically known as High Contrast Mode. While
the recent focus has been on improving battery life, millions of users
with low vision have been relying on dark mode on desktop computers
because they find it easier to see, and this has included support in
web browsers.

### Approach 1: Overriding CSS colors and background images

When you enable High Contrast mode on Windows (Alt+Shift+PrtScr), both
IE and Firefox load an alternate stylesheet that uses white and blue
as foreground colors and black as the background color, and completely
disables web pages from overriding these. Notably, they also
completely disable background images.

Pros: Text is guaranteed to be readable, foreground images look
fine. The whole UI is consistently dark. Sites can detect it's enabled.

Cons: Background images are often important, so many sites become
completely broken. Site layouts become hard to understand because
borders, shadows, and background colors are all ignored.

### Approach 2: Inverting Colors for the Whole Screen

MacOS, iOS, Android, and Chrome OS all support this option to invert
at the pixel level. People who use this feature report that they
toggle it frequently, hundreds of times a day, as some text is easier
inverted and other text is not.

Pros: Simple, foolproof, and quick to toggle. Safari on MacOS supports
the media query, so a few web sites look better.

Cons: Images look terrible, and color schemes often look strange.

### Approach 3: Rewriting CSS

Extensions like Dark Reader (~700k users on Google Chrome) fetch all
of the stylesheets for a page and then override them with new
stylesheets that replace colors with their complement.

Pros: Images look great, extension is fine-tuned to make top sites
look really good.

Cons: Flickers when a site is loading, depends on hundreds of custom
rules for top sites, so doesn't scale well. Impacts performance.

### Approach 4: Smart inversion

[iOS has a Smart Invert feature](https://support.apple.com/en-us/HT207025)
that reverses all colors on the display,
except for images, media, and some apps that use dark color styles.
Developers can opt out of smart invert and handle it themselves on
a per-view level using
[accessibilityIgnoresInvertColors](https://developer.apple.com/documentation/uikit/uiview/2865843-accessibilityignoresinvertcolors).

Pros: This provides a good experience for most apps while giving
developers the option to have full control.

## Scope

How a web browser might implement forced dark mode is beyond the scope
of this document. Browsers may use heuristics to improve the
appearance of legacy sites who don't update their sites based on the
new standards. They may use any permutation of the approaches
described above. For all of those approaches, it should be possible
for a web site to opt-out and get more control over their own theme.

## Existing prefers-color-scheme media query

The [Media Queries Level 5](https://drafts.csswg.org/mediaqueries-5)
spec proposes a new
[prefers-color-scheme](https://drafts.csswg.org/mediaqueries-5/#prefers-color-scheme)
media feature, with possible values of no-preference, light, and
dark.

This enables a website that offers both light and dark themes to
automatically select the right theme.

## Meta element for color scheme support

Without a way for a site to clearly specify that it uses a dark theme,
the possible interaction would be poor between forced dark mode and a site that
respects the new media query, or a site that's always dark.

Instead, a site should be able to specify its support for the
prefers-color-scheme media query by listing all of the color
schemes it supports, like this:

```<meta name="color-schemes" value="light,dark">```

The value of the name attribute must be "color-schemes".

The value of the value attribute must contain a comma-separated
list of color schemes, with no whitespace. Initially only
"light" and "dark" would be allowed.

If a web browser is in forced-dark mode, it must not modify
the presentation of any web page that indicates it supports a
dark color scheme.

Similarly, if a web browser is in forced-light mode, it must not
modify the presentation of any web page that indicates it
supports a light color scheme.

## CSS color-scheme property

Many web sites are built out of a mix of components from many different
sources. It might not be realistic for a site to create a custom dark
scheme for the entire site. Rather than only allowing web authors a
way to either completely provide their own dark scheme or use the
browser's force dark, a CSS property could allow sites to control
how each individual element is rendered:

```color-scheme: light | dark | no-modify;```

If the value of the color-scheme property is set to ```light```, a
browser in force-dark mode may invert this element's colors.
Similarly, if the value of the color-scheme property is set to
```dark```, a browser in force-light mode may invert this element's
colors.

If the value of the color-scheme property is set to ```no-modify```,
the web browser must never modify the color scheme of this element
and its descendants. This might be appropriate for a photographic
image, for example, that should never be inverted.

## CSS invert property

As an alternative or complement to the color-scheme property, a
CSS property could be used to explicitly invert an element and its
descendants.

```invert: rgb | luminance | none | auto;```

If both color-scheme and invert were to be implemented, they could
overlap and it would have to be decided which takes precedence if
both are provided and conflict.

If the value of the invert property is set to ```rgb```,
all colors for that element and its descendants are painted with
the RGB inverse color (R' = 1.0 - R, G' = 1.0 - G, B' = 1.0 - B).

If the value of the invert property is set to ```luminance```,
all colors for that element and its descendants are painted with
the luminance component of the color in HSL space replaced with its
inverse (L' = 1.0 - L).

If the value of the invert property is set to ```none```,
colors will be unchanged.

If the value of the invert property is set to ```auto```
(the default), the browser would be able to choose what modifications
to make to any colors based on the color scheme and force-dark modes.

### How is invert different than the CSS filter?

When a CSS filter is applied to an element, that element is put
into a graphics layer and the filter is applied to the layer
before compositing it to the page. This not only adds computational
cost, but it necessarily applies to an entire subtree of eleemnts.
It's not possible to only apply a filter to a parent element but
not its child elements, for example - unless you can apply a
complementary filter to the children.

In comparison, the invert property is applied separately to each
element as it's painted to the viewport. A div with a black
background and invert: rgb applied to it will now have a white
background, but if its children have invert: none applied, then
they'll still be painted with their original colors.

### Why both color-scheme and invert?

The color-scheme and invert properties attack the problem from
different directions.

The color-scheme property is only meaningful if the browser has
enabled a force dark mode. It has no effect otherwise. Sites could
add the color-scheme property to parts of the site in order to fix
rendering issues but never actually use the media query or build
two themes.

The invert property is more low-level. If implemented, it takes
effect completely independent of the current color scheme or the
browser's support for a force dark mode.

In particular, the invert property could be used to quickly and easily
implement a dark color scheme in browsers that expose the
prefers-color-scheme media query but don't have any force-dark mode.
Some sites have hundreds of stylesheetes spread across many components.
Updating all of them to give them a dark scheme might be practically
impossible, but the invert property could be used to quickly retrofit
an entire site with just a few small changes.
