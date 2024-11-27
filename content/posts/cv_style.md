+++
date = '2024-11-27T09:40:07Z'
title = 'Making an easy CV'
+++

Designing your CV is a great way to procrastinate from doing any actual useful work, so of course I spent most of yesterday doing just that, rather than finishing my thesis corrections.

I'm not a fan of word or microsoft, so previously I've used latex to make my CV which has been fine, but there have always been a few issues with it:

1) I am overall not a fan of the latex langauge - it's fairly verbose and normally involves importing a dozen packages from CTAN, and mining stackoverflow for fixes to those annoying quirks that latex has.
2) As a result of those challenges, I've previously leaned into using existing templated to generate my CV. My problem with existing templates is that they often overcomplicate things with multilanguage support, hundreds of functions to define content, or really stupid CV formats (why is there a picture of your face; why do you have a side bar?; why do you have a barchart for your 'skills'?).
3) Given that it's an academic CV, I want to include my publications, which you'd expect `.bib` files to be the answer to, but again needs a fair amount of sleuthing to figure out how to implement this.
4) As a result of both of the above, when the infrequent time comes to re-do your CV, you've probably forgetten what anything does in the code.

This time, I decided to use [typst](https://typst.app) to generate my CV.
It's a pretty interesting project which I hope continues to grow in popularity, but I've never really had the chance to use it apart from testing it's capacity to build my thesis via pandoc (had to fallback to latex for this).
Like latex, they have a [template library](https://typst.app/universe/package/modern-cv/) but a quick scan of this reveals multifile projects heaving with custom functions - many broken with the latest `typst` release - and again just that really overkill formatting which these templates go for.

First, here's the [final product](https://github.com/SwiftSeal/cv/blob/master/main.pdf).

To begin my CV in `typst`, I began by defining some basic page features:

```
// This sets useful metadata for the document
#set document(
  title: "Moray Smith CV",
  author: "Moray Smith",
)

// General page formatting
#set page(
  margin: 30mm,
  footer: context [
    #datetime.today().display()
    #h(1fr)
    #counter(page).display("1/1", both: true)
    #h(1fr)
    Moray Smith
  ]
)
```

The `document()` block just defines metadata for readers and the like, whilst the `page()` block mostly keeps things simple with a footer that displays the date of compilation, the page count, and my name.

When adding my name and some useful links, I afforded myself the *luxury* of some simple `.svg` icons for each link.
To do this:

```
#import "@preview/scienceicons:0.0.6": github-icon, orcid-icon, linkedin-icon

#set align(center)

= Moray Smith

#github-icon(baseline: 20%, height: 1em)
#link("https://github.com/swiftseal")[SwiftSeal]
#h(.25cm)
#orcid-icon(baseline: 20%, height: 1em)
#link("https://orcid.org/0000-0001-9363-3170")[0000-0001-9363-3170]
#h(.25cm)
#linkedin-icon(baseline: 15%, height: 1em)
#link("https://www.linkedin.com/in/moray-smith/")[moray-smith]

#set align(left)
```

The `science-icons` package is relatively simple to work with, although you can see I had to do some manual tweaking to get the icons to align with the text properly.
I would like to change the ORCiD and LinkedIn icons to their respective colours, but the method to do this seems to be broken - ah well.

Another luxury - a horizontal line to separate the header, with a gradient!

```
#rect(
  width: 100%,
  height: .1cm,
  fill: gradient.linear(
    ..color.map.magma,
  ),
)
```

For producing entries in the document, I avoided using functions as although I can appreciate they help reduce some redundancy, I do find it just overcomplicates things for such a simple document.
Let's take an entry for my education as an example:

```
*University of St Andrews*, _PhD_ #h(1fr) 2020 - 2024
 - Thesis: "Applications of next-generation sequencing towards resistance gene identification"
 - EASTBIO Doctoral Training Programme
```

I *could* define a function for that, but it's fairly readable on its own.

Rest of the document is fairly straightforward, except for the bibliography which can be defined as followed:

```
#bibliography("references.bib", title: none, full: true, style: "apa")
```

The only thing I'd like to change with this is potentially putting my name in bold for each entry, and also sorting them by year, but it's more than enough for now!