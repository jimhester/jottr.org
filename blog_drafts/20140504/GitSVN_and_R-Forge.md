
# How to: Git SVN for R-Forge
by Henrik Bengtsson on 2014-05-04

This document explains how to (bi-directionally) access your [R-Forge] repository using [Git] instead of [Subversion].  We will cover three important things:

1. How to setup a Subversion-to-Git map for author annotations.
2. How to clone ("checkout") a Subversion repository.
3. How to setup an "R-Forge" branch in Git that represents whats on R-Forge.
4. How to to update and commit via the R-Forge branch.


## Introduction
Although the instructions herein are long, they are only so to make sure that we carefully walk through the details at least once.  As soon as you get the gist of if, everything is very simply and you'll be very quick in doing this yourself.

So, when you understand Git, it is very easy, but the learning curve might be a bit steep.  The only complicated task is to under how author annotations are mapped between Git and Subversion and to accept that it may be too much work to have Subversion reflect the proper author information when contributed via Git (the R-Forge submitter [=you] will become the author of those commits at R-Forge).

Cheat sheet:

* `svn checkout` = `git svn clone <url>`
* `svn update`   = `git svn rebase`
* `svn commit`   = `git svn dcommit`
* `svn info`     = `git svn info`


## A one time important step before anything else
```
> git config --global svn.authorsfile ~/.gitauthors
```
On Windows, you have to do:
```
> git config --global svn.authorsfile %HOME%\.gitauthors
```
Using `git config --global` will query/modify/create `~/.gitconfig`, which is a "global" user-specific Git configuration file that will be used by all `git` calls, meaning the above setting `svn.authorsfile` will be used in all your Git interactions for all your R packages.  Don't worry, it's settings can be overridden locally by `git config --local` which edits `./.git/config`, but now where getting ahead of ourselves.


## Checkout your R-Forge package as a Git repository

```
> git svn --use-log-author clone svn+ssh://henrikb@svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.synchronize

Initialized empty Git repository in x:/_GITHUB/R-Forge exports/r-dots/foo/R.methodsS3/R.menu/.git/
W: Ignoring error from SVN, path probably does not exist: (160013):
   Filesystem has no item: File not found: revision 100, path '/pkg/R.synchronize'
W: Do not be alarmed at the above message git-svn is just searching aggressively for old history.
This may take a while on large repositories
Checked Ahrough R/000.R
        A       R/006.fixVarArgs.R
        A       R/SocketMutexServer.R
        A       R/NullFileLock.R
        A       R/999.package.R
        A       R/999.NonDocumentedObjects.R
        A       R/AbstractMutex.R
        A       R/FileLock.R
        A       R/999.missingdocs.txt
        A       R/SocketMutex.R
        A       R/NullMutex.R
        A       R/zzz.R
        A       DESCRIPTION
        A       incl/BengtssonH_2003.bib.Rdoc
        A       incl/SocketMutexServer.Rex
        A       incl/FileLock.Rex
        A       incl/SocketMutex.Rex
        A       man/SocketMutex.Rd
        A       man/NullMutex.Rd
        A       man/R.synchronize-package.Rd
        A       man/AbstractMutex.Rd
        A       man/FileLock.Rd
        A       man/Non-documented_objects.Rd
        A       inst/testScripts/system/locks/test20090526-lock.Rex
        A       inst/NEWS
r238 = 2f8f4021436d10a2808b80c18b0d0b97d767b118 (refs/remotes/git-svn)
        D       R/999.missingdocs.txt
        M       R/999.NonDocumentedObjects.R
        A       incl/999.missingdocs.txt
W: -empty_dir: R/999.missingdocs.txt
r243 = 123a39e8c1fb324d7f40c3feee3629fa3e600ca5 (refs/remotes/git-svn)
Checked out HEAD:
  svn+ssh://henrikb@svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.synchronize r243
```
That's pretty much it.  The reason we `--use-log-author` is "just in case" and will be explained further below.

To verify that it worked, you can for instance check the _Git_ log for the most recent commit;
```
> git log -1
commit 123a39e8c1fb324d7f40c3feee3629fa3e600ca5
Author: henrikb <henrikb@edb9625f-4e0d-4859-8d74-9fd3b1da38cb>
Date:   Fri Jun 5 18:37:04 2009 +0000

    Further rearrangements for passing R CMD check without prefiltering files.

    git-svn-id: svn+ssh://svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.synchro
```

Note that the above displays the log of your (local) Git repository.  To check the log of the (R-Forge) Subversion repository, do
```
> git svn log -1
------------------------------------------------------------------------
r243 | henrikb | 2009-06-05 11:37:04 -0700 (Fri, 05 Jun 2009) | 2 lines

Further rearrangements for passing R CMD check without prefiltering files.

------------------------------------------------------------------------
```

