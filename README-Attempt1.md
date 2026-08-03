Target Hardware: Samsung Chromebook 2 (11.6", Exynos 5420 ARMv7-A)
SPI Flash Chip: Winbond W25Q32FW (4096 kB / 4 MB)
Build Environment: Native 32-bit armhf (and 64-bit aarch64) chroot running Trisquel 11 (Aramo) / GCC 11.4.0.

So I have a arm (32-bit) Trisquel chroot alongside a arm64 Trisquel chroot for this build. They're minimal tarballs with the bare minimum within. Dependencies installed are :
apt-get install acpica-tools libfwtsacpica1 libfwtsiasl1 build-essential bison flex git make patch zlib1g-dev libncurses5-dev
arm-linux-gnueabihf-gcc was also used for one compilation attempt. What wasn't used was crossgcc from coreboot itself - to save time mostly as the CPU on this machine isn't present for building coreboot.

Fetched using Coreboot's download page (not git) - coreboot v24.05 and coreboot blobs v24.05. As well as U-boot 2024.04.

U-boot 2024.04 was built simply on both the 32-bit and 64-bit rootfs :
make peach-pit_defconfig
make CPUS=8
(64 bit uses CROSS_COMPILE=arm-linux-gnueabihf- make ARCH=arm CPUS=8)
And I moved u-boot.bin to coreboot's build dir as payload.bin. Maybe I didn't include it right in the final output. Just maybe.

Went into Coreboot and ran menuconfig.
Selected "Any Toolchain" from General Setup, the Peach-Pit board from Mainboard, and added payload.bin in Payload as just payload.bin in the path.

The compilation failed on static.o and cpu.o : cc1: error: '-mfloat-abi=hard': selected architecture lacks an FPU
Went into src/arch/arm/armv7/Makefile.mk and added 'armv7_flags += -march=armv7-a+mp+sec+vfpv4 -mfpu=neon-vfpv4' under armv7_asm_flags which resolved it.

I also tried to use the 64-bit chroot at first because since the cross compilers for arm has a gnueabi and gnueabihf version - I thought maybe the hf version is designed for hard float (no difference through).

Had a whack-a-mole with vboot - failing constantly on extract_vmlinuz.o with warnings. error: '__builtin_memcpy' specified bound... exceeds maximum object size...
Finally in util/cbfstool/Makefile.mk - I turned CC="$(HOSTCC) into 'CC="$(HOSTCC) -Wno-stringop-overflow -Wno-error -Wno-array-bounds -w' in a attempt to silence that error, and it finished building.

Of course I extracted the first count=8192 from my dumped stock firmware and planted that in 3rdparty/blobs/cpu/samsung/exynos5420/bl1.bin (or something like that), and the screen never turned on - it just blinks it's red LED.
So I wrote stock firmware back - and it boots ChromeOS fine.

So I found a download link in the dummy bl1.bin and downloaded that as the actual bl1.bin. However that also failed to turn the screen on.

Maybe I will try using crossgcc after all. Maybe with a newer version of Coreboot.
