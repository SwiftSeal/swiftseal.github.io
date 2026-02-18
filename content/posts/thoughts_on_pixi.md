+++
date = '2026-02-16'
title = 'Thoughts on Pixi for bioinformatics'
+++

I've been using the [Pixi package manager](https://pixi.prefix.dev/latest/) for about a year now and thought I'd put some of my thoughts down on its current status and applicability to bioinformatics.
For those not familiar with Pixi, it is a package manager which allows users to rapidly install software into dedicated environments without fear of manual installations and frustrating dependency incompatibilities.
It is somewhat of a spiritual successor to Conda (many of the developers of Pixi came from the conda-forge project), but does have some differences in how it is used which have an impact on how it can be used for bioinformatics.

Here are the five ways in which I tend to handle dependencies, and how Pixi can help or hinder.

## Mono-environment

I use a single environment for the whole project which contains all dependencies.
I mostly do this when developing a tool myself, rather than for projects such as genome assembly where dozens of potentially clashing dependencies are required.
Pixi is great for this!
Dependency resolving is super fast and stable, and it is superior to conda in basically every way.
I've also used this approach for building software that isn't available on conda and is in a state of "dependency hell"
Here, the performance is critical in allowing me to quickly iterate through different dependencies and find a working solution - doing this in conda can genuinely take hours longer due to slower resolves and installs.

[They describe this in more detail in their docs](https://pixi.prefix.dev/latest/switching_from/conda/#environment-vs-workspace), where they describe Pixi as more of a "workspace" manager than an "environment" manager like conda.

## Multiple environments

As mentioned, for projects involving large sets of incompatible software, the go-to method tends to be to create dedicated Conda environments for each tool.
I'd say this is the most common approach I see in bioinformatics, probably because it's so easy to go `conda create -n blah blahblahblah` and have a saved copy of whatever tool you need.

Side note, but I also think this causes problems, as some bioinformaticians tend to lean too heavily on these environments, then forget which version they actually installed, or struggle to rebuild them if they get corrupted, or struggle to teach other users how to do XYZ because they have a 5TB rats nest of 1600 environments (`genome_assembler_env`, `genome_assembler_env_new`, `genome_assembler_env_new_better` etc) that they have collected over the past 4 years...

Pixi really struggles to replicate this in my opinion.
They do have documentation on "multiple environments", but this is largely about creating extensions of the workspace, for example creating environments for GPU vs CPU dependencies, or perhaps holding some optional development tools which aren't always required in the environment.
I'm sure you could hack this into some sort of equivalent to conda environments, but this will be a pain point for bioinformaticians transitioning from conda.

That being said, perhaps this barrier actually helps us in the long term?
For example, if I created a Conda environment for a tricky tool like the transposable element editor [HiTE](https://github.com/CSU-KangHu/HiTE), my process might be follow their instructions for creating the conda environment (which includes `pip` dependencies as of writing), perhaps install some extra stuff like samtools or whatnot, patch a few bugs in the dependencies, then forget what I did and just rely on that environment.

With pixi, I might instead clone their repository, run `pixi init` to get an empty environment, then build the environment in-place, essentially tying the code to the environment.
I can then share the respective pixi files onto my own fork on github, which other users can enjoy, or perhaps the developers can adopt.
Then, when I need to run `HiTE`, I can reuse that environment, with the caveat of having to execute `HiTE.py` from the base directory containing the pixi environment.


## Isolated environments

My go-to method these days is to create an isolated environment for every program.
For this, I initially switched to using singularity/apptainer/docker containers, for example:

```
singularity exec docker://user/sometool:v1.0 sometool ...
```

This will fetch the container, cache it, then run whatever code I specify in that environment.
The advantage of this approach is that the container, and as a result exact version of software, is saved *inside* the script so there is no uncertainty in what was run.
It is also much easier to share with other users, as if they run my script, the dependency will be installed and run just the same (assuming their singularity is set-up in the same way).

With pixi, this is even easier:

```
pixi exec -s "sometool==1.0" sometool ... 
```

It also has the edge on situations where multiple dependencies are required.
For example, let's say I want to align reads with `minimap2` and create a sorted BAM file with `samtools`.
With containers, I could do:

```
singularity exec docker://quay.io/biocontainers/minimap2:2.1.1--0 minimap2 $GENOME $READS > $SAM
singularity exec docker://quay.io/biocontainers/samtools:1.23--h96c455f_0 samtools sort -o $BAM $SAM
```

Ugly!
And annoying that I can't use it in a single pipe and need to use that intermediate SAM file.
I can improve this by using a container with both dependencies (Seqera's [wave](https://seqera.io/wave/) can help here), but it's still clunky.

With pixi, I can just do this:

```
pixi exec -s minimap2==2.1.1 -s samtools==1.23 minimap2 $GENOME $READS | samtools sort -o $BAM
```

Much nicer!
Pixi can also create shells with dependencies inside, which might be easier, or for SLURM you can use my favourite incantation where you submit a job that creates an on-the-fly environment!

```
#!/bin/env -S pixi exec -s minimap2==2.1.1 -s samtools==1.23 -- bash

#SBATCH -p long
#SBATCH -c 32
#SBATCH --mem 64gb
#SBATCH --export ALL

minimap2 $GENOME $READS | samtools sort -o $BAM
```

An independent SLURM job script that automatically creates the environment it runs in!
What a time to be alive.

## Global tools

I have some tools that I wouldn't really classify as part of a project, but are nice to have.
For example, I use [dust](https://github.com/bootandy/dust) a lot to quickly check project folder sizes.
You can install global tools in pixi with `pixi global install`.
Not much to say on this, it's nice to have!
Analogous to the base conda environment.

## Workflow managers

Pixi isn't supported in Nextflow yet, so I can't comment on this much, but it does look like they're progressing to make this a core feature.
It makes sense, as it won't have an impact on end-users beyond increased performance, and perhaps greater stability of environments?
It'll be interesting to see how this is implemented in the end!
I've seen some hacked together methods to get it working with Nextflow already, but they do look a bit brittle.

## Concluding

It'll be interesting to see how widely adopted Pixi becomes by the bioinformatics community.
I hope it remains in development for the long future, as it is genuinely a great piece of software, but it does run the standard risk of being developed by a small team with financial pressures!
