---
title: "Using Cpp17 or higher version of Cpp on Vscode extension Clangd"
date: 2022-08-07T21:49:29+08:00
tags: [clangd, VSCode, Cpp]
---
Recently I've switched my code completion tool in VScode from Microsoft's official plugin to clangd. On the one hand, clangd can provide a better user experience, on the other hand, I also use this plugin in my company (Bytedance).

I joked to myself: Two of the most interesting things I learned when I interned at the company were that indentation changed from four spaces to two, and braces changed from wrap to no wrap.

Now, there is a problem. I installed clangd, switched over and everything worked fine. But when I use string_view, clangd told me it didn't recognize this thing. I quickly realized that this is due to the fact that the default Cpp version of clangd is still 14. So changing its default version became something I had to do.

I googled but didn't find any useful information.

I try to use compile_commands.json file. But on the one hand, it needs to be configured separately for each project, while the effect I expect is to use Cpp17 for any file that is opened at random. On the other hand, it turns out that it does not work. (maybe I didn't configure it properly)

So, here is the final solution.

Open (create if not already there) `~/.config/clangd/config.yaml`
and write the following information

```yaml
CompileFlags: 
    Add: [-std=c++20]
```

Cool. Now let's have fun with Cpp20.

