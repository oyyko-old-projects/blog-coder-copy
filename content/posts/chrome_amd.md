---
title: In archlinux, chrome cannot be displayed normally under AMD.
date: 2023-06-07
tags: [archlinux, bug]
---
Solution:
```
rm -rf .config/google-chrome/Default/GPUCache
```