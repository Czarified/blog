+++
date = '2026-07-23T16:37:00-04:00'
draft = true
title = 'Tailbliss Struggles'
+++


This is quickly exploding in complexity beyond what I'm willing to deal with... I finally got the GitHub Actions working with Tailbliss,
and I can change all the text on the home page and add this new blog post. However, I cannot figure out how to post images on the home page, or tweak
some of the more complex layout elements.

So, let's see if we can even include images in the a blog post...

![Alt text](/images/fbd.png)

Okay so that's working... I put a leading slash on the relative URL. My image is saved in the root `static/images` directory. So the relative URL that worked was `![Alt text](/images/fbd.png)`

This doesn't seem to have fixed the home page, though. There's something else going on there with Hugo Pipes.