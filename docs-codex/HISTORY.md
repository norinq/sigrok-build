# 操作记录

- 2026-10-22：按用户要求创建本地提交，提交 MinGW Python 3.13.17 构建与打包适配、直接变量别名清理及操作记录，作者使用 codex。检查 .gitignore 已覆盖 build 产物，无需增加规则；保留现有版权和许可，不提交 dependencies 的本地改动及构建输出。

- 2026-10-22：将 MinGW Makefile 的 Python 切换为 3.13.17，按架构下载官方嵌入包及同版本 NuGet 开发头文件，生成 MinGW 导入库和 embedding pkg-config 文件；运行目录包含 DLL、PYD、标准库、路径配置及许可，NSIS 仅在构建目录适配完整运行文件和卸载清单。为避免写入 dependencies，使用构建目录中的源码副本重编译 libsigrokdecode、sigrok-cli，重新构建 PulseView，实际生成两个 64 位 Release installer 与两个 portable。两个 EXE 均导入 python313.dll；四包完整性及全部 31 个 Python 运行文件内容核对通过，四种配置 all 命令预览与 Shell 语法检查通过。未修改 dependencies，未验证 Windows 实机、实际构建 32 位/Debug 或创建提交。

- 2026-10-22：清理 MinGW Makefile 的直接变量别名，将 PATCHDIR、PYTHON_DOWNLOADS 的全部引用分别统一为 PATH_MAKEFILE、DOWNLOADDIR 并删除别名定义；保留具有绝对路径转换作用的变量及用户已有修改。四种架构/调试配置的 Python、安装包、免安装包和 clean-mxe 命令预览与修改前一致；未修改 dependencies、执行实际构建或创建提交。

- 2026-10-22：检查 MinGW Makefile 的 Python 升级可行性，确认现有 Python 3.4 SDK、导入库、运行时与 NSIS 打包规则均需配套迁移，不能仅替换版本号；核对 Python 官方最新稳定版为 3.15.1。只读检查 dependencies，未修改 Makefile、执行构建或创建提交；尚未验证新版编译及 Windows 运行兼容性。

- 2026-10-22：按用户要求将最新 readme_cn.md 同步翻译到英文 readme.md，保留已有对应译文，补充 20261022 说明、Windows 64 位验证表述及 MXE 构建命令；命令参数与中文原文一致，仅翻译 Shell 注释。差异格式与命令内容核对通过，未执行构建或创建提交。

- 2026-10-22：按用户要求恢复 PulseView NSIS 模板至 HEAD，PulseView 与 sigrok-cli 子模块均保持干净；将 Makefile 的安装路径替换统一用于两个构建目录生成的 NSIS 文件。使用 Makefile 的配置命令和导出环境完成 PulseView CMake 配置；复用现有安装产物实际生成两个 64 位 installer 和两个 portable，解包及 ZIP CRC 检查通过，四包中的 25 个 Python/Zadig 文件均与源文件哈希一致。四种配置的 all 及四种打包目标共 20 组预览与 Shell 语法检查通过，未重编译全部组件、验证 Windows 实机或创建提交。

- 2026-10-22：按用户要求恢复 sigrok-cli 的 NSIS 模板至 HEAD，子模块状态干净；原修改仅涉及 Zadig/Python 打包来源和卸载清单。将原始模板所需的 Zadig 缓存及 Python/runtime 路径适配移到主仓库 Makefile，仅修改构建目录生成的 NSIS 文件。复用安装产物实际生成 64 位 sigrok-cli 安装包，解包核对全部 25 个 Python/Zadig 文件哈希一致；四种配置完整 all 命令预览、12 组打包目标预览与 Shell 语法检查通过。未重新编译全部组件，未创建提交。

- 2026-10-22：按用户要求，将主仓库剩余 .gitignore、FileAssociation.nsh、pyconfig.patch 和 readme_cn.md 一并合并到上次提交，保留原有许可与补丁所需 CRLF；检查忽略规则已覆盖构建目录，无需增加规则。子模块不创建提交、不更改提交引用，保留所有本地修改。

- 2026-10-22：按用户要求撤销 libsigrok 子模块本地提交，将 HEAD 恢复到 origin/HEAD（0bc24877），保留驱动源码、暂存内容及其他本地修改；修正主仓库提交中的子模块引用。后续不提交子模块。

- 2026-10-22：按用户要求以 codex 作者提交本次修复；先提交 libsigrok 子模块的 U3Pro32 源码及构建注册，再提交主仓库子模块引用、MinGW Makefile 和操作记录。检查现有 .gitignore，无需新增忽略规则；保留其他未提交改动，不提交构建产物。
