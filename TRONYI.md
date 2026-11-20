# Tronyi 远程桌面

本项目 fork 自开源项目 [rustdesk/rustdesk](https://github.com/rustdesk/rustdesk)，用于**内部/自用 + 公网远程协助**。
上游许可为 **AGPL-3.0**（见 `LICENCE`）：内部自用无额外义务；一旦修改后作为网络服务对外提供，修改部分也必须开源。

## 这个仓库里有什么

- `src/` `libs/` `flutter/` —— RustDesk 客户端完整源码（Rust 核心 + Flutter 界面）。
- `deploy/server/` —— **本项目新增**：自建信令/中继服务器（hbbs/hbbr）的部署脚本与说明。
  详见 [deploy/server/README.md](deploy/server/README.md)。
- 其余文件均为上游原样内容，尽量不改，方便日后同步上游。

## 分支约定

| 分支 | 用途 |
|---|---|
| `rustdesk` | 上游 `rustdesk/rustdesk` 的纯净镜像，跟踪 `upstream/master`，只用于拉取更新，不要在上面提交业务改动 |
| `main` | 我们的开发主线，从 `rustdesk` 切出，所有定制提交在这里 |
| `master` / `dev` / `devrun` | Gitee 初始化时留下的空脚手架（只有一个初始 commit），可忽略/删除 |

### 同步上游

```bash
git fetch upstream
git checkout rustdesk && git merge --ff-only upstream/master   # 更新纯净镜像
git checkout main && git merge rustdesk                         # 合入我们的主线
```

## 直接可用（推荐先走这条）

纯内部自用、要公网远程协助，**不需要自己编译客户端**：自建服务器 + 官方发行版客户端即可。
先按 [deploy/server/README.md](deploy/server/README.md) 把服务器跑起来，再用官方客户端连自己的服务器。

## 从源码构建客户端（可选，工作量大）

上游 `README.md` 的 *Raw Steps to build* 与 `.github/workflows/flutter-build.yml` 是权威说明。Windows 上大致需要：

1. Rust 工具链（本机已装：`C:\Users\zr\.cargo\bin`）
2. C++ 构建环境（MSVC Build Tools）+ LLVM/clang
3. [vcpkg](https://github.com/microsoft/vcpkg)，设 `VCPKG_ROOT`，安装：
   `vcpkg install libvpx:x64-windows-static libyuv:x64-windows-static opus:x64-windows-static aom:x64-windows-static`
4. Flutter SDK（**本机当前未安装**），版本需匹配 `flutter/pubspec.yaml` / 上游 CI 的 Flutter 版本
5. 子模块已初始化：`libs/hbb_common`

> 注意（上游 AGENTS.md）：不要运行 `flutter_rust_bridge_codegen`；新增 FFI 需手写 Dart 包装。

## 常用命令

```bash
git remote -v            # origin=Gitee 私有仓库, upstream=rustdesk 官方
git branch -vv
git submodule status
```

## 许可

AGPL-3.0。保留上游版权与许可声明。
