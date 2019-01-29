# Fearless Git

### About

This will be a random collection of git tips and tricks from the
trenches.  It will explain the nitty gritty of rewriting branch
history to make clean pull requests.

### A Little Bit Of Background

Git was made by Linus Torvalds for managing Linux source code.  I
think it helps to understand how it was managed before Linus used
source control.  There were releases: tarballs which snapshotted the
source tree.  And there were patch series: a series of diffs building
on each others, usually posted to a mailing list.

The tarball snapshots map quite well to git version history.  In some
contexts, a git commit can be viewed as just a plain snapshot of the
entire source tree.  When you check out a commit, it's like deleting
all files and extract a tarball.

A patch series is similar to a pull request.  In this view, it's all
about the diffs.  Each diff in the series is requred to make one
logical, atomic change.  When a new Linux version was released, you
were required to redo the patch series by applying it to the new
version and update any mismatches.  This is exactly like rebasing.

### Why rewrite pull requests?

It's not just for fun, rewriting your pull request have actual
benefits.

- It's easier to review a pull request that consists of small atomic
  commits.  Perhaps more importantly, sometimes a a bug appears and
  someone else has to go back and look at your commits to understand
  their intent.

- If branch goes out of sync with the upstream master, rebasing it
  will make it possible to a fast-forward merge.  This makes for a
  linear history which is easier to understand than a history which
  branches and joins in several parallel arcs.

- Sometimes a change needs to be backed out.  If the commit does
  several unrelated things, a simple revert operation removes
  everything.

- It's generally accepted that code should be clean and readable.
  Just extend the notion to the time dimension.  Changes should be
  readable too.  Avoid spaghetti history like you would spaghetti
  code.

### Commit messages

I'll just copy from the "git commit" man page:

> it’s a good idea to begin the commit message with a single short
> (less than 50 character) line summarizing the change, followed by a
> blank line and then a more thorough description.

### Reorder commits

This is easy with interactive rebasing: `git rebase -i master`.  This
will bring up your editor with text like this, plus some help text:

```
pick 1234567 Commit 1.
pick 89abcde Commit 2.
pick 6c437ee Commit 3.
```

This is the Swiss army knife of git.  Each line is a command in git's
special rebase command language.  Each command will be applied in
top-down order to the base commit of the rebase, `master` in this
example.  Here we see `pick` which just adds a commit to the branch.
To reorder the commits, just reorder the lines as you see fit.  Then
save the file and exit the editor.

If the commits overlap, there will be merge conflicts.  Git will
announce this and stop for you to fix the conflict.  To do this, you
edit the files and stage them.  To proceed with the rebase, type `git
rebase --continue`.

### Squash commits

Merging two or more commits into one is called "squashing".  This can
also be done with `rebase -i`.  If, say, commit 3 is to be squashed
into commit 1, edit the text to say

```
pick 1234567 Commit 1.
squash 6c437ee Commit 3.
pick 89abcde Commit 2.
```

When git has finished, there will only be commit 1 and commit 3 left,
but the first will include the changes from commit 2.

You will have the option of editing the commit message, merging text
from both commits.  If this is not necessary, you can use `fixup`
instead of `squash`.  The commit message of the fixup commit will be
discarded.
