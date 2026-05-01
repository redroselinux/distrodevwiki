# Making a Live System

Let's take our Busybox compiled binary into the `live-initrd` folder we created before:

```bash
mkdir live-initrd/bin
cp ../busybox/busybox live-initrd/bin/busybox
# We wont use --install because it breaks; instead symlink manually
find live-initrd/bin ! -name 'busybox' -type l -delete
for applet in $(live-initrd/bin/busybox --list); do
    ln -s busybox live-initrd/bin/$applet
done
```

This installs the Busybox utilities. Now create an init script:

```bash
cat > live-initrd/init << 'EOF'
#!/bin/sh
mkdir /proc /sys /dev /tmp
mount -t proc none /proc
mount -t sysfs none /sys
mount -t devtmpfs none /dev
mount -t tmpfs none /tmp
exec /bin/sh
EOF
chmod 0755 live-initrd/init
```

Now, we will make a simple initramfs cpio:

```bash
(cd live-initrd && find . -print0 | cpio --null -ov -H newc > ../initramfs.cpio) && \  # Create a CPIO archive
gzip initramfs.cpio && \                                                               # Compress the CPIO archive
mv initramfs.cpio.gz fs/boot/live-initrd.cpio.gz                                       # Move it to the boot directory
```

Now, we need to add this to our `grub.cfg`, right after the `linux` command:

```
echo "Loading initrd from /boot/live-initrd.cpio.gz"
initrd /boot/live-initrd.cpio.gz
```

Congratulations! You have a simple Linux system.
