# Android设备的userdata.img的打包分析
### 核心打包流程
下面使用一个在hikey960打包一个文件系统为f2fs的userdata.img作为例子
#### 用户通过以下命令进行打包
make BOARD_USERDATAIMAGE_FILE_SYSTEM_TYPE=f2fs TARGET_USERIMAGES_USE_F2FS=true userdata -j4
这个命令首先会进入 build/core/main.mk
.PHONY: userdataimage
userdataimage: $(INSTALLED_USERDATAIMAGE_TARGET)

这个匿名目标会跳转到 build/core/Makefile
\$(INSTALLED_USERDATAIMAGE_TARGET): $(INSTALLED_USERDATAIMAGE_TARGET_DEPS)
    $(build-userdataimage-target)
INSTALLED_USERDATAIMAGE_TARGET 的编译依赖于 INSTALLED_USERDATAIMAGE_TARGET_DEPS，满足了依赖条件后执行 $(build-userdataimage-target) 进行打包。
变量的分析
INSTALLED_USERDATAIMAGE_TARGET_DEPS 它的值由如下的变量构成：
INSTALLED_USERDATAIMAGE_TARGET_DEPS := \
    $(INTERNAL_USERIMAGES_DEPS) \
    $(INTERNAL_USERDATAIMAGE_FILES) \
    $(BUILD_IMAGE_SRCS)
核心变量是 INTERNAL_USERIMAGES_DEPS， 用于进行解包和重新打包，它的构成如下，分别为用于解包的 simg2img 命令，以及 EXT4 工具组，以及 F2FS 工具组。
ifeq ($(INTERNAL_USERIMAGES_USE_EXT),true)
INTERNAL_USERIMAGES_DEPS := $(SIMG2IMG)
INTERNAL_USERIMAGES_DEPS += $(MKEXTUSERIMG) $(MAKE_EXT4FS) $(E2FSCK)
ifeq ($(TARGET_USERIMAGES_USE_F2FS),true)
INTERNAL_USERIMAGES_DEPS += $(MKF2FSUSERIMG) $(MAKE_F2FS)
endif
endif
因此这里 INTERNAL_USERIMAGES_DEPS 的展开后的值如下，其实就是一系列命令的保存点
\$(SIMG2IMG) $(MKEXTUSERIMG) $(MAKE_EXT4FS) $(E2FSCK) $(MKF2FSUSERIMG) $(MAKE_F2FS)
下面列出了这些值具体的含义 (部分值在 config.mk 中定义)，:
\$(SIMG2IMG) = $(HOST_OUT_EXECUTABLES)/simg2img = out/host/linux-x86/bin/simg2img
\$(MKEXTUSERIMG) = MKEXTUSERIMG := $(HOST_OUT_EXECUTABLES)/mkuserimg_mke2fs.sh = out/host/linux-x86/bin/mkuserimg_mke2fs.sh
\$(MAKE_EXT4FS) = $(HOST_OUT_EXECUTABLES)/mke2fs$(HOST_EXECUTABLE_SUFFIX) = \out/host/linux-x86/bin/mke2fs
\$(E2FSCK) = $(HOST_OUT_EXECUTABLES)/e2fsck$(HOST_EXECUTABLE_SUFFIX) = out/host/linux-x86/bin/e2fsck
\$(MKF2FSUSERIMG) = $(HOST_OUT_EXECUTABLES)/mkf2fsuserimg.sh = out/host/linux-x86/bin/mkf2fsuserimg.sh
\$(MAKE_F2FS) = $(HOST_OUT_EXECUTABLES)/make_f2fs$(HOST_EXECUTABLE_SUFFIX) = out/host/linux-x86/bin/make_f2fs
打包的具体流程
上面提及到，满足了相应的依赖条件之后，会通过 $(build-userdataimage-target) 进行打包，它的定义如下：
define build-userdataimage-target
  $(call pretty,"Target userdata fs image: $(INSTALLED_USERDATAIMAGE_TARGET)")
  @mkdir -p $(TARGET_OUT_DATA)
  @mkdir -p $(userdataimage_intermediates) && rm -rf $(userdataimage_intermediates)/userdata_image_info.txt
  $(call generate-image-prop-dictionary, $(userdataimage_intermediates)/userdata_image_info.txt,userdata,skip_fsck=true)
  $(hide) PATH=$(foreach p,$(INTERNAL_USERIMAGES_BINARY_PATHS),$(p):)$$PATH \
      build/make/tools/releasetools/build_image.py \
      $(TARGET_OUT_DATA) $(userdataimage_intermediates)/userdata_image_info.txt $(INSTALLED_USERDATAIMAGE_TARGET) $(TARGET_OUT)
  $(hide) $(call assert-max-image-size,$(INSTALLED_USERDATAIMAGE_TARGET),$(BOARD_USERDATAIMAGE_PARTITION_SIZE))
