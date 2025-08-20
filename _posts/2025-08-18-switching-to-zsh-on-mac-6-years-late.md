---
layout: post
title: Switching to zsh on Mac, 6 years late
date: 2025-08-21 09:00 +0200
published: true
image: /assets/img/2025-08-21-bash-zsh-switch-2.png
---

For six years, I've seen this every time I open a new terminal:

```
The default interactive shell is now zsh.
To update your account to use zsh, please run `chsh -s /bin/zsh`.
For more details, please visit https://support.apple.com/kb/HT208050.
```


I finally changed it last week. I started writing this earlier, but children, family, vacations and homemade pizza got in the way — giving a strong contribution to my happiness.

I was never a fan of `zsh` for two reasons:

**1.** The way it treats wildcards. `bash` is smart enough to determine when input should be passed to the command instead of being interpreted as wildcards for the files in the current directory.

For example:

```bash
grep Salary.*named *
```

This behaves very differently in `bash` and `zsh`. `zsh` will respond with:

```
zsh: no matches found: Salary.*named
```

assuming there are no matching file names.

`bash` will go ahead and run `grep` the way I intended.

**2.** Licensing (perhaps — not sure about this point). I have the impression that several organisations are switching to `zsh` mainly because of `bash`'s licensing change.
With version 4, `bash` switched to GPLv3, which is less favourable for a proprietary company like Apple. There are several 
reasons to criticise GPLv3, and even Linus Torvalds recognises that <i class="fa-brands fa-youtube fa-sm" style="color: gray;"></i> [the GPLv3 is less free than GPLv2](https://www.youtube.com/watch?v=PaKIZ7gJlRU)
as it imposes new restrictions. And logically, with more restrictions come less freedom. I usually tend to agree with Linus Torvalds more than with
Richard Stallman (whom I met when he gave a talk at my university — maybe I should talk about that one day). And to be fair, Apple did contribute
to both `bash` and `ksh` in the spirit mentioned by Torvalds in the same talk "I give you source code, you give me changes back, we're even".

However, it doesn't seem fair that after serving reliably for decades, `bash` is sidelined not for technological reasons but for licensing politics, should that really be the case here.

So, despite the wildcard matching (the licensing thing is more of a rant and has no actual impact on my daily life) 
why am I switching to `zsh` after six years? One could still install an updated version of the _Bourne Again Shell_ with `brew`

Because of muscle memory. I have already been using `zsh` for probably more than a decade at work, and it just feels natural to have the same setup on both my work and personal laptops so that I don't need to remember the little differences between the two.

Thank you `bash`, you have served me well. From 1999 to 2027.

### Note to self should I decide to use `bash` again.

To use the version distributed with `brew` just:

```bash
chsh -s /usr/local/bin/bash
```

And remember that [spaceship](https://starship.rs/) works with both.
