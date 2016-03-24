---
layout: post
title:  "git subtree - alternative way to manage submodules with Git"
date:   2013-10-20
categories: git
---

### Update on 19/09/2015:

Git submodules are often considered as de-facto standard to incorporate one Git module to another, yet there is an
option which slightly less known - git subtree. For a long time `git subtree` was a part of `git-contrib` and we
had to install it manually. In recent versions `git subtree` became a part of standard git distribution, which makes
it officially an alternative to `git submodule`.

The text which follows is the original article I posted in 2014, providing one of the many examples of where
`git subtree` may be used, and showing how to use it.


## Git subtree

One of the Git-based solutions to modularize a large code base, or to reuse existing repositories in multiple places -
is using a `subtree`. `subtree` is basically a directory in one branch of the repository, which may be managed
in the other branch of same or another repository, and both can be well synchronized.


### Why?

I have found `git subtree` very useful in the transition period when splitting large modular code base into a multitude
of the repositories.

For example, let's say there is a project consisting of several modules: every module is managed in a sub-directory of
a single source code repository. Now, every module can be moved to its own "clean" repository (or even a branch of
the same repository; although..., don't do that) while those sub-directories in existing branches will reference
newly created repositories (branches). New changes continue in the new repositories, and are merged back to
sub-directories of previously existing "super" branches.

This clearly sounds very similar to `git submodule`, but it is completely different inside, and its implementation
differences render several advantages:

1. Full transparency - any tools or workflows that currently depend on the project structure will continue to work
well - they'd be using same old branches, and won't require anything additional, not even a single additional Git command.
So, your CI server won't notice anything, for example.

2. Similarly, if necessary, development can still continue on previously existing development "super" branches, and can
be periodically synchronized back to module repositories or branches.

So, neither of workflows are changed, but every module is already in its own repository. This may be nicely used in the
transition period - when not all of the tools or processes are ready for the change.


## Hands on git subtree

### git subtree - installation

*Update 15/09/2015*: depending on how you installed Git (Homebrew on Mac, apt-get on Ubuntu, or else) and very likely
if you are using Git 1.8 or later, `git subtree` is normally already bundled in, and you may skip
this section. Check if `git subtree` exists, and proceed with installation instructions if it doesn't.

If `git subtree` command is not available, it can be installed from its Git contrib project. Installation is
[explained in its GitHub repo](https://github.com/git/git/blob/master/contrib/subtree/INSTALL), or one may try on of
the [alternative ways](http://stackoverflow.com/questions/11603510/how-do-i-correctly-install-the-tools-in-gits-contrib-directory),
especially on Windows where `make` is not available in Git Bash.

And yes, this is only required on those systems, where a subtree will be manipulated, but it isn't required where
the repository will be used for regular fetch-merge-commit-push workflow.

### Moving directories to external repositories or branches

Let's say, the project under question consists of several modules ("module-1".."module-n") , and one would like to
split some of those (or maybe all of them).

    $ git checkout source-branch
    $ git subtree split --rejoin --prefix=module-n --branch module-n

Notice now, that new branch `module-n`, contains now entire history of all changes made to the directory `module-n`
of a source branch, and only changes made to its files. Newly created `module-n` branch contains only `module-n` directory
content, all in its root now, not in `module-n` subdirectory. It is also an orphan Git branch - that is it starts its
own history from zero, just like `master` branch for example. Finally, `--rejoin` also merged this branch to the source
branch, to make it explicit that this is incorporated into the source branch from this new module branch.

This is the initial setup. Like this, it is pretty nice - basically the module is fully extracted into its own branch,
with its own history. New module branch can be pushed to a new repository for example, but the advantage of git subtree
that this is not even necessary - you can still keep everything in one repository.

### Synchronizing subtree and its branch

Now, both, module branch and branch including it, can be well synchronized, whether the changes are made on either of
them, or even on both of them.

If one wants to focus on one module only, checkout module branch, make feature branches, commit one or multiple
changes, do the usual stuff. Unless explicitly requested, all your work rests in this branch only. But then later, this
can be all delivered back to the "super" branch, with:

    $ git checkout target-merge-branch
    $ git subtree merge [--squash] --prefix=module-n module-n

Using `--squash` will deliver everything in one commit, while otherwise every commit to `module-n` branch will be
replayed in the target merge branch.

The good thing is that in the target branch, module source can be updated to any point of the module branch, not
necessarily the latest one. And, when using `--squash`, not even sequentially (that's right, you can basically rollback
to older state if needed). Just specify the commit or tag to merge, just like you would do with usual `git merge`.

Next, it isn't required that this is only merged with the original source branch. Let's say for example, that you as
well want to deliver some older, more stable version of module to the release branch:

    $ git checkout release_1.0
    $ git subtree add --prefix=module-n modn.v.0.9

Now, release branch contains an older version of one module (found by tag `modn.v.0.9`), but likewise it can contain
other versions of other modules (`mod2.v.1.2`, etc.). Same wouldn't be possible if all modules are managed in one
development branch only, would it?

But what about changes in the original source branch? What if one still wants to work on old source branch, not
switching to module specific branches? Yes, this is also possible. The point about `git subtree split`, is that
subsequent splits are consistent. Even if one did multiple changes in the original source branch after the split, and
even if they were mixed with other changes in other modules, and even if they were mixed in same commits, all this is
still OK. Repeating same `git subtree split` command over the time will just update the module branch with changes made
to the module as a subdirectory in the original branch:

    $ git subtree merge [--squash] --prefix=module-n module-n

This will as well work fine if some changes were made in both branches - regular git merge happens in this case, and
regular conflict resolution, if any exist.

## Sum up

So, when compared to `git submodule`, I would say that `git subtree` differs in following:

 - `subtree` is more complex to set-up correctly: unlike `submodule` it's setup is just a little more complicated, and
   it may require quite complex merges if changes are expected to be made on both sides;

 - on the other hand, `subtree` is much more easy to use by those who don't set it up or merge - it is absolutely
   transparent to the users after it was configured and doesn't require any additional knowledge or any additional Git
   commands to issue; unless, again, you do changes in both places in parallel;

 - and `subtree` is quite flexible in how it allows making changes in both places

A verdict? I have found a `subtree` very useful in the scenario presented above - in the transition period when
splitting large modular code base into a multitude of the repositories. For other use cases, I think `submodule` is
better suited to reuse a repository in the another one - even though it requires a couple of additional commands, it is
more straightforward to use - exactly because it is explicit in what it does.

