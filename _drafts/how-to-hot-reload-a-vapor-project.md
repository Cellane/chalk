---
title: "How to hot reload a Vapor project"
description: "One question that gets frequently asked in Vapor’s Slack channel
    is whether is it possible to not have to recompile the entire project upon
    the slightest change in the source code. Until recently, I would have told
    you that this is just the unavoidable reality of compiled languages, but
    today I have a solution – sort of."
tags: [tidbit, vapor, web]
---

We all know that Swift Package Manager... well, while it works most of the time,
it could really use some improvements: it could do more with the `Package.swift`
than just to read it (what about adding new dependencies on-demand?), it could
be more developer-friendly (`swift package generate-xcodeproj`, really? I still
can't believe that's an actual command name :scream:) and in general, it
could... just be a little bit more like [Ice](https://github.com/jakeheis/Ice),
actually.

Ice is a much more user-friendly interface to Swift Package Manager while
retaining a full compatibility with SPM, meaning you can mix and match SPM
commands with Ice commands and your project should safely survive. Its output
is clean and colorful[^1], it can add new dependencies to your `Package.swift`
file as well as manage targets and projects. It even has its own small registry
for all sorts of packages you can add to your project[^2]!

[^1]: Actually, the output should be colorful but for some reason, it's not in
    my terminal, as I just realized. I wonder if that's an issue of Ice or of my
    shell configuration :no_mouth:

[^2]: I wonder if that could be inter-connected with the
    [Awesome Vapor](https://github.com/Cellane/awesome-vapor) repository I'm
    maintaining.

But why am I talking about some sort of SPM interface when I promised to be
talking about hot reload? As it turns out, one of the features that SPM doesn't
have and Ice adds to the mix is watching the filesystem for changes and
rebuilding your project anytime it detects one. You can do so by issuing the
`ice run -w` command, where the `-w` stands for "watch".

I've been playing with this feature on and off for the past few days and...
while it's useful, it's definitely not perfect. Rebuilding a nearly empty
project after a tiny change takes about ~25 seconds on my machine, which can
lead to situations in which you expect the API to already include your latest
changes when in fact, Swift is still building your project and preparing for
relaunching your app.

So while it may not be a "hot reload" per se, it's at the very least a "lukewarm
reload" :trollface:
