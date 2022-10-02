# make menuconfig出现一大堆未定义的错误
运行make menuconfig后出现一大堆：scripts/kconfig/mconf.o：在函数‘show_help’中：mconf.c:(.text+0x704)：对‘stdscr’未定义的引用

运行
```
make menuconfig
```
后出现一大堆：
```
scripts/kconfig/mconf.o：在函数‘show_help’中：
mconf.c:(.text+0x704)：对‘stdscr’未定义的引用
scripts/kconfig/lxdialog/checklist.o：在函数‘print_arrows’中：
checklist.c:(.text+0x40)：对‘wmove’未定义的引用
checklist.c:(.text+0x60)：对‘acs_map’未定义的引用
checklist.c:(.text+0x68)：对‘waddch’未定义的引用
checklist.c:(.text+0x7a)：对‘waddnstr’未定义的引用
checklist.c:(.text+0x8a)：对‘wmove’未定义的引用
checklist.c:(.text+0xb4)：对‘acs_map’未定义的引用
checklist.c:(.text+0xbc)：对‘waddch’未定义的引用
checklist.c:(.text+0x10b)：对‘acs_map’未定义的引用
checklist.c:(.text+0x113)：对‘waddch’未定义的引用

。。。。。。。。。。。。。。。。。。
```
的错误。

解决：
```
sudo apt-get install libncursesw5-dev
```

注：非lib64ncurses5-dev 或者 lib32ncurses5-dev 