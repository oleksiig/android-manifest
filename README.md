* **Prepare workspace:**
```
  $ mkdir ~/mydroid && cd ~/mydroid
```

* **Clone manifest and AOSP from Google**
```
  $ repo init -u https://github.com/oleksiig/android-manifest.git -b android-X.X.X_rXX --partial-clone
```

* **Sync sources**
```
  $ repo sync -c --no-tags
```

* **Initialize build environment and build Android**

```
   $ source build/envsetup.sh
   $ lunch <target>
   $ make
```

* **Build and flash Bootloaders**

1. Download, Build and Install **rkdeveloptool** from https://github.com/rockchip-linux/rkdeveloptool

2. Set environment variables
```
  $ export CROSS_COMPILE=~/mydroid/prebuilts/gcc/linux-x86/aarch64/gcc-linaro-X.X.X-XXXX.XX-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-
  $ export BL31=~/mydroid/device/rockchip/bootloaders/rkbin/bin/rk33/rk3399_bl31_vX.XX.elf
```
2. Build **Uboot**
```
  $ cd ~/mydroid/device/rockchip/bootloaders/uboot
  $ make <target>_defconfig
  $ make
```
3. Prepare bootloader images for flashing: **uboot.img**, **trust.img** and **rk3399_loader_vX.X.X.bin**
```
  $ cd ~/mydroid/device/rockchip/bootloaders/rkbin
  $ ./tools/loaderimage --pack --uboot ../uboot/u-boot-dtb.bin uboot.img 0x200000
  $ ./tools/trust_merger RKTRUST/RK3399TRUST.ini
  $ ./tools/boot_merger RKBOOT/RK3399MINIALL.ini
```
  Make shure that in the directory (~/mydroid/device/rockchip/bootloaders/rkbin) were created following files:
```
  rk3399_loader_vX.X.X.bin
  trust.img
  uboot.img
```

  Now we have all necessary files to flash the target board.

4. Power on the target board and make sure that SoC is in "rom mask" mode.

  To confirm that the board running in required mode:
```
  $ lsusb | grep Rockchip
  Bus 003 Device 110: ID 2207:330c Fuzhou Rockchip Electronics Company RK3399 in Mask ROM mode
```

5. Upload loader and flash bootloader images.

  Following method is used for initial board flashing, and when loader should be updated.
```
  $ cd ~/mydroid/device/rockchip/bootloaders/rkbin
  $ rkdeveloptool db rk3399_loader_vX.X.X.bin
  $ rkdeveloptool ul rk3399_loader_vX.X.X.bin
  $ rkdeveloptool wl 0x4000 uboot.img
  $ rkdeveloptool wl 0x6000 trust.img
```
  After successful flashing, reset the target board and let it boot into Uboot.

6. Preparing environment variables and Android partition table.

  U-Boot
```
  => env default -a
  ## Resetting to default environment

  => saveenv
  Saving Environment to MMC... Writing to MMC(0)... OK

  => gpt write mmc 0 $partitions
  Writing GPT: success!

  => gpt verify mmc 0 $partitions
  Verify GPT: success!

  => mmc part
  (Here should be displayed new GPT partition table)

  => fastboot usb 0
```
  From now target board is in **fastboot** mode, and Android images can be flashed.


* **Flashing Android Images**
```
  $ fastboot oem format
  $ fastboot -u format userdata
  $ fastboot format metadata
  $ fastboot erase misc
  $ fastboot erase frp
  $ fastboot flashall
```
 *When flashing become finished, borad will reboot automatically.*

* **Done**
