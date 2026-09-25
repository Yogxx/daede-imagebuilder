# imagebuilder

[英文](#english)

## English

Build an ImmortalWrt x86/64 KVM test image with
[`luci-app-daede`](https://github.com/kenzok8/openwrt-daede) installed by
default, plus the runtime dependencies needed by `dae` / `daed`.

### Default Image

- Version: ImmortalWrt `25.12-SNAPSHOT`
- Target: `x86/64`
- Profile: `generic`
- Rootfs partition: `1024` MB
- ImageBuilder URL:
  `https://downloads.immortalwrt.org/releases/25.12-SNAPSHOT/targets/x86/64/immortalwrt-imagebuilder-25.12-SNAPSHOT-x86-64.Linux-x86_64.tar.zst`
- Shortcut:
  [`kenzok8/openwrt-daede`](https://github.com/kenzok8/openwrt-daede)

### Default Packages

The workflow downloads the matching `luci-app-daede-*-x86_64.apk` from the
[`kenzok8/openwrt-daede`](https://github.com/kenzok8/openwrt-daede) GitHub
Release, places it in ImageBuilder's local package directory, and installs these
packages:

```text
luci
luci-i18n-base-zh-cn
luci-i18n-package-manager-zh-cn
luci-app-daede
kmod-sched-core
kmod-sched-bpf
kmod-veth
kmod-xdp-sockets-diag
curl
nano
```

ImageBuilder does not compile the LuCI app from source. This repository bakes the
prebuilt `luci-app-daede` APK into the image. `dae` / `daed` and kernel
dependencies still come from the selected ImmortalWrt feed. If a dependency is
missing from that feed, the build will fail.

The workflow runs `make manifest` before building the image. This catches common
feed problems earlier, especially:

- snapshot ImageBuilder and package feeds are out of sync
- a required `kmod-*` dependency is missing for the selected target and kernel
- the `luci-app-daede` release APK architecture does not match the selected
  target

About BTF:

- ImmortalWrt 25.12 kernels enable `CONFIG_DEBUG_INFO_BTF` by default. `dae` /
  `daed` read BTF directly from `/sys/kernel/btf/vmlinux` at runtime and do not
  require a separate `vmlinux-btf` package.
- Do not add `vmlinux-btf` to `EXTRA_PACKAGES`. It is usually not published in
  the feed, and ImageBuilder cannot build it.
- If you target an older OpenWrt release whose kernel lacks built-in BTF, build
  `vmlinux-btf` with a full SDK build first.

### First Boot Defaults

The generated image applies these defaults on first boot:

- LAN IP: `192.168.3.252/24`
- Gateway: `192.168.3.254`
- DNS: `192.168.3.254`, `223.5.5.5`
- SSH port: `9167`

No root password is written into this repository or the generated image. The
image keeps the OpenWrt default empty root password, so the first LuCI/console
login should set a new password. For unattended access, inject an SSH public key
using a private workflow/secret-based step.

### Build

Run the `Build daede image` workflow manually from GitHub Actions.

The workflow uploads generated images as an artifact. When `publish_release` is
set to `true`, it also publishes a GitHub release.

Common inputs:

- `publish_release`: publish the generated image to GitHub Releases, defaults to
  `false`
- `imagebuilder_url`: ImmortalWrt ImageBuilder URL, defaults to `25.12-SNAPSHOT`
  x86/64
- `preflight`: run the package manifest check before building, defaults to
  `true`
- `rootfs_partsize`: rootfs partition size, defaults to `1024` MB
- `install_daede`: bake `luci-app-daede` into the image, defaults to `true`
- `daede_release_tag`: `luci-app-daede` release tag to use, defaults to `latest`
- `daede_apk_url`: direct APK download URL; when set, it takes priority

For the normal x86/64 build, leave the inputs unchanged and run the workflow.
The generated image will include `luci-app-daede`.

You can override the daede APK source with environment variables:

- `DAEDE_RELEASE_TAG`: defaults to `latest`
- `DAEDE_ARCH`: defaults to `x86_64`
- `DAEDE_APK_URL`: direct APK URL override
- `INSTALL_DAEDE`: set to `0` to skip baking daede into the image
