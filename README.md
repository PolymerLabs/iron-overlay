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
<!-- it will contain all the overlay renderers -->
<iron-overlay-container></iron-overlay-container>

<!-- this creates a new stacking context -->
<div style="transform: translateZ(0);">
  <button onclick="this.nextElementSibling.open()">Open overlay</button>
  <iron-overlay>
    <template>
      <!-- overlay content -->
      This text will be contained in iron-overlay-content
    </template>
  </iron-overlay>
</div>
```

## iron-overlay

It delegates rendering of the overlay content to a renderer (`iron-overlay-renderer`).
It won't host the renderer, but request another element to host it through the
events `iron-overlay-attach` and `iron-overlay-detach`. It requires overlay contents
to be contained in a `<template>` (since they need to be hosted in the renderer).

## iron-overlay-renderer

Takes care of the rendering (e.g. center overlay), and handles the switch from
opened state to closed state & viceversa.

## iron-overlay-container

Hosts the overlay renderers, keeps track of the opened overlays and delegates `tap`
event and `esc` keyboard event to the top overlay. This element should be placed
in a stacking-context safe node (e.g. `document.body`).

## Styling

Content will be stamped outside the styling scope of `iron-overlay`. To ensure
there is no leaking of styles for the content, the best approach is to create a
custom element for your content.

To help styling the content, `iron-overlay` sets the renderer's `data-overlay`
attribute to be its id, so that styling can be done like this:

```html
<style is="custom-style">
  iron-overlay-renderer {  
    --iron-overlay-background-color: yellow;
  }
  iron-overlay-renderer[data-overlay="overlay1"] {
    --iron-overlay-background-color: orange;
  }
</style>

<iron-overlay-container></iron-overlay-container>

<iron-overlay>
  <template>Overlay Content</template>
</iron-overlay>
<iron-overlay id="overlay1">
  <template>Overlay 1 Content</template>
</iron-overlay>
```

Content can be styled as well by passing a `<style>` element into the template,
but beware of possible conflicts with classes, as selectors will apply to all
matching elements contained in `iron-overlay-container`:

```html
<iron-overlay-container></iron-overlay-container>

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
