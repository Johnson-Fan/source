#
# Copyright (C) 2009-2013 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

NAND_BLOCKSIZE := 2048-128k

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/image.mk

DEVICE_VARS += DEVICE_DTS KERNEL_SIZE PAGESIZE BLOCKSIZE SUBPAGESIZE KERNEL_IN_UBI
KERNEL_LOADADDR:=0x8000
TARGET_DEVICES = linksys-audi linksys-viper dockstar goflexnet goflexhome

define Device/Default
  KERNEL_DEPENDS = $$(wildcard $(DTS_DIR)/$$(DEVICE_DTS).dts)
  KERNEL := kernel-bin | append-dtb | uImage none
  KERNEL_NAME := zImage
  KERNEL_SUFFIX  := -uImage
  KERNEL_INSTALL := 1
  KERNEL_IN_UBI :=
endef

define Device/dockstar
  DEVICE_DTS := kirkwood-dockstar
  PAGESIZE := 2048
  SUBPAGESIZE := 512
  BLOCKSIZE := 128KiB
  FILESYSTEMS := squashfs
  PROFILES := Generic DOCKSTAR
  IMAGES := factory.bin sysupgrade.tar
  IMAGE/factory.bin := append-ubi
  IMAGE/sysupgrade.tar := sysupgrade-nand
  KERNEL_IN_UBI := 1
  KERNEL := kernel-bin | append-dtb
endef

define Device/goflexnet
$(Device/dockstar)
  PROFILES := Generic GOFLEXNET
  DEVICE_DTS := kirkwood-goflexnet
endef

define Device/goflexhome
$(Device/dockstar)
  PROFILES := Generic GOFLEXHOME
  DEVICE_DTS := kirkwood-goflexhome
endef

define Device/linksys-audi
  DEVICE_DTS := kirkwood-linksys-audi
  PAGESIZE := 512
  SUBPAGESIZE := 256
  BLOCKSIZE := 16KiB
  KERNEL_SIZE := 2624k
  FILESYSTEMS := squashfs
  PROFILES := Generic AUDI
  IMAGES := factory.bin sysupgrade.tar
  IMAGE/factory.bin := append-kernel $$$$(KERNEL_SIZE) | append-ubi
  IMAGE/sysupgrade.tar := sysupgrade-nand
endef

define Device/linksys-viper
  DEVICE_DTS := kirkwood-linksys-viper
  PAGESIZE := 2048
  SUBPAGESIZE := 512
  BLOCKSIZE := 128KiB
  KERNEL_SIZE := 2688k
  FILESYSTEMS := squashfs
  PROFILES := Generic VIPER
  IMAGES := factory.bin sysupgrade.tar
  IMAGE/factory.bin := append-kernel $$$$(KERNEL_SIZE) | append-ubi
  IMAGE/sysupgrade.tar := sysupgrade-nand
endef

