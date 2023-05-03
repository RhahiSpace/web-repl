---
title: "automated testing"
date: 2023-05-04T00:22:23+02:00
mathjax: false
draft: false
series: ""
tags: []
---

Programming a large program in Julia is not fun, because it is very slow to get feedback from new revision. This can be sped up by using Revise.jl or Jupyter Notebook, but it discouraged me from writing and revising tests as my unstable code changed over time.

To solve this problem, I would like to add a server dedicated to [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration). Let's call this machine `builder`. Builder will be hosting:

- the source code and the remote development environment.
- a copy of Kerbal Space Program.
- required mods (kRPC and kOS).

## Requirement

What other requirements should it have?

1. Builder must run [headless](https://en.wikipedia.org/wiki/Headless_computer).
1. Reduce power consumption. Builder should shutdown itself if no remote is connected, and no job is running.
1. Builder should not lose connection to GitHub. That is, it should have scheduled reboot to check for jobs.
1. Enforce conventional commit and use automatic versioning.
1. Make it secure.

## Security

It is not recommended to use [self hosted runners for public repositories](https://github.com/orgs/community/discussions/26722). However, my project is an open-source project. What can I do?

One way to solve this problem is by having the CI mirror the development repository. I work on my private repo, and then when a job succeeds, it publishes the resulting code into a public facing mirror.

## Next steps

- Read [GitHub documentation](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
- Learn how to get Kerbal Space Program, possibly by installing Steam in Builder.
- Setup automatic shutdown and startup.
