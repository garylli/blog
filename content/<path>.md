---
title: <path> element
tags:
  - "#svg"
draft: "true"
---
When drawing with the `<path>` element, there are some peculiarities in how the browser determines whether to loop back to the beginning of the path or not.

> When the path has a fill specified to `none`, the browser will not loop the stroke back to the beginning.
> However, if you append the draw command with `z`, it will close the path.

##### Intersecting Paths
> “ Since paths are, by default, filled with black, it is natural to wonder what happens when the path crosses itself.”
>
> — [*An SVG Primer*](https://www.w3.org/Graphics/SVG/IG/resources/svgprimer.html#preface)

You can specify the behavior intersecting paths will exhibit using the `fill-rule` property.  This allows you to cut out overlapping parts