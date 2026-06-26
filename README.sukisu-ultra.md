# SukiSU-Ultra / SUSFS integration

This kernel tree is wired to build with the SukiSU-Ultra `builtin` kernel source layout.

The upstream setup command is:

```sh
curl -LSs "https://raw.githubusercontent.com/SukiSU-Ultra/SukiSU-Ultra/main/kernel/setup.sh" | bash -s builtin
```

The repository keeps the kernel build-system wiring in-tree:

- `drivers/Makefile` builds `drivers/kernelsu/` when `CONFIG_KSU` is enabled.
- `drivers/Kconfig` sources `drivers/kernelsu/Kconfig` so SukiSU-Ultra options are visible to Kconfig.
- `drivers/kernelsu` is a symlink to `../KernelSU/kernel`.

For local builds, populate `KernelSU` before running `make *defconfig`:

```sh
git clone --depth=1 --branch builtin https://github.com/SukiSU-Ultra/SukiSU-Ultra KernelSU
```

The GitHub Actions workflow does this automatically before configuring the kernel, so CI has `drivers/kernelsu/Kconfig` available during `defconfig`.

The workflow mirrors the upstream setup script behavior: it clones the SukiSU-Ultra repository first, then fetches the requested `sukisu_ref` and checks out `FETCH_HEAD`. If that ref is unavailable, the workflow continues with the repository default branch instead of failing before configuration.

The workflow also sets `CONFIG_LOCALVERSION="-SukiSU"` and disables `CONFIG_LOCALVERSION_AUTO` after defconfig so the generated `UTS_RELEASE` stays below the Linux 64-character limit when the SukiSU-Ultra Git checkout is present.

The workflow also uses Android `prebuilts/misc` for `DTC_EXT`, `DTC_OVERLAY_TEST_EXT`, and `mkdtimg`, matching the kernel build config expectations for Google device-tree overlays.

The default workflow `sukisu_ref` is `builtin` because SukiSU-Ultra currently exposes `main`, `builtin`, `dev`, and `old` branches; `susfs-main` is documented as an experimental setup argument but is not a remote branch to check out directly.
