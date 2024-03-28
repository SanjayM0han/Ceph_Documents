### Creating an rbd pool

First, we will create a pool to store the images we create later. We could supply placement groups and other information if needed.
```
sudo ceph osd pool create rbdpool
```

Next, we will initialize the pool so it will have the application of the rbd set. This command will add the meta required to host images.
```
sudo rbd pool init rbdpool
```

### Creating an rbd image

We will create our first image called `disk1`. It will be 4G in size and have the layering feature. There is a bunch of different image features to choose from:

* layering: layering support
* striping: striping v2 support
* exclusive-lock: exclusive locking support
* object-map: object map support (requires exclusive-lock)
* fast-diff: fast diff calculations (requires object-map)
* deep-flatten: snapshot flatten support
* journaling: journaled IO support (requires exclusive-lock)
* data-pool: erasure-coded pool support

The one that is most appropriate here is layering. Next, if not already mentioned in your configuration(`/etc/ceph/ceph.conf`), we could supply the monitor IP. Next, we could provide the key file if not present in the regular place. And lastly, we will add which pool to create the image.
```
sudo rbd create disk1 --size 4096 --image-feature layering -m 192.168.6.43 -k /etc/ceph/ceph.client.admin.keyring -p rbdpool
```

### Temporary mounting

On the client, we can now map the drive. This command will use the same parameters with the addition of the client's name to connect.
```
sudo rbd map disk1 --name client.admin -m 192.168.6.43 -k /etc/ceph/ceph.client.admin.keyring -p rbdpool
```

Next, we will create a filesystem on the image available as a device on your system under `/dev/rdb`, then there will be a directory with the name of your pool and lastly, the name of the image as a link to the rbd device created.
```
sudo mkfs.ext4 -m0 /dev/rbd/rbdpool/disk1
```

Lastly, mounting the device is just using the device and mounting to a mount-point exactly as typical mounting.
```
sudo mount /dev/rbd/rbdpool/disk1 /mnt/ceph-block-device
```

### Permanent mounting

First, we need to add a configuration for the rbdmap service in the file `/etc/ceph/rbdmap`. This configuration will map the image `disk1` on pool `rbdpool`. Next, we add the parameter id of the client to connect. Finally, we need to add the keyring with the access key for this pool.
```
rbdpool/disk1           id=admin,keyring=/etc/ceph/ceph.client.admin.keyring
```

To get this drive mapped, we start the rbdmap service.
```
sudo systemctl start rbdmap
```

Next, we will create a filesystem on the image available as a device on your system under `/dev/rdb`, then there will be a directory with the name of your pool and lastly, the name of the image as a link to the rbd device created.
```
sudo mkfs.ext4 -m0 /dev/rbd/rbdpool/disk1
```

Next, we add a configuration in `/etc/fstab`. Pretty much what you would expect to mount the `disk1` image into our mount-point `/mnt/ceph-block-device`. The filesystem is ext4, and we need to supply the flag `noauto` as the system might try to mount the device before it's actually mapped. The rbdmap service will mount the drive when the image is mapped.
```
/dev/rbd/rbdpool/disk1     /mnt/ceph-block-device   ext4   noauto  0       0
```
