+++
content = ["test"]
title = "Test & Template"
author = "Raul Morales Delgado"
date = "2020-09-01"
description = "This is just a test and template."
toc = false
tags = ["template", "markdown", "latex"]
ismath = true
+++

## Test
The sole purpose of this post is to serve as a test for deploying new capabilities and to be a template holder.

## Template
### Frontmatter
The following is a frontmatter template for all documents. Values for taxonomies depend on key.
```toml
+++
content = ["test", "tutorials", "projects", "nuggets"]  # This is closed to these values
title = "Title for Document"
author = "Raul Morales Delgado"
date = "2000-01-01"
lastmod = ""  # Not necessary since git and file metadata is enabled
description = "Description for the document. Intro-like text"
toc = true
tags = ["a", "b", "c"]  # This is open
ismath = true  # Only necessary if document contains latex-like formulae.
+++
```

### Emojis
The objective of this test is to test a lot of things. :smile:

### Mathjax for LaTeX
The following section contains equations for general reference.

Block equations must be written with `$$` at the beginning and end. This is a sample output:
$$\int_{a}^{b} x^2 dx$$

Inline equations only need one `$` at each side.

For instance, integrating inline $\int x^3 dx$ is so cool.

Matrices can use `\\` at the beginning and end:
\\begin{array}{cc}
  a & b \\\\
  c & c
\\end{array}

### Plotly
*Coming soon.*