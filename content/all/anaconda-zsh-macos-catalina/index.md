+++
categories = ["nuggets", ]
author = "Raul Morales Delgado"
title = "Switching to zsh & Re-Enabling Anaconda in the New MacOS Catalina"
description = "This post shows how to *re-enable* command-line applications, namely the default ones in MacOS distributions, like `git`, and user-installed Anaconda, that would not launch or work after installing MacOS Catalina and its new default shell, `zsh`."
date = "2019-11-15"
toc = true
tags = ["devops", "conda", "macos", "shell", ]
+++


In this post, I will show how to set `zsh` as the new default shell — although this is set by Apple, it is necessary to explicitly change it on the Terminal —, how to install the latest version of Xcode Command Line Tools, and how to re-initialize Anaconda for `zsh`. The latter part will only be useful for users who installed Anaconda in their home folder; this will not be useful if you installed it on root.


## 1. Introduction

Like many other Apple users, eager to try out the new MacOS stable release, *Catalina*, I ushered myself to download it and try out some of its new eye-candy features. While I had already read somewhere before that this new distribution would come with `zsh` as the new default shell — replacing good ol' `bash` —, I totally forgot about it until I had to use `git` and `conda` again. Long story short, after installing Catalina you will not be able to run `conda`, `git`, and other `bash`-related things, but there are some fixes — to some extent — that I summarize in this post.

MacOS Catalina makes `zsh` the default shell from now on, and if you are new to `zsh`, worry not because it is not that much different from `bash` but it is full of *new* functionalities (*new*, but note that it was originally released back in 1990 [^1] that make it a more powerful shell. However, all manual modifications that you did on `.bash_profile` (equivalent to `.bashrc` for Linux users) will now require some modifications in the new `.zshrc` file. Similarly, you will have to install the updated Xcode and to re-initialize `conda`.


## 2. Changing the Shell for Good

The first thing after installing Catalina and opening the Terminal again is a message that asks you to make `zsh` the new default shell. At this point, there are command-line applications that are not working anymore (for me that was `git`).

To make `zsh` the new shell, run:
```bash
$ chsh -s /bin/zsh
```

Now, proceed to restart your Terminal.


## 3. Installing Xcode Command Line Tools

Apple's Xcode Command Line Tools is a package that provides a set of basic developer command line tools, like `git`, `pip`, `gcc`, `svn`, and so on. As clearly explained in this Stack Overflow [post](https://stackoverflow.com/questions/52522565/git-is-not-working-after-macos-update-xcrun-error-invalid-active-developer-pa), after every major OS release it is necessary to update Xcode Command Line Tools. To do so, run:
```bash
% xcode-select --install
```

This will open MacOS' default installer and will ask you if you want to proceed with the installation through the GUI. Agree with all there is to agree with and that's it. This enabled `git` back for me, but not `conda`.


## 4. Re-enabling Anaconda

Before you read this whole section, note that there are two main scenarios here: the better-and-easy-to-solve one, where Anaconda was installed in your home directory — e.g. `~/anaconda3`, and the worse-and-complicated one, where it was installed on root. As stated in this [post](https://www.anaconda.com/how-to-restore-anaconda-after-macos-catalina-update/) by Anaconda, if you are in the latter case then it might be a bumpy ride. As stated at the beginning of this document, a solution for this case will not be presented here — there are plenty sub-scenarios which involve Catalina moving files previously located in root to a folder called `Relocated Items`, which will be sitting in your desktop. If this is your case you will have to do *rehoming*, which is explained in the link above.

If you are in the "better" case, however, the solution that worked for me is really simple and is the following one. First, re-activate `conda` using `source`:
```bash
% source ~/anaconda3/bin/activate
```

Then, just re-initialize conda but for `zsh`:
```bash
% conda init zsh
```

After doing this, you should be able to find `conda` and `python` using `which conda` and `which python`, having both returning a path to `~/anaconda3/`.

This is it. Please let me know if there's any bug here.

[^1]: https://en.wikipedia.org/wiki/Z_shell