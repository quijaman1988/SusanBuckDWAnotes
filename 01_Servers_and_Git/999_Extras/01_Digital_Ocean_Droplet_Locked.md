# DigitalOcean Droplet Locked / DDoS

Below are two ominous messages we've occasionally seen students face in regards to their DigitalOcean Droplets:

<img src='http://making-the-internet.s3.amazonaws.com/vc-message-ddos-on-digital-ocean@2x.png' style='max-width:100%; width:1000px' alt='Message notifying you of a DDoS attack on DigitalOcean'>

When this happens, it means DigitalOcean has locked down your droplet because, as the message states, it detected a &ldquo;large flood of traffic from one or more of your servers.&rdquo;

When this happens, it likely means a malicious hacker gained access to your Droplet and used it to perform DDoS attacks, which caused that large flood of outgoing traffic.

DDoS stands for *Distributed Denial of Service* and is a attack in which hackers use &ldquo;innocent&rdquo; servers to do things like send spam mail, or try to overwhelm some other target server.


## What happened
So how did this happen? Well, it can be hard to track down exactly where the problem started.

One possible cause is a hacker gained access to your root password, allowing them to access your server, create themselves an account, and then install their DDoS program to start sending traffic out.

Because of this, it's always preferable to *not* use a password when logging into your server, which is one of the reasons we use the more secure alternative of SSH keys.

If you're not using a root password, another possible cause could have come from outdated or insecure software on your server.

DDoS is one of the various plagues web developers and system administrators face in the real world, across all hosting providers.

In these notes we're going to describe what to do to get back on track if your DigitalOcean Droplet has been suspended.


## What to do next
If you read the support ticket DigitalOcean creates for you after they shut you down, you'll see they link to this article: [How To Recover from a Compromised Droplet Sending an Outgoing Flood or DDoS](https://www.digitalocean.com/community/tutorials/how-to-recover-from-a-compromised-droplet-sending-an-outgoing-flood-or-ddos).

The long and short of their plan for recovery involves creating a new Droplet, and &ldquo;hardening&rdquo; that Droplet to try and prevent further DDoS attacks from happening. The article also outlines how to recover your data from your compromised Droplet, so that you can move it to your new Droplet.

Following in DigitalOcean's lead, we're going to outline similar steps, but under the context of our class setup.

Fortunately, in our cases, recovery is simple for the following reasons:

1. All our code is backed up via git.
2. All our databases are built with Laravel Migrations, making the structure easy to create.
3. We're not dealing with real-world applications, so any database data is still disposable.

That being said, here's what you should do to get back on track...


## Create a new droplet
Start by creating a new Droplet.

As a reminder on what settings you need for the Droplet, [here's the flowchart](http://making-the-internet.s3.amazonaws.com/vc-digital-ocean-new-droplet.png) taken from the notes, [Deploy to Digital Ocean](https://github.com/susanBuck/dwa15-fall2015-notes/blob/master/01_Servers_and_Git/10_Deploy_to_Digital_Ocean.md#new-droplet).

Don't forget to choose your existing SSH key stored on DigitalOcean when creating this new Droplet.

Once your Droplet is created, note the IP address, and then log in via SSH to confirm the SSH key connection is working:

```bash
$ ssh root@your.new.ip.address
```

__If you got in, your next steps is to proceed to read the instructions on [Increasing server security](https://github.com/susanBuck/dwa15-fall2015-notes/blob/master/01_Servers_and_Git/99_Extras/Increasing_server_security.md) which will take you through the procedures of creating a more secure server setup.__

After you finish those instructions, come back here for the next steps.


## Rebuilding your data
With your new Droplet created, and secured, you want to re-clone your projects from Git.

First, install git:

```bash
$ apt-get git
```

Next, set up the SSH key between DigitalOcean and Github. You've done this before, but it was on the compromised Droplet. Now, you're going to do it again for the new, healthy Droplet.

While SSH'd into your new Droplet, run this command to generate a new SSH key pair:

```bash
$ ssh-keygen -t rsa -C "your@email.com"
```

Get the public key:

```bash
$ cat ~/.ssh/id_rsa.pub
```

Outputs a public key that looks something like this:

```bash
ssh-rsa [LONG STRING OF RANDOM CHARACTERS] your@email.com
```

Copy the public key, and go to Github.com -> *Settings* -> *SSH Keys* -> *Add SSH Key*. Paste in the key, give it a name and save it.

Confirm the SSH key works by running this command from your Droplet:

```bash
ssh -T git@github.com
```


## Clone your repositories/projects
To load up your existing project code on your new Droplet, first move into your web accessible directory:

```bash
$ cd /var/www/html
```

Then, clone each project you've created so far. For example, to clone p1:
```bash
$ git clone git@github.com:yourUsername/p1.git
```

Reminder: You can get the SSH clone url for any repository by looking at the bottom right on your repository homepage [screenshot](http://making-the-internet.s3.amazonaws.com/vc-ssh-clone-url.png).

Simply cloning your projects is enough to get p1 and p2 working again.

For p3 and p4 (if you're there yet), you'll need to `cd` into those directories and run through the following steps:

1. Recreate your database environment config file.
2. Run `composer update`
3. Open write access on the storage directory: `chmod -R 777 app/storage`
5. Run `php artisan migrate` to build your database structure


## Re-setup your Subdomains
Finally, lets get your subdomains back in working order. To do this, we'll recreate the steps taken in the notes [Live domain setup](https://github.com/susanBuck/dwa15-fall2015-notes/blob/master/01_Servers_and_Git/11_Live_domain_setup.md).

The following is an &ldquo;express&rdquo; run down of the procedures:

__DNS__
Visit Namecheap (or whoever your domain provider is) and update your DNS settings to point your main domain and any subdomains to your new Droplet ip address.

<img src='http://making-the-internet.s3.amazonaws.com/vc-updating-dns-after-droplet-was-locked@2x.png' style='width:100%; max-width:1000px' alt='Point your domains to your new IP'>

__VirtualHost__
Then, set up a new VirtualHost block for each subdomain in `000-default.conf`:

```
$ sudo nano /etc/apache2/sites-enabled/000-default.conf
```

And one new VirtualHost block for *each* project at the end of the file.

For example, here's the VirtualHost block for p1:

```bash
<VirtualHost *:80>
  ServerName p1.yourdomain.com
  DocumentRoot "/var/www/html/p1"
  <Directory "/var/www/html/p1">
    AllowOverride All
  </Directory>
</VirtualHost>
```

Save and exit.

Restart apache so your new VirtualHost settings will take effect:

```bash
$ sudo service apache2 restart
```

Test out your subdomains. If they're not working, yet, revisit the section on [DNS cache](https://github.com/susanBuck/dwa15-fall2015-notes/blob/master/01_Servers_and_Git/11_Live_domain_setup.md#dns-cache) in the original notes.


## Conclusion
Having your server compromised in any way is never fun. Resolving a compromise is often frustrating and time consuming.

Unfortunately, though, preventing and dealing with hacking is par for the course when it comes to web applications, and no server is foolproof when it comes to attacks.

The best we can do to mitigate risks is put proper security measures in place and make sure all software is kept up to date.
