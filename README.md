# alma-auto-updater

自动跟踪 [`yetone/alma-releases`](https://github.com/yetone/alma-releases) 的新版本，并将 `alma-desktop-bin` 的 AUR 包定义同步更新与发布。

## 项目目标

这个仓库用于维护 `alma-desktop-bin` AUR 包的自动化更新流程，减少手动改版本、算 checksum、更新 `.SRCINFO` 和推送 AUR 的重复工作。

## 仓库内容

- `PKGBUILD`：AUR 包构建脚本（版本、依赖、下载源等）。
- `.SRCINFO`：AUR 元数据（由 `makepkg --printsrcinfo` 生成）。
- `nvchecker.toml`：版本检查配置，监控 `yetone/alma-releases` 的最新 release tag（`v*`）。
- `nvchecker_oldver.json`：版本状态记录，供 `nvcmp` 比较新旧版本。
- `.github/workflows/update_and_pack_alma.yml`：自动更新、验证构建并发布到 AUR 的 CI 工作流。

## 自动化流程

GitHub Actions 工作流会在以下时机运行：

- 每 4 小时一次（cron）
- 手动触发（`workflow_dispatch`）

当发现新版本时，流程会：

1. 用 `nvchecker`/`nvcmp` 检查新版本。
2. 更新 `PKGBUILD` 中的 `pkgver`，并重置 `pkgrel=1`。
3. 运行 `updpkgsums` 更新校验和。
4. 运行 `makepkg` 验证可构建，并生成新的 `.SRCINFO`。
5. 推送 `PKGBUILD` 和 `.SRCINFO` 到 AUR 仓库 `alma-desktop-bin`。
6. 回写并提交本仓库状态（包含 `nvchecker_oldver.json`）。

## 所需配置

在 GitHub 仓库 Secrets 中配置：

- `AUR_SSH_PRIVATE_KEY`：用于推送到 AUR 的 SSH 私钥。

## 手动维护说明

如需本地手动更新，可参考：

```bash
nvchecker -c nvchecker.toml
nvcmp -c nvchecker.toml
updpkgsums
makepkg --printsrcinfo > .SRCINFO
```

## 相关链接

- Alma 官网: https://alma.now
- AUR 包名: `alma-desktop-bin`
- 上游发布仓库: https://github.com/yetone/alma-releases
