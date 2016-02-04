## Beginner workflows

Below are some common, less than ideal workflows used by beginner developers.
Each workflow hinges heavily on the use of FTP to edit files on a server.

__The Live Audience Workflow__

1. FTP into your live server.
2. Make changes to files.

__The Copy Wrangling Workflow__

1. FTP into your live server.
2. Create &ldquo;development&rdquo; copies of the files you want to work on.
3. Make changes to the copy.
4. When done with edits, replace the original with the edited copy.

__The Local to Live Sync Workflow__

1. Run a local server, maintain a full clone of your site.
2. Make changes on the local server.
3. When done FTP changes to the live site, or use something like [*rsync*](https://en.wikipedia.org/wiki/Rsync) to keep the local and live files in sync.




## Introducing version control

We're going to modernize and streamline the above workflows by implementing version control.

Dual purpose:
+ Track code changes over time
+ Mechanism for transferring code to/from servers (replaces FTP)

There are different implementations of version control software; in this course we'll use __Git__:

>> Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency. Git is easy to learn and has a tiny footprint with lightning fast performance.

### Benefits of Version Control

[Github.com preview via https://github.com/laravel/laravel](https://github.com/laravel/laravel)

__Workflow enhancement__
+ Because Git keeps a running history of your changes, you can quickly revert to a previous state of your code if you mess something up. Think cmd+z for programming.
+ Ability to create *branches* of your code for working on new features, starting on different versions, etc.

__Teamwork__
+ Helps developers from overwriting each other's work.
+ Accountability&mdash; every change shows who authored it.
+ Having a history of changes can solve a lot of mysteries.

__Backup__
+ Storing your code in a Git repository serves as an ongoing backup of your code.

__Mechanism for transferring code to/from servers__


## How it works
<img src='http://making-the-internet.s3.amazonaws.com/vc-local-to-git-and-live-server-alternative@2x.png' style='max-width:540px;' alt='Version Control Workflow'>

Working with git, you'll end up with at least three copies of your code base.

__The first copy will be on your computer.__ For this you'll need to set up a local server so you can run the code just like you do on your live server (you'll need Apache, PHP, MySQL). Only you will be able to see this copy of the code, never live users.

Each collaborator that is working with you on a project will also have a copy of the repository on their computers.  

__The second copy will be the main repository__, and, for this course, it will be stored at Github.com.

__The third copy of your site is the code that runs on your live server__; this is what the rest of the world sees.




## Learn more

This is not a git course. We're covering just enough git for our purposes.

In the interest of time, we are omitting certain git concepts that are common in real-world settings such as branching, forking, staging servers, etc.

To continue your education on git, you can refer to the following resources:

+ [Scott Chacon's Pro Git Book](http://git-scm.com/book)
+ [Chacon's Git Videos](http://git-scm.com/videos) (about 25 min total)




## Summary

1. Local server
    + MAMP or XAMPP
    + Run and test web applications locally
2. Live server
    + Via DigitalOcean
    + Where you'll deploy your web applications so the rest of the world can see your work
3. Git/Version Control
    + Manage changes to code over time
    + Mechanism for transferring code to and from servers
4. Domains
    + Via Namecheap
    + Configuring




## Challenges

The challenge ahead: Lots of different variables...

+ Software (versions, OS's, code editors, etc.)
+ Servers
+ Settings

Expect troubleshooting; expect things to not work on the first try; expect having to seek out answers.

Good news: set it and (mostly) forget it.
