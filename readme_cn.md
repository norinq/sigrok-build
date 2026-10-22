# sigrok综合编辑脚本
我真的受够了用官方的sigrok-util进行跨平台编译，每次编译前都要删除重新下载每个git仓库，然后从头编译，浪费了大量时间，甚至在我尝试16路并行编译的时候，直接让我的linux系统OOM

## 项目简介
- 可能因为项目创立的时间很早，所以用了大量复杂的插件，编译门槛也是很高，需要个人去做大量的操作，这个项目目的在于使用makefile脚本配合git submodule一键编译，而不是原项目的bash脚本
- 因为我比较懒，所以我编写并验证了在debian上编译本机和交叉编译windows版本软件的脚本，应该可以满足大家八成的需求了

## 用前注意
- 这个项目顺便把哪些上古的库都升级成了最新版库，带来新鲜感的同时也带来了不确定性，如果有遇到问题，欢迎大家提交issue，如果我没有疲于应付温饱，我会很乐意去帮我少的可怜的粉丝的。当然大家也可以自行尝试用AI分析解决问题
- 如果是用AI分析解决的问题，请不要轻易推送PR。另外可以试着提醒一下AI，修改代码时尽量减少diff差异，不然review和merge起来真的头大

## 20261022
- AI生成的makefile有点乱，想改却无从下手，大家都知道代码如果能跑起来就不要动它，第一版先这样了，我的主要目的还是改libsigrok添加设备的
- 参考官方sigrok-util/cross-compile/mingw/README 的说明, 已验证编译windows64位可以运行
```bash
git clone https://github.com/mxe/mxe.git $HOME/mxe
cd $HOME/mxe
make MXE_TARGETS=x86_64-w64-mingw32.static.posix \
   MXE_PLUGIN_DIRS=plugins/examples/qt5-freeze \
   gcc glib libzip libusb1 libftdi1 hidapi glibmm qtbase qtimageformats \
   qtsvg qttranslations boost check gendef libieee1284 \
   qtbase_CONFIGURE_OPTS='-no-sql-mysql'
# 切换到本项目路径
make -f ./cross-compile/mingw/makefile
```
