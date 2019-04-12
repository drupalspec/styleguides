# SVGs

Scalable Vector Graphics - they're Retina-friendly by default and lighter than most other image formats. Use them whenever you can! (Here's how…)

## Best Practices

There are multiple ways to use an SVG in a webpage (but we stick to two):

* 👉 inline directly in page markup
* 👉 as an inlined CSS `background-image` with `url('data:image/svg+xml;utf8,<svg></svg>)`
* as the `src`/`srcset` for an `img` tag (or other media element, like `picture > source` - you might use this for art direction but not responsive images, since SVGs are resolution-independent already)
* as the `data` attr in an `<object>` tag
* as a CSS `background-image` with `url('path/to/file.svg')`

By default, we prefer **inline SVG** (see [SVGJar](#SVGJar)), especially when CSS styling of element colors is necessary, or where the layout requires the SVG to define its own space.

For background decorations, we use **background-image SVGs**, but we don't manually manage the inlining. [`narwin-pack`](https://github.com/DockYard/narwin-pack/blob/master/index.js) uses [`postcss-inline-svg`](https://github.com/TrySound/postcss-inline-svg) to convert those to inline data SVGs. This `background-image` technique has the advantages using an SVG without creating additional markup, but it lacks the ability to style the SVG or its children with CSS.

```postcss
.icon--background {
  background-image: svg-load('assets/icons/download.svg', fill=#283ae2, stroke=#fff);
}
```

You may have heard about icon fonts. [SVG icons are better](https://blog.github.com/2016-02-22-delivering-octicons-with-svg/).

## Accessibility

If you're using an SVG as an image's `src`, follow the normal rules for `alt` text and accessibility.

If the SVG is inline, add `role="presentation"` if it's purely decorative. However, if it needs descriptive text, add an `aria-label` to the SVG tag and consider putting that text in a [`<title>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/desc) element as well. The `aria-label` will be announced to screen readers and the `<title>` will create a hover "tooltip" for sighted users on devices with pointers.

You may also consider adding [`<desc>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/desc) elements. If an SVG is rendered in a non-graphical way, `<desc>` elements may be shown/read to users. You can add a `<title>` or a `<desc>` to every `<g>` if needed. Keep your users' needs in mind, however: if assistive tech reading _multiple_ descriptions or titles would be tedious or pedantic, stick with a single description.

```xml
<svg aria-label="DockYard Logo">
  <title>DockYard Logo</title>
  <path />
</svg>
```

## Styling SVGs with CSS

SVG children element have lots of [attributes that define their visual presentation](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/Presentation) (the most common are `fill`, `stroke`, and all their sub-attributes). Many of those attributes are available to CSS as well.

_This_

```xml
<svg class="icon">
  <path fill="#283ae2">
</svg>
```

_is equivalent to this:_

```postcss
.icon path {
  fill: #283ae2;
}
```

### BEM

Using plain tag names can cause unintended style overlap, so it's safest to use BEM classes to select and style inner elements:

```xml
<svg class="icon">
  <path class="icon__path-1" />
  <path class="icon__path-2" />
</svg>
```

```postcss
.icon__path-1 {
  fill: #283ae2;
}

.icon__path-2 {
  fill: #3ae228;
}
```

### Hover (and Other Pseudo-Selector State) Effects

If you need color changes on `:hover` or other pseudo-selector states, remember to put those on the right containing element:

_Not gonna do what you want:_

```postcss
.icon path {
  fill: #283ae2

  &:hover {
    fill: #3ae228;
  }
}
```

_You probably want this:_

```postcss
.icon path {
  fill: #283ae2;

  .button:hover & {
    fill: #3ae228;
  }
}
```

### Using `currentColor`

Don't forget, `currentColor` is a native CSS variable that lets SVG icons inherit colors from their parent elements. This is particularly useful for single-color icons that need to match the color of the text they're set next to:

_The `.link__icon` SVG will inherit the `.link`'s colors:_

```postcss
.link {
  color: #283ae2;

  &:hover {
    color: #3ae228
  }
}

.link__icon {
  fill: currentColor;
}
```

### Stroke Scaling

Sometimes you'll need the `stroke` on an SVG to stay at a fixed pixel width, regardless of the image's size. (Normally, `stroke-width` will scale in line with the SVG's coordinate system as the SVG itself is resized.) To do this, add the attribute `vector-effect="non-scaling-stroke"` to the stroke elements (or to the SVG itself, if all elements should have non-scaling `stroke-width`).

_Try this, per element:_
```xml
<svg>
  <circle stroke="red" stroke-width="1" />
  <circle stroke="blue" stroke-width="1" vector-effect="non-scaling-stroke" />
</svg>
```

_Or this, for the whole image:_
```xml
<svg vector-effect="non-scaling-stroke">
  <circle stroke="red" stroke-width="1" />
  <circle stroke="blue" stroke-width="1" />
</svg>
```

## Safe Click Events

SVGs (and all their child elements) can intercept a user's click event. Occasionally, you'll find this causing problems when an inline SVG is inside an `a` or `button`. To prevent an SVG or its children from intercepting a click event that should bubble up to the intended parent element, use `pointer-events` - this feature can be set using either attributes or CSS.

_CSS_

```css
a {
  svg,
  svg * {
    pointer-events: none;
  }
}
```

_XML_

```xml
<svg pointer-events="none"></svg>
```

## Optimization

Start by taking your SVG (from Design handoff, or exported from a design app) over to [SVGOMG](https://jakearchibald.github.io/svgomg/). The default options will be just what you need most of the time. You can see both the visual result and code output by toggling "Image" and "Markup" in the top navigation. Occasionally, an optimization may break the visual rendering. In this case, toggle the settings in the "Features" list until it's no longer broken and optimize the image without that setting enabled. Don't hesitate to ask another UX Dev for help - SVG spec is complicated and none of us can memorize all the tiny gotchas that cause big problems. Rarely, you may need to go back to Design and pair to get an SVG built and exported in a way that can be optimized better.

SVGOMG is a PWA that will work offline. Also, it processes everything in your browser; it's safe to use on projects with "no 3rd party apps" NDAs.

### Keep `viewBox`

The `viewBox` attribute defines the SVG's internal dimensions/measurements and should be preserved. Whether you size the SVG with CSS properties or HTML attributes, keeping the `viewBox` ensures the SVG's inner elements will be scaled correctly across browsers.

_This is stable:_

```xml
<svg viewBox="0 0 160 90" width="32" height="18"></svg>
```

_So is this:_

```postcss
.icon {
  width: 32px
  height: 18px;
}
```
```xml
<svg class="icon" viewBox="0 0 160 90"></svg>
```

_This, not so much…_

```xml
<svg></svg>
```

### Keep `<defs>` containers

The `<def>` element is used to store graphical objects that will be used at a later time, like `linearGradient` for example. This element will appear to be an extraneous element that would commonly get removed during SVG optimization but due to issues with graphical elements in Safari, this should not be removed. [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/defs) explains:

> Defining these objects inside of a `<defs>` element promotes understandability of the SVG content and is beneficial to the overall accessibility of the document.

### Remove `<style>` tags

Many image apps add `fill`, `stroke` or other visual properties to inner elements using classes and a `style` tag. When you're using multiple SVGs in a page, those classes may overlap and cause unexpected style issues. Convert all `style` blocks to inline attributes on child elements.

If an SVG is intended for use as a `background-image`, all `stroke` and `fill` values must be inline on the elements that need them. If an SVG is intended for inline use and interactive styling is necessary, [BEM class names](https://github.com/dockyard/styleguides/blob/master/ux-dev/class-naming-conventions.md) should be added where [CSS style hooks](#BEM) are necessary. These classes allow us to style states (`:hover`, etc) easily. For inline SVGs that contain static illustrations (especially complex ones, [example: DY Design hero image](https://dockyard.com/design-consulting)), inline `stroke` and `fill` are preferable to BEM classes and CSS styling.

_Change this:_

```xml
<svg>
  <style>
    .cls-1 {
      fill: #283ae2;
    }
  </style>
  <path class="cls-1" />
</svg>
```

_…into this:_
```xml
<svg>
  <path fill="#283ae2" />
</svg>
```

### Transforms

Occasionally, a design app will apply unnecessary transforms to inner element (and maybe even unnecessarily offset the `viewBox` too). SVGOMG doesn't always catch these, but with a little addition/subtraction, it's not too terrible to optimize them manually.

_Nope:_

```xml
<svg viewBox="-15 -15 33 33" xmlns="http://www.w3.org/2000/svg">
  <path d="..." transform="translate(-15 -15)"/>
</svg>
```

_Yep:_

```xml
<svg viewBox="0 0 33 33" xmlns="http://www.w3.org/2000/svg">
  <path d="..." />
</svg>
```

Sub-par exports could affect SVG child element positioning too:

_Still nope:_

```xml
<svg viewBox="0 0 32 32" xmlns="http://www.w3.org/2000/svg">
  <circle r="12" cx="24" cy="24" transform="translate(-8 -8)" />
</svg>
```

_Yes, please:_

```xml
<svg viewBox="0 0 32 32" xmlns="http://www.w3.org/2000/svg">
  <circle r="12" cx="16" cy="16" />
</svg>
```

These manual optimizations can get much trickier inside a `path` values list, however. (If there are capital letters scattered throughout the values, those point to absolute coordinates, so each following x/y pair needs to be adjusted to offset a stray `transform` or `viewBox` offset.) Typically, SVGOMG will fix those once non-`path` elements have been cleaned up from unnecessary transforms. But if not, and you're stuck, holler at your UXD pals for help - either we'll figure it out, or we might just stop optimizing if it's too tricky.

## SVGJar

Many DY projects use [ember-svg-jar](https://www.npmjs.com/package/ember-svg-jar). If you have a question about it, start with its [docs](https://github.com/ivanvotti/ember-svg-jar#installation) and [issues](https://github.com/ivanvotti/ember-svg-jar/issues)

### Adding SVGs

Optimize new SVGs, then drop them in your project's `/public/assets/` directory. If your project has more specific rules (like a `/svgs/` dir in `/assets/`), follow those rules. (SVGJar will find SVGs anywhere in `/public/` and can be configured to look in other directories if a project requires it.)

### Using SVGs in Templates

In your template file, use the SVGJar component:

```hbs
{{svg-jar "path/to/file-name"}}
```

This component also handles several other useful properties, including `class`, `height`, `width`, and more:

```hbs
{{svg-jar "icons/logo" class="header__logo" height="70" width="140"}}
```

### Previewing Available SVGs

To preview all the SVGs available to SVGJar, go to `{your local dev domain & port number}/ember-svg-jar/index.html`.

This previewer page will also let you quickly copy the Ember component code needed for any available SVG.

## Resources

* [MDN](https://developer.mozilla.org/en-US/docs/Glossary/SVG)
* [Jenkov's Tutorials](http://tutorials.jenkov.com/svg/index.html)
* Sara Soueidan: [her blog](https://www.sarasoueidan.com/blog/), [her CSS-Tricks posts](https://css-tricks.com/author/sarasoueidan/), [anything](https://duckduckgo.com/?q=sara+soueidan+svg)
* [Sarah Drasner](https://sarahdrasnerdesign.com/)
* [Una Kravets](https://una.im/archive/)
