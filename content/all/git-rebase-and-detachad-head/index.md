+++
categories = ["nuggets", ]
title = "git rebase -\\-onto and the Detached HEAD"
author = "Raul Morales Delgado"
date = 2020-10-08
description = "In this post, a solution is presented to the problem of rebasing the section of a branch to a new parent commit, and making that new section a new branch on which to work."
toc = true
tags = ["git", "devops", ]
+++

## Introduction
I was recently facing a Git situation in which I had to rebase a section of a branch onto a specific commit of `master`. That is, I wanted to rebase only a portion of my branch onto a different parent commit of `master` than the current one. When doing so, however, I wanted to keep the previous branch and make the new one, a detached `HEAD` by default, into a branch I can work on.

If you've never got a problem like this in Git, don't worry — you *will*. I looked for solutions and I found this extremely helpful blog [post](https://womanonrails.com/git-rebase-onto) — it provides an answer on how to cut a section of a branch and rebase it onto master. However, it lacked the answer as to how to recover the detached `HEAD` pointing to the commits that form the section of the branch that was cut.

In this post, I will present a solution for this problem.

## git rebase -\-onto
To those familiar with `git rebase`, its basic syntaxis is the following:[^1]
```bash
git rebase [-i | --interactive] [<options>] [--exec <cmd>]
	[--onto <newbase> | --keep-base] [<upstream> [<branch>]]
```

where `<newbase>`, `<upstream>` and `<branch>` can be a commit-like arguments — e.g. a branch name, a short SHA-1, a revision parameter like `HEAD~3`, etc. Also, by default, if `<branch>` is not specified, `HEAD` will be considered.

In this post, I will explore this command:
```bash
git rebase --onto <newbase> <upstream> <branch>
```

The best way — for me — to parse this command out and make sense of it is like this:
```text
git rebase, from <upstream>'s child until <branch>, --onto <newbase>
```

Put in a different way, the resulting branch will go from `<upstream>` to `<branch>`, exclusive of the former and inclusive of the latter. (Don't worry, this will be shown in an example in the following section.)

Remember that a branch is just a pointer to a commit — the last one in that branch. Also, when successfully executing `git rebase`, a piece of the commit history will have been rewritten — a group of existing commits will get new SHA-1 hashes. After the rebase, `HEAD` will end up pointing to the rewritten `<branch>`, whether it was explicitly provided or not (in the latter case, `HEAD` will point to "`HEAD'`").

>**Note:** This post will not delve into the `git rebase` command. There is plenty official [documentation](https://git-scm.com/docs/git-rebase) and [nice tutorials](https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase) on how to use it.


## Sectioning and Rebasing a Branch
As mentioned earlier, the command presented above allows not only to rebase a branch to a new parent but also to section it — to extract only a part of it. Imagine we have the following commit history:
```text
1 — 2 — 3 — 4 — 5 — 6 — 10 — 11 — 12 — 13 — 14 (master)
                     \ 
                      7 — 8 — 9 (dev)
```

Remember that, in a branch, every commit has a parent (or parents, in case of a merge). In the example above, commit `6` is the parent of commit `7`. If we wanted to section branch `dev` to extract **only** commit `8`, and rebase it — that is, change its parent commit —, onto commit `11`, we would use `git rebase` in the following way:
```bash
git rebase --onto 11 7 8
```

As you might have figured it out already, non of the above arguments is a branch. They are all commits. Parsing out the expression above in a *human* way, we are asking `git (to) rebase, from 7's child until 8, --onto 11`. (Of course, `7's child` is `8`, but the section is exclusive of `<upstream>` and inclusive of `<branch>`.)

The resulting tree would look like this:
```text
                               8' (HEAD)
                              /
1 — 2 — 3 — 4 — 5 — 6 — 10 — 11 — 12 — 13 — 14 (master)
                     \ 
                      7 — 8 — 9 (dev)
```

Because `git rebase` rewrites history, `8'` is a new commit with a new SHA-1 hash. By executing the previous command, we have conserved branch `dev`, and we have created a new *branch*, with only one commit in this case — `8'` —, and `HEAD` is pointing to it. 

But is it a real branch? Not yet. Regardless if it is 1 or 100 commits, if we ran `git branch -a` to retrieve all available branches, we would get something like this:
```text
* (HEAD detached from 6)
  dev
  master
```

The `*` above indicates where `HEAD` is pointing to — we explained before that a successful execution of `git rebase` will leave the `HEAD` pointing to the argument passed in to `<branch>` — in this case commit `8'`. So, how do we recover that new branch where `HEAD` is pointing to and make it a new branch on which we can start working?


## Making the Detached HEAD a Branch
If you were to start working straight away, you would be able to make changes and commit them, undo commits, and so on. However, all these changes will only exist as long as you are *in this branch*. If you checkout another branch, e.g. `git checkout master`, then changes made on this `HEAD detached` *branch* will stop existing and will be lost.[^2] 

In order to start working on this branch, a new branch needs to be created. We can do so using the following syntaxis of `git checkout`:
```bash
git checkout -b <new_branch> <start_point>
```

where `<new_branch>` is the name of the new branch you want to create, and `<start_point>` is the name of the commit at which to start the new branch. Option `-b` allows to create a new branch and checkout on it (make the `HEAD` point to it)[^3].

For instance, if we run:
```bash
git checkout -b test HEAD
```

the resulting tree would look like this:
```text
                               8' (HEAD -> test)
                              /
1 — 2 — 3 — 4 — 5 — 6 — 10 — 11 — 12 — 13 — 14 (master)
                     \ 
                      7 — 8 — 9 (dev)
```

Finally, imagine that you did all this because you already had a branch `test`, which turned out not being as good as just grabbing commit `8` and start working on it. To get rid of your previous `test` branch and make it a new one that points to `8'`, you can use the following oneliner:
```bash
git checkout -B <branch> HEAD
```

Option `-B` on `git checkout` allows reset an existing branch and point it to `<start_point>`.

[^1]: Git rebase [documentation](https://git-scm.com/docs/git-rebase)
[^2]: There are [ways](https://stackoverflow.com/a/14757539/11905552) to recover that last commit you made on the `HEAD detached` *branch*. However, this ways rely on the reflog, which is periodically cleaned by Git — it is not a permanent log.
[^3]: Git checkout [documentation](https://git-scm.com/docs/git-checkout)
