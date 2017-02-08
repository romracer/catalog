## Rancher OCFS2 Filesystem Volume Plugin Driver

Mount an OCFS2 filesystem for shared usage

### Requirements

* Host kernel support for recent versions of the OCFS2 file system (RancherOS tested with kernel-extras)
* Network IP address and hostname for each node (a private network is recommended)
  - Port 7777/tcp should be open between the hosts on this network
* Pre-partitioned and OCFS2-formatted block device attached to each node at the same location (ex. `/dev/disk/by-path/pci-0000:03:00.0-scsi-0:0:1:0-part1` or `/dev/disk/by-label/rancher-ocfs2`)
* The cluster name used for mkfs.ocfs2 **must** match the cluster name specified when deploying the driver in Rancher (alpha-numeric, max 16 characters).

An OCFS2 filesystem can be created in this container. A recommended sequence of commands to create one might be:
```
$ docker run -it --rm --privileged -v /dev:/host/dev romracer/storage-ocfs2:v0.6.6 /bin/bash
# mount --rbind /host/dev /dev
# parted --script /dev/disk/by-path/pci-0000:03:00.0-scsi-0:0:1:0 \
    mklabel gpt \
    mkpart primary 1MiB 100\%
# mkfs.ocfs2 --node-slots 16 --label rancher-ocfs2 -T mail --fs-feature-level=max-features --mount cluster --cluster-stack=o2cb --cluster-name=rancherocfs2 /dev/disk/by-path/pci-0000:03:00.0-scsi-0:0:1:0-part1
# exit
```
You might need to re-read the block device on your nodes now:
```
# blockdev --rereadpt /dev/disk/by-path/pci-0000:03:00.0-scsi-0:0:1:0
```
It is recommended to set the following sysctls for appropriate recovery of a failed node:
```
# sysctl -w kernel.panic=30
# sysctl -w kernel.panic_on_oops=1
```

### Limitations

* A maximum of 16 nodes is supported
* Only the o2cb cluster stack is supported
* Only local heartbeat mode is supported

### Usage
Rancher hosts will have backing device environment variable set identifying default block device.  Driver will use it to mount OCFS2 filesystem and create a directory for each volume during create command and delete it at delete command.  Directory is bind mounted to appropriate Rancher volume location on container start and unmounted on container stop.

For instance:
```
export BACKING_DEVICE=/dev/disk/by-label/rancher-ocfs2
```

### Additional Info
* See [the plugin code][1] for details on the implementation.  
* Reference [the Oracle troubleshooting guide][2] for issues with OCFS2.  These commands will need to be done in the ocfs2 driver container.  
* PDF man pages for the 1.8 release of OCFS2 tools (used in this container) can be found [here][3].  

[1]: https://github.com/romracer/storage/tree/add_ocfs2_driver/package/ocfs2
[2]: https://docs.oracle.com/cd/E52668_01/E54669/html/ol7-tshoot-ocfs2.html
[3]: https://oss.oracle.com/projects/ocfs2/dist/documentation/v1.8/ocfs2-1_8_2-manpages.pdf
