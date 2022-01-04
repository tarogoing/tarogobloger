# 安卓ro.build.fingerprint cannot exceed 91 bytes

### vote 2 down vote favorite

```
I'm building an android rom from the android source code but after about 5 minutes it gives this error.

error: ro.build.fingerprint cannot exceed 91 bytes: Android/mini_emulator_x86/mini-emulator-x86:5.0.555/AOSP/username02280306:userdebug/test-keys (97)
make: *** [out/target/product/mini-emulator-x86/system/build.prop] Error 1
make: *** Deleting file `out/target/product/mini-emulator-x86/system/build.prop'
make: *** Waiting for unfinished jobs....
How do I increase the ro.build.fingerprint size limit?

Plus I'm building on a Mac.

java android build terminal android-source
share|improve this question
asked Feb 28 '15 at 1:20

shoriwa-shaun-benjamin
15411
add a comment | 
1 Answer

active oldest votes
up vote 5 down vote
Edit build/tools/post_process_props.py. Change lines as follows:

PROP_NAME_MAX = 31
# PROP_VALUE_MAX = 91
PROP_VALUE_MAX = 128
Edit bionic/libc/include/sys/system_properties.h. Change lines as follows:

#define PROP_NAME_MAX   32
// #define PROP_VALUE_MAX  92
#define PROP_VALUE_MAX  128
Do

make clean
make
You can also run the second make command in parallel using syntax such as
```

