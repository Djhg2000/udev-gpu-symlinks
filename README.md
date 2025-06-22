# udev-gpu-symlinks
Rules for udev to add GPU aliases independent of slot or init order.

Useful for things like overriding kwin card selection (see `KWIN_DRM_DEVICES`).

## How to use
Put `99-gpu-symlinks.rules` in `/etc/udev/rules.d` and reboot or reload udev.

In most cases, this would be how you'd do that:

```
git clone https://github.com/Djhg2000/udev-gpu-symlinks.git
cd udev-gpu-symlinks
sudo cp 99-gpu-symlinks.rules /etc/udev/rules.d
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Afterwards you might something like this in `/dev/dri/` (if you have 2 GPUs):

```
$ ls -l /dev/dri/
total 0
lrwxrwxrwx  1 root root          5 jun 23 00:15 0x1002:0x67df-115-D009PI2-101 -> card0
lrwxrwxrwx  1 root root          5 jun 23 00:15 0x1002:0x687f-0213faa68d4a4084 -> card1
lrwxrwxrwx  1 root root          5 jun 23 00:15 0x1002:0x687f-xxx-xxx-xxx -> card1
lrwxrwxrwx  1 root root          5 jun 23 00:15 AMD:0x67df-115-D009PI2-101 -> card0
lrwxrwxrwx  1 root root          5 jun 23 00:15 AMD:0x687f-0213faa68d4a4084 -> card1
lrwxrwxrwx  1 root root          5 jun 23 00:15 AMD:0x687f-xxx-xxx-xxx -> card1
drwxr-xr-x  2 root root        120 jun 23 00:15 by-path
crw-rw----+ 1 root video  226,   0 jun 23 00:15 card0
crw-rw----+ 1 root video  226,   1 jun 23 00:15 card1
crw-rw----+ 1 root render 226, 128 jun 23 00:15 renderD128
crw-rw----+ 1 root render 226, 129 jun 23 00:15 renderD129

```

Several symlinks are created, due to the quirky nature of GPU PCIe descriptors; some include a unique ID, some at least give you the VBIOS version (and some have a dummy version string), some may not give you anything uniquely identifying at all. Therefore these rules first try to identify the card by the `unique_id` attribute from udev, then the VBIOS version string.

For convenience these symlinks also appear with an interpreted PCIe `Vendor ID`. Currently only AMD/ATI (identified only as AMD for brevity) is tested but it should also correcty identify NVIDIA and Intel GPUs. These rules will not attempt to interpret the `Device ID`.

These are the formats of the symlinks created:

```
<Vendor ID>:<Device ID>-<udev unique_id>
<Vendor ID>:<Device ID>-<VBIOS version>
<Vendor name>:<Device ID>-<udev unique_id>
<Vendor name>:<Device ID>-<VBIOS version>
```

If you find a card which does not give you any symlinks at all then please open an issue.
