# Start

To start, ensure you have tools as `grub-mkrescue` installed. We will need them for making a simple .iso with GRUB installed.

## Compiling Busybox

Next up, we need a Busybox static binary. While we will switch away from Busybox later and use shared libraries as an optional step, we will use it as a starting point.

To compile Busybox, head to https://busybox.net/ to get a source tarball, and extract it. We will be doing a lot of compiling while making a distro, so this is a outline of what to do:

```bash
tar -xf busybox.tar.gz                # -xf = extract
cd busybox/                           #
make defconfig ; make menuconfig      # Busybox uses this layout; not every project is built like this
make -j$(nproc)                       # The nproc command checks how many CPU cores you have
ldd ./busybox                         # Check if the file was compiled statically
```

Make sure to enable static compiling in the menu (General). Also, the `make menuconfig` command requires that you have `ncurses` development headers installed, which may be in a package named like `libncurses-devel`.

If the `ldd` output says `not a dynamic executable`, that means you have done everything correctly, so far. If not, make sure the static compiling option is enabled in `make menuconfig`.

## Compiling the Linux kernel

For now, we will use the `defconfig` configuration, which takes much less time to compile than you would expect - around 3 mins on my 12 core CPU.

As before with Busybox, go to https://kernel.org/ to grab a release tarball. The Linux kernel is compiled similarly to Busybox:

```bash
tar -xf linux.tar.gz                  # -xf = extract
cd linux/                             #
make defconfig ; make menuconfig      # Linux uses the same layout as Busybox
make -j$(nproc)                       # The nproc command checks how many CPU cores you have
file ./arch/x86/boot/bzImage          # Check for the file (might be in arch/x86_64 instead)
```

## Creating the base project

Now, we will create our project directory. An outline of the workflow would look like:

```bash
mkdir my_distro
mkdir my_distro/live-initrd
mkdir my_distro/fs
mkdir my_distro/fs/boot
cp ../linux/arch/x86/boot/bzImage my_distro/fs/boot/vmlinuz
touch my_distro/Makefile # Optional, you can write one if desired.
cd my_distro
```

Now we will make a GRUB config.

```
set default=0
set timeout=5

set gfxmode=auto
set menu_color_normal=light-gray/black
set menu_color_highlight=white/light-blue

menuentry "My Distro Live" {
    echo "Loading kernel from /boot/vmlinuz"
    linux /boot/vmlinuz nomodeset
}

menuentry "Boot existing OS" {
    exit
}

menuentry "Reboot"  {
    reboot
}

menuentry "Shutdown" {
    halt
}
```

Which goes into the file `fs/boot/grub/grub.cfg`.

## Our first bootable system

Try running this command to create a bootable ISO:

```bash
grub-mkrescue -o ./mydistro_linux.iso fs       # Create a bootable disk image
qemu-system-x86_64 -cdrom mydistro_linux.iso   # Run it in a VM
```

<div style="border-left:4px solid #42b983; background:#f5f5f5; padding:10px 14px; margin:12px 0; border-radius:4px;">
  <strong style="display:block; margin-bottom:6px;">Tip</strong>

  <div>
    Remember - QEMU is never a copy of real hardware. You should always test on real hardware.
  </div>

  <div style="margin-top:6px;">
    QEMU emulates hardware in software, which means its behaviour can differ from real devices in subtle ways. For example, the system may blackscreen on real hardware!
  </div>
</div>

After you press ENTER on the first option, you will see a kernel panic. That means you did everything you were meant to do correctly.

<a href="#/live-system.md" style="
  display:inline-block;
  padding:10px 16px;
  background:#42b983;
  color:white;
  border-radius:6px;
  text-decoration:none;
  font-weight:600;
">
Creating a Live ISO
</a>
