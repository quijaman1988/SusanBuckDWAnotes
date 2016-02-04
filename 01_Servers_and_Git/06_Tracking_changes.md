With your local repository set up and connected your remote repository at Github.com, you're ready to start tracking changes to your project.

Everything we have done to this point has been setup steps that you only do once when starting a new project. Everything that follows reflects the kind of git work you'll do on a day-to-day basis.

Within your `hello-world` directory, run `git status` to see what changes git has tracked:

```bash
$ git status
```

At this point, you should have one untracked file&mdash; the `README.md` file.

<img src='http://making-the-internet.s3.amazonaws.com/vc-git-status@2x.png' class='' style='max-width:757px; width:100%' alt='Git status'>

Stage (aka `add`) this new file with the following command:

```
$ git add README.md
```

Now, try `git status` again, and you should see you have a file waiting to be committed:

```
$ git status
```

<img src='http://making-the-internet.s3.amazonaws.com/vc-git-add@2x.png' style='max-width:757px; width:100%' alt='Git add'>

Next, commit this change using the `commit` command, plus the `-m` flag which indicates we'll also be including a commit message:

```bash
$ git commit -m "Add README file."
```

<img src='http://making-the-internet.s3.amazonaws.com/vc-git-commit@2x.png' class='' style='max-width:757px; width:100%' alt='Git add README'>

Finally, push this change to Github.com.

```bash
$ git push origin master
```

<img src='http://making-the-internet.s3.amazonaws.com/vc-git-push@2x.png' style='max-width:757px; width:75%' alt='Git push'>

To double check your commit made it to Github.com, look for your new `README.md` file on Github.com:

<img src='http://making-the-internet.s3.amazonaws.com/vc-first-commit-in-github@2x.png' style='max-width:849px; width:100%' alt=''>




## Practice

Edit your `README.md` file adding the date to the top of the file.

Also, add a new file called `index.php` and include this code:

```html
<h1>Hello World!</h1>
```

Now lets track these changes in Github (one edit of an existing file, one new file).

You can stage the files one at a time:

```bash
$ git add README.md
$ git add index.php
```

Or you can use this command to stage any changed or new files:

```bash
$ git add --all
```

Once your changes are staged, commit:

```bash
$ git commit -m "Update README file; add index file."
```

And finally, push your commit:

```bash
$ git push origin master
```

Check your repository in github.com to make sure the commit is there.



## Recap

1. Make changes on a local copy of your code
2. Save changes
3. Test your work via your local server, is everything good?

Steps 1,2,3 repeat over and over until the feature or change you're working on is ready.
When everything is set and you're ready to go live...

4. __Stage__ individual files that have changed
5. __Commit__ everything you staged
6. __Push__ your commits to Github




## View hello-world on your local server

Before deploying this hello-world example to a liver server in the next steps, make sure you can view it on your local server:

1. Change your local server's document root to point to `/path/to/your/htdocs/hello-world/`
2. Restart your local server
3. Visit http://localhost and make sure you see the `index.php` file




## Tips

* If you get a message about `src refspec master does not match any.` after trying to push&mdash; it means you have not yet done a commit.

* `git status` is a command you should take advantage of frequently; it will give you a sense of what's going on your repository: what has changed, if any files are untracked, etc.

* `git diff` is another useful command to show what changes have been made. You can see all changes with just `git diff` or see just the changes on a particular file with `git diff <file>`.

* If you want to undo changes made to a file that haven't yet been staged, you can use the command `git checkout <file>`.

* If you stage a file but decide you want to unstage it, run this command: `git reset HEAD <file>`. This won't revert any changes to that file (that's what `checkout` does), it will just remove it from the stage.

* If you delete a file from a project, you'll use the `git rm <file>` command instead of `git add <file>`.
