# Distributed Workflows

Suppose that Alice has started a new project with a git repository in
/home/alice/project, and that Bob, who has a home directory on the
same machine, wants to contribute.

Bob begins with:

    git clone /home/alice/project myrepo

This creates a new directory "myrepo" containing a clone of Alice's
repository.  The clone is on an equal footing with the original
project, possessing its own copy of the original project's history.

Bob then makes some changes and commits them:

    (edit files)
    git commit -a
    (repeat as necessary)

When he's ready, he tells Alice to pull changes from the repository
at /home/bob/myrepo.  She does this with:

    cd /home/alice/project
    git pull /home/bob/myrepo master

This merges the changes from Bob's "master" branch into Alice's
current branch.  If Alice has made her own changes in the meantime,
then she may need to manually fix any conflicts.  (Note that the
"master" argument in the above command is actually unnecessary, as it
is the default.)

The "pull" command thus performs two operations: it fetches changes
from a remote branch, then merges them into the current branch.

When you are working in a small closely knit group, it is not
unusual to interact with the same repository over and over
again.  By defining 'remote' repository shorthand, you can make
it easier:

    git remote add bob /home/bob/myrepo

With this, Alice can perform the first operation alone using the
"git fetch" command without merging them with her own branch,
using:

    git fetch bob

Unlike the longhand form, when Alice fetches from Bob using a
remote repository shorthand set up with `git remote`, what was
fetched is stored in a remote tracking branch, in this case
`bob/master`.  So after this:

    git log -p master..bob/master

shows a list of all the changes that Bob made since he branched from
Alice's master branch.

After examining those changes, Alice
could merge the changes into her master branch:

    git merge bob/master

This `merge` can also be done by 'pulling from her own remote
tracking branch', like this:

    git pull . remotes/bob/master

Note that git pull always merges into the current branch,
regardless of what else is given on the command line.

Later, Bob can update his repo with Alice's latest changes using

    git pull

Note that he doesn't need to give the path to Alice's repository;
when Bob cloned Alice's repository, git stored the location of her
repository in the repository configuration, and that location is
used for pulls:

    $ git config --get remote.origin.url
    /home/alice/project

(The complete configuration created by git-clone is visible using
"git config -l", and the [git config](https://git-scm.com/docs/git-config) man
page explains the meaning of each option.)

Git also keeps a pristine copy of Alice's master branch under the
name "origin/master":

    $ git branch -r
      origin/master

If Bob later decides to work from a different host, he can still
perform clones and pulls using the ssh protocol:

    git clone alice.org:/home/alice/project myrepo

Alternatively, git has a native protocol, or can use rsync or http;
see [git pull](https://git-scm.com/docs/git-pull) for details.

## Pushing changes to a public repository

To make your changes accessible to others, you will need to update the public
repository with the latest changes created in your private repository through
[git push](https://git-scm.com/docs/git-push).

As with git-fetch, you may also set up configuration options to
save typing; so, for example, after

    $ cat >>.git/config <<EOF
    [remote "public-repo"]
        url = ssh://yourserver.com/~you/proj.git
    EOF

you should be able to perform the above push with just

    git push public-repo master

See the explanations of the remote.*name*.url, branch.*name*.remote,
and remote.*name*.push options in [git config](https://git-scm.com/docs/git-config)
for details.

## What to do when a push fails

If a push would not result in a fast forward of the
remote branch, then it will fail with an error like:

    error: remote 'refs/heads/master' is not an ancestor of
    local  'refs/heads/master'.
    Maybe you are not up-to-date and need to pull first?
    error: failed to push to 'ssh://yourserver.com/~you/proj.git'

This can happen, for example, if you:

    - use `git-reset --hard` to remove already-published commits, or
    - use `git-commit --amend` to replace already-published commits, or
    - use `git-rebase` to rebase any already-published commits.

You may force git-push to perform the update anyway by preceding the
branch name with a plus sign:

    git push ssh://yourserver.com/~you/proj.git +master

Normally whenever a branch head in a public repository is modified, it
is modified to point to a descendant of the commit that it pointed to
before.  By forcing a push in this situation, you break that convention.

Nevertheless, this is a common practice for people that need a simple
way to publish a work-in-progress patch series, and it is an acceptable
compromise as long as you warn other developers that this is how you
intend to manage the branch.

It's also possible for a push to fail in this way when other people have
the right to push to the same repository.  In that case, the correct
solution is to retry the push after first updating your work: either by a
pull, or by a fetch followed by a rebase.
