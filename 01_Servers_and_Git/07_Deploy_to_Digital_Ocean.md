## DigitalOcean

So far we have:

1. A local server
2. A remote repository at Github.com

The third and final piece of the puzzle we need to complete your workflow is a live server where you can publish your work online so that it's available for the world to see.

The live server platform we'll use in this course is **DigitalOcean**; DigitalOcean is a straightforward, developer friendly VPS (Virtual Private Server) provider.

To get started, head over to <https://digitalocean.com> and create a new account. Setting up your account will be free; it's once you're set up and start creating *Droplets* that pricing will come into play; more on that later.




## SSH Key: Your computer <-> DigitalOcean

After setting up your account, the first thing you'll want to do is **set up a SSH key**. This will prevent you from having to enter your password every time you communicate with DigitalOcean from command line. It will also prevent DigitalOcean from sending you server passwords via email, which is insecure.

For this course, you can use the same `id_rsa.pub` key you created when you configured Github. Use the `cat` command to open this file, then copy its contents.

Mac:

```bash
$ cat ~/.ssh/id_rsa.pub
```

Windows:

```bash
$ cat %UserProfile%/.ssh/id_rsa.pub
```

Back in DigitalOcean, click the Gear icon (top right) then go to *Settings* and then *Security*.

Within *Security* find the **SSH Keys** section and the blue button to **Add SSH Key**.

Paste in the contents of the `id_rsa.pub` file you copied above, and give the key a descriptive name.

<img src='http://making-the-internet.s3.amazonaws.com/vc-do-ssh-key@2x.png' style='max-width:888px; width:100%' alt='New SSH key on DigitalOcean'>

That's it for the SSH key for now. In a few steps, we'll test to confirm it's working.

<small>
Note: Instead of using your Github key, you could have generated a unique one for DigitalOcean. This latter technique is more secure and suggested for projects beyond the scope of this class.
</small>




## Droplet Pricing

With some initial DigitalOcean configurations behind you, it's time to create your first *Droplet*.

DigitalOcean calls their virtual servers **Droplets**; each Droplet that you spin up is a new virtual server for your personal use.

In this course, you'll use **one single Droplet** to host all of your class projects.

The base plan which costs $5/month should be enough to serve your needs for this course. For the duration of the semester (4 months), your total cost will be $20.

If you want to save some money though, here are a couple options:

1. DigitalOcean has a **refer-a-friend program** which you can read about here: <https://cloud.digitalocean.com/settings/referrals>.
2. Github offers a **Student Developer Pack** which includes a DigitalOcean discount; you can read more about that here: <https://education.github.com/pack>. Note: Your student developer pack might not be approved right away, so if you go this route, don't wait to the last minute.




## New Droplet
From your DigitalOcean dashboard, find the big green button labeled *Create Droplet* to initiate a new Droplet.

On the screen that follows, make your Droplet settings match the following options:

<img src='http://making-the-internet.s3.amazonaws.com/vc-digital-ocean-new-droplet@2x.png' style='max-width:1143px; width:100%' alt='New Droplet at Digital Ocean'>






## Log in to your new server droplet via SSH

Once your Droplet is created, make note if its IP address:

<img src='http://making-the-internet.s3.amazonaws.com/vc-do-new-ip-address@2x.png' style='max-width:1350px; width:100%' alt='New IP address on Droplet'>

From your local command line, SSH into your DigitalOcean droplet using the username `root` and the IP address:

```bash
$ ssh root@your-digital-ocean-ip-address
```

When you first connect, you'll see the following message indicating it's a connection your computer does not recognize. Type `yes` and hit *Enter* to confirm the connection.

```bash
The authenticity of host '104.236.13.65 (104.236.13.65)' can't be established.
RSA key fingerprint is 4a:6e:8b:f2:39:27:ec:05:e1:46:e2:a6:80:e4:e9:d3.
Are you sure you want to continue connecting (yes/no)? yes
```

After you hit enter, if your SSH key is set up properly, you should be logged into your server. You should *not* be prompted for a password (the SSH key is your security credential for connecting, so no password is required).



## Install Git

Your new server does not come with git, so lets install that.

First, while still SSH'd into your server, run this command to update *apt-get*:

```bash
apt-get update
```

apt-get is a package manager for Linux machines (your server is running with Linux as its operating system); it's used to install and manage software.

