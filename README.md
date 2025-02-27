![boringtun logo banner](./banner.png)

# BoringTun

## Warning
Boringtun is currently undergoing a restructuring. You should probably not rely on or link to 
the master branch right now. Instead you should use the crates.io page.

[![crates.io](https://img.shields.io/crates/v/boringtun.svg)](https://crates.io/crates/boringtun)

BoringTun是WireGuard®协议的一种实现，旨在提高可移植性和速度。

BoringTun已成功部署在数百万iOS和Android消费设备以及数千台Cloudflare Linux服务器上。

该项目由两部分组成：

可执行文件boringtun，一个用于Linux和macOS的用户空间WireGuard实现。
库boringtun，可用于在各种平台（包括iOS和Android）上实现快速高效的WireGuard客户端应用程序。它实现了底层的WireGuard协议，没有网络或隧道栈，这些都可以用平台惯用的方式实现。

### Installation

您可以使用cargo安装此项目:

```
cargo install boringtun
```

### Building

- Library only: `cargo build --lib --no-default-features --release [--target $(TARGET_TRIPLE)]`
- Executable: `cargo build --bin boringtun-cli --release [--target $(TARGET_TRIPLE)]`

默认情况下，可执行文件放在 `./target/release` 文件夹中。您可以手动将其复制到所需的位置，或使用 `cargo install --bin boringtun --path .`进行安装

### Running

根据规范，开始隧道使用：

`boringtun-cli [-f/--foreground] INTERFACE-NAME`

然后可以使用 [wg](https://git.zx2c4.com/WireGuard/about/src/tools/man/wg.8), 作为常规的WireGuard隧道或任何其他工具来配置隧道。

可以与wg-quick一起使用 [wg-quick](https://git.zx2c4.com/WireGuard/about/src/tools/man/wg-quick.8) 通过将环境变量 `WG_QUICK_USERSPACE_IMPLEMENTATION` 设置为 `boringtun`.例如：

`sudo WG_QUICK_USERSPACE_IMPLEMENTATION=boringtun-cli WG_SUDO=1 wg-quick up CONFIGURATION`

### Testing

Testing this project has a few requirements:

- `sudo`: required to create tunnels. When you run `cargo test` you'll be prompted for your password.
- Docker: you can install it [here](https://www.docker.com/get-started). If you are on Ubuntu/Debian you can run `apt-get install docker.io`.

### Benchmarking

To benchmark this project you can run this command:

```
cargo +nightly bench
```

This command depends on the unstable `test` feature of the Rust compiler. As a result, you'll need to use the `nightly` channel of Rust when you run it.

## Supported platforms

Target triple                 |Binary|Library|                 |
------------------------------|:----:|:-----:|-----------------|
x86_64-unknown-linux-gnu      |  ✓   |   ✓   |[![Build Status](https://dev.azure.com/cloudflare-ps/wireguard-cf/_apis/build/status/cloudflare.boringtun?branchName=master&jobName=Linux%20armv7)](https://dev.azure.com/cloudflare-ps/wireguard-cf/_build/latest?definitionId=4&branchName=master)
aarch64-unknown-linux-gnu     |  ✓   |   ✓   |[![Build Status](https://dev.azure.com/cloudflare-ps/wireguard-cf/_apis/build/status/cloudflare.boringtun?branchName=master&jobName=Linux%20aarch64)](https://dev.azure.com/cloudflare-ps/wireguard-cf/_build/latest?definitionId=4&branchName=master)
armv7-unknown-linux-gnueabihf |  ✓   |   ✓   |[![Build Status](https://dev.azure.com/cloudflare-ps/wireguard-cf/_apis/build/status/cloudflare.boringtun?branchName=master&jobName=Linux%20armv7)](https://dev.azure.com/cloudflare-ps/wireguard-cf/_build/latest?definitionId=4&branchName=master)
x86_64-apple-darwin           |  ✓   |   ✓   |[![Build Status](https://dev.azure.com/cloudflare-ps/wireguard-cf/_apis/build/status/cloudflare.boringtun?branchName=master&jobName=macOS)](https://dev.azure.com/cloudflare-ps/wireguard-cf/_build/latest?definitionId=4&branchName=master)
x86_64-pc-windows-msvc        |      |   ✓   |[![Build Status](https://dev.azure.com/cloudflare-ps/wireguard-cf/_apis/build/status/cloudflare.boringtun?branchName=master&jobName=Windows)](https://dev.azure.com/cloudflare-ps/wireguard-cf/_build/latest?definitionId=4&branchName=master)
aarch64-apple-ios             |      |   ✓   |FFI bindings
armv7-apple-ios               |      |   ✓   |FFI bindings
armv7s-apple-ios              |      |   ✓   |FFI bindings
aarch64-linux-android         |      |   ✓   |JNI bindings
arm-linux-androideabi         |      |   ✓   |JNI bindings

<sub>Other platforms may be added in the future</sub>

#### Linux

`x86-64`, `aarch64` and `armv7` architectures are supported. The behaviour should be identical to that of [wireguard-go](https://git.zx2c4.com/wireguard-go/about/), with the following difference:

`boringtun` will drop privileges when started. When privileges are dropped it is not possible to set `fwmark`. If `fwmark` is required, such as when using `wg-quick`, instead running with `sudo`, give the executable the `CAP_NET_ADMIN` capability using: `sudo setcap cap_net_admin+epi boringtun`. Alternatively run with `--disable-drop-privileges` or set the environment variable `WG_SUDO=1`.

#### macOS

The behaviour is similar to that of [wireguard-go](https://git.zx2c4.com/wireguard-go/about/). Specifically the interface name must be `utun[0-9]+` for an explicit interface name or `utun` to have the kernel select the lowest available. If you choose `utun` as the interface name, and the environment variable `WG_TUN_NAME_FILE` is defined, then the actual name of the interface chosen by the kernel is written to the file specified by that variable.

---

#### FFI bindings

The library exposes a set of C ABI bindings, those are defined in the `wireguard_ffi.h` header file. The C bindings can be used with C/C++, Swift (using a bridging header) or C# (using [DLLImport](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.dllimportattribute?view=netcore-2.2) with [CallingConvention](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.dllimportattribute.callingconvention?view=netcore-2.2) set to `Cdecl`).

#### JNI bindings

The library exposes a set of Java Native Interface bindings, those are defined in `src/jni.rs`.

## License

The project is licensed under the [3-Clause BSD License](https://opensource.org/licenses/BSD-3-Clause).

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you, as defined in the 3-Clause BSD License, shall be licensed as above, without any additional terms or conditions.

If you want to contribute to this project, please read our [`CONTRIBUTING.md`].

[`CONTRIBUTING.md`]: https://github.com/cloudflare/.github/blob/master/CONTRIBUTING.md

---
<sub><sub><sub><sub>WireGuard is a registered trademark of Jason A. Donenfeld. BoringTun is not sponsored or endorsed by Jason A. Donenfeld.</sub></sub></sub></sub>
