---
sidebar_position: 2
---

# Hosting docusaurus with domain ZN2D.com

    The Fact : I have docusaurus running in a Docker container. I would like to do the same thing with Apache and host my own site.

    The problem : How to get both containers to talk to each other and how to update everything locally and remotely (Through a git repo?)

    Hypothese : There must be a way to get them to communicate

After searching for a while to find a way to expose or better yet, to get Docker containers to communicate between each other and for which, I didn't find a easy way to do this, I decided to change my approach.

Since I might be hosting different websites, the Docker approach seemed a little bit complicated...ish. So instead, I installed apache directly and now I'm hosting the built version on it.