We're going to use `apt-get` to install git:

```bash
$ sudo apt-get install git
```

Follow any given instructions to complete the git installation. When the install is done, confirm git is installed by checking the version:

```bash
$ git --version
git version 1.9.1
```




## SSH Key: DigitalOcean <-> Github.com

In order to communicate between your DigitalOcean droplet and Github, you need to set up *another* SSH key.

While still SSH'd in to your DigitalOcean droplet, generate a new SSH key:

```bash
$ ssh-keygen -t rsa -C "your@email.com"
```

Press enter:

```bash
$ Enter file in which to save the key (/root/.ssh/id_rsa):
```

Press enter:

```bash
$ Enter passphrase (empty for no passphrase):
```

Press enter:

```bash
$ Enter same passphrase again:
```

You'll now have two new files in `/root/.ssh/`:

+ `id_rsa`
+ `id_rsa.pub`

Run the `cat` command to view the contents of the `id_rsa.pub` file.

```bash
$ cat /root/.ssh/id_rsa.pub
```

Copy the contents of `id_rsa.pub`

Add this new key via [Github.com SSH settings](https://github.com/settings/ssh).

Finally, test that the SSH keys work by running this command:

```bash
$ ssh -T git@github.com
```

If all went well, you should see a message like this:
```
Hi susanBuck! You've successfully authenticated, but GitHub does not provide shell access.
```




## Explore your new droplet
Set up of your new server is complete, so lets take a look around and see what files you have to start with.

First, change into your document root which is located at `/var/www/html/`:

```bash
$ cd /var/www/html
$ ls
```

You should see two files:

* `index.html`
* `info.php`

If you access your site in the browser (via the IP address DigitalOcean gives you), you should see the default contents of `index.html` which looks like this:

<img src='http://making-the-internet.s3.amazonaws.com/vc-digital-ocean-default-index@2x.png' class='' style='max-width:625px; width:100%' alt=''>



## Clone a repository

Now that you've set up your server and established the document root is at `/var/www/html/`, your next step is to clone your hello-world repository there.

In the `/var/www/html/` directory on DigitalOcean, run the following command, replacing the git URL for your hello-world repository:

```bash
$ cd /var/www/html/
$ git clone git@github.com:username/hello-world.git
```

Your directory structure on DigitalOcean should now look something like this:

* `hello-world/`
	* `index.html`
	* `README.md`
* `index.html`
* `info.php`

When you visit your DigitalOcean IP and tack on the `hello-world` subdirectory to the URL, you should see your `hello-world` project, just as it looked when you ran it locally:

<img src='http://making-the-internet.s3.amazonaws.com/version-control-hello-world-on-digital-ocean@2x.png' class='' style='max-width:580px; width:100%' alt=''>




## Deployment

Deployment is the process of moving changes from your local environment to production.

Once your repository is cloned from Github.com to DigitalOcean, the steps for deploying changes looks like this:

1. SSH in to your Droplet
2. Change directories into your project
3. Run `git pull origin master` to sync any new changes


So a typical workflow might look like this:

1. Sit down for the afternoon to work on your project. Make lots of changes to your local files, testing the changes on your local server.
2. After a couple hours, you're done for the day, so you want to check in your latest changes and update your live server.
3. You stage, commit and push all your local changes to your remote repository at Github.
4. Finally, you SSH into your Droplet and pull down the latest changes.




## Tips/Notes

Apache error and configuration files on DigitalOcean:

+ Apache error log: `/var/log/apache2/error.log`
+ Apache configuration: `/etc/apache2/httpd.conf`

SSH Keys between your computer and DigitalOcean will only work on Droplets that were created *after* you set up the key. To set up a key for an existing Droplet, navigate into your computer's .ssh directory and run this command:

```bash
cat id_rsa.pub | ssh root@[your.ip.address.here] "cat >> ~/.ssh/authorized_keys"
```

## Reference

* [Transferring Digital Ocean snapshots between accounts](https://www.digitalocean.com/company/blog/easily-transfer-snapshots-between-accounts/)

* [How To Create Your First DigitalOcean Droplet Virtual Server](https://www.digitalocean.com/community/tutorials/how-to-create-your-first-digitalocean-droplet-virtual-server)

* [How To Use SSH Keys with DigitalOcean Droplets](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-keys-with-digitalocean-droplets)
