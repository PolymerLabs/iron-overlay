[![Build Status](https://travis-ci.org/PolymerLabs/iron-overlay.svg?branch=master)](https://travis-ci.org/PolymerLabs/iron-overlay)

## The problem

Overlays should always render on the top layer. This is achieved using the css
properties `position: fixed` and `z-index`. When an overlay is contained in a node
that creates a new [stacking context](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Positioning/Understanding_z_index/The_stacking_context),
it causes the overlay to be trapped within it (z-index and position will be relative to
the new stacking context), resulting in problems like [this one](http://jsbin.com/kuboqa/1/edit?html,output):

```html
<style>
  .backdrop {
    position: fixed;
    top: 0; bottom: 0;
    left: 0; right: 0;
    z-index: 100;
    background-color: rgba(0, 0, 0, 0.5);
  }
  .overlay {
    position: fixed;
    z-index: 999;
  }
</style>
<div class="backdrop"></div>
<!-- this creates a new stacking context -->
<div style="transform: translateZ(0);">
  <div class="overlay">
    <button>click me!</button>
  </div>
</div>
```

`iron-overlay` works around this issue by "teleporting" its content to a
stacking-context-safe node.

```html
<!-- this creates a new stacking context -->
<div style="transform: translateZ(0);">
  <button onclick="this.nextElementSibling.opened = true">Open overlay</button>
  <iron-overlay>
    <template>
      <!-- overlay content -->
      This text will be stamped and appended to the document body.
    </template>
  </iron-overlay>
</div>
```

## iron-overlay

It delegates rendering of the overlay content to a renderer (`iron-overlay-renderer`).
It won't host the renderer, but request another element to host it through the
events `iron-overlay-attach` and `iron-overlay-detach`, or append it to `<body>`.
It requires overlay contents to be contained in a `<template>` (since they need
to be hosted in the renderer).

## iron-overlay-renderer

Takes care of the rendering (e.g. center overlay), and handles the switch from
opened state to closed state & viceversa.

## iron-overlay-container

Hosts the overlay renderers, keeps track of the opened overlays. This element should 
be placed in a stacking-context safe node (e.g. `document.body`).

```html
<div style="transform: translateZ(0);">
  <button onclick="this.nextElementSibling.opened = true">Open overlay</button>
  <iron-overlay>
    <template>
      <!-- overlay content -->
      This text will be contained in iron-overlay-content
    </template>
  </iron-overlay>
</div>

<!-- it will contain all the overlay renderers -->
<iron-overlay-container></iron-overlay-container>
```

## Styling

Styling must be done in the context of where iron-overlay will be hosted.

### Styling the renderer

`iron-overlay` sets the renderer's `data-overlay` attribute to be its id, so
that styling of the overlay can be done like this:

```html
<custom-style><style is="custom-style">
  [data-overlay] {  
    --iron-overlay-background-color: yellow;
  }
  [data-overlay="overlay1"] {
    --iron-overlay-background-color: orange;
  }
</style></custom-style>

<div style="transform: translateZ(0);">
  <iron-overlay>
    <template>Overlay Content</template>
  </iron-overlay>
  <iron-overlay id="overlay1">
    <template>Overlay 1 Content</template>
  </iron-overlay>
</div>
```

### Styling the content

Content can be styled by passing a `<style>` element into the template,
but beware of possible conflicts with classes, as selectors will apply to all
matching elements in the styling context where they're hosted:

```html
<iron-overlay>
  <template>
    <style>
      .my-content {
        background-color: yellow;
      }
    </style>
    <div class="my-content">Content</div>
  </template>
</iron-overlay>

<iron-overlay>
  <template>
    <!-- Will have yellow background as well -->
    <div class="my-content">Other Content</div>
  </template>
</iron-overlay>
```

The best approach to ensure style encapsulation is to create a custom element
for your content.

```html
<iron-overlay>
  <template>
    <my-content>Content</my-content>
  </template>
</iron-overlay>

<iron-overlay>
  <template>
    <my-other-content>Other Content</my-other-content>
  </template>
</iron-overlay>
```
