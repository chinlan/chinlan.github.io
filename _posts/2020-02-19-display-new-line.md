---
layout: post
title: "Display new line in strings"
description: ""
categories: [css]
tags: [react]
redirect_from:
  - /2020/02/19/
---


2 ways to display new line `\n` in strings:

1. Add this to the element which contains the target strings:
~~~css
.target-strings-class {
  white-space: pre-wrap;
}
~~~

2.
~~~react
string.split('\n').map((text, index) => (
  <p key={`${text}-${index}`}{text}<br /></p>
));
~~~

[reference](freecodecamp.org/forum/t/newline-in-react-string-solved/68484/12)
