Android repo manifest for OP-TEE development

mkdir -p $HOME/devel/optee
cd $HOME/devel/optee
repo init -u https://github.com/jbech-linaro/manifest.git -m ${TARGET}.xml [-b ${BRANCH}]
repo sync
cd build
make -f toolchain.mk toolchains
make -f $TARGET.mk

FVP
After running 'make' above, run 'make run'.

Hikey
After running 'make' above, follow the instructions at https://github.com/96boards/documentation/wiki/UEFI#flash-binaries-to-emmc to flash all the required images to and boot the board.

Location of files/images mentioned in the link above:
$HOME/devel/optee/burn-boot/hisi-idt.py
$HOME/devel/optee/l-loader/l-loader.bin
$HOME/devel/optee/l-loader/ptable.img
$HOME/devel/optee/arm-trusted-firmware/build/hikey/release/fip.bin
$HOME/devel/optee/out/boot-fat.uefi.img
