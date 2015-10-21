=====
 dim
=====

---------------------------
drm-intel maintainer script
---------------------------

:Author: Daniel Vetter <daniel.vetter@ffwll.ch>
:Author: Jani Nikula <jani.nikula@intel.com>
:Date: 2014-05-15
:Copyright: 2012-2015 Intel Corporation
:Manual section: 1
:Manual group: maintainer tools

SYNOPSIS
========

**dim** [*option* ...] *command*

DESCRIPTION
===========

drm-intel maintainer script.

Branch Model
------------

The dim flow has 3 main development branches:

- drm-intel-next-queued for feature work. This branch gets regularly pushed to
  drm-intel-next and tagged and then sent on to the upstream drm-next branch
  using *update-next*.  The cut-off for the last pull request to drm-next is
  around -rc5. After that point patches in drm-intel-next-queued already aim at
  the next but one and not the next merge window.

- drm-intel-next-fixes is to collect fixes for the current merge window after
  the -rc5 feature cut-off in drm-next.

- drm-intel-fixes is for fixes for the current -rc1 kernel. This is separate
  from drm-intel-next-fixes since the merge window feature cutoff at -rc5 is a
  few weeks ahead of the final release of the previous kernel.

  There's separate tracking branches for inclusion into linux-next to make sure
  that the feature work in drm-intel-next-queued aimed for the next but one
  merge window doesn't cause unecassary conflicts in linux-next - in that case
  only drm-intel-next-fixes is included in linux-next. The switchover happens
  when drm-intel-fixes has caught up (in git terms: drm-intel-next-fixes is
  direct ancestor of drm-intel-fixes). Therefore only roll drm-intel-fixes
  forward once -rc1 is released

In addition there's 2 permanent topic branches:

- topic/drm-misc carries core drm patches aimed at the next merge window.

- topic/drm-fixes carries core drm fixes for the current -rc kernels.

Additional topic branches are created as needed using *create-branch* and
*remove-branch*.

OPTIONS
=======

-f		Ignore some error conditions.
-d		Dry run.
-i		Interactive mode.

COMMANDS
========

setup *prefix*
--------------
Setup git maintainer branches in the given prefix.

nightly-forget
--------------
Forget git rerere solutions for nightly merges in case they
contain a bogus merge resolution.

update-branches
---------------
Updates all maintainer branches. Only needs to be run to synchronize branches
between different machines (or maintainers fwiw). As long a given branch is
always maintained from the same machine, even if different branches are
maintained on different machines (by different maintainers), there's no need to
run this command.

rebuild-nightly
---------------
Rebuilds the nightly branch. Useful when ad-hoc trees are
included in -nightly.

cat-to-fixup
------------

Pipes stdin into the fixup patch file for the current -nightly merge.

push-branch branch [*git push arguments*]
-----------------------------------------

push-fixes|pf [*git push arguments*]
------------------------------------

push-next-fixes|pnf [*git push arguments*]
------------------------------------------

push-queued|pq [*git push arguments*]
-------------------------------------

Updates the named branch, or drm-intel-fixes, drm-intel-next-fixes or the
drm-intel-next-queued branch respectively. Complains if that's not the current
branch, assuming that patches got merged to the wrong branch. After pushing also
updates linux-next and drm-intel-nightly branches.

checkout|co *branch*
--------------------
Checks out the named branch.

conq
----

cof
---

conf
----
Checks out the drm-intel-fixes branch, dinf or dinq respectively for merging
patches.

apply-branch|ab|sob branch [*git am arguments*]
-----------------------------------------------
Applys a patch to the given branch, complaining if it is not
checked out yet.

apply-fixes|af [*git am arguments*]
-----------------------------------

apply-next-fixes|anf [*git am arguments*]
-----------------------------------------

apply-queued|aq [*git am arguments*]
------------------------------------
Applies a patch to -fixes, -next-fixes or -next-queued respectively, complains
if it's not the right branch. Additional arguments are passed to git am.

magic-patch|mp [-a]
-------------------
Apply a patch using patch and then wiggle in any conflicts. When passing the
option -a automatically changes the working directory into the git repository
used by the last previous branch-specific command. This is useful with the
per-branch workdir model.

magic-rebase-resolve|mrr
------------------------
Tries to resolve a rebase conflict by first resetting the tree
and the using the magic patch tool. Then builds the tree, adds
any changes with git add -u and continues the rebase.

cd
--
Changes the working directory into the git repository used by the last previous
branch-specific command. This is implemented as a bash-function to make it
useful in interactive shells and scripts. Only available when the bash
completion is sourced.

