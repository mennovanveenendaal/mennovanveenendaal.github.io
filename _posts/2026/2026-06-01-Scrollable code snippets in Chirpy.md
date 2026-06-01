---
title: "Scrollable code snippets in Chirpy"
layout: post
categories: [Github Pages, Chirpy]
description: How to make the code snippets scrollable in a Chirpy themed website.
image:
  path: /assets/2026/scrollbars/scrollable.png
  alt: Scrollable
---

When writing blog posts, I often include example code. Some code blocks are only a few lines long, while others can contain dozens of lines of YAML, Bash, or configuration data. Small code snippets fit naturally within the article, but larger blocks can quickly take up too much space and disrupt readability.

## Maximum lines
After quite a bit of searching, I found a useful suggestion on [GitHub](https://github.com/cotes2020/jekyll-theme-chirpy/discussions/1319). The solution uses custom CSS in `./assets/css/jekyll-theme-chirpy.scss` to make code blocks scroll vertically once they exceed a defined height.

```css
/* append your custom style below */
$max-lines: 10; // max lines of code to be scrolled

.highlight {
    overflow-y: auto;
    max-height: $max-lines * 1em;
}
```

## Specific code blocks
This worked well, but I also wanted the ability to only set the maximum height restriction for specific code blocks. For example, log output or shorter examples often fit better when displayed in full.

To achieve this, I combined the two examples into a single solution and added it to `./assets/css/jekyll-theme-chirpy.scss`.

```css
/* append your custom style below */
$max-lines: 25; // max lines of code to be scrolled

div.scroll > .highlight  {
    overflow-y: auto;
    max-height: $max-lines * 1em;
}
```

Now, whenever I want a code block to become scrollable, I simply add `{: .scroll}` directly after the closing backticks. This keeps long code examples compact while allowing readers to scroll through the content when needed.

![.scroll](/assets/2026/scrollbars/scroll.png)
_Fig.1 Applying the `.scroll` class_

![no vertical bar](/assets/2026/scrollbars/no_vertical_bar.png)
_Fig.2 No vertical scroll bar_

![vertical bar](/assets/2026/scrollbars/vertical_bar.png)
_Fig.3 Vertical scroll bar_

## Horizontal scroll bar
Long lines can also create horizontal scroll bars, which are often less convenient for readers. To wrap long lines automatically, I added the following CSS to `./assets/css/jekyll-theme-chirpy.scss`.

This combines [line wrapping](https://www.dhiwise.com/post/display-code-snippets-with-an-html-code-block) and [word wrapping](https://stackoverflow.com/a/7760844) techniques.

```css
pre, code {
  white-space: pre-wrap; /* Wraps long lines */
  word-wrap: break-word;
}
```

![horizontal bar](/assets/2026/scrollbars/horizantal_bar.png)
_Fig.4 Example of horizontal scrolling_

![no horizontal bar](/assets/2026/scrollbars/no_horizantal_bar.png)
_Fig.5 Long lines wrapped without horizontal scrolling_