Of course, since there hasn't been any commits (to neither the Git nor the Subversion repository), the above two logs should the same information.


### Windows only: Fix Git mistake
Git "cloning" does not handle the fact that you have a colon in your settings:
```
git config --global svn.authorsfile
C:\\Users\\hb/.gitauthors
```
Because of this, it will incorrectly add a "local" version of this setting that will override the "global" one:
```
git config --local svn.authorsfile
x:\\work\\path\\C;C:\\Program Files (x86)\\Git\\Users\\hb\\.gitauthors
```
 which later will cause error messages about not finding the authors file.
To fix this, let's just drop it again:
```
git config --local --unset svn.authorsfile
```


## Creating a "R-Forge" branch reflecting what's on R-Forge

While developing new features to your package, you have probably sometimes found yourself copying the complete package directory to an separate location to avoid having your new trial-and-error development clutter up your "master" version.  Also, while working on this "devel" branch, you no longer keep track of versions and your modifications are not backed up (at least not via R-Forge).  Finally, when you've finished this work, you've then copied files back into the "master" directory and commiting to R-Forge.

With Git, the above manual copying is completely unnecessary.  Instead Git will let you switch to your "devel" branch with a simple "checkout" command, while keeping your "master" branch safe and hidden.  Git will automatically switch your files "in and out" between branches.  The best thing, all your work is under version control and part of the Git repository.

Since you will wonder, working with multiple branches is lightweight, i.e. the overhead in additional disk space when adding a new branch basically corresponds to the size of the text message you add to a commit.  Also, Git switches between branches ridicolously fast - it's so fast that first timers tend to verify it manually, because they didn't even notice the switch.  In other words, there is no overhead costs working with branches in Git.

Although you can always to it later (also retroactively), it's easier to create the "R-Forge" branch immediately after "cloning" the Subversion to Git.  Any Git repository always has a "master" branch, so does ours:
```
> git branch
* master
```
Now, lets create the "R-Forge" branch:
```
> git branch R-Forge
```
and confirm that it's added:
```
> git branch
  R-Forge
* master
```
The asterisk in front of 'master' indicates that this is the currently "checked out" branch, which means that the files you see in the working directory reflects those in the "master" branch.  To switch to the "R-Forge" branch, do:
```
> git checkout R-Forge
Switched to branch 'R-Forge'
```
which you can confirm with:
```
> git branch
* R-Forge
  master
```
At this point, the "master" and the "R-Forge" branch contain identical file sets.  If you would edit any of the files right now, those modifications would only affect the "R-Forge" branch, because that's the currently checked out branch.

TIPS: You can create and checkout the R-Forge branch in one go via `git checkout -b R-Forge`.


## Never edit files in the "R-Forge" branch
While it's possible, I recommend that you never edit files while in the "R-Forge" branch.  Instead do all your work in the "master" branch  Later we'll might refine this to always do work in a third "devel" branch.  Either way, you should never have to do edits in the "R-Forge" branch.  That exists only to reflect what's on R-Forge, and for committing to and updating from R-Forge.

## Do some edits and committing those to R-Forge
Let's bump the version number of the package but editing the `DESCRIPTION` file.  However, first, remember to not modify the R-Forge branch, so let's switch to "master":
```
> git checkout master
Switched to branch 'master'
```
and verify:
```
> git branch
* R-Forge
  master
```

