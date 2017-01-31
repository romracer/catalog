## Rancher OCFS2 Filesystem Volume Plugin Driver

Mount an OCFS2 filesystem for shared usage

### Requirements

* Host kernel support for recent versions of the OCFS2 file system (RancherOS tested with kernel-extras)
* Network IP address and hostname for each node (private network recommended)
* Pre-partitioned and OCFS2-formatted block device attached to each node at the same location (ex. `/dev/disk/by-path/pci-0000:03:00.0-scsi-0:0:1:0-part1` or `/dev/disk/by-label/rancher-ocfs2`)

An OCFS2 filesystem can be created in Ubuntu 16.04 or this container.
A recommended command to create one using this container might be:
```
$ docker run -it --rm --privileged -v /dev:/host/dev romracer/storage-ocfs2:v0.6.6 /bin/bash
# mount --rbind /host/dev /dev
# parted --script /dev/disk/by-path/pci-0000:03:00.0-scsi-0:0:1:0 \
    mklabel gpt \
    mkpart primary 1MiB 100\%
# mkfs.ocfs2 -N 16 -L rancher-ocfs2 -T mail --fs-features=sparse,backup-super,unwritten,inline-data,xattr,indexed-dirs -M cluster /dev/disk/by-path/pci-0000:03:00.0-scsi-0:0:1:0-part1
# exit
```
You might need to re-read the block device on your nodes now:
```
# blockdev --rereadpt /dev/disk/by-path/pci-0000:03:00.0-scsi-0:0:1:0
```

### Limitations

* A maximum of 16 nodes is supported

### Usage
Rancher hosts will have default backing device environment variable set identifying default block device.  Driver will use it to mount OCFS2 filesystem and create a directory for each volume during create command and delete it at delete command.  Directory is bind mounted to appropriate Rancher volume location on container start and unmounted on container stop.

For instance:
```
export BACKING_DEVICE=/dev/disk/by-label/rancher-ocfs2
```

### Additional Info
See [the plugin code][1] for details on the implementation.

[1]: https://github.com/romracer/storage/tree/add_ocfs2_driver/package/ocfs2