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

By supporting Bokeh, this website also supports [Holoviews](https://holoviews.org/index.html) plots using a Bokeh backend:

{{< bokeh "bokehs/holotest.json" >}}

Note, though, that Holoviews is a tool to simplify plotting with several backends — a high-level API to make plotting easier; similar to what Seaborn is to Matplotlib. When a plot is made with Holoviews, it needs to be rendered as a Bokeh object to then be dumped as a JSON file. 


## Social Icons
To allow the user present its social networking links, this website features social icons that have been wrapped up in a partial and a shortcode, such that the icons can be easily modified (e.g. add a new icon for a new social service) and the set of icons can be easily added anywhere in this site by calling the shortcode:

{{< socialicons >}}


## Raw HTML
In order to add raw HTML code into a Markdown file, this website features a shortcode that does so. While this feature might not be something frequently needed — part of the objective of using Markdown with Hugo is to avoid writing HTMLs altogether —, there can be cases in which granular customization or embedding of specific objects might be needed. For instance, an embedded Google map would look like this:

{{< rawhtml >}}
<p style="text-align:center">
<iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d5774.538501083268!2d-79.38924548498251!3d43.642566179121665!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x882b34d68bf33a9b%3A0x15edd8c4de1c7581!2sCN%20Tower!5e0!3m2!1sen!2sca!4v1601877315775!5m2!1sen!2sca" width="600" height="450" frameborder="0" style="border:0;" allowfullscreen="" aria-hidden="false" tabindex="0"></iframe>
</p>
{{< /rawhtml >}}

And that's a wrap! This post will be updated if new features are deployed.

