## Git via the command line
Git can be operated via either GUI applications or via command line.

We're going to use the command line because this gives you the most powerful and consistent access to git.

Once you get the hang of command line git you can independently explore some GUI applications.


## See if Git is installed
First, let's see if your computer already has Git installed by running the `git` command:

```bash
$ git
```

If you see a readout of a bunch of commands, it means Git is installed and you're good to go; skip down to the section titled *Initial Git Configuration*.

*Git is installed:*

<img src='http://making-the-internet.s3.amazonaws.com/vc-git-success@2x.png' style='max-width:589px; width:100%'>

If, however, you see an error telling you the command `git` is not recognized or not found, you'll need to install git, as described in the next section.

Windows Users: If you're using *Cmder* and you downloaded the full version (as suggested), it comes with Git installed, so the above command should definitely show that Git ready to go.




## Download and install Git
If git is *not* yet installed on your computer, head to the [download page](http://git-scm.com/downloads) and choose your operating system. Download the appropriate program and run through installation.

**Once installation is complete, close and re-open your command line program.**

Now, try the `git` command again. Do you see a bunch of instructions and commands related to Git? Good, you're all set!

If you don't&mdash; are you sure you completely closed and re-opened your command line program?

If you're running an **older operating system** and the latest Git build doesn't work, you may have to try an older version of git. [You can find older builds of git here.](https://code.google.com/p/git-osx-installer/)

__Tips:__

Find out what version of git you're running:
```bash
$ git --version
```

Find out where git is installed on Mac:

```bash
$ which git
```

Find out where git is installed on Windows/Cmder:

```bash
$ where.exe git
```




## Initial Git configuration

Once you've confirmed Git is installed, you need to do some initial setup.

Run the following commands to set a user name and email key to be associated with any commits coming from your computer. Replace the name and email with your own details. The details you enter here does *not* have to match the credentials you use on Github, but it's okay if they do.

```bash
$ git config --global user.name "Sam Seaborn"
$ git config --global user.email sam@gmail.com
```

Next, run the following command to make any Git input color coded (i.e. easier to read):

```bash
$ git config --global color.ui true
```

Next, run this command so that git will ignore filemode changes:

```bash
$ git config --global core.filemode false
```

Finally, we're going to tell git what to use *nano* as your command line text editor. If you have another favorite CL text editor, you can use that instead.

```bash
$ git config --global core.editor nano
```


### Config tips
If you want to see what one of your git configs are set to, use `git config` followed by the particular configuration you want to read. For example, to double check your user.name, run:

```bash
$ git config user.name
```

You can also see *all* your configs with this command:

```bash
$ git config --list
```
