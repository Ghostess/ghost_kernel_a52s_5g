# Ghost's Custom Android Kernel for the Samsung A52s 5G (Snapdragon 778G - SM7325)

Using [Eureka kernel](https://github.com/saadelasfur/android_kernel_samsung_sm7325/) as the source
<br>

Linux 5.4.254, built with Clang 19.0-r530567 (plus other compilation optimizations)

### Features
- KernelSU-Next as the root solution (manual hooks)
- Supports both AOSP* and One UI** roms
- Optimized for battery life and performance
- Disabled several kernel debugging tools, flags, and features
- Enabled CONFIG_TMPFS_XATTR for [mountify](https://github.com/backslashxx/mountify) KernelSU module mounting compatibility
- Disabled Samsung Knox
- Switchable SELinux policy
- Backported NTSync*** (for better performance in Winlator/GameNative/etc)
- Full [Droidspaces](https://github.com/ravindu644/Droidspaces-OSS) support

> Compatibility Notes: <br>
> \* Is fully untested, I don't know if it works or not. <br>
> \** Should work on all One UI versions, but I've only tried it on UN1CA 3.1.0 (OneUI 8), so ymmv. <br>
> \*** Requires newer versions of Wine/Proton with NTSync support

> [!WARNING]
I am NOT a kernel developer. this is just my personal kernel that i made for myself to daily-drive. If you have any issues i may or may NOT fix them. im not responsible for anything that happens to your device or your data.

---

# Installation
1. Download the appropriate flashable .zip file from the [Releases](https://github.com/Ghostess/android_kernel_samsung_sm7325/releases/) (Or [Github Actions](https://github.com/Ghostess/android_kernel_samsung_sm7325/actions) for nightly builds)
   - `*_AOSP_*.zip` for AOSP-based ROMs
   - `*_One-UI_*.zip` for Samsung One UI ROMs
2. Reboot into your recovery environment (see [TWRP](https://xdaforums.com/t/recovery-official-twrp-3-7-1-0-for-galaxy-a52s-5g.4488419/))
3. Flash the .zip file
    - If this is your first time flashing the kernel, make sure to wipe Cache/Dalvik. Otherwise, you can skip this step
4. Reboot
5. Download the KernelSU-Next manager app [here](https://github.com/KernelSU-Next/KernelSU-Next/releases) and install it
    - If the KernelSU-Next Manager app reports "Unsupported", completely uninstall and reinstall it. Your existing modules will be preserved

> [!TIP]
> Use [mountify](https://github.com/backslashxx/mountify) or [hybrid-mount](https://github.com/Hybrid-Mount/meta-hybrid_mount) as the primary metamodule. If your KSU modules, such as GPU or audio driver modules, don't work, it's because you don't have a [metamodule](https://kernelsu.org/guide/metamodule.html) installed.
>
> Use this [KSU Module](https://github.com/user-attachments/files/25517721/A16StorageFix-v2.0.zip) if your apps can't save data in AOSP Android 16 ROMs. (There's also [this](https://github.com/omersusin/StorageFixer/) and [this](https://gist.github.com/Loukious/d7f6da0bdc13556d2cde84123fe4f794). Your pick)
>
> *The following are optional recommendations; feel free to use any, all, or none of them.*
>
> Update GPU drivers with this [KSU module](https://t.me/adrenolabsupport/242/1157). Newer versions of this module aren't compatible with this device. One notable issue is that you won't be able to upload stories on Instagram or send any media through DMs if you do update it. Stick with this one. You also need a metamodule (hybrid-mount, mountify) for it to work
>
> Use [Zygisk-Next](https://github.com/Dr-TSNG/ZygiskNext), and this version of [LSPosed (Github)](https://github.com/JingMatrix/Vector) if needed. you can also use [LSPosed (Telegram)](https://t.me/LSPosed/314) but its closed source which i'd avoid (check for newer versions on that Telegram group)
>
> For ad-blocking, just use [bindhosts](https://github.com/bindhosts/bindhosts)

---

# How to build
Run `build_kernel_zip.sh` for a fully automated kernel build (builds both oneui and aosp flashables and places them in the release/ folder along with the .config from the build)

The script downloads and extracts required build tools (`clang` and `magiskboot`) into the local `toolchain/` directory. No system-wide installation is performed.

You may edit hard-coded values in the script (such as AUTHOR or build metadata) to match your setup.

**Notes**:

- If prompted during kernel configuration, you can simply press enter to go with the default options (which is what i use)

- The build script relies on pre-packaged images located in `base-images` for both AOSP (crDroid) and One UI (UN1CA 3.1.0) (For this one, i modified its fstab to support all oneui versions). If you provide your own `boot.img` or `vendor_boot.img`, make sure you replace both `boot.img` and `vendor_boot.img` in either `aosp` or `oneui` folder, depending on which ROM you got them from.

---

## Dependencies

### Fedora / RHEL
```bash
sudo dnf install -y \
  curl git make binutils bc bison flex python3 \
  zip unzip cpio tar \
  findutils sed grep coreutils kmod openssl-devel
```

### Arch Linux
```bash
sudo pacman -S \
  curl git make binutils bc bison flex python \
  zip unzip cpio tar \
  findutils sed grep coreutils kmod openssl
```

### Debian / Ubuntu
```bash
sudo apt update && sudo apt install -y \
  curl git make binutils bc bison flex python3 \
  zip unzip cpio tar \
  findutils sed grep coreutils kmod libssl-dev
```

---

## Manually
[Check this tutorial if you have any questions, or if you don't know where, or how, to start](https://github.com/ravindu644/Android-Kernel-Tutorials)

Most of the next steps are outdated, but it will still build successfully.

### Requirements
- [Clang-19-r530567](https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86/+archive/refs/heads/main/clang-r530567.tar.gz)
- [Magiskboot](https://github.com/topjohnwu/Magisk/releases/download/v30.7/Magisk-v30.7.apk)

### Clone this repository

`git clone git@github.com:Ghostess/android_kernel_samsung_sm7325.git`

### Update git submodules

`git submodule update --init --recursive`

### Export PATH
`export PATH="/{path-to-your-clang-bin-folder}/clang/bin:$PATH"`

### Make defconfig
`make O=$(pwd)/out ARCH=arm64 vendor/a52sxq_kor_single_defconfig`

### Make
```
make -j$(nproc) \
  O=$(pwd)/out \
  ARCH=arm64 \
  CC=clang \
  LLVM=1 \
  LLVM_IAS=1 \
  CROSS_COMPILE=aarch64-linux-gnu- \
  KBUILD_BUILD_USER=example \
  KBUILD_BUILD_HOST=kernel \
  CONFIG_SECTION_MISMATCH_WARN_ONLY=y
```

If prompted during configuration, use the following options:

`Enable vDSO for 32-bit applications (COMPAT_VDSO)` **y**

`Clang Shadow Call Stack (SHADOW_CALL_STACK)` **y**

`Use virtually mapped shadow call stacks (SHADOW_CALL_STACK_VMAP)`  **y**

`Link-Time Optimization (LTO)` **2**

`Use Clang's Control Flow Integrity (CFI) (CFI_CLANG)` **n** (or **yes** if not integrating KSU-Next root solution)

`Use CFI shadow to speed up cross-module checks (CFI_CLANG_SHADOW)` **y**

`Use CFI in permissive mode (CFI_PERMISSIVE)` **n**

`Use RELR relocation packing (RELR)` **y**

`Use Clang's ThinLTO (EXPERIMENTAL) (THINLTO)` **y**

### Prepare module files
`cd out/`

`mkdir -p modules_for_zip`

`find . -path ./modules_for_zip -prune -o -type f -name "*.ko" -exec cp -t modules_for_zip/ {} +`

`find modules_for_zip -type f -name "*.ko" -exec llvm-strip --strip-unneeded {} \;`

### Prepare flashable .zip file
Extract `boot.img` and `vendor_boot.img` from ROM's .zip file (or use the ones from `base-images` folder, located at the kernel root directory)

### boot.img
See [this](https://github.com/ravindu644/Android-Kernel-Tutorials#01-downloading-and-extracting-the-latest-magisk-apk) if you have any questions about magiskboot

`magiskboot unpack boot.img`

Copy `Image` file, replace with `kernel` file

Repack with `magiskboot repack boot.img` and place the generated .img file in `template-zip-file/images/`, change its name to `boot.img`

### dtbo.img
Place `dtbo.img` from `out/arch/arm64/boot/` in `template-zip-file/images/`

### vendor_boot.img
`magiskboot unpack -h vendor_boot.img`

Replace `dtb` file with `arch/arm64/boot/dts/vendor/qcom/yupik.dtb`, rename to `dtb`

Open header file and replace first line with `SRPUE26A001` (probably not necessary)

`mkdir ramdisk && cd ramdisk`

`cpio -idm < ../ramdisk.cpio`

`sudo chown -R $(whoami):$(whoami) .`

`rm -rf lib/modules/5.4-gki/*`

Place .ko files from `modules_for_zip/` and `modules.alias`, `modules.dep`, `modules.softdep` and `modules.load` in `lib/modules/`. Make sure the `modules.dep` entries for each module point to `/lib/modules/` and not `/vendor/lib/modules/`

if using `vendor_boot.img` from `template-zip-file` folder, it already has the proper `modules.*` files (assuming you did not add or remove any modules)

`cp -rf {kernel-source-directory}/firmware/tsp_stm/fts5cu56a_a52sxq* lib/firmware/tsp_stm/` (not necessary if using `vendor_boot.img` from `template-zip-file` folder, since it is already present)

`sudo chown -R root:root .`

`sudo find . -type d -exec chmod 755 '{}' \;`

`sudo find . -type f -exec chmod 644 '{}' \;`

`sudo find .  -print0 | cpio --null -o -H newc --owner root:root > ../ramdisk.cpio`

`cd ..`

`sudo rm -rf ramdisk/`

Repack with `magiskboot repack vendor_boot.img` and place the generated .img file in `template-zip-file/images/`, rename it to `vendor_boot.img`

### Make flashable .zip file
Go to `template-zip-file` folder and run the following command:

`zip -r -9 {flashable-kernel-zip-file-name}.zip META-INF/ images/`

Now flash the .zip file in your recovery environment

### Start-over
`make ARCH=arm64 mrproper CONFIG_KSU_MANUAL_HOOK=y` (for whatever reason, KSU-Next needs that last flag enabled)

`rm -rf out/`

## Update KSU-Next definitions
```
cd KernelSU-Next
git fetch --tags
git checkout v3.2.0-legacy
cd ..
git add KernelSU-Next
git commit -m "Update KernelSU-Next to v3.2.0-legacy"
```

---

# Credits (*)
**salvogiangri** (kernel, UN1CA ROM), **Simon1511** (AOSP related changes), **Frax3r/utkustnr** (kernel, update-binary shell script and README.md instructions), **RisenID** (kernel), **saadelasfur** (kernel),  **MySelly** (crDroid's Nothing-Phone-1 kernel, SUSFS implementation), **Haky86** (kernel A23 5G), **DrRoot85** (kernel S23), **0xSecureByte** (kernel msm-5.4), **rifsxd** (KSU-Next), **backslashxx** (Manual hook implementation for KSU-Next), **osm0sis** (Recovery Flashable Zip shell script), **ravindu644** (kernel compilation), **Samsung** (original kernel source code), **CodeLinaro** (kernel Qualcomm msm-5.4)

Special thanks to bone-machine (https://github.com/bone-machine/android_kernel_samsung_sm7325_a52s_5g). I forked saadelasfur's repo as the base for this kernel, but cherry picked a lot of commits from bone-machine's repo and use his build script too (it would take me a long time to make one on my own)

I also used bone-machine's README as the base for my own (his detailed build instructions helped me out so it'll help yall out too).

<sup>* There are several commits which do not have the original author's name. In most cases, you can find the source for each change inside each commit. In any case, I do not take credit for them.</sup>

---

# Resources
- https://github.com/Mesa-Labs-Archive/android_kernel_samsung_sm7325/

- https://github.com/salvogiangri/android_kernel_samsung_sm7325

- https://github.com/utkustnr/android_kernel_samsung_sm7325/

- https://github.com/RisenID/kernel_samsung_ascendia_sm7325

- https://github.com/saadelasfur/android_kernel_samsung_sm7325/

- https://github.com/LineageOS/android_kernel_samsung_sm7325

- https://github.com/crdroidandroid/android_kernel_nothing_sm7325/

- https://github.com/Haky86/android_kernel_samsung_sm6375

- https://github.com/DrRoot85/kernel_samsung_sm8550-commom

- https://github.com/0xSecureByte/platform_kernel_msm-5.4

- https://github.com/KernelSU-Next/KernelSU-Next

- https://github.com/backslashxx/KernelSU/issues/5#event-24583207399

- https://github.com/ravindu644/Android-Kernel-Tutorials

- https://opensource.samsung.com/uploadList?menuItem=mobile (SM-A736B, SM-A528B, SM-A528N)

- https://git.codelinaro.org/clo/la/kernel/msm-5.4

- https://github.com/WildKernels/kernel_patches/
