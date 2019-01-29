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

### Commit messages

I'll just copy from the "git commit" man page:

> it’s a good idea to begin the commit message with a single short
> (less than 50 character) line summarizing the change, followed by a
> blank line and then a more thorough description.
