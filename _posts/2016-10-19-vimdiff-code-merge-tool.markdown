---
layout: post
title: "Vimdiff - code merge tool"
date: 2016-10-19 21:47:36 +0530
comments: true
categories: 
---
Just thought to post about vimdiff which can be used as a code merging tool. vimdiff package comes by default in the bundle of vim editor.

vim is favourite part of any C/Unix developer. Though I worked in many GUI text editor ( EditPlus, Notepad++, gedit etc.. ) I still like to write coding stuff using vim editor. Here is some tips for using vimdiff as merging tool.

```
do   -   diffget from other window
dp    -   diffput to other window
]c    -   go to the next change
[c    -   go the previous change
zo    -   open the folded lines
zc    -   close the folded lines
ctrl+w, w  -  change active working window
:only | wq -      quit all other windows, write and quite current window
```
