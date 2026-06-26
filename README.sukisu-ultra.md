# SukiSU-Ultra / SUSFS integration

This kernel tree is wired for the SukiSU-Ultra `susfs-main` kernel source layout.

The upstream setup command is:

```sh
curl -LSs "https://raw.githubusercontent.com/SukiSU-Ultra/SukiSU-Ultra/main/kernel/setup.sh" | bash -s susfs-main
```

In this environment, direct `curl`/`git clone` access to GitHub was blocked by the proxy, so the kernel-side wiring performed by the setup script was applied manually:

- `drivers/Makefile` builds `drivers/kernelsu/` when `CONFIG_KSU` is enabled.
- `drivers/Kconfig` sources `drivers/kernelsu/Kconfig`.
- `drivers/kernelsu` is a symlink to `../KernelSU/kernel`, matching the upstream setup script expectation that the SukiSU-Ultra repository is checked out at the repository root as `KernelSU`.

Before building from a fresh checkout, populate `KernelSU` with SukiSU-Ultra and check out the SUSFS branch:

```sh
git clone https://github.com/SukiSU-Ultra/SukiSU-Ultra KernelSU
cd KernelSU
git checkout susfs-main
```

Then enable the desired SukiSU/SUSFS Kconfig options in the target defconfig.
