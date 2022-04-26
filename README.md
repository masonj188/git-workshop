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

Now let's look at the commit history after a _rebase_ from my-feature onto main has been performed by running `git rebase main` from the `my-feature` branch:
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

#### Interactive Rebasing
Rebasing has another cool trick - the ability to do an _interactive_ rebase. Interactive rebasing allows you to re-order, squash, or remove commits while rebasing. You can even do an interactive rebase onto your _own_ branch in order to modify your previous commits. You can start an interactive rebase by adding the `-i` option. For example, if I wanted to fix a typo on a previous commit, I could add a new commit that fixes the typo and then run `git rebase -i HEAD~2` to mark my most recent commit as a fixup and add it into the previous commit.

### Partial File Commits
I've found occasionally while working on a project that I've added enough code to a _single file_ that it no longer makes sense for all the work to be a single commit. It would make more sense for the work to have been split up in multiple commits. This becomes a problem when you go to make a commit as you can only `git add` an entire file! Luckily, there is a way to only add a portion of a file into a commit and leave the rest of the changes unstaged.

You can accomplish this with `git add --patch <file>` (or `git add -p <file>`). Git will automatically try to split the file into different "hunks" and ask you if you'd like to stage the first chunk it found. You can see all the possible actions you can take by typing `?` into the prompt. Typically, I just use the `e` option which opens the file in your text editor and lets you manually pick which lines you'd like to stage.

### Git Config Tips/Tricks

#### SSH Config
When using SSH to connect to git servers a simple ssh config in `~/.ssh/config` streamlines the process immensely:
```
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/<your-github-key>
  IdentitiesOnly yes

Host gitlab.ssc.dev
  HostName gitlab.ssc.dev
  User git
  IdentityFile ~/.ssh/<your-gitlab-key>
  IdentitiesOnly yes 
```

#### Local vs Global Settings
Git has many options to tweak - some of which are more or less required to set. In general, options are set using `git config <option>`. There are two levels you can set git options at, globally and per-project. By default, setting an option only sets it within the current project, in order to make it apply globally, add the `--global` flag. Project-local settings are stored in `<project-root>/.git/config` while global options are stored in `~/.gitconfig` by default.

Some options that make sense to set globally are your name (which will show up in every commit you make), the editor you'd like git to open files in, modifications to the way diffs are shown (which we'll talk about later), and any other options that you want to set no matter which project you're working in:
```
git config --global user.name "John Doe"
git config --global core.editor nvim
```

Other options make more sense to set on a per-project basis, such as the email you'd like to associate with your commits. If you have a personal GitHub account, a work GitHub (or other VC) account, and a school GitLab account (like `gitlab.ssc.dev`), you'll likely want to associate commits to each of those accounts with the emails associated with those accounts. Git makes this pretty simple, simply `cd` into a directory within a locally cloned repository and any settings you set there will apply to that repository/project only:
```
cd ~/my-school-project
git config user.email "johndoe@uwyo.edu"

cd ~/my-personal-project
git config user.email "johndoe@gmail.com"
```
Remember that since we set our name globally, that setting will be inherited in each project unless it is specifically overridden in an individual project.

#### Parent Directory Settings
It can get quite tedious to set your email in each project as you clone them or create new projects if you're frequently switching between work, school, and personal projects. One way to simplify things is to keep a separate parent directory for each different email/account you use and keep all the projects associated with that email/account within that parent directory. For example:
```
~/work-projects/workproj1
~/work-projects/workproj2

~/personal-projects/personalproj1
~/personal-projects/personalproj2

~/work-projects/workproj1
```
Once you've organized your projects this way, you can modify your global git configuration to look for different git config settings depending on which directory you are in. For example, your `~/.gitconfig` may look like so:
```
[user]
    name = John Doe

[includeIf "gitdir:~/work-projects/"]
    path = ~/work-projects/.gitconfig

[includeIf "gitdir:~/personal-projects/"]
    path = ~/personal-projects/.gitconfig

[includeIf "gitdir:~/school-projects/"]
    path = ~/school-projects/.gitconfig
```
Then, create a new `.gitconfig` file in each of those parent directories and include the options you'd like to override in there. For example, your `~/school-projects/.gitconfig` file may look like this:
```
[user]
    email = johndoe@uwyo.edu
```
Now every project within the `school-projects` directory will have the email set to the uwyo email address.

#### Better Diffs
When using `git diff` the default diff display is okay, but there are other alternatives that look better in my opinion - one of those is [`delta`](https://github.com/dandavison/delta). Simply download and install `delta` and then add these settings to your `~/.gitconfig`:
```
[core]
    pager = delta

[interactive]
    diffFilter = delta --color-only

[delta]
    navigate = true  # use n and N to move between diff sections
    side-by-side = true
    hyperlinks = true

[merge]
    conflictstyle = diff3

[diff]
    colorMoved = default
```
Make sure you don't add another "[core]" section if you already have one because you've set your editor, just add the `pager = delta` line to the existing "[core]" section.
