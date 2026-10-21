# sigrok Build Scripts

I've had enough of using the official sigrok-util scripts for cross-platform builds. Before every build, I have to delete and clone each Git repository again, then rebuild everything from scratch. This wastes a lot of time, and when I tried building with 16 parallel jobs, my Linux system even ran out of memory (OOM).

## About This Project

- Perhaps because the original project dates back quite a while, it uses a lot of complicated tooling. Getting it to build takes considerable effort and many manual steps. This project aims to provide a one-command build using Makefiles and Git submodules in place of the original Bash scripts.
- Since I'm rather lazy, I've written and tested scripts for native builds on Debian and cross-compiling Windows versions from Debian. That should cover about 80% of what most people need.

## Before You Start

- This project also updates those ancient libraries to newer versions. That brings something fresh, but also some uncertainty. If you run into problems, feel free to open an issue. When I'm not too busy trying to make ends meet, I'll be happy to help my tiny handful of followers. Of course, you can also try using AI to investigate and solve problems yourself.
- If you use AI to diagnose and fix a problem, please don't rush to submit a PR. You can also try reminding the AI to keep code diffs as small as possible—otherwise, reviewing and merging them becomes a real headache.
