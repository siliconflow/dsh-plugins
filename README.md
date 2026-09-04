# DSH 插件存储仓库

DeepSeek Harness 本地插件集合。本仓库用于统一管理和分发 DSH 的自定义插件，后续新增插件均追加至此仓库。

## v1.3.0：归档删除修复

[下载 Release](../../releases/tag/v1.3.0) · 归档面板 **0.2.1** · 对应独立插件 **0.4.1**

本次修复历史恢复和分叉会话误报 `session-live`、删除等待写入时卡住、日志或展示缓存残留。面板支持搜索、工作区筛选、更新时间排序、删除确认、防重复提交及失败重试。

### 更新归档插件

重新运行下方归档插件安装命令，或下载 Release 中的 `dsh-ui-archived-local-0.2.1.tar.gz`，解压覆盖 profile 下的 `local-plugins/dsh-ui-archived-local/`。本地依赖和 composition 配置见后文。

**安装脚本仅更新插件，不会自动修复 Host。** 根据 [补丁说明](dsh-ui-archived-local/patches/README.md) 选择：

| Host 当前状态 | 应用的补丁 |
|---|---|
| 已应用独立插件 0.4.0 的完整后端补丁 | 仅 `deleteSession-0.4.1.diff` |
| fork 基线 `8eb6aa069af605a3d6277dc301190ddeea3bb972` | 更新后的 `deleteSession-complete.diff` 与测试补丁 |
| 其他版本 | 迁移匹配的改动，先运行 `git apply --check` |

应用后执行 `pnpm run build:lib:host`，**重启 DSH Host，再刷新浏览器**。不要叠加完整补丁和增量升级补丁。运行中的会话仍需先完成或停止；删除不操作项目工作目录和用户导出的文件。

Release 提供已构建的本地插件、后端补丁及 `SHA256SUMS`。真实 Web 测试覆盖新建、恢复历史、分叉三条删除路径；JSONL / SQLite 测试覆盖删除后重新打开存储。

### English upgrade note

Release **v1.3.0** includes archive panel **0.2.1**, corresponding to standalone panel **0.4.1**. It fixes false ownership rejection for resumed/forked sessions, deletion deadlocks and persistent checkpoint residue. Updating the frontend is insufficient: apply the matching [Host patch](dsh-ui-archived-local/patches/README.md), rebuild, restart DSH and reload the browser. Use the incremental patch only when 0.4.0 is already applied. Release assets include the built local plugin and checksums.

## 快速安装

### 安装单个插件

```bash
# 只装 OSS 文件浏览器
curl -fsSL https://raw.githubusercontent.com/chensl139-ok/dsh-plugins/main/install.sh | bash -s -- dsh-tool-oss

# 只装归档面板
curl -fsSL https://raw.githubusercontent.com/chensl139-ok/dsh-plugins/main/install.sh | bash -s -- dsh-ui-archived-local
```

### 安装多个插件

```bash
curl -fsSL https://raw.githubusercontent.com/chensl139-ok/dsh-plugins/main/install.sh | bash -s -- dsh-tool-oss dsh-ui-archived-local
```

### 交互式选择

```bash
curl -fsSL https://raw.githubusercontent.com/chensl139-ok/dsh-plugins/main/install.sh | bash
```

会列出所有可用插件，输入数字选择（支持多选，`0` 全选）。

### 指定 profile

```bash
curl -fsSL https://raw.githubusercontent.com/chensl139-ok/dsh-plugins/main/install.sh | bash -s -- --profile my-profile dsh-tool-oss
```

### 安装后配置

如果安装了 `dsh-tool-oss`，在 `~/.zshrc` 中设置环境变量：

```bash
export SILICON_OSS_AK='你的AccessKey'
export SILICON_OSS_SK='你的SecretKey'
export SILICON_OSS_ENDPOINT='https://s3.6scloud.com'
export SILICON_OSS_REGION='cn-east-1'
export SILICON_OSS_BUCKET='你的bucket名'
```

然后：

```bash
source ~/.zshrc
pnpm dsh web
```

刷新浏览器即可使用。`dsh-ui-archived-local` 无需额外配置。

> 安装脚本自动完成：克隆仓库 → 复制选中插件 → 添加依赖 → pnpm install → 配置 cordis.patch.yml。未选中的插件不会被安装。

## 包含的插件

| 插件 | 版本 | 说明 |
|---|---|---|
| [dsh-tool-oss](./dsh-tool-oss/) | v0.3.0 | OSS 对象存储文件浏览器，支持多 Bucket、文件/文件夹上传、递归删除，对接硅基流动 / 腾讯云 COS / 阿里云 OSS 等任意 S3 兼容存储 |
| [dsh-ui-archived-local](./dsh-ui-archived-local/) | v0.2.1 | 归档面板插件，侧边栏「已归档」面板，支持查看/打开/取消归档/永久删除，自定义居中确认弹窗替代 `window.confirm` |