define Image/BuildKernel/Template

	$(CP) $(KDIR)/zImage $(BIN_DIR)/$(IMG_PREFIX)-zImage
	echo -ne '\x00\x00\x00\x00' >> $(BIN_DIR)/$(IMG_PREFIX)-zImage
	$(call Image/BuildKernel/MkuImage, \
		none, 0x8000, 0x8000, \
		$(BIN_DIR)/$(IMG_PREFIX)-zImage, \
		$(BIN_DIR)/$(IMG_PREFIX)-uImage \
	)

 ifneq ($(CONFIG_TARGET_ROOTFS_INITRAMFS),)
	$(CP) $(KDIR)/zImage-initramfs $(BIN_DIR)/$(IMG_PREFIX)-zImage-initramfs
	echo -ne '\x00\x00\x00\x00' >> $(BIN_DIR)/$(IMG_PREFIX)-zImage-initramfs
	$(call Image/BuildKernel/MkuImage, \
		none, 0x8000, 0x8000, \
		$(BIN_DIR)/$(IMG_PREFIX)-zImage-initramfs, \
		$(BIN_DIR)/$(IMG_PREFIX)-uImage-initramfs \
	)
 endif

 ifneq ($(1),)
	$(CP) $(DTS_DIR)/kirkwood-$(1).dtb $(BIN_DIR)/$(IMG_PREFIX)-$(1).dtb

	$(CP) $(KDIR)/zImage $(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-zImage
	cat $(BIN_DIR)/$(IMG_PREFIX)-$(1).dtb >> $(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-zImage
	$(call Image/BuildKernel/MkuImage, \
		none, 0x8000, 0x8000, \
		$(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-zImage, \
		$(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-uImage \
	)

  ifneq ($(CONFIG_TARGET_ROOTFS_INITRAMFS),)
	$(CP) $(KDIR)/zImage-initramfs $(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-zImage-initramfs
	cat $(BIN_DIR)/$(IMG_PREFIX)-$(1).dtb >> $(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-zImage-initramfs
	$(call Image/BuildKernel/MkuImage, \
		none, 0x8000, 0x8000, \
		$(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-zImage-initramfs, \
		$(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-uImage-initramfs \
	)
  endif
 endif
endef

define Image/InstallKernel/Template

 ifneq ($(CONFIG_TARGET_ROOTFS_INCLUDE_KERNEL),)
	$(INSTALL_DIR) $(TARGET_DIR)/boot
   ifneq ($(CONFIG_TARGET_ROOTFS_INCLUDE_UIMAGE),)
	$(CP) $(BIN_DIR)/$(IMG_PREFIX)-uImage $(TARGET_DIR)/boot/
	ln -sf $(IMG_PREFIX)-uImage $(TARGET_DIR)/boot/uImage
   endif
   ifneq ($(CONFIG_TARGET_ROOTFS_INCLUDE_ZIMAGE),)
	$(CP) $(BIN_DIR)/$(IMG_PREFIX)-zImage $(TARGET_DIR)/boot/
	ln -sf $(IMG_PREFIX)-zImage $(TARGET_DIR)/boot/zImage
   endif
 endif

 ifneq ($(CONFIG_TARGET_ROOTFS_INCLUDE_DTB),)
	$(INSTALL_DIR) $(TARGET_DIR)/boot
  ifneq ($(1),)
	$(CP) $(BIN_DIR)/$(IMG_PREFIX)-$(1).dtb $(TARGET_DIR)/boot/
	ln -sf $(IMG_PREFIX)-$(1).dtb $(TARGET_DIR)/boot/$(1).dtb
  endif
 endif
endef

define Image/Build/jffs2-nand-2048-128k
	dd if=$(KDIR)/root.$(1) of=$(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-$(1).img \
		bs=2048 conv=sync
endef

define Image/Build/squashfs
	$(call prepare_generic_squashfs,$(KDIR)/root.squashfs)
	( \
		dd if=$(KDIR)/zImage bs=4096k conv=sync; \
		dd if=$(KDIR)/root.$(1) bs=128k conv=sync; \
	) > $(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-$(1).img
endef

define Image/Build/ubifs

 ifneq ($($(PROFILE)_UBIFS_OPTS),)
	$(CP) $(KDIR)/root.ubifs $(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-rootfs.ubifs
 endif
endef

define Image/Build/ubi

 ifneq ($($(PROFILE)_UBI_OPTS),)
	$(CP) $(KDIR)/root.ubi $(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-rootfs.ubi
 endif
endef

DOCKSTAR_UBIFS_OPTS:="-m 2048 -e 126KiB -c 4096"
DOCKSTAR_UBI_OPTS:="-m 2048 -p 128KiB -s 512"

GOFLEXHOME_UBIFS_OPTS:="-m 2048 -e 126KiB -c 4096"
GOFLEXHOME_UBI_OPTS:="-m 2048 -p 128KiB -s 512"

GOFLEXNET_UBIFS_OPTS:="-m 2048 -e 126KiB -c 4096"
GOFLEXNET_UBI_OPTS:="-m 2048 -p 128KiB -s 512"

Image/BuildKernel/Template/Generic=$(call Image/BuildKernel/Template)
Image/InstallKernel/Template/Generic=$(call Image/InstallKernel/Template)

IB62X0_UBIFS_OPTS:="-m 2048 -e 126KiB -c 4096"
IB62X0_UBI_OPTS:="-m 2048 -p 128KiB -s 512"

Image/BuildKernel/Template/IB62X0=$(call Image/BuildKernel/Template,ib62x0)
Image/InstallKernel/Template/IB62X0=$(call Image/InstallKernel/Template,ib62x0)

ICONNECT_UBIFS_OPTS:="-m 2048 -e 126KiB -c 4096"
ICONNECT_UBI_OPTS:="-m 2048 -p 128KiB -s 512"

Image/BuildKernel/Template/ICONNECT=$(call Image/BuildKernel/Template,iconnect)
Image/InstallKernel/Template/ICONNECT=$(call Image/InstallKernel/Template,iconnect)

IOMEGA_IX2_200_UBIFS_OPTS:="-m 512 -e 15872 -c 2048"
IOMEGA_IX2_200_UBI_OPTS:="-m 512 -p 16384 -s 256"

Image/BuildKernel/Template/IOMEGA_IX2_200=$(call Image/BuildKernel/Template,iomega_ix2_200)
Image/InstallKernel/Template/IOMEGA_IX2_200=$(call Image/InstallKernel/Template,iomega_ix2_200)

NSA310S_UBIFS_OPTS:="-m 2048 -e 126KiB -c 4096"
NSA310S_UBI_OPTS:="-m 2048 -p 128KiB -s 512"

Image/BuildKernel/Template/NSA310S=$(call Image/BuildKernel/Template,nsa310s)
Image/InstallKernel/Template/NSA310S=$(call Image/InstallKernel/Template,nsa310s)

POGOE02_UBIFS_OPTS:="-m 2048 -e 126KiB -c 4096"
POGOE02_UBI_OPTS:="-m 2048 -p 128KiB -s 512"

Image/BuildKernel/Template/POGOE02=$(call Image/BuildKernel/Template,pogo_e02)
Image/InstallKernel/Template/POGOE02=$(call Image/InstallKernel/Template,pogo_e02)

SHEEVAPLUG_UBIFS_OPTS:="-m 2048 -e 126KiB -c 4096"
SHEEVAPLUG_UBI_OPTS:="-m 2048 -p 128KiB -s 512"

Image/BuildKernel/Template/SHEEVAPLUG=$(call Image/BuildKernel/Template,sheevaplug)
Image/InstallKernel/Template/SHEEVAPLUG=$(call Image/InstallKernel/Template,sheevaplug)

SHEEVAPLUGSATA_UBIFS_OPTS:="-m 2048 -e 126KiB -c 4096"
SHEEVAPLUGSATA_UBI_OPTS:="-m 2048 -p 128KiB -s 512"

Image/BuildKernel/Template/SHEEVAPLUGSATA=$(call Image/BuildKernel/Template,sheevaplug-esata)
Image/InstallKernel/Template/SHEEVAPLUGSATA=$(call Image/InstallKernel/Template,sheevaplug-esata)

GuruplugServerPlus_UBIFS_OPTS:="-m 2048 -e 126KiB -c 4096"
GuruplugServerPlus_UBI_OPTS:="-m 2048 -p 128KiB -s 512"

Image/BuildKernel/Template/GuruplugServerPlus=$(call Image/BuildKernel/Template,guruplug-server-plus)
Image/InstallKernel/Template/GuruplugServerPlus=$(call Image/InstallKernel/Template,guruplug-server-plus)

Topkick1281P2_UBIFS_OPTS:="-m 2048 -e 126KiB -c 4096"
Topkick1281P2_UBI_OPTS:="-m 2048 -p 128KiB -s 512"

Image/BuildKernel/Template/Topkick1281P2=$(call Image/BuildKernel/Template,topkick)
Image/InstallKernel/Template/Topkick1281P2=$(call Image/InstallKernel/Template,topkick)

define Image/BuildKernel
	$(call Image/BuildKernel/Template/$(PROFILE))
endef

define Image/InstallKernel
	$(call Image/InstallKernel/Template/$(PROFILE))
endef

define Image/Build
	$(if $(Image/Build/$(1)), \
		$(call Image/Build/$(1),$(1)), \
		$(CP) $(KDIR)/root.$(1) $(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-$(1).img \
	)
endef

$(eval $(call BuildImage))
