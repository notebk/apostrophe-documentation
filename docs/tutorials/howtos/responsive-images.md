---
title: "Responsive images"
layout: tutorial
---

The `apostrophe-images` widget provides a powerful way to select and display images on the site, including the ability to manually crop images. But sometimes, this isn't best for mobile-responsive site designs.

## Responsive images can meet mobile design needs

Rather than scaling all of the images to have the same height while displaying at their full width, it can be more appropriate to allow CSS to crop the image, creating a "vignette" effect in which the design doesn't require that users see the entire image — they just need a background to fill, for instance, the top half the screen on their device of choice, whether it is a smartphone or a desktop browser. This isn't hard to achieve with the `background-size: cover` CSS property and the `noHeight` option of the `apostrophe-images` widget:

```less
// In your own .less file
.my-marquee .apos-slideshow-item
{
  background-size: cover;
  max-width: 100%;
  height: 50vh;
}
```

```markup
{# in your own page template #}

<div class="my-marquee">
  {{ apos.area(data.page, 'marquee', {
    widgets: {
      'apostrophe-images': {
        size: 'full',
        noHeight: true
      }
    }
  }) }}
</div>
```

> Normally, Apostrophe ensures that all images in a single widget, i.e. a single slideshow, are displayed at the same height and allows their widths to vary. This is done to prevent the rest of the page from "jumping" every time the slideshow advances to a slide with a different aspect ratio. This is great until what we really want is responsive CSS cropping. The `noHeight` option disables this behavior, making us responsible for achieving a consistent height via our CSS.

## Everything was great until the boss lost his head

Unfortunately, this strategy has a key flaw. Everything is great until the marquee is a photo of the boss, and you can't see his or her face on a smartphone.

This problem can be solved with the `focalPoint` option, which gives the user the ability to choose a "focal point" within the image that is guaranteed to be visible:

```markup
{# in your own page template #}

<div class="my-marquee">
  {{ apos.area(data.page, 'marquee', {
    widgets: {
      'apostrophe-images': {
        size: 'full',
        noHeight: true,
        focalPoint: true
      }
    }
  }) }}
</div>
```

Users will now see an "eyeball" icon in the list of images they have selected for the widget. Clicking that icon opens a dialog box in which they can click to choose a focal point.

## What the focal point actually does

The focal point, which is stored as a percentage on the X and Y axes, is output as a `background-position` CSS property. Note that this only makes a difference in practice if `background-size: contain` is not in effect and the `noHeight` option has been passed to the widget.

> If you have overridden `widget.html` for the `apostrophe-images` widget, take a look at the latest version of `widgetBase.html` to see how this can be applied in your override.

## Other ways to use the focal point data

This is great if you are rendering the widget normally with `apos.area` or `apos.singleton`, but what if you are fetching the images directly, for instance with `apos.images.first`?

Here's an example of how you can combine that technique with a focal point:

```markup
{% set image = apos.images.first(data.page.main) %}
{% if image %}
  <div class="focal-point-test" style="
    {%- if apos.attachments.hasFocalPoint(image) -%}
      background-position: {{ apos.attachments.focalPointToBackgroundPosition(image) }};
    {%- endif -%}
    background-image: url({{ apos.attachments.url(image, { size: 'one-half' }) }})"></div>
{% endif %}
```

## Accessing the the focal point data directly

If you are not setting `background-position` and wish to use the focal point percentages in another way, you can call `{{ apos.attachments.getFocalPoint(image) }}` to obtain an object with `x` and `y` properties. These numbers will be percentages of the width and height of the image, as cropped by the user.

## Focal points for freestanding attachments

Freestanding attachment schema fields — those that are part of your own piece types, and thus never in Apostrophe's main "images" media library — can also have focal points. You can enable that by setting the `focalPoint: true` option for the schema field in question.

## Alternatives

Sometimes even the focal point strategy doesn't give end users enough confidence that critical elements won't be cropped out.

One alternative is to provide two `apostrophe-images` widgets, which can be manually cropped for two fixed aspect ratios, and use CSS media queries to decide which one appears on a given device.