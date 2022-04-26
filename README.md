## Git Workshop

### Rebasing
`git rebase` changes the "base" that your changes build upon. As an example, look at the following diagram, where each letter represents a single commit.
```
      E---F---G my-feature
     /
A---B---C---D   main
```

We created a new branch off of the main branch when commit "B" was the newest commit. We added three commits (E, F, G) in our branch while the main branch continued to receive additional commits (C and D). Now, we want to merge our changes into the main branch!

#### Git Merge

The strategy you are likely most familiar with is the `git merge` strategy - let's see how that works in our example. 
```
      E---F---G   my-feature
     /         \
A---B---C---D---E main
```

With the `git merge` strategy, a brand-new commit "E" is created in the main branch. This commit contains the *combined the diffs from the _merge base_ to the ends of each branch*. The merge base is the commit where the branches diverged, in this case "B". 

#### Git Rebase

Let's move on to the `rebase` strategy. Let's start with the same set of commits as we did earlier:
```
      E---F---G my-feature
     /
A---B---C---D   main
```

Now let's look at the commit history after a _rebase_ from my-feature onto main has been performed:
```
              E`---F`---G`   my-feature
             /
A---B---C---D                main
```

`git rebase` takes a set of commits and reapplies them on top of another tip. The first thing you might notice is that `main` hasn't actually changed - we'll still need a `git merge` to take care of that - but now the `git merge` results in a _fast-forward_ instead of a merge commit, the end result being:

```
A---B---C---D---E`---F`---G` main
```

The second thing to note is that the commits have changed. E became E\`, F is now F\`, etc. The _content_ of the commits stays the same, but the commit itself is in a different location and therefore a distinctly different commit. Conceptually, git takes the commits from the feature branch and "replays" them one-by-one onto the tip of the main branch.

#### Interactive rebasing

Rebasing has another cool trick - the ability to do an _interactive_ rebase. Interactive rebasing allows you to re-order, squash, or remove commits while rebasing. You can even do an interactive rebase onto your _own_ branch in order to modify your previous commits.