endef
#### 第一行打印debug的信息，打印出打包后文件的保存位置：
$(call pretty,"Target userdata fs image: $(INSTALLED_USERDATAIMAGE_TARGET)")
这里的 INSTALLED_USERDATAIMAGE_TARGET 一般的值类似如下路径：
INSTALLED_USERDATAIMAGE_TARGET=BUILT_USERDATAIMAGE_TARGET=out/target/product/hikey960/userdata.img
#### 第二、三行用于创建文件夹，删除上次使用的配置文件 userdata_image_info.txt。
\@mkdir -p $(TARGET_OUT_DATA)
\@mkdir -p $(userdataimage_intermediates) && rm -rf \$(userdataimage_intermediates)/userdata_image_info.txt
第四行用于重新生成配置文件 userdata_image_info.txt:
\$(call generate-image-prop-dictionary, \$(userdataimage_intermediates)/userdata_image_info.txt,userdata,skip_fsck=true)
其中 generate-image-prop-dictionary 的关键部分如下，echo 了一些信息到 $(1) 传入的其实就是 userdata_image_info.txt 的路径
define generate-image-prop-dictionary
...
\$(if $(filter $(2),userdata),\
    \$(if \$(BOARD_USERDATAIMAGE_FILE_SYSTEM_TYPE),\$(hide) echo "userdata_fs_type=$(BOAOARD_USERDATAIMAGE_FILE_SYSTEM_TYPE)" >> $(1))
    \$(if $(BOARD_USERDATAIMAGE_PARTITION_SIZE),$(hide) echo "userdata_size=$(BOARD_USERDATAIMAGE_PARTITION_SIZE)" >> $(1))
)
...
endef
其中 \$(BOARD_USERDATAIMAGE_FILE_SYSTEM_TYPE) 和 \$(BOARD_USERDATAIMAGE_PARTITION_SIZE)等值在编译的时候，或者 BoardConfig.mk 中指定了。
第四步先配置一下环境变量，然后调用一个python脚本 build_image.py 去打包
\$(hide) PATH=$(foreach p,$(INTERNAL_USERIMAGES_BINARY_PATHS),$(p):)$$PATH \
      build/make/tools/releasetools/build_image.py \
      $(TARGET_OUT_DATA) $(userdataimage_intermediates)/userdata_image_info.txt $(INSTALLED_USERDATAIMAGE_TARGET) $(TARGET_OUT)
第一行的 PATH=$(foreach p,$(INTERNAL_USERIMAGES_BINARY_PATHS),$(p):)$$PATH 主要作用是配置环境变量，将一些用于格式化和打包的工具如 mkfs_f2fs、mkf2fsuserimg.sh 等打包工具放入环境变量中，以便于直接调用。 第二行就是执行python脚本，传入4个参数，分别为：
\$(TARGET_OUT_DATA)=out/target/product/hikey960/data
\$(userdataimage_intermediates)/userdata_image_info.txt=out/target/product/hikey960/obj/PACKAGING/userdata_intermediates/userdata_image_info.txt
\$(INSTALLED_USERDATAIMAGE_TARGET)=out/target/product/hikey960/userdata.img
\$(TARGET_OUT)=out/target/product/hikey960/system
python脚本的核心是调用shell脚本 mkf2fsuserimg.sh，调用的形式如下：
mkf2fsuserimg.sh \
    out/target/product/hikey960/userdata.img \
    25845301248 \
    -f out/target/product/hikey960/data \
    -D out/target/product/hikey960/system \
    -s out/target/product/hikey960/obj/ETC/file_contexts.bin_intermediates/file_contexts.bin \
    -t data -L data
脚本 mkf2fsuserimg.sh 一般在 out/host/linux-x86-bin 可以找到它的定义:
\#!/bin/bash
#
\# To call this script, make sure make_f2fs is somewhere in PATH
...

MAKE_F2FS_CMD="make_f2fs -S $SIZE -f -O encrypt -O quota -O verity $MKFS_OPTS $OUTPUT_FILE"
echo $MAKE_F2FS_CMD
\$MAKE_F2FS_CMD
if [ $? -ne 0 ]; then
  exit 4
fi

SLOAD_F2FS_CMD="sload_f2fs -S $SLOAD_OPTS $OUTPUT_FILE"
echo $SLOAD_F2FS_CMD
\$SLOAD_F2FS_CMD
if [ $? -ne 0 ]; then
  rm -f $OUTPUT_FILE
  exit 4
fi
可以看到分别使用了 make_f2fs 对userdata.img进行格式化，然后使用 sload_f2fs 将预置app或者文件保存到userdata.img中
发布于 2018-12-22
https://zhuanlan.zhihu.com/p/53009043