---

## dsh-tool-oss

OSS 对象存储文件浏览器。

> **零外部依赖** — 仅用 Node.js 内置 `crypto`（AWS SigV4 签名）和全局 `fetch`，不安装任何 SDK。

## 功能一览

| 功能 | 说明 |
|---|---|
| 📁 文件浏览 | 居中面板，支持目录层级导航（面包屑 + 子目录点击进入） |
| 📤 文件上传 | 单文件 / 多文件，任意格式（文本、图片、二进制） |
| 📁 文件夹上传 | 递归上传整个文件夹，保留完整目录结构（含文件夹名） |
| 📝 文本上传 | 直接输入 key + 文本内容上传 |
| 👁 文件查看 | 点击文件名内联查看内容 |
| 🗑 文件删除 | 单个文件删除，居中确认弹窗 |
| 🗑 文件夹删除 | 递归删除整个文件夹及其下所有对象 |
| 🔔 居中弹窗 | 所有确认 / 错误提示均为自定义居中 ModalDialog（非原生 `window.confirm`） |
| 🪣 多 Bucket | 同时配置多个 Bucket，UI 顶部一键切换 |
| 🔐 环境变量 | 凭证全部从 `~/.zshrc` 环境变量读取，composition 文件零密钥 |

## 包结构

```
local-plugins/
├── dsh-tool-oss/              # OSS 主插件（Host + Client 双面粉）
│   ├── index.js               # Host: oss 模型工具 + /oss RPC 通道
│   ├── client.js              # Client: 文件浏览面板 + 上传对话框
│   └── package.json           # dsh.client 声明
└── dsh-ui-archived-local/     # 归档面板覆盖插件
    ├── index.js               # Host: 空 apply 占位
    ├── client.js              # Client: 自定义居中确认弹窗替代 window.confirm
    └── package.json
```

## 安装

### 1. 放置插件文件

将 `local-plugins/` 目录放到 DSH profile 下：

```bash
~/.dsh/profiles/web/local-plugins/
```

### 2. 添加依赖

编辑 `~/.dsh/profiles/web/package.json`，在 `dependencies` 中加入：

```json
"dsh-tool-oss": "link:./local-plugins/dsh-tool-oss",
"dsh-ui-archived-local": "link:./local-plugins/dsh-ui-archived-local"
```

然后安装：

```bash
cd ~/.dsh/profiles/web && pnpm install
```

### 3. 配置环境变量

在 `~/.zshrc` 中添加（以硅基流动两个 Bucket 为例）：

```bash
# 共用配置（同账号）
export SILICON_OSS_AK='你的AccessKey'
export SILICON_OSS_SK='你的SecretKey'
export SILICON_OSS_ENDPOINT='https://s3.6scloud.com'
export SILICON_OSS_REGION='cn-east-1'

# Bucket 1
export SILICON_OSS_BUCKET='silicon'
# Bucket 2
export SILICON_OSS_BUCKET_2='model'
```

```bash
source ~/.zshrc
```

### 4. 配置 composition

编辑 `~/.dsh/profiles/web/cordis.patch.yml`：

```yaml
- insert:
    - id: tool-oss
      name: dsh-tool-oss
      config:
        timeoutMs: 60000
        providers:
          silicon:
            endpointEnv: SILICON_OSS_ENDPOINT
            regionEnv: SILICON_OSS_REGION
            bucketEnv: SILICON_OSS_BUCKET
            accessKeyIdEnv: SILICON_OSS_AK
            secretAccessKeyEnv: SILICON_OSS_SK
          model:
            endpointEnv: SILICON_OSS_ENDPOINT
            regionEnv: SILICON_OSS_REGION
            bucketEnv: SILICON_OSS_BUCKET_2
            accessKeyIdEnv: SILICON_OSS_AK
            secretAccessKeyEnv: SILICON_OSS_SK
          # 腾讯云 COS
          # tencent:
          #   endpointEnv: TENCENT_COS_ENDPOINT
          #   regionEnv: TENCENT_COS_REGION
          #   bucketEnv: TENCENT_COS_BUCKET
          #   accessKeyIdEnv: TENCENT_COS_AK
          #   secretAccessKeyEnv: TENCENT_COS_SK

# 归档面板覆盖（可选）
- id: ui-archived
  disabled: true
- insert:
    - id: ui-archived-local
      name: dsh-ui-archived-local
```

### 5. 重启 DSH

```bash
pnpm dsh web
```

刷新浏览器后，左下角出现 `☁ OSS` 按钮。