apply-resolved|ar
-----------------
Compile-test the current tree and if successfully resolve a
confilicted git am. Also runs the patch checker afterwards.

apply-igt|ai
------------
Apply a patch to the i-g-t repository.

tc *commit-ish*
---------------
Prints the linux kernel tag which contains the supplied commit-ish

check-patch|cp [*commit-ish* [.. *commit-ish*]]
-----------------------------------------------
Runs the given commit range commit-ish..commit-ish through the check tools. If
no commit-ish is passed, defaults to HEAD^..HEAD. If one commit-ish is passed
instead of a range, the range commit-ish..HEAD is used.

cherry-pick *commit-ish* [*git cherry-pick arguments*]
------------------------------------------------------

Improved git cherry-pick version which also scans drm-intel-nightly for any
mentions of the cherry-picked commit. Should be used when cherry-pick from -next
to -fixes to make sure all fixups are picked, too. In dry-run mode/-d only the
patch list is generated.

pull-request *branch* *upstream*
--------------------------------
Fetch the *upstream* remote to make sure it's up-to-date, create and push a date
based tag for the *branch*, generate a pull request template with the specified
*upstream*, and finally start \$DIM_MUA with the template with subject and
recipients already set.

Since the tag for the *branch* is date based, the pull request can be
regenerated with the same commands if something goes wrong.

pull-request-fixes [*upstream*]
-------------------------------
This is a special case of **pull-request**, with *drm-intel-fixes* as the
branch and *origin/master* as the default upstream.

pull-request-next-fixes [*upstream*]
------------------------------------
This is a special case of **pull-request**, with *drm-intel-next-fixes* as
the branch and *\$DRM_UPSTREAM/drm-next* as the default upstream.

pull-request-next [*upstream*]
------------------------------
This is similar to **pull-request**, but for feature pull requests, with
*drm-intel-next* as the branch and *\$DRM_UPSTREAM/drm-next* as the default
upstream.

The difference to **pull-request** is that this command does not generate a
tag; this must have been done previously using **update-next**. This also means
that the pull request can be regenerated with the same commands if something
goes wrong.

update-next
-----------
Pushes out the latest dinq to drm-intel-next and tags it. Also
pushes out the latest nightly to drm-intel-testing. For an
overview a gitk view of the currently unmerged feature pile is
opened.

Also checks that the drm-intel-fixes|-next-queued are fully
merged into -nightly to avoid operator error.

tag-next
--------

Pushes a new tag for the current drm-intel-next state after checking that the
remote is up-to-date. Useful if drm-intel-next has been changed since the last
run of the update-next command (e.g. to apply a hotfix before sending out the
pull request).

checker
-------
Run sparse on the kernel.

create-branch *branch* [*commit-ish*]
-------------------------------------

Create a new topic branch with the given name. Note that topic/ is not
automatically prepended. The branch starts at HEAD or the given commit-ish.

remove-branch *branch*
----------------------

Remove the given topic branch.

create-workdir (*branch* | all)
-------------------------------

Create a separate workdir for the branch with the given name (requires that
git-new-workdir from git-core contrib is installed), or for all branches if
"all" is given.

for-each-workdir|fw *command*
-----------------------------

Run the given command in all active workdirs including the main repository under
\$DIM_DRM_INTEL.

help
----
Show this help. Install **rst2man(1)** for best results.

ENVIRONMENT
===========

DIM_CONFIG
----------
Path to the dim configuration file, \$HOME/.dimrc by default, which is sourced
if it exists. It can be used to set other environment variables to control dim.

DIM_PREFIX
----------
Path prefix for kernel repositories.

DIM_DRM_INTEL
-------------
The main maintainer repository under \$DIM_PREFIX.

DIM_DRM_INTEL_REMOTE
--------------------
Name of the $drm_intel_ssh remote within \$DIM_DRM_INTEL.

DIM_DRM_UPSTREAM_REMOTE
-----------------------
Name of the $drm_upstream_git remote within \$DIM_DRM_INTEL.

DIM_MUA
-------
Mail user agent. Must support the following subset of **mutt(1)** command line
options: \$DIM_MUA [-s subject] [-i file] [-c cc-addr] to-addr [...]

DIM_TEMPLATE_HELLO
------------------
Path to a file containing a greeting template for pull request mails.

DIM_TEMPLATE_SIGNATURE
----------------------
Path to a file containing a signature template for pull request mails.

CONTRIBUTING
============

Submit patches for any of the maintainer tools to
intel-gfx@lists.freedesktop.org with [maintainer-tools PATCH] prefix. Use

$ git format-patch --subject-prefix="maintainer-tools PATCH"

for that. Push them once you have
an ack from maintainers (Jani/Daniel).
