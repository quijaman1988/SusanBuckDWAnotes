## Domain setup
Right now you can access your DigitalOcean droplet by typing its IP address into your browser. This works but is hard to remember, and it'd also be nice to run several web applications on your one Droplet, each with a unique subdomain.

To set this up, we're going to configure a domain and subdomains for use with DigitalOcean.

If you have an existing domain you'd like to use, that's fine. If not, you can create a new domain name via a service like **[Namecheap](http://www.namecheap.com/?aff=61057)**. As of this writing, Namecheap is offering free domain names for students (with a `.edu` email address) via <https://nc.me/>.

After your create your domain, find your **DNS settings** within your domain companies' control panel. In Namecheap, these settings are found by finding the domain you'd like to configure in your **Domain List** and then clicking **Manage**. Then, find the tab for **Advanced DNS**

In your DNS settings, you'll set both your `@` and `www` hostname to your DigitalOcean IP address.

Also, while you're there, add a new *A Record* with the *host* `hello-world` which also points to the same IP. This will create the subdomain `http://hello-world.yourdomain.com`.

<img src='http://making-the-internet.s3.amazonaws.com/version-control-namecheap-dns@2x.png' style='max-width:1210px; width:100%' alt='Namecheap DNS'>

When you're done, click **Save all Changes**.

Give the above settings a few minutes to take effect, then test out your domain. You should see the same results you saw above when you tested your IP address, but this time it's loaded via your domain name:

<img src='http://making-the-internet.s3.amazonaws.com/vc-namecheap-domain-first-working@2x.png' style='max-width:958px; width:100%' alt='First domain working'>

### DNS Cache
If your domain is not working yet, it could because your DNS settings are being cached.

To get around this, try the following steps:

1. Clear your browser cache.
2. Clear your DNS cache
    + [Mac instructions](https://support.apple.com/en-us/HT202516)
    + Windows/Cmder run this command: `ipconfig /flushdns`.

Try again. Still not loading?

3. Try a different browser.
4. Try accessing your URL via [this proxy](http://www.megaproxy.com/freesurf/). Does it load there? If it loads there, it just means your computer is still caching old settings.


## Virtual Host / Subdomain setup
Your primary domain is working, now lets get a subdomain working (`http://helloworld.domain.com`).

In the above step, you already set up your DNS for the subdomain, making it so that traffic hitting `http://helloworld.domain.com` will point to your IP address.

Now we need to give the server instructions on what to do with this traffic.

This is done via a Virtual Host configuration; **Virtual Hosts allow Apache (your web server software) to be configured for multiple sites that have different configurations**.

To start, copy the following code into a new text editor window (your code editor, Notepad, Text Editor, whatever):

```bash
<VirtualHost *:80>
  ServerName hello-world.dwa15-practice.biz
  DocumentRoot "/var/www/html/hello-world"
  <Directory "/var/www/html/hello-world">
    AllowOverride All
  </Directory>
</VirtualHost>
```

In plain english, this VirtualBlock block code says: *when traffic comes in via `helloworld.dwa15-practice.biz`, point to the `/var/www/html/hello-world` directory*.

Edit your VirtualHost block so that the the `ServerName` value matches *your* subdomain. In this example, it's set to `hello-world.dwa15-practice.biz` but you'll need to change it to match your domain.

Also change the `DocumentRoot` and `Directory` path to match the path to your hello-world directory on DigitalOcean, if it's different from our example.

With your edits in place, we'll now show you where to put this code on your server...

SSH into your DigitalOcean server.

Open `/etc/apache2/sites-enabled/000-default.conf` in nano:

```bash
$ nano /etc/apache2/sites-enabled/000-default.conf
```

This file is where all your VirtualHost specifications live.

In nano, at the *bottom* of `000-default.conf`, paste in your edited VirtualHost block.

<img src='http://making-the-internet.s3.amazonaws.com/vc-pasting-virtual-host-block@2x.png' style='max-width:75%; width:877px'>

Tip for Cmder users: If you try and paste multiple lines into Cmder using keyboard shortcuts, it will paste just the first line. To get it to paste *all* the lines, right click the top bar in Cmder and choose `Edit` -> `Paste` ([screencast demo](http://screencast.com/t/u43zTSEx4GKl)).



When you're done editing the new VirtualHost block, save your changes to `000-default.conf` (`ctrl` + `x`, then `y`, then *Enter*).

(If for some reason you make a mistake in `000-default.conf`, [here's a copy of the original](https://gist.github.com/susanBuck/790ea5a0d1ad7d02e586).)

To make your VirtualHost changes take effect, restart Apache with this command:

```bash
$ service apache2 restart
```

Once the restart is complete, test out your subdomain `helloworld.yourdomain.com`.

<img src='http://making-the-internet.s3.amazonaws.com/version-control-subdomain-good@2x.png' class='' style='max-width:337px; width:75%' alt=''>


## Recap
Keep these notes handy for when you set up projects, because each project should have its own subdomain.

To recap, here were the steps we took to set up a new subdomain:

1. Open your DNS settings in Namecheap (or whoever you domain provider is) and point your subdomain to your server's IP address.
2. Add a *new* VirtualHost `/etc/apache2/sites-enabled/000-default.conf`. Each Subdomain will have its own VirtualHost block.
3. Restart Apache


## Tips / Notes
Find out the IP address of a domain:

```bash
$ ping domain.com
```

Find out the nameserver of a domain:

```bash
$ dig +short NS domain.com
```
