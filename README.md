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
- bootm is the command in uEnv.txt which talks about handing over. before handing over check-sum validation, uimage header printing, etc happens.
- "starting kernel..." is the last message of uboot.
- After handover, linux has its bootstrap loader, misc.c which is part of this laoder in linux, which takes care of uncompressing kernel and further step.
- to check uimage header contents. step in boot mode, uboot# help load /* load command help to load uimage from mSD(dev - 0) or eMMC(dev - 1) to DDR */
- uboot# load mmc 0:2 DDR_Loading_Addr /location/to/boot/uImage /* for fat based file loading, use fatload command */
- uboot# md DDR_Addr Number_Of_LiLEndian_DWords /* metadata command */
- uboot# imi DDR_Loading_ADDr /* header info for application image */
- many other uboot commands are also present.

##uimage (ELF extension)
- uimage is uboot friendly image, cuz it contains uboot image header (64B) on top of zimage.
- before loading kernel, uimage header info gets printed.

##hand-off from uboot to kernel
- uboot handsoff control to head.S in linux bootstrap loader.
- u-boot-2017.05-rc2/arch/arm/lib/bootm.c /* code which picks up linux kernel from memory, and handsoff control to kernel -- boot_jump_linux(*image, flag) */
- this function has imaege structure, this structure holds the entry point of kernel image in DDR memory, right after header
- image struct has other members initialized and points to device tree blob/binary in DDR. ft_addr or fdt_addr or flatten_tree_binary_address to describe peripheral on the board
- at last call the kernel entry point. first args is zero, second is machine_id identified by uboot passed by r1, and third arg is device tree binary address /* kernel_entry(0, machid, r2) */
- head.S -> misc.C // uncompress, compressed image
- head.S -> kernel's head.s -> head-common.c -> main.c
- main.c-> init // user application
