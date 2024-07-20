---
title: Setting up a Documentation Site with Jekyll and Github Pages
date: 2024-07-19 16:20:42 -0500
categories: [homelab,hardware]
tags: [servers,lenovo,intel,tp-link] # TAG names should always be lower case
---

_tldr: Jump straight to the [implementation](https://joschua.io/posts/2022/12/04/obsidian-grid-callouts/#implementation)._

## Arranging Obsidian content in columns without any plugins

Recently, I’ve been looking to rearrange the way I structure notes on books in Obsidian. In order to be able to display, sort and query book notes in various ways, I usually add a ton of metadata in the note’s frontmatter.

It works, but it’s not pretty. Anytime I open a book note there is a gray wall of text to scroll past.

![Screenshot of book note in Obsidian. Frontmatter takes up a lot of space.](https://joschua.io/_astro/1-obsidian-note-metadata.501d124e_Z2bCUpO.webp)

To make this a bit more manageable, I separated the metadata into fields related to the book (“About”) and my progress (“Reading”). Especially with Obsidian’s 1.0 redesign, [callouts](https://help.obsidian.md/How+to/Use+callouts) have become as nice looking as practical. I moved the fields from the YAML metadata to [inline dataview fields](https://blacksmithgu.github.io/obsidian-dataview/data-annotation/#field-types).

![Screenshot of book note with two callouts full of metadata.](https://joschua.io/_astro/2-callouts.7eb85d43_2wX5cc.webp)

Looks much cleaner and is much easier to scan! However, it still takes up a fair bit of vertical space despite the content not being too long.

Some callout and CSS grid magic and here we go:

![Screenshot of Obsidian note with two callouts horizontally next to each other.](https://joschua.io/_astro/3-callout-grid.e0b9047a_Zhtvy8.webp)

I added a bit of styling and am pretty happy with the result:

![Screenshot of Obsidian note with both callouts being styled nicely.](https://joschua.io/_astro/4-callout-styling.55936e8e_1l7vWa.webp)

## Implementation

The two callouts that are displayed side by side are actually encapsulated by another callout. This callout arranges all its content in columns.

![Obsidian note showing one larger callout containing two smaller ones within.](https://joschua.io/_astro/5-even-columns.b06c2b03_Z1rHdJF.webp)

This is what it looks like in the note:

```code
<span><span>&gt; [</span><span>!even-columns</span><span>]</span></span>
<span><span>&gt;</span></span>
<span><span>&gt; &gt; [</span><span>!abstract</span><span>] About</span></span>
<span><span>&gt; &gt;</span></span>
<span><span>&gt; &gt; - Type: #book/nonfiction</span></span>
<span><span>&gt; &gt; - [Author:: [[Cal Newport]]]</span></span>
<span><span>&gt; &gt; - [pages:: 305]</span></span>
<span><span>&gt; &gt; - [ddc:: 650.1]</span></span>
<span><span>&gt; &gt; - [Year published:: [[</span><span>2012</span><span>]]]</span></span>
<span><span>&gt;</span></span>
<span><span>&gt; &gt; [</span><span>!bookinfo</span><span>] Reading</span></span>
<span><span>&gt; &gt;</span></span>
<span><span>&gt; &gt; - [status:: read]</span></span>
<span><span>&gt; &gt; - [rating:: 4.75]</span></span>
<span><span>&gt; &gt; - [added:: 2022-10-29]</span></span>
<span><span>&gt; &gt; - [started:: 2022-10-29]</span></span>
<span><span>&gt; &gt; - [read:: 2022-10-29]</span></span>
```

A [CSS snippet](https://help.obsidian.md/How+to/Add+custom+styles#Use+Themes+and+or+CSS+snippets) in Obsidian applies the styling.

`.obsidian/snippets/callouts.css`:

```code
<span><span>/* Even columns */</span></span>
<span><span>.callout</span><span>[</span><span>data-callout</span><span>=</span><span>"even-columns"</span><span>] {</span></span>
<span><span>  /* Removes padding and background colour from the container */</span></span>
<span><span>  padding</span><span>:</span><span> 0</span><span>;</span></span>
<span><span>  background-color</span><span>:</span><span> transparent</span><span>;</span></span>
<span><span>}</span></span>
<span><span>.callout</span><span>[</span><span>data-callout</span><span>=</span><span>"even-columns"</span><span>] </span><span>&gt;</span><span> .callout-content</span><span> {</span></span>
<span><span>  /* Arranges the content in columns */</span></span>
<span><span>  display</span><span>:</span><span> grid</span><span>;</span></span>
<span><span>  /* minmax sets the minimum width of a column. Make the columns 'skinnier' by setting 15rem to a smaller number */</span></span>
<span><span>  grid-template-columns</span><span>:</span><span> repeat</span><span>(auto-fit</span><span>,</span><span> minmax</span><span>(15</span><span>rem</span><span>,</span><span> 1</span><span>fr</span><span>))</span><span>;</span></span>
<span><span>  gap</span><span>:</span><span> 12</span><span>px</span><span>;</span></span>
<span><span>}</span></span>
<span><span>.callout</span><span>[</span><span>data-callout</span><span>=</span><span>"even-columns"</span><span>] </span><span>&gt;</span><span> .callout-title</span><span> {</span></span>
<span><span>  /* Hides the callout title */</span></span>
<span><span>  display</span><span>:</span><span> none</span><span>;</span></span>
<span><span>}</span></span>
```

That’s it! Simply copy the above code into your snippets folder and activate it. Anytime you add a callout with the name `even-columns`, the content inside is arrange in, well, even columns.

What we are using here is a CSS property called `grid`. You can read more about grid at [CSS-tricks](https://css-tricks.com/snippets/css/complete-guide-grid/).

The great thing about `grid` is that it is really flexible. Each new line in our callout automatically moves into a new column.

```code
<span><span>&gt; [</span><span>!even-columns</span><span>]</span></span>
<span><span>&gt; left</span></span>
<span><span>&gt;</span></span>
<span><span>&gt; center</span></span>
<span><span>&gt;</span></span>
<span><span>&gt; right</span></span>
```

![Three columns in an Obsidian note.](https://joschua.io/_astro/6-three.9861c745_ZVkmth.webp)

On a smaller display size, for example on mobile, the columns stack responsively depending on how much space is available.

There are a couple of plugins out there that implement this functionality and might offer a bit more fine-grained control. For a basic use-case like mine, it is quite impressive how far you can get with a callout class and some CSS. It’s great how much customisation Obsidian already supports out of the box.
