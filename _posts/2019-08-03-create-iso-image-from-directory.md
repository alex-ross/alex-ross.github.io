---
layout:     post
title:      "Create ISO image from directory"
date:       2019-08-03
---

I was using VirtualBox to run an older version of OS X. I had some files that I needed from my host computer but hadn't set up any shared directories. I was using `hdiutil` to create an ISO for the OS X and figured that I could move files from my host computer using another ISO because it can be mounted without any restarts.

No additional tools is needed since `hdiutil` comes with OS X/macOS.

This is basically it:

```bash
hdiutil makehybrid -o ~/Desktop/image.iso /directory/to/make/iso/of -iso -joliet
```

You could use the command above as is and just change `/directory/to/make/iso/of` to the path you would like an iso of. But if you are interested in a breakdown of the command it comes here.

- `hdiutil` a tool that comes with macOS to manipulate disk images
- `makehybrid -o [image] [source]` the hdiutil verb to generate a read-only disk image with given image and source path
- `-iso` create the disk image as an ISO which is an cross-platform format
- `-joliet` the filesystem for the disk image. OS X/macOS will prefer the Joliet filesystem according to the man pages of hdiutil.