+++
categories = ["templates"]
title = "Website Showcase & Template"
description = "Template and showroom for website's features."
author = "Raul Morales Delgado"
date = "2020-09-01"
toc = true
tags = ["hugo", "markdown", "latex", "bokeh"]
ismath = true
isbokeh = true
+++


## Introduction
The purpose of this post is to serve as a test for deploying new website features and showcase them, and to serve as a template holder. If you would like to know how these features have been enabled, please refer to my post on [deploying a website with Hugo](/all/deploying-website-hugo/). 


## Front Matter
The following code is a `toml` front matter template used for all documents in this website. Values for taxonomies depend on the key.
```toml
+++
content = ["tutorials", "projects", "nuggets", "templates"]  # This is closed to these values
author = "Raul Morales Delgado"
title = "Title for Document"
description = "Description for the document. Intro-like text."
cover = "/path/to/absolute/img.jpg"  # if absolute path
# or
cover = "img.jpg" and useRelativeCover = true  # if relative path
coverAlt = "Description of image"
coverCaption = "Image credit to [text](https://example.com/)"
date = ["date", "publishDate", "lastmod"]  # e.g. "2000-01-01"
lastmod = [":git", "lastmod", "date", "publishDate"]  # Not necessary since git file's metadata is enabled. Add ":fileModTime" to config.toml to enable file's metadata. 
publishDate = ["publishDate", "date"]  # Won't appear until publishDate
expiryDate = ["expiryDate",]  # Won't appear anymore after expiryDate
toc = true  # Table of Contents; default is "false", but can be manually set to "true" on a per-post basis
tags = ["a", "b", "c"]  # Open list with post's tags
ismath = true  # Only necessary if post contains latex-like formulae.
isbokeh = true  # Only necessary if post contains Bokeh plots.
+++
```

## Accessing Resources
Accessing resources — i.e. calling files in a post — depends on the resource type: static (i.e. absolute) from the `/static` directory, or relative, located at post's current directory. 

This website has been designed to have each post in a unique subdirectory, `/content/../post/`. The post itself will be an `index.md` file located in such path and ad hoc content for a post can be organized in subdirectories, e.g. `/contents/projects/my-first-post/img/photo-1.jpg`.

To access static resources, a file must be called using an absolute path whose root is at `/static`. For instance, a static picture can be accessed using `/my-photo.jpg`, or `/img/my-photo.jpg` (if its located at `/static/img/my-photo.jpg`).

To access relative resources, a file must be called using a relative path to the post's `index.md` file. For instance, to access `/content/projects/my-first-post/img/photo-1.jpg`, one must call `img/photo-1.jpg`. Note that there is no `/` at the beginning — making it a relative path — and that `index.md` would be at the same level as the `img/` subdirectory.


## Headings with Anchors
This website supports automatic anchors for headings — all headings (i.e. `h{2..6}`) will have a `#` next to the heading referencing the URL of the heading. These anchors have set to be invisible by default, and only visible when hovering.


## Image Embedding
This website features image embedding using regular Markdown embedding and using shortcodes for image embedding.

### Markdown Embedding
Regular Markdown image embedding can be performed easily. For block image embedding, use the following code patterns:
```text
![Alt text](<absolute or relative path to image, or URL> "<Image Title>")  # Without hyperlink
```
or
```text
[![Alt text](<absolute or relative path to image, or URL> "<Image Title>")](<URL for hyperlink>)  # With hyperlink
```

For instance, this code:
```text
[![Test image](img/test.jpg "Bruce Trail, Bruce Peninsula, ON, Canada. (Source: Raul Morales Delgado)")](https://rmoralesdelgado.com/about/)
``` 
yields the following:
[![Test image](img/test.jpg "Bruce Trail, Bruce Peninsula, ON, Canada. (Source: Raul Morales Delgado)")](/about/)

Note, though, that images embedded this way will be displayed in their original size (capped by `max-width`).

Inline images are discouraged since the result will be identical to that of block embedding.

### Shortcodes for Image Embedding
This theme also includes three shortcodes for image embedding, namely, `image`, `figure` and `imgproc`.

The `image` shortcode is the simplest one, and can be used as follows:
```text
{{</* image src="img/test.jpg" alt="Test image" position="center" style="border-radius: 8px; max-width: 400px" */>}}
```
{{< image src="img/test.jpg" alt="Test image" position="center" style="border-radius: 8px; max-width: 400px" >}}

