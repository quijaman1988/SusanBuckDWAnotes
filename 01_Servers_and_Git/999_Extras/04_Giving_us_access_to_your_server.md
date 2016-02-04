In order for us (TA or instructor) to be able to SSH into your DigitalOcean Droplet to help you, we need you to add our SSH public key on to your Droplet.

To do that, run the following command from your local machine (i.e *not* while SSH'd into your Droplet).

Before running the command, replace `xxx.xxx.xxx.xxx` with *your* Droplet's IP address.

```bash
curl http://dwa15.com/dwa15-teaching-team-access.pub | ssh root@xxx.xxx.xxx.xxx "cat >> ~/.ssh/authorized_keys"
```

FYI:
+ What this command does is adds a copy of our public key into the `~/.ssh/authorized_keys` file on your Droplet.
+ Only the TAs and myself have the corresponding private key on our machines.
