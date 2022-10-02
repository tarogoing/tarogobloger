# Android和kernel编译版本显示更改
### 1，android版本：
在android/build/core/version_defaults.mk中红色部分
```
ifeq "" "$(BUILD_NUMBER)"
  # BUILD_NUMBER should be set to the source control value that
  # represents the current state of the source code.  E.g., a
  # perforce changelist number or a git hash.  Can be an arbitrary string
  # (to allow for source control that uses something other than numbers),
  # but must be a single word and a valid file name.
  #
  # If no BUILD_NUMBER is set, create a useful "I am an engineering build
  # from this date/time" value.  Make it start with a non-digit so that
  # anyone trying to parse it as an integer will probably get "0".
  BUILD_NUMBER := user.root.$(shell date +%Y%m%d.%H%M%S)
endif
```
### 2，kernel版本：
在kernel/script/mkcompile_h中红色部分：
```
  echo #define UTS_MACHINE "$ARCH"
  echo #define UTS_VERSION "`echo $UTS_VERSION | $UTS_TRUNCATE`"
  echo #define LINUX_COMPILE_BY "`echo ZT-10003G`"
  echo #define LINUX_COMPILE_HOST "`echo ZT-10003G`"
  echo #define LINUX_COMPILER "`$CC -v 2>&1 | tail -n 1`"
) > .tmpcompile
```