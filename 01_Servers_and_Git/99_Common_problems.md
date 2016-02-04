The following document is a work in progress, updated to reflect common problems and solutions discovered by students.




## Problem: Local server: Permissions Issue / You're getting a error 500 / "Internal Server Error"

Run this command on the document root:

```bash
chmod -R g-w /path/to/document/root
```




## Problem: Mac users: When you try to use any git command, you're told git is not installed

Symptoms look like this:

```bash
$ git
-bash: git: command not found
```

First thing to try: Double check you completely closed and re-opened Terminal window after installing Git.

If that doesn't fix it...

This issue has only turned up on Macs so far, so the following instructions apply to Mac users:

See if the files for Git exist on your machine. The common git installation location on a Mac is in the `/usr/local/dir` directory. See if you can move into that directory:

```bash
cd /usr/local/git/
```

Either it will work, or it will tell you that directory does not exist:

```bash
-bash: cd: /usr/local/git/: No such file or directory
```

If it does not exist it's possible you may have missed the step of installing git. Confirm you visited [the git download page](http://git-scm.com/downloads) and downloaded Git.

If it does exist, the problem could be that your computer doesn't know the path to Git. Let's fix that.

Run this command to edit your `~/.bashrc` file using nano:

```bash
sudo nano ~/.bashrc
```

At the end of the `~/.bashrc` file, add this line:

```bash
export PATH=$PATH:/usr/local/git/bin/
```

Save and close your file, then run this command to refresh the `~/.bashrc` file:

```bash
source ~/.bashrc
```

Now, try running a git command again.

<small>
Source: ([mountain-lion-git-command-not-found-2-min-fix](http://dwainm.wordpress.com/2012/12/19/mountain-lion-git-command-not-found-2-min-fix/))
</small>




## You're told your branch is ahead by x commits

Example:

```bash
# On branch master
# Your branch is ahead of 'origin/master' by 5 commits.
#
nothing to commit (working directory clean)
```

This isn't indicative of any major problem but can be cleared up with the following command:

```bash
$ git fetch origin
```

Fetch seems to update the local representation of the remote branch, which doesn't necessarily happen when you do the git pulls.

Note, this only happens when you do `git pull origin master` not `git pull`

<small>
Reference: [StackOverflow 1](http://stackoverflow.com/questions/14835515/local-git-clone-is-ahead-of-origin-even-though-no-local-changes-were-made) [StackOverflow 2](http://stackoverflow.com/questions/2432579/git-your-branch-is-ahead-by-x-commits)
</small>
