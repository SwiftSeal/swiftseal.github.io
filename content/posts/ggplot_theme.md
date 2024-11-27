+++
date = '2024-11-27T09:30:32Z'
title = 'A simple ggplot2 style'
+++

Upon first discovering [ggplot2](https://ggplot2.tidyverse.org), peoples' first reaction is normally "Oh my goodness why is this so much better than everything else?".
Their second reaction is normally "Oh my goodness why is the default theme so ugly?".
What follows tends to be a bit of a frenzy of learning how `theme()` works, customising it to the Nth degree, making a custom colour palette, importing a thousand different libraries, maybe considering a career change to graphic design...

I did all that (bar the career change) and essentially reached ggplot2 nirvana where I strongly considered going back to the default theme because "what does it all matter anyway?".
Anyway I settled on a happy middle ground:

```
library(ggplot2)
library(ggthemes)

ggplot(iris) +
  geom_point(aes(x = Sepal.Length, y = Sepal.Width, colour = Species)) +
  theme_bw(base_size = 8) + theme(panel.grid = element_blank()) +
  scale_colour_tableau()
```

Wow so exciting.
I like nice colours and have always been interested in accessible palettes, and there is a [great post](https://www.tableau.com/blog/colors-upgrade-tableau-10-56782) by the Tableu team about the rationale behind their colour scheme, so might as well just borrow that.
The palette comes with ten colours - sometimes I find myself wishing more were available, but I've come to use this instead as an opportunity to ask myself whether or not the plot is beginning to become too crowded, and to seek alternative ways to visualise the data.

I often like to make figures that include a couple of subfigures, and for that I use ggpubr and its `ggarrange()` function.
It's sufficient for simple grids and I like the inbuilt labelling system.
Anything more complicated than that and I'd opt for illustrator, but again I think there is a lot to be said for keeping things simple - particularly for a thesis!

Something I've also learnt is that it is probably a good idea to keep the output dimensions consistent across plots.
I like to use a width of 5.9 inches, which corresponds to the width of an A5 page with three centimetre margins.
This way, all text should be readable and in proportion with the plot, and the figure should be easily inserted into a document.
Once again, if at this width the plot is crowded and bugging out - make it simpler!

And that's it!
I've read dozens of "how to make sick plots in ggplot2/matplotlib/seaborn/whatever" posts before and this is the happiest I've been with plotting.
Ultimately there are only so many hours in the day and who actually cares what your plots look like?
Spend that extra time doing something more useful!