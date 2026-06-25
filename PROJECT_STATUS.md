# 项目状态

更新时间：2026-06-25（Asia/Shanghai）

## 当前同步状态

- 当前 fork 分支：`main`
- 当前 fork 发布提交：以 `v1.17.6` tag 指向为准
- 当前 fork 合并提交：`de2c4f3`（`Merge remote-tracking branch 'refs/remotes/upstream/tags/v1.17.6'`）
- 当前 fork release：`v1.17.6`
- 当前官方上游：`upstream/main` = `d7847da`
- 当前官方最新 tag：`v1.17.6`
- 当前 origin release：`v1.17.6`
- Release 页面：https://github.com/lucian-zh/Cli-Proxy-API-Management-Center/releases/tag/v1.17.6
- Release 产物：https://github.com/lucian-zh/Cli-Proxy-API-Management-Center/releases/download/v1.17.6/management.html

## 当前上游版本说明

- 官方 `v1.17.6` 新增 APIKEY.FUN provider/quick start 管理入口、plugin OAuth 支持，以及 Codex reset credits 展示。
- 普通 Gemini API key 管理、Gemini 模型获取、`generateContent` 连接测试仍存在。
- Gemini CLI OAuth/配额/`enable-gemini-cli-endpoint` 仍按官方 `v1.17.1` 起的策略移除。

## 维护策略

- 以官方上游为主，fork 只保留必要的小改动。
- 版本号与官方同步；官方发布 `vX.Y.Z` 后，本 fork 合并官方 `main`，再用同名 `vX.Y.Z` 发布包含 fork 改动的版本。
- fork 的 release tag 是本仓库自己的 annotated tag，通常指向“合并官方 + fork 改动”的提交；官方同名 tag 指向官方 commit。不要假设两边同名 tag 的 commit 完全一致。
- 拉取上游时优先使用 `--no-tags`，避免官方同名 tag 与本 fork release tag 冲突。

推荐同步流程：

```bash
git status --short --branch
git fetch --no-tags upstream main:refs/remotes/upstream/main
git fetch --no-tags origin main:refs/remotes/origin/main
git ls-remote --tags upstream 'v*' | awk '{print $2}' | tr -d '\r' | sed 's#refs/tags/##; s#\^{}##' | sort -uV | tail -20
git merge --no-edit upstream/main
```

发布流程：

```bash
./node_modules/.bin/tsc --noEmit
./node_modules/.bin/tsc && ./node_modules/.bin/vite build
git tag -a vX.Y.Z -m "Release vX.Y.Z" HEAD
git push origin main
git push origin vX.Y.Z
```

tag 推送会触发 `.github/workflows/release.yml`，自动构建并发布 `management.html`。

## 当前保留的 fork 改动

相对官方 `upstream/main`，当前保留的差异是有意的，后续合并上游时应尽量保留：

1. 防止浏览器或密码管理器误填管理密钥/API key。
   - `src/pages/LoginPage.tsx`
   - `src/components/config/VisualConfigEditor.tsx`
   - `src/components/config/VisualConfigEditorBlocks.tsx`
   - `src/features/providers/sheets/forms/BaseProviderForm.tsx`

2. 在供应商列表和详情里展示 provider priority。
   - `src/features/providers/components/ProviderResourceTable.tsx`
   - `src/features/providers/components/ProviderResourceTable.module.scss`
   - `src/features/providers/sheets/ResourceDetailView.tsx`
   - `src/i18n/locales/*.json`

3. 修复日志页面 `fileLoggingRequired` 状态清理时序，避免刷新/重连时提示状态短暂不一致。
   - `src/pages/LogsPage.tsx`

4. 修复 Claude/Codex 上游模型获取和连接测试的鉴权问题。
   - `src/components/providers/utils.ts`
   - `src/features/providers/sheets/forms/useConnectivityTest.ts`
   - `src/features/providers/sheets/forms/useModelDiscovery.ts`
   - `src/services/api/models.ts`

   重点行为：
   - Codex `/v1/responses` 连接测试需要有效 `Authorization: Bearer ...`，不能只检查 header 名是否存在。
   - Claude `/v1/messages` 连接测试保留 `x-api-key`，同时补 `Authorization: Bearer ...`，兼容要求 Bearer token 的 Claude-compatible 上游。
   - 模型获取时同样补齐有效 Bearer token。
   - 完整 base URL 或 endpoint（如 `/v1/models`、`/v1/messages`、`/v1/responses`、不同端口、`/v0/management`）会归一化到对应的测试/模型接口。

## 最近验证记录

最近一次完整验证在 `v1.17.6` 合并发布前完成：

- `git diff --check`：通过
- `./node_modules/.bin/tsc --noEmit`：通过
- `./node_modules/.bin/tsc && ./node_modules/.bin/vite build`：通过
- 关键触点 ESLint：仍被仓库既有 React hooks 规则问题拦住，当前命中 `src/features/providers/useProviderWorkbench.ts` 的 `react-hooks/set-state-in-effect`。
- 全量 ESLint：仍失败，当前是仓库既有的 27 个 React hooks 规则错误，主要为 `react-hooks/set-state-in-effect` 和 `react-hooks/refs`。这不是最近 fork 改动新增的问题。

## 本地清理约定

- `dist/` 是本地构建产物，已在 `.gitignore` 中；发布由 GitHub Actions 重新构建，平时可以删除。
- `node_modules/` 是本地依赖目录，已在 `.gitignore` 中；不要提交。
- 不要提交 `dist/management.html` 或 `dist/index.html`。
