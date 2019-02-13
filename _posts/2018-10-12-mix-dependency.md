---
layout: post
title: "Mix: dependency"
description: ""
categories: [phoenix]
tags: [elixir mix]
redirect_from:
  - /2018/10/12/
---
[ref](https://groups.google.com/forum/#!topic/elixir-lang-talk/jadW1ybxfWw)

When removing gem from Rails gemfile, all we need to do is delete the line, and bundle again. The removed gem will be removed from gemfile.lock after bundle.

It is a little different when it comes to Mix,
it will not remove the dependency from `.lock` automatically if just remove a dependency from `mix.exs` and ran `mix deps.get` .

To also remove the unwanted dependency from `.lock` file, we will have to run `mix deps.unlock dep1 dep2` or `mix deps.unlock --unused` .
