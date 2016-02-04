# Digital Ocean Snapshots

DigitalOcean has a feature called *Snapshots* where you can take a, well, snapshot of your Droplet in it's current state, including all configurations and data.

It's similar to DigitalOcean's backup features with these differences:

+ Backups are created automatically, wheras Snapshots are created manually
+ Backups cost extra money, wheras Snapshots are free

If your Droplet is currently in a healthy state, it's wise to take a Snapshot. That way, if something bad happens you can rebuild a new Droplet from the healthy Snapshot.


## Taking a Snapshot

To take a Snapshot, you must first Power Off your Droplet via the *Power* section:

<img src='http://making-the-internet.s3.amazonaws.com/vc-power-off-droplet@2x.png' style='width;100%; max-width:100px'>

Then, in the *Snapshot* section, you can create a new snapshot:

<img src='http://making-the-internet.s3.amazonaws.com/vc-take-the-snapshot@2x.png' style='width;100%; max-width:100px'>

In this example, we used the current date for the snapshot name, but you can use whatever name you'd like.

And that's it.


## Restoring/Starting from a Snapshot
You can now restore from this Snapshot in one of two places:

1. Droplet settings -> *Snapshots* -> *Restore from Snapshot*
2. When creating a new Droplet, under *Images*, you can select a previously created snapshot ([screenshot](http://making-the-internet.s3.amazonaws.com/vc-creating-new-droplet-from-snapshot.png)).
