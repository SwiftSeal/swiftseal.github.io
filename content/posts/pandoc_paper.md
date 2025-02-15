+++
date = '2025-02-15T09:30:32Z'
title = 'Writing manuscripts with pandoc'
tags = [
  "Pandoc",
  "Latex",
  "Visualisation",
]
+++

If you're like me, you are probably not a fan of writing in microsoft word.
There are many drawbacks to using a [WYSIWYG](https://en.wikipedia.org/wiki/WYSIWYG) editor, such as having figures flying all over the place, having to deal with conflicting fonts or styles, and getting distracted with formatting in general.
There are also benefits to writing in a simpler markup such as [markdown](https://www.markdownguide.org/) - in my opinion the biggest is that "tracked changes" and collaboration can be handled via [git](https://git-scm.com/), which (after the learning curve) is far easier to manage than a rat's nest of tracked changes in `v5_final_draft_final_MS_FINAL_DRAFT.docx`.

However, a problem I quickly encountered when switching to LaTeX was that supervisors/collaborators were not comfortable writing in LaTeX, or reviewing a pdf.
They wanted a `.docx` for the tracked changes.
I also [don't like writing in LaTex]({{ ref "/posts/cv_style.md" }}), as I find its syntax to be pretty dated and hard to manage! 

The solution I found was to use [`pandoc`](https://pandoc.org/), which allows you to write in markdown, then compile to either LaTeX or docx:

```
# Create a pdf via latex
pandoc main.md -o main.pdf

# and a word document!
pandoc main.md -o main.docx
```

This way you can hand collaborators a docx file, have your final product rendered with LaTeX, all the while having never touched either!

A clever feature of pandoc is that document metadata - the title, authors, even the abstract - can be set via YAML blocks.
These be specified via a separate `.yaml` file, or directly in the `.md` document:

```
---
title: My paper
author: Moray Smith
abstract: >
  Blah blah blah.
  Here is my abstract.
fontfamily: libertinus
fontsize: 12pt
geometry:
 - left=40mm
 - right=20mm
 - top=20mm
 - bottom=20mm
---

And here's the rest of my paper.

## Results

Blah.
```

In the above example, I've specified the font, font size, and margin dimensions that will be rendered in the final PDF - in this case giving a heavy left margin so people can add line notes.
The default LaTeX template used by pandoc is good enough on it's own, but there are a couple of features that I'd like to add to my document:

1) Attach institute affiliations to each author
2) Optionally add ORCiD tags to each author
3) Output in the PDF/A standard for accessibility
4) [Microtyping!](https://ctan.org/pkg/microtype)
5) Have line numbering!

How to do this?
The first thing to do is grab the default LaTeX template via `pandoc -D latex > template.tex`.
I then added the following packages:

```
\usepackage{pdfx} % adds PDF/A support
\usepackage{lineno} % gives me line numbering
\usepackage{microtype} % for better typesetting

\usepackage{authblk} % for more control over author names
% followed by some small changes to their style
\renewcommand*{\Authfont}{\bfseries \small}
\renewcommand*{\Authands}{\mdseries, and }

\usepackage{orcidlink} % support ORCiD tags, neat package!
```

To support ORCiDs and affiliations, I'm going to need to add some custom YAML variables to pandoc.
The ideal format is something like:

```
---
title: My paper
author:
 - name: Moray Smith
   orcid: 0000-0001-9363-3170
   affiliation: 
    - 1
    - 2
institution:
 - id: 1
   name: James Hutton Institute, Dundee, DD2 5DA, UK
 - id: 2
   name: University of St Andrews, St Andrews, KY16 9AJ, UK
abstract: >
  ...
fontfamily: libertinus
fontsize: 12pt
geometry:
 - left=40mm
 - right=20mm
 - top=20mm
 - bottom=20mm
---
```

To support these, all I need to add to the template is:

```
$for(author)$
\author[$for(author.affiliation)$$author.affiliation$$sep$, $endfor$]{$author.name$ $if(author.orcid)$\orcidlink{$author.orcid$}$endif$}
$endfor$

$for(institution)$
\affil[$institution.id$]{\footnotesize $institution.name$}
$endfor$
```

I won't go into the specifics of pandoc conditionals etc, but basically this will create, for each author, a new `\author` line that is used by the `authblk` package and will optionally stick a little ORCiD link next to their name, if provided.
Then, each institute is built again via `authblk`.
These are rended when `\maketitle` is called.

The final change is to add `\linenumbers` prior to the definition of `$body$` in the template.

## Building via Github actions

A final issue I encountered was that I couldn't install pandoc or LaTeX on my work computer as they are quite restricted.
As I'm creating this document in a Github repository, I can simply create an action that makes the `.tex`, `.pdf`, and `.docx` for me!

```
name: Build
on: push
jobs:
  pandoc:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
      - name: create output folder
        run: |
          mkdir output

      - uses: docker://ghcr.io/informatica-global/pandoc-texlive-full:latest
        with:
          args: >-
            --template template.tex
            --bibliography=citations.bib
            --wrap=none
            -F pandoc-crossref
            --citeproc
            -o output/main.tex
            main.md

      - uses: docker://ghcr.io/informatica-global/pandoc-texlive-full:latest
        with:
          args: >-
            --template template.tex
            --bibliography=citations.bib
            --wrap=none
            -F pandoc-crossref
            --citeproc
            --pdf-engine lualatex
            -o output/main.pdf
            main.md

      - uses: docker://ghcr.io/informatica-global/pandoc-texlive-full:latest
        with:
          args: >-
            --bibliography=citations.bib
            --wrap=none
            -F pandoc-crossref
            --citeproc
            -o output/main.docx
            main.md
      
      - uses: actions/upload-artifact@v4
        with:
          name: output
          path: output
```

Then just stick this under `.github/workflows/build.yml`.
Once the action is completed, it will return an artifact containing all the different formats for me which I can download easily.
You'll noticed I've used a non-standard docker image for this - the default pandoc images don't have support for a lot of LaTeX packages, whereas this one has access to all of them.

Here's the final product:

![Example of document](/static/images/example_paper.png)

Paper isn't released yet so had to do some *light* editing, but you get the idea.
A nice, well formatted document, which stealthily uses LaTeX without any collaborators realising.
The `.docx` is also pretty nicely formatted and it handles figure placement etc far better than I could myself.
