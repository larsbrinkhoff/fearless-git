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

### Commit messages

I'll just copy from the "git commit" man page:

> it’s a good idea to begin the commit message with a single short
> (less than 50 character) line summarizing the change, followed by a
> blank line and then a more thorough description.