## 使用

1. 点击左下角 `☁ OSS` → 居中弹出文件浏览面板
2. 顶部 `Bucket:` 按钮切换不同桶
3. 面包屑导航目录层级，`📁` 文件夹可点击进入
4. 点击 `📤 上传` → 居中弹出上传对话框
   - **文件 / 文件夹** Tab：选择文件或文件夹上传
   - **文本** Tab：输入 key + 内容上传
5. 文件右侧 `🗑` 删除单个对象，文件夹右侧 `🗑` 递归删除整个文件夹

## 对接其他云厂商

所有 S3 兼容存储都可对接，只需设置对应的环境变量：

| 云厂商 | Endpoint 示例 |
|---|---|
| 硅基流动 | `https://s3.6scloud.com` |
| 腾讯云 COS | `https://cos.ap-guangzhou.myqcloud.com` |
| 阿里云 OSS | `https://oss-cn-hangzhou.aliyuncs.com` |
| AWS S3 | `https://s3.us-east-1.amazonaws.com` |
| MinIO | `http://localhost:9000` |

## 技术细节

- **签名**：AWS Signature V4，纯 `node:crypto` 实现
- **传输**：Host 端 `fetch` + S3 REST API；Client 端 `connection.rpc.call` → Host
- **二进制安全**：文件以 base64 编码传输，Host 端 `Buffer.from(content, 'base64')` 解码后 PUT
- **文件夹删除**：S3 服务强制 `delimiter=/`，采用递归方式逐层删除（先删文件，再递归子目录）
- **生命周期**：所有 RPC 通道和 Tool 注册均 fiber-scoped，插件卸载时自动清理

---

## dsh-ui-archived-local

侧边栏「已归档」面板插件。在 DSH 侧边栏提供归档会话列表，支持查看、打开、取消归档和永久删除。将原 shipped `ui-archived` 的 `window.confirm` / `window.alert` 替换为自定义居中 `ModalDialog`。

### 功能

- 显示已归档会话的标题、工作区和相对时间
- 点击会话即可重新打开
- Host 支持 `unarchiveSession` 时可以取消归档
- Host 支持 `deleteSession` 时可以永久删除（删除前居中确认弹窗）
- Host 尚未安装补丁时自动隐藏对应操作按钮，查看和打开功能不受影响
- 所有确认/错误弹窗均为自定义居中 `ModalDialog`（非原生 `window.confirm`）

### 修复删除不彻底（必须更新 Host）

0.2.1 补齐了历史恢复和分叉会话的所有权记录，修复误报 `session-live`，并清理持久化展示缓存。已应用上一版后端修复的部署可使用 `deleteSession-0.4.1.diff` 增量升级，重建后重启。

前端现已保留删除失败的归档项，显示错误并允许重试；删除期间禁用重复操作。支持搜索、工作区筛选、更新时间排序和居中确认框。

后端修复见 [补丁与适用基线](dsh-ui-archived-local/patches/README.md)：先释放自己持有的闲置 Agent / Session，等待最终写入结束，再删除 JSONL / SQLite 记录、工作区成员和归档索引。**安装脚本只更新插件；后端补丁需要应用、构建并重启 DSH，不能只刷新浏览器。**

浏览器代码由 [dsh-archived-panel](https://github.com/chensl139-ok/dsh-archived-panel) 源码生成。维护时在该仓库执行 `pnpm build` 和 `pnpm export:local /path/to/dsh-plugins/dsh-ui-archived-local`，再同步版本与文档。

### Complete deletion fix

The frontend preserves failed archive entries for retry and blocks duplicate operations. The [Host patch](dsh-ui-archived-local/patches/README.md) disposes owned idle agents, drains pending writes, then deletes durable records and workspace/archive indexes. Installing the plugin alone is insufficient: apply the matching Host patch, rebuild and restart DSH. The browser artifact is generated from dsh-archived-panel source.

### Host 补丁（可选）

`patches/` 目录包含为 DSH 源码添加 `unarchiveSession` 和 `deleteSession` 的可选补丁：

```
patches/
├── unarchiveSession.diff          # 为 workspaces 服务添加 unarchiveSession 方法
├── unarchiveSession.tests.diff    # 对应测试
├── deleteSession.diff             # 为 workspaces 服务添加 deleteSession 方法
├── deleteSession.tests.diff       # 对应测试
└── README.md                      # 补丁说明和应用步骤
```

不安装补丁时面板自动降级为查看 + 打开模式。

### 安装

在 `cordis.patch.yml` 中禁用 shipped `ui-archived` 并挂载本地版：

```yaml
- id: ui-archived
  disabled: true

- insert:
    - id: ui-archived-local
      name: dsh-ui-archived-local
```

## License

MIT