### Edit files and ask Git what's changed
Now, bump the version in `DESCRIPTION` and save.  After doing this, we can ask Git for what has changed (relative to what's in the Git repository):
```
> git diff
diff --git a/DESCRIPTION b/DESCRIPTION
index df2846d..088c05a 100644
--- a/DESCRIPTION
+++ b/DESCRIPTION
@@ -1,6 +1,6 @@
 Package: R.synchronize
-Version: 0.0.3
-Date: 2009-05-30
+Version: 0.0.4
+Date: 2014-05-04
 Title: Synchronious locking mechanisms
 Author: Henrik Bengtsson <henrikb@braju.com>
 Maintainer: Henrik Bengtsson <henrikb@braju.com>
 ```
We can also get a brief summary of what changed:
```
> git diff --stat
 DESCRIPTION | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)
```

Note, it's safe to try to switch to another branch.  Files will be preserved even if they are not committed, and if there is a situation when it is not safe to switch, Git will prevent you from doing it and explain why.


### Commit changes to R-Forge
After having tested our package in the "master" branch, for instance by running `R CMD check --as-cran`, we can commit our changes to R-Forge.  This will be done in three steps:

1. Commit changes to Git (branch "master")
2. Merge changes in "master" into the "R-Forge" branch.
3. Commit to the R-Forge server.

First, we need to commit the above modications of the "master" branch, because currently those are only available as "tracked but not committed" files.  This is done by:
```
> git commit -a -m "Bumped the version"
[master 7a7f3ce] Bumped the version
 1 file changed, 2 insertions(+), 2 deletions(-)
```
The log will tell you that it went through:
```
> git log -1
commit 7a7f3ce61ac5a32a51c02790e3f0af9fbdbb97bc
Author: hb <hb@aroma-project.org>
Date:   Sun May 4 13:20:51 2014 -0700

    Bumped the version
```
and so does `git diff` which will given an empty output.

Second, let's bring those changes over to the "R-Forge" branch.  To do this we first have to switch over:
```
git checkout R-Forge
Switched to branch 'R-Forge'
```
If we forgot what has changed since last R-Forge commit, we can do:
```
> git diff --stat R-Forge master
 DESCRIPTION | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)
```
which will tell us that one file has been modified in the "master" branch that is still not in the "R-Forge" branch.

Let's bring in those modifications by _merging_ the "master" branch into the "R-Forge" branch.  This part may sound like voodoo, but these days merging of text-based files works almost flawlessly and pretty much emulates what you would have done manually.  Also, if there are potential conflicts, e.g. the exact same line has been modified in both branches (remember, you shouldn't edit the "R-Forge" branch anyway), Git will detect this, inform you and let you edit it manually.  Git is always your friend - not you enemy.  So, let's merge:
```
> git merge "master"
Updating 70ac435..7a7f3ce
Fast-forward
 DESCRIPTION | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)
```
That's it!  Now, let's check the log of this ("R-Forge") branch:
```
> git log -1
commit 7a7f3ce61ac5a32a51c02790e3f0af9fbdbb97bc
Author: hb <hb@aroma-project.org>
Date:   Sun May 4 13:20:51 2014 -0700

    Bumped the version
```
So, the commit we did in the "master" branch is now incorporated in the "R-Forge" branch.  Also, since we haven't modified the "R-Forge" branch, the "R-Forge" and the "master" branches our now identical, which you can verify by `git diff R-Forge master` giving an empty output.

Oops, we almost forgot, let's rebase the "R-Forge" branch such that it points the the current state of the "master" branch:
```
> git rebase master
First, rewinding head to replay your work on top of it...
```

```
> git rebase master
First, rewinding head to replay your work on top of it...
Fast-forwarded R-Forge to master.
```

Note, all we have done this far have only affected out Git repository.  So, as a last step we have to commit the "R-Forge" branch to the R-Forge servers:
```
> git svn --use-log-author dcommit
Committing to svn+ssh://henrikb@svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.synchronize ...
        M       DESCRIPTION
Committed r2140
        M       DESCRIPTION
r2140 = 90a69bf3ab12e51b1c034f4960554fe8193c25fb (refs/remotes/git-svn)
No changes between 7a7f3ce61ac5a32a51c02790e3f0af9fbdbb97bc and refs/remotes/git-svn
Resetting to the latest refs/remotes/git-svn
```
As before, we use `--use-log-author` just in case as explained later.

We're done.  We can verify by asking R-Forge about the Subversion log:
```
> git svn log -1
------------------------------------------------------------------------
r2140 |  | 2014-05-04 13:38:49 -0700 (Sun, 04 May 2014) | 2 lines

Bumped the version

------------------------------------------------------------------------
```

It is still the same as before.  This is because a commit to a Git repository ony happens locally.  The logic behind this is that all Git repository are local.  In order to "push" the commits (now in the Git repository) to R-Forge, do:
```
> git svn --use-log-author dcommit
Committing to svn+ssh://henrikb@svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.synchronize ...
        M       DESCRIPTION
Committed r2139
        M       DESCRIPTION
r2139 = 70ac435dda41ff4be501ddb36ba524cb16376754 (refs/remotes/git-svn)
No changes between 96a4f58cb22c7cb621df40c570d8055d11515c5a and refs/remotes/git-svn
Resetting to the latest refs/remotes/git-svn
```

```
> git svn log -1
------------------------------------------------------------------------
r2139 | henrikb | 2014-05-03 22:54:26 -0700 (Sat, 03 May 2014) | 2 lines

Bumped the version

------------------------------------------------------------------------
```

## What about the author information?
If we compare the `git log` and the `git svn log` further, we notice that the timestamp and
the username of the Subversion log do not match those of the Git log.

