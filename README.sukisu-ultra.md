# SukiSU-Ultra / SUSFS 集成说明

本内核源码树已按 SukiSU-Ultra 的 `builtin` 内核源码布局接入，并针对 4.14 Non-GKI 内核启用 Manual hook 与 SUSFS。

## 上游接入方式

SukiSU-Ultra 上游文档要求 Non-GKI / 内置内核使用 `builtin` 参数拉取内核侧源码：

```sh
curl -LSs "https://raw.githubusercontent.com/SukiSU-Ultra/SukiSU-Ultra/main/kernel/setup.sh" | bash -s builtin
```

仓库中的固定接入点如下：

- `drivers/Kconfig` 会 source `drivers/kernelsu/Kconfig`，因此 SukiSU-Ultra 配置项会出现在 Kconfig 中。
- `drivers/Makefile` 会在启用 `CONFIG_KSU` 时编译 `drivers/kernelsu`。
- `drivers/kernelsu` 是指向 `../KernelSU/kernel` 的符号链接。
- `arch/arm64/configs/floral_defconfig` 和 `arch/arm64/configs/vendor/atoll_defconfig` 默认启用 `CONFIG_KSU`、`CONFIG_KSU_MANUAL_HOOK` 与基础 SUSFS 选项。

本地编译前需要先准备 `KernelSU` 目录：

```sh
git clone --depth=1 --branch builtin https://github.com/SukiSU-Ultra/SukiSU-Ultra KernelSU
```

## Manual hook

当前树按 Manual hook 接入，不依赖 kprobes。为避免手动 hook 与 kprobe hook 同时生效，`floral_defconfig` 和 `atoll_defconfig` 中已关闭 `CONFIG_KPROBES`，并启用 `CONFIG_KSU_MANUAL_HOOK`。

已接入的 Manual hook 位置包括：

- `fs/exec.c`：`do_execveat_common()`
- `fs/open.c`：`faccessat` syscall
- `fs/read_write.c`：`vfs_read()`
- `fs/stat.c`：`vfs_fstatat()`
- `drivers/input/input.c`：`input_handle_event()` 安全模式入口
- `fs/devpts/inode.c`：`devpts_get_priv()` 兼容入口
- `kernel/sys.c`：`setresuid()`，用于 SUSFS 模式下给管理器安装 KernelSU fd
- `kernel/reboot.c`：`reboot()` supercall 入口

## SUSFS

已导入 `susfs4ksu` 的 `kernel-4.14` 分支补丁，包含：

- `include/linux/susfs.h`
- `include/linux/susfs_def.h`
- `include/linux/sus_su.h`
- `fs/susfs.c`
- `fs/sus_su.c`
- 对 VFS、procfs、overlayfs、namespace、stat、kallsyms 等位置的 SUSFS hook

这样可以解决启用 `CONFIG_KSU_SUSFS` 后编译 SukiSU-Ultra 时缺少 `linux/susfs.h` / `linux/susfs_def.h` 的问题。

## GitHub Actions 注意事项

工作流中的默认 `sukisu_ref` 仍应使用 `builtin`，因为 SukiSU-Ultra 远端实际提供的是 `main`、`builtin`、`dev`、`old` 等分支；`susfs-main` 是 setup 脚本的实验参数，并不是可直接 checkout 的远端分支。

工作流在 defconfig 后还会设置 `CONFIG_LOCALVERSION="-SukiSU"` 并关闭 `CONFIG_LOCALVERSION_AUTO`，避免存在 SukiSU-Ultra Git checkout 时生成的 `UTS_RELEASE` 超过 Linux 64 字符限制。
