# Fearless Git

Learn to love interactive rebasing in a handful of easy steps!

### Table of contents

1. [About](#about)
2. [A little bit of background](#a-little-bit-of-background)
3. [Why rewrite pull requests?](#why-rewrite-pull-requests)
4. [Keep in sync with upstream](#keep-in-sync-with-upstream)
5. [Commit messages](#commit-messages)
6. [If you mess up, don't panic](#if-you-mess-up-dont-panic)
7. [Edit a single commit](#edit-a-single-commit)
8. [Reorder commits](#reorder-commits)
9. [Squash commits](#squash-commits)
10. [Split commits](#split-commits)

### About

This will be a random collection of git tips and tricks from the
trenches.  It will explain the nitty gritty of rewriting branch
history to make clean pull requests.

It's not a substitute for reading the git documentation.  The reader
is supposed to know the basics of git, and to be able to look up
details about git commands in the manual pages.

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

### Keep in sync with upstream

I recommend never adding any commits to the master branch.  Always
make a new branch for new features or bug fixes.

Before you make a new branch, it's a good idea to update your
repository from upstream.  Do a `git fetch` to bring in new commits,
check out your local master branch, and the `git merge --ff-only
origin/master`.

Sometimes, there can be two remote repositories: your own copy, and
the upstream repository to which you will post pull requests.  I have
my own remote as `origin` and the other I call `upstream`.  I fetch
from upstream to update the local master branch, and then I push this
to `origin`.  This way, my local and remote mater always tracks the
upstream.

### Commit messages

I'll just copy from the "git commit" man page:

> it’s a good idea to begin the commit message with a single short
> (less than 50 character) line summarizing the change, followed by a
> blank line and then a more thorough description.

### If you mess up, don't panic

Sometimes you're in the middle of a messy rewrite operation and you
feel things aren't coming together like you wish.  Don't worry, most
of the time you can type `git rebase --abort` to reset back to before
the operation.  Reconsider your plan and start over.

If you completed the rebase (or other operation) and have regrets, you
can try the reflog.  This is a long log of every commit git recorded.
Type `git reflog` and look at the top of the output.  If you want to
go back to a previous good commit FOO, type `git reset --hard FOO`.
**Warning:** this will overwrite files.  If you have any pending
changes that are not committed, they will be lost.

### Edit a single commit

To update the very latest commit, just stage your changes and type
`git commit --amend -C HEAD`.  Amending means to add changes to an
existing commit, and `-C HEAD` is used to keep the commit message as
is.

To update another commit, do `git rebase -i FOO~`, where FOO is that
commit.  It can be a hexadecimal commit ID, or a revision like
`HEAD~3` to select a commit relative to the current head.  This will
bring up your editor with text like this, plus some help text:

```
pick 1234567 Commit 1.
pick 89abcde Commit 2.
pick 6c437ee Commit 3.
```

On the line indicating the commit you want to edit, change `pick` to
`edit`.  When you exit the editor, git will start a rebase operation
and stop at that commit.  Now you can amend the commit like described
above.  When done, type `git rebase --continue`.

### Reorder commits

This is easy with interactive rebasing.  Type `git rebase -i master`.
This will offer up all commits on your branch:

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

### Split commits

Sometimes you make a commit and later want to split it in two.  If you
want to split commit FOO, do an interactive rebase against the the
commit two before it: `git rebase -i FOO~2`.  Your editor will display
the commit immediately before FOO at the top, then FOO, and following
commits if any:

```
pick QUUX Commit quux.
pick FOO Commit foo.
pick BAR Commit bar.
pick BAZ Commit baz.
```

Edit this to look like

```
edit QUUX Commit quux.
pick FOO Commit foo.
pick BAR Commit bar.
pick BAZ Commit baz.
```

This instructs git we want to make a change to a commit.  When you
exit the editor, git will stop at commit QUUX and let you edit the
files.  Note at this point we're immediately before FOO which we want
to split.

Now type `git checkout -p FOO` to make a **partial** check out.  If
you want just one particular file, type `git checkout -p FOO --
filename`.  Git will present you with a series of questions whether
you want a particular diff applied or not.  Answer `y` or `n` as
appropriate, or `s` to split a diff in even smaller parts.

When this is finished, you will have staged the changes.  Commit this,
and continue the rebase operation.  Git will see that FOO already has
the changes you split off, so it will not complain.

### Move a change to an earlier commit

Sometimes you commit some changes, and later want to move some change
to an earlier commit instead.  Doing this is similar to splitting a
commit described in the previous section.

Do an interactive rebase against the the commit before the one you
want to add to: `git rebase -i FOO~1`.  Your editor will display this:

```
pick FOO Commit foo.
pick BAR Commit bar.
pick BAZ Commit baz.
```

If you want to move something from BAZ to FOO, instruct git you want
to edit FOO:

```
edit FOO Commit foo.
pick BAR Commit bar.
pick BAZ Commit baz.
```

Exit the editor.  Git will stop right after FOO and allow you to do
edits.  Now type `git checkout -p BAZ`.  Like in *"Split commits"*
above, you can pick and choose changes.  Say `y` those you want.  When
done, your changes are in the staging area.  Now amend FOO: `git
commit --amend -C HEAD`.  Continue and finish the rebase.

### Update your pull request

With GitHub or GitLab, you can just do `git push -f` to forcibly push
your branch and update the pull request.
