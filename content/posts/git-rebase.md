---
title: Git Rebase 
date: "2019-05-14T11:46:37.121Z"
template: "post"
draft: false
slug: "/posts/git-rebase/"
category: "Tutorials"
tags:
  - "git"
  - "rebase"
  - "tutorial"
description: "Git rebase jutsu "
---

In Git, there are two main ways to integrate changes from one branch into another: the merge and the rebase. 

---

    Branch early, branch often

---


Say Dan and  Moses are working on the domains branch and  John is working on the master branch. Now, what we need to do is to integrate the changes that are on master into our branch.  If you have pushed your changes back into the remote branch, you should merge.

But, if you didn't push yet, probably what you want to use is not a merge, probably it's a rebase. Rebase is a way of changing the history and basically what the rebase does is it takes the changes that you have on your branch and reapplies on the top of the branch that you specified. 

Rebasing enables fast-forward merges by moving a branch to the tip of another branch. It effectively eliminates the need for merge commits, resulting in a completely linear history.

Fast forward merges are merges that can be applied when the the current branch is the ancestor of the branch you want to merge. Fast forward merges simply move the ancestor’s hash to be the new master hash.



## In summary

git rebase allows you to rewrite your history. It alters the repository so that the commits you can see before and after its usage are different. It can change, merge, split, add, remove, modify commits.

In case the changes are pushed upstream already and  you change history, you will break merges for EVERYONE that has already ‘pulled’ your branch. That’s why you only use rebase when you haven’t pushed your changes. 


---

NOTE: Don’t rebase if you have already pushed your changes upstream.

---

## Simple Rules

1.No commit should EVER be rebased if
    -fit is already public.
    - If you have already pushed it
    - If you have pulled or fetched it, or
    - You have merged it on a different branch




There are 2 types of rebases:
   - Batch
   - Interactive


# BATCH Rebase

git rebase master 
Takes a branch, and modify it to make it look like the branch never existed, and the dev. Was done directly off another branch.

Example:


A-B-C-D              master
        \-E-F-G      topic

Becomes:

A-B-C-D-E’-F’-G’   master
From ‘topic’ branch: ‘git rebase master’

git rebase does an actual merge and Merges may have conflict.  You have three choices

    1.  Solve the conflicts:
          git add the resolved-files
          git rebase –continue
    2.  Skip this commit
          git rebase –skip
    3. Abort the rebase
         git rebase --abort



## Change Tree Structure

You can also change the tree structure of your repository.

E.g: you have master, branch A forked from master, branch B forked from A

After: git rebase --onto master A B

Now, branch B is forked from the HEAD of master


## Delete Commits


You  can delete commits:
    A-B-C-D-E-F      master
    git rebase --onto master~5 master~1 master
    A-E-F master



##Interactive Rebase

---

git rebase -i <commit/branch>

---

This will open an editor with the buffer

      pick bdffc87 [some tag] Some commit
      pick fed0f4c [game API] Another commit
      pick 5df5842 [docs] [minor] Second! commit
      pick 104c3c8 [data representation] First commit!

      #
      # Commands:
      #  p, pick = use commit
      #  r, reword = use commit, but edit the commit message
      #  e, edit = use commit, but stop for amending
      #  s, squash = use commit, but meld into previous commit
      #  f, fixup = like "squash", but discard this commit's log message
      #
      # If you remove a line here THAT COMMIT WILL BE LOST.
      # However, if you remove everything, the rebase will be aborted.
      #


If you do nothing, or remove all the lines, nothing happens
squash – This commit gets deleted, but its contents are added to the previous commit.  Commit messages are merged.
fixup – Like squash, but the commit message gets lost
reword – change the commit message
edit – stop there to allow modifying the commit





##Useful Commands

Pulling

git pull --rebase origin master  or git pull --rebase=interactive origin master


The local changes you made will be rebased on top of the remote changes

The interactive option gives you more control on the rebase process.


Documentation

https://developer.atlassian.com/blog/2014/12/pull-request-merge-strategies-the-great-debate/
