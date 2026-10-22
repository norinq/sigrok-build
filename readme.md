# sigrok Build Scripts

I've had enough of using the official sigrok-util scripts for cross-platform builds. Before every build, I have to delete and clone each Git repository again, then rebuild everything from scratch. This wastes a lot of time, and when I tried building with 16 parallel jobs, my Linux system even ran out of memory (OOM).

## About This Project

- Perhaps because the original project dates back quite a while, it uses a lot of complicated tooling. Getting it to build takes considerable effort and many manual steps. This project aims to provide a one-command build using Makefiles and Git submodules in place of the original Bash scripts.
- Since I'm rather lazy, I've written and tested scripts for native builds on Debian and cross-compiling Windows versions from Debian. That should cover about 80% of what most people need.

## Before You Start

- This project also updates those ancient libraries to newer versions. That brings something fresh, but also some uncertainty. If you run into problems, feel free to open an issue. When I'm not too busy trying to make ends meet, I'll be happy to help my tiny handful of followers. Of course, you can also try using AI to investigate and solve problems yourself.
- If you use AI to diagnose and fix a problem, please don't rush to submit a PR. You can also try reminding the AI to keep code diffs as small as possible—otherwise, reviewing and merging them becomes a real headache.

## 20261022

- The AI-generated Makefile is a bit messy, and I'm not sure where to start cleaning it up. As everyone knows, if the code works, don't touch it. I'll leave the first version as it is for now; my main goal is still to add device support to libsigrok.
- Following the instructions in the official sigrok-util/cross-compile/mingw/README, I've verified that the 64-bit Windows build runs successfully.

```bash
git clone https://github.com/mxe/mxe.git $HOME/mxe
cd $HOME/mxe
make MXE_TARGETS=x86_64-w64-mingw32.static.posix \
   MXE_PLUGIN_DIRS=plugins/examples/qt5-freeze \
   gcc glib libzip libusb1 libftdi1 hidapi glibmm qtbase qtimageformats \
   qtsvg qttranslations boost check gendef libieee1284 \
   qtbase_CONFIGURE_OPTS='-no-sql-mysql'
# Switch to this project's directory.
make -f ./cross-compile/mingw/makefile
```
