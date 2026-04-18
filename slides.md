---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: ReScript-Shadcn
info: |
    Presentation of ReScript-Shadcn.

    ReScript Retreat 2026, Vienna.

    Learn more at [rescript-shadcn.miriad.studio](https://rescript-shadcn.miriad.studio/)
# apply UnoCSS classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
    persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable Comark Syntax: https://comark.dev/syntax/markdown
comark: true
# duration of the presentation
duration: 25min
---

# <img src="./icon.svg" class="inline-block w-16 mb-4" /> ReScript-Shadcn 

by Paul Tsnobiladzé

ReScript Retreat 2026, Vienna

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
class: text-2xl
---

# What is ReScript-Shadcn?

ReScript-Shadcn is a rewrite of shadcn design system in ReScript.

- same philosophy as shadcn
- pixel-perfect rewrite
- compatible with shadcn ecosystem (CLI, MCP, etc)
- not a library 
- a collection of components you can import individually
- meant to be customized to your needs
- built-in dependency tracking
- based on base-ui
- comprehensive documentation

---
class: text-3xl
layout: center
---

# The challenges I've faced 

- Bindings / rewrite in ReScript
- ReScript and next.js
- dependency-tracking

---
class: text-2xl
---

# Bindings / rewrite in ReScript

Main characteristics: 
- mostly done by AI with rescript-bindings skill
- bindings to base-ui components:
    - some custom props + common DOM props
    - use of `render` prop
- rewrite of shadcn components in ReScript
    - heavy use of props spreading 
    - heavy use of `data-*` attributes

--- 
class: text-2xl
---

## The implementations I've tried

1. write a custom type for each base-ui component:
    - too time-consuming
2. write a common type for every base-ui components
    - clashing types for some common props (value, onChange, etc)
3. back to custom type for each base-ui component
    - after some guidance AI produced overall correct code
    - added some DOM/pixel comparison to help AI

--- 
class: text-xl
---

## The issue with props drilling

1. cumbersome for large props (inheriting from DOM props)
    - could start with a limited set of props thanks to customizability
2. Different semantics between optional field in ReScript and JS
    - incompatible with object spreading:
```js
function MyJsComponent({ className, ...otherProps }){
    const props = {
    "data-slot": "some-default-slot",
    ...parentProps
    }
    return (<SomeOtherComponent ...props />)
}
```
the following ReScript code:
```js
<MyJsComponent ?dataSlot />
```
produces: 
```js
const props = { "data-slot" : undefined }
```

--- 
class: text-2xl
layout: center
---

# Solution: support destructuring record rest

Implemented in [rescript-lang/rescript#8317](https://github.com/rescript-lang/rescript/pull/8317)

```js
@react.component
let make = ({ ?className, ...MyJsComponent.props as props}) => {
  <MyJsComponent 
    {...props} 
    className=`text-lg ${className->Option.getOr("")}` />
}

```

--- 
class: text-2xl
---

# Next.js and ReScript

Overall, the experience is not that great.

1. App router based on filenames:
    - fixed with JS file shims 
    - cumbersome, what other solutions could we offer?
2. Nested components
    - next can't find components inside submodules from a server component
    - potential fix (but breaking) in [rescript-lang/rescript#8293](https://github.com/rescript-lang/rescript/pull/8293)

--- 
class: text-2xl
---

# dependency-tracking 

1. originally analyzed imports from JS files
    - missed the rescript type dependencies 
2. fixed by adding custom .ast analysis 
    - should be provided by rescript-tools
3. you need both to track both rescript and JS dependencies

--- 
class: text-2xl
layout: center
---


# Potential reuse?
1. could be a great fit for binding distribution
2. any other ideas?