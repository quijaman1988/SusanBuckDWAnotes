# Digital Ocean Snapshots

DigitalOcean has a feature called *Snapshots* where you can take a, well, snapshot of your Droplet in it's current state, including all configurations and data.

It's similar to DigitalOcean's backup features with these differences:

+ Backups are created automatically, wheras Snapshots are created manually
+ Backups cost extra money, wheras Snapshots are free

If your Droplet is currently in a healthy state, it's wise to take a Snapshot. That way, if something bad happens you can rebuild a new Droplet from the healthy Snapshot.


## Taking a Snapshot

To take a Snapshot, you must first turn off your Droplet. This can be done while SSH'd into your server with this command:

```bash
$ sudo poweroff
```

This will turn off your droplet and exit you out of your SSH connection.

Example:

```bash
root@Feb4DynamicWebApplications:~# sudo poweroff
root@Feb4DynamicWebApplications:~#
Broadcast message from root@Feb4DynamicWebApplications
	(/dev/pts/0) at 21:59 ...

The system is going down for power off NOW!
Connection to 162.243.95.209 closed by remote host.
Connection to 162.243.95.209 closed.
```

Once your Droplet is powered off, via the *Snapshot* section in DigitalOcean, you can create a new Snapshot:

<img src='http://making-the-internet.s3.amazonaws.com/vc-take-the-snapshot@2x.png' style='width:100%; max-width:1000px'>

In this example, I used the current date for the snapshot name, but you can use whatever name you'd like.

How long it takes your Snapshot to complete will vary; in my tests it took anywhere from a few seconds to a few minutes. The more data you have on your server, the longer the Snapshot creation will take.

After DigitalOcean is done making the Snapshot you can power your server back on via the *Power* section in DigitalOcean.

<img src='http://making-the-internet.s3.amazonaws.com/sysadmin-power-on-do@2x.png' style='max-width:1186px; width:100%' alt=''>

## Restoring/Starting from a Snapshot
You can now restore from this Snapshot in one of two places:

1. Droplet settings -> *Snapshots* -> *Restore from Snapshot*
2. When creating a new Droplet, under *Images*, you can select a previously created snapshot ([screenshot](http://making-the-internet.s3.amazonaws.com/vc-creating-new-droplet-from-snapshot.png)).