The `figure` shortcode provides support for adding a caption and modifying the caption's style:
```text
{{</* figure src="img/test.jpg" alt="Test image" position="center" style="border-radius: 8px; max-width: 400px; margin: auto" caption="Bruce Trail, Bruce Peninsula, ON, Canada. (Source: Raul Morales Delgado)" captionPosition="center" captionStyle="color: currentColor;" */>}}
```
{{< figure src="img/test.jpg" alt="Test image" position="center" style="border-radius: 8px; max-width: 400px; margin: auto" caption="Bruce Trail, Bruce Peninsula, ON, Canada. (Source: Raul Morales Delgado)" captionPosition="center" captionStyle="color: currentColor;" >}}

Finally, the `imgproc` shortcode allows to `Resize`, `Fill` or `Fit` an image. It can be used as follows:
```text
{{</* imgproc "img/test.jpg" Resize "350x" center */>}} 
Bruce Trail, Bruce Peninsula, ON, Canada. (Source: Raul Morales Delgado)
{{</* /imgproc */>}}
```
{{< imgproc "img/test.jpg" Resize "350x" center >}} 
Bruce Trail, Bruce Peninsula, ON, Canada. (Source: Raul Morales Delgado)
{{< /imgproc >}}

>**Note:** The `imgproc` shortcode has been modified to allow centering the image over the caption. Please refer to my post on [deploying a website with Hugo](/all/deploying-website-hugo/).


## Code Snippets
This website supports code snippets inline, as a block and as an interactive collapsable block using a shortcode.

### Inline Code Snippets
Inline code snippets are included by placing backticks around the desired text. For instance, `` `this little snippet` `` will look like `this little snippet`.

### Block Code Snippets
As with other Markdown flavours (e.g. Github), block code snippets have to be wrapped at top and bottom by three consecutive backticks. For instance, the following code:
```text
```python
import numpy as np
``` 
```

will produce the following block:
```python
import numpy as np
```

### Shortcode for Code Snippets
Finally, this theme includes a shortcode, `code` that enables collapsable block code snippets. For example, this code:
```text
{{</* code language="python" title="Test code" id="1" expand="show" collapse="╳" isCollapsed="true" */>}}
import numpy as np
{{</* /code */>}}
```
will yield the following block:
{{< code language="python" title="Importing Numpy" id="1" expand="show" collapse="╳" isCollapsed="true" >}}
import numpy as np
{{< /code >}}


## Emoji Support
>**Note:** To enable emoji support, the `enableEmoji = true` key-value must be placed in the front matter.

This website also features emoji support :see_no_evil:. The reference for supported emojis can be found [here](https://www.webfx.com/tools/emoji-cheat-sheet/).

## Mathjax for LaTeX Support
>**Note:** To enable $\LaTeX$ support, the `ismath = true` key-value must be placed in the front matter.

This website supports $\LaTeX$ using MathJax. This feature uses the `ismath` key-value in the front matter in order to load the MathJax JavaScript scripts on-demand (thus reducing loading times when it is not needed). 

Below are examples of what's possible to embed.

### Inline LaTeX
Inline equations only need one `$` at each side. For instance, this integral `$\int x^3 dx$` will be transformed into this $\int x^3 dx$.

### Block LaTeX
Block and multi-line LaTeX code must be written with `$$` at the top and bottom. For instance, this:
```text
$$
\int_{a}^{b} x^2 dx
$$
```

will be transformed into this:
$$
\int_{a}^{b} x^2 dx
$$

Matrices can use `\\` at the top and bottom. For instance, this code:
```text
\\begin{array}{cc}
  a & b \\\\
  c & c
\\end{array}
```
will yield this result:
\\begin{array}{cc}
  a & b \\\\
  c & c
\\end{array}

## Bokeh Support
>**Note:** To enable Bokeh support, the `isbokeh = true` key-value must be placed in the front matter.

This website also supports [Bokeh](https://docs.bokeh.org/en/latest/index.html) plot embedding. This feature uses the `isbokeh` key-value in the front matter in order to load the Bokeh JavaScript scripts on-demand (thus reducing loading times when it is not needed).

The following is an [example](https://docs.bokeh.org/en/latest/docs/gallery/hexbin.html) from Bokeh's gallery that has been slightly modified.

{{< bokeh "bokehs/test.json" >}}