> "... the subversion repository format is poorer than the git one. It doesn't support merging well, or committers vs authors or commit times distinct from push times. git-svn is OK to get the git ui locally, but it can't do much about the data model." [http://stackoverflow.com/questions/5327924/git-svn-dcommit-with-svn-usernames]

```
> git config --global svn.authorsfile ~/.gitauthors
> git config --local --unset svn.authorsfile
```
```
--use-log-author

--authors-prog=<filename>
If this option is specified, for each SVN committer name that does not exist in the authors file, the given file is executed with the committer name as the first argument. The program is expected to return a single line of the form "Name <email>", which will be treated as if included in the authors file.
```

## Check if the Subversion repository has any modifications

```
x:\_GITHUB\R-Forge exports\r-dots\foo\R.synchronize>git svn info
Path: .
URL: svn+ssh://svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.synchronize
Repository Root: svn+ssh://svn.r-forge.r-project.org/svnroot/r-dots
Repository UUID: edb9625f-4e0d-4859-8d74-9fd3b1da38cb
Revision: 243
Node Kind: directory
Schedule: normal
Last Changed Author: henrikb
Last Changed Rev: 243
Last Changed Date: 2009-06-05 11:37:04 -0700 (Fri, 05 Jun 2009)
```

## All existing commit authors

All existing authors who even committed to the Subversion repository:
```
> git svn log | grep "r[0-9]" | cut -d" " -f3 | sort -u
henrikb
```

Now create an `svn-git-authors.txt` file:
```
henrikb = Henrik Bengtsson <henkebenke@aroma-project.org>
```


# Appendix

## Don't use svn2git
Although it's neat, it's unnecessary, but what's worse is that it uses `--no-metadata` which will prevent you from having your Git repository to sync towards R-Forge.  See `git svn --help` for more details.


## Tips for Windows users

### "Terminal not fully functional"
Do you see this?
```
> git log
WARNING: terminal is not fully functional
...
(END)
```
If so, then do:
```
set TERM=msys
```
and you'll get rid of the message and at the same time highlighted (color annotated) output from Git that is much easier to read.  To set it permanently, use `setx TERM msys /M`

## Validate Git repository
```
git fsck --full --strict
git gc
```

## Shortcuts
```
# Clone and add R-forge branch
git rfcl r-dots R.cache  # clone R-forge package r-dots:R.cache

# Update .Rbuildignore
git ci "Ignoring more files"

# Add .travis.yml
git add .travis.yml
git ci "TRAVIS: Setup"

# Add R package version tags
git log
git tag v0.9.0 e87d3c7ee66bd79f8a90e63eb7499eb5f84a42cb

# Commit to R-Forge
git cor
git rebase master
git rfci
```

## GitHub
```
git remote add origin git@github.com:HenrikBengtsson/R.utils.git
```


> git log

WARNING: terminal is not fully functional
commit 123a39e8c1fb324d7f40c3feee3629fa3e600ca5
Author: henrikb <henrikb@edb9625f-4e0d-4859-8d74-9fd3b1da38cb>
Date:   Fri Jun 5 18:37:04 2009 +0000

    Further rearrangements for passing R CMD check without prefiltering files.

    git-svn-id: svn+ssh://svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.synchro

commit 2f8f4021436d10a2808b80c18b0d0b97d767b118
Author: henrikb <henrikb@edb9625f-4e0d-4859-8d74-9fd3b1da38cb>
Date:   Fri Jun 5 00:59:03 2009 +0000

    Added R.synchronize.

    git-svn-id: svn+ssh://svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.synchro
(END)



> git svn info

Path: .
URL: svn+ssh://svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.synchronize
Repository Root: svn+ssh://svn.r-forge.r-project.org/svnroot/r-dots
Repository UUID: edb9625f-4e0d-4859-8d74-9fd3b1da38cb
Revision: 243
Node Kind: directory
Schedule: normal
Last Changed Author: henrikb
Last Changed Rev: 243
Last Changed Date: 2009-06-05 11:37:04 -0700 (Fri, 05 Jun 2009)



> git config --local --list

core.repositoryformatversion=0
core.filemode=false
core.bare=false
core.logallrefupdates=true
core.symlinks=false
core.ignorecase=true
core.hidedotfiles=dotGitOnly
svn-remote.svn.url=svn+ssh://henrikb@svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.synchronize
svn-remote.svn.fetch=:refs/remotes/git-svn

```
git config --local svn-remote.svn.url svn+ssh://henrikb@svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.methodsS3
git config --local svn-remote.svn.fetch :refs/remotes/git-svn
```

> git svn dcommit

Committing to svn+ssh://henrikb@svn.r-forge.r-project.org/svnroot/r-dots/pkg/R.synchronize ...



[Git]: http://git-scm.com/
[Subversion]: http://subversion.apache.org/
[R-Forge]: http://r-forge.r-project.org/


