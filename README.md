# embedded_linux_bbb_AM355x_soc

## SDcard files pre-requisite.
- Create a paritition using gparted (ubuntu platform), select correct device on startup eg. /dev/sdx, unmount, delete the partition, create file system fat16 1GB, label the partition as boot,
  now create filesystem ext3 for rootfs partition. name it as ROOTFS, click add and then apply changes to create such partition. set flag for boot partition, right click on partition - manage flags
  - check boot and apply. 
- MLO loader file in BOOT partition for RBL
- u-boot.img file in BOOT partition for MLO
- uEnv.txt to help uboot in getting to know the image location to boot, where to load /boot/platform-dtb and /boot/uImage (kernel image)
- rootfs in ROOTFS partition.

## How to boot BBB
- Insert SD card in BBB
- Connect board to host sytem using serial debug wires. USB to UART converter for debug log or via Mini USB cabel (P4 connector)
- Open minicom in terminal, give power to board (it tries to boot from emmc), put board in power-down mode by longpress s3(all LEDs off), press and hold boot button s2(for SD card boot), gently press and release power button s3, and release boot button s2.

## U-boot
- File name has to be u-boot.img
- uEnv.txt is environment config file
- Uboot uses uimage. uimage is zimage + header (describes arch, image name, kernale relase, platform etc. e.g. uname -r dump)
- Uboot is responsible for up and running all peripheral which are supported for booting by platform, loading kernel and hand-out control to kernel.
- bootm is the command in uEnv.txt which talks about handing over. before handing over check-sum validation, header printing, etc happens.
- "starting kernel..." is the last message of uboot.
- After handover, linux has its bootstrap loader, misc.c which is part of this laoder in linux, which takes care of uncompressing kernel and further step.
