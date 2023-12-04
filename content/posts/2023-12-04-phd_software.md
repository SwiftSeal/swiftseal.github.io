---
title: 'Software I found useful during my PhD'
date: 2023-12-01
tags:
  - PhD
  - Software
---

A quick post to highlight some of the most useful software I've encountered during my PhD 😊.

## Termius

Most of my work is done on a shared computer cluster, so having a decent SSH client is a priority.
I've used a couple of different solutions - MobaXterm, PuTTY, and also just ssh'ing directly.
Since trying Termius I've stuck with it for a couple of reasons:

* Allows cross-platform SSH, even on a phone. Useful if hopping between multiple devices.
* Easy to manage multiple hosts.
* In-built file transfer system is pretty straightforward.
* Free for students!
* It's pretty.

I never really use it's other features like code snippets or sharing hosts.
I could see the team aspect being useful if everyone in a lab used it, but never tried it personally.
I did try to use it's terminal sharing feature once to help an undergraduate student I was supervising, but it didn't work 🤷‍♀️.

## VS Code

I don't like microsoft, but VS Code is a game changer for writing code.
With GitHub Student (free!) you also get access to GitHub copilot which I struggle to do without now.
I use it for everything - python, rust, R, markdown, this blog...

## Mamba

I made the switch to the faster `mamba` from `conda` recently and it's saved me a lot of hassle.
In general though, definitely a good idea to get comfortable with `conda` - at the start of your PhD get into the habit of making a folder of any `conda` environment `.yaml` recipes that you use.
It saves you from trying to recreate them if they get deleted or lost!

## Zotero

Zotero is the best citation manager - don't use anything else.
It can export `.bib` files which was essential for me!

## Inkscape

I use Inkscape to make all my figures.
I find it particularly useful for tidying up `ggplot` pdfs and making subfigures.
To be honest if I get more funding I'd switch to adobe - I do like that it's open source but it is pretty buggy and also not very good at formatting text, so posters are tricky.
That being said, with some learning it is much better than using powerpoint which so many people still seem to use...
Just get used to saving your work every five seconds.

## Pandoc

Not going to turn this into another Pandoc post, but if you hate formatting text and are sick of Word breaking all the time, give it a shot!

## Various Rust CLI tools

* `btm` - Useful for monitoring SLURM jobs
* `ripgrep` - Faster than `grep`
* `dust` - Easy way to spot big files that are using up your disk space
