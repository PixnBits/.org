---
title: Jekyll in Docker on a Pixelbook
date: 2019-01-26 08:05:00 -0700
summary: Linux in ChromeOS running containers FTW
---

I've switched out the platter HDD for a solid state on my laptop and replaced Windows with Ubuntu, but there's only so much physical customization you can do with laptops. While it still runs decently five years in, it is pretty thick which limits portability. At React Rally 2018 someone showed my their Chromebook and how the OS is adding experimental Linux machine support. That was exciting because macOS, while still closer to \*nix than Windows, has some odd behaviors like case-insensitive filesystem paths. Ubuntu is great, but as I haven't invested the time into writing drivers myself I have to hope someone else has done so for my specific machine. My trackpad is really frustrating, even after tuning.

A few months ago there was a sale on Pixelbooks. Perfect time to try out a Chromebook. _Sooo_ portable; no comparison, even to MacBook Pros.

At the time I had to switch to the development channel to run Linux. There are some oddities with that, but nothing _too_ bad. Atom installed easily, exciting! Then for docker...and no dice. The dev channel has weekly updates, but every time I tried to run busybox there was some issue.

Until this week. Docker now runs!!

I switched this site to Jekyll a while ago only because that's how GitHub pages work, but having missed the height of the Ruby era it's always been hard to get working correctly locally (at least, without breaking something else). I've been looking forward to running a docker container to develop this site for a while. I've added a `serve.sh` that I can run:
```
#!/bin/bash
JEKYLL_VERSION=stable
docker run --rm --volume="$PWD:/srv/jekyll" -p 3000:4000 -it jekyll/jekyll:$JEKYLL_VERSION jekyll serve --watch --drafts
```

Now maybe I can document various findings a bit better (low bar) and figure out if Jekyll can run node tools for building sites 🤞.
