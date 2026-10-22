# 操作记录

- 2026-10-22：按用户要求将最新 readme_cn.md 同步翻译到英文 readme.md，保留已有对应译文，补充 20261022 说明、Windows 64 位验证表述及 MXE 构建命令；命令参数与中文原文一致，仅翻译 Shell 注释。差异格式与命令内容核对通过，未执行构建或创建提交。

- 2026-10-22：按用户要求恢复 PulseView NSIS 模板至 HEAD，PulseView 与 sigrok-cli 子模块均保持干净；将 Makefile 的安装路径替换统一用于两个构建目录生成的 NSIS 文件。使用 Makefile 的配置命令和导出环境完成 PulseView CMake 配置；复用现有安装产物实际生成两个 64 位 installer 和两个 portable，解包及 ZIP CRC 检查通过，四包中的 25 个 Python/Zadig 文件均与源文件哈希一致。四种配置的 all 及四种打包目标共 20 组预览与 Shell 语法检查通过，未重编译全部组件、验证 Windows 实机或创建提交。

- 2026-10-22：按用户要求恢复 sigrok-cli 的 NSIS 模板至 HEAD，子模块状态干净；原修改仅涉及 Zadig/Python 打包来源和卸载清单。将原始模板所需的 Zadig 缓存及 Python/runtime 路径适配移到主仓库 Makefile，仅修改构建目录生成的 NSIS 文件。复用安装产物实际生成 64 位 sigrok-cli 安装包，解包核对全部 25 个 Python/Zadig 文件哈希一致；四种配置完整 all 命令预览、12 组打包目标预览与 Shell 语法检查通过。未重新编译全部组件，未创建提交。

- 2026-10-22：按用户要求，将主仓库剩余 .gitignore、FileAssociation.nsh、pyconfig.patch 和 readme_cn.md 一并合并到上次提交，保留原有许可与补丁所需 CRLF；检查忽略规则已覆盖构建目录，无需增加规则。子模块不创建提交、不更改提交引用，保留所有本地修改。

- 2026-10-22：按用户要求撤销 libsigrok 子模块本地提交，将 HEAD 恢复到 origin/HEAD（0bc24877），保留驱动源码、暂存内容及其他本地修改；修正主仓库提交中的子模块引用。后续不提交子模块。

- 2026-10-22：按用户要求以 codex 作者提交本次修复；先提交 libsigrok 子模块的 U3Pro32 源码及构建注册，再提交主仓库子模块引用、MinGW Makefile 和操作记录。检查现有 .gitignore，无需新增忽略规则；保留其他未提交改动，不提交构建产物。

- 2026-10-22：修复 MinGW 构建缺少 U3Pro32 protocol.c：从相邻 libsigrok 开发仓库原样恢复 api.c、protocol.c、protocol.h，并在子模块中暂存，保留版权及许可。将 clean 的 git clean -dfx 改为 -dfX，仅清理忽略的生成文件，防止删除未跟踪源码。实际完成 64 位 MinGW libsigrok 及 C++ 绑定编译链接；临时仓库验证未跟踪源码保留、忽略产物删除，清理命令预览、Shell 语法及差异格式检查通过。未执行完整应用构建、打包或 Windows 实机验证。

- 2026-10-22：按 docs/REVIEW.md 检查 MinGW Makefile 依赖和并行调度，确认默认子 Make -j4 会重置 jobserver（临时双分支工程复现），并记录 PHONY 导致重复配置/安装、跨架构共用源码 autogen 的竞争风险及手册生成可并行的优化点；现有应用、运行库及打包依赖总体完整。仅更新审查记录，保留最近 10 条，未修改 Makefile、执行实际构建或联网。

- 2026-10-22：参考用户提供的官方式安装树和脚本，将 Python34 SDK/根目录运行文件整理为 Python/include、Python/libs、Python/runtime，并生成 runtime-uninstall.nsh；保留 Python 3.4 ABI，按用户选择 Zadig 仅保留 downloads 缓存。更新 python3.pc、两个 NSIS 模板及 portable 规则，配置 PulseView 时刷新 pkg-config 缓存以消除旧 Python34 绝对路径。旧布局移至临时备份；实际重建 libsigrokdecode、PulseView、sigrok-cli 并重打四个发布包（复用 libsigrok/固件/示例安装产物）。两个 portable CRC/运行文件哈希、两个 installer 完整性与运行文件内容、四种配置预览及 shell 语法检查通过，未验证 Windows 实机运行。

- 2026-10-22：Zadig 仅保留 downloads 下载缓存，删除 Makefile 向 PREFIX 复制的中间文件目标；portable 直接取缓存，两个子模块 NSIS 模板支持 ZADIG_DIR 参数（未指定时兼容 CROSS），Makefile 传入绝对缓存路径。核对内容相同后删除现有 sr_ming 两个 Zadig EXE；临时使用完整 NSIS 脚本构建两个 installer 成功，实际重打两个 portable 并逐字节核对 Zadig 缓存，四种配置预览及 shell 语法通过，未重编译应用。
