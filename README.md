# Deepflood Sign

自动完成 Deepflood 社区每日签到、统计签到收益，并在需要时更新 Cookie 的 Python 脚本。脚本既可以在本地直接运行，也可以投放到青龙面板或 GitHub Actions 等自动化环境中使用。

## ✨ 功能

- **多运行环境自动适配**：支持普通本地环境、青龙面板、GitHub Actions 等，自动决定 Cookie 存储策略。
- **多账号支持**：同时处理多组账户或 Cookie，按顺序逐一签到。
- **自动补救**：Cookie 失效时，自动调用 YesCaptcha 完成登录并刷新 Cookie。
- **收益统计**：拉取近月签到明细，展示总收益、天数及日均数据。
- **通知生态丰富**：通过内置的 `notify.py`，可接入 Telegram、企业微信、Bark、PushDeer、SMTP 等几十种通知渠道。

## YesCaptcha

1. 访问 [YesCaptcha](https://yescaptcha.com/i/k2Hy3Q) 注册账号
2. 注册后联系客服可免费获得余额（约可使用60次登录）

> **提示**：YesCaptcha 提供两个服务节点，可根据网络情况选择：
> - 国际节点：`https://api.yescaptcha.com`（默认）
> - 国内节点：`https://cn.yescaptcha.com`

## 🧩 核心环境变量

| 变量 | 必需 | 默认值 | 说明 |
| :-- | :--: | :--: | :-- |
| `DF_COOKIE` | 是 | 空 | Deepflood Cookie，多个账号用 `&` 或换行符分隔。仅依赖 Cookie 签到时必填。|
| `USER` / `PASS` | 否 | 空 | 第一组账号的用户名、密码。|
| `USER1` / `PASS1` ... | 否 | 空 | 更多账号配置，编号递增。|
| `CLIENTT_KEY` | 账号登录必需 | 空 | YesCaptcha API Key，用于解析 Turnstile 验证码。|
| `API_BASE_URL` | 否 | `https://api.yescaptcha.com` | YesCaptcha 接口地址，可改为国内节点 `https://cn.yescaptcha.com`。|
| `DF_RANDOM` | 否 | `true` | 默认随机鸡腿签到False是固定5鸡腿。|
| `GH_PAT` | GitHub Actions 可选 | 空 | Personal Access Token，用于把刷新后的 Cookie 回写到仓库。|
| 各类通知变量 | 否 | - | 详见 `notify.py`，按需设置对应推送渠道的环境变量。|

> 💡 青龙面板会在运行时注入 `QLAPI` 对象，脚本会自动调用它来创建/更新面板变量；本地单机或 GitHub Actions 则不会触发这些调用。

### 多账号示例

```env
DF_COOKIE=df_cookie_1 & df_cookie_2
USER=userA
PASS=passA
USER1=userB
PASS1=passB
CLIENTT_KEY=your_yescaptcha_key
```

脚本会优先使用 Cookie；当 Cookie 失效时，会尝试使用对应的账号和密码重新登录。

## 🚀 快速上手

### 本地运行

1. 克隆仓库，安装依赖（见上）。
2. 通过系统环境变量或 `.env` 文件写入上表中的配置。
3. 执行

  ```bash
  python deepflood_sign.py
  ```

脚本会输出每个账号的签到结果、收益统计及 Cookie 更新情况。

### 青龙面板

在青龙面板中执行以下命令克隆本仓库：

```bash
ql repo https://github.com/yowiv/NodeSeek-Signin.git
```

然后在环境变量中添加所需配置。

### GitHub Actions

为了实现自动更新Cookie功能，脚本需要使用GitHub Personal Access Token (PAT)将获取的新Cookie保存到仓库变量中。

#### 1. 创建Personal Access Token

1. 登录您的GitHub账户
2. 点击右上角头像 → 选择 "Settings"（设置）
3. 滚动到页面底部 → 点击 "Developer settings"（开发者设置）
4. 在左侧菜单选择 "Personal access tokens" → 点击 "Tokens (classic)"
5. 点击 "Generate new token" → 选择 "Generate new token (classic)"
6. 配置token：
   - 名称：填写一个描述性名称，如 "NodeSeek签到脚本"
   - 过期时间：根据需要选择（推荐90天或更长）
   - 勾选权限：
     - `repo` (完整的仓库访问权限)
     - `workflow` (用于管理GitHub Actions)
7. 点击页面底部的 "Generate token" 按钮
8. **立即复制生成的token**，关闭页面后将无法再次查看

#### 2. 添加到仓库Secrets

1. 进入您的NodeSeek-Signin仓库
2. 点击 "Settings" → "Secrets and variables" → "Actions"
3. 点击 "New repository secret"
4. 名称填写：`GH_PAT`
5. 值填写：刚才复制的Personal Access Token
6. 点击 "Add secret" 保存

完成以上设置后，签到脚本可以自动将有效的Cookie保存到GitHub仓库变量中，下次运行时直接使用，减少重复登录和验证码操作。

仓库已经包含 `deepflood.yml` 工作流，默认会在每天北京时间 0 点运行，也可以手动触发。需要在仓库的 **Settings → Secrets and variables → Actions** 中配置下列条目：

| 名称 | 说明 |
| :-- | :-- |
| `DF_COOKIE` | 初始 Cookie（可为空） |
| `USER` / `PASS`、`USER1` / `PASS1` ... | 账号密码，用于 Cookie 失效时自动登录 |
| `CLIENTT_KEY` | YesCaptcha API Key |
| `API_BASE_URL` | YesCaptcha API 地址，留空使用默认国际节点 |
| `GH_PAT` | (可选) 用于把刷新后的 Cookie 写回仓库变量 |

其他通知相关 Secrets 也可按需添加。运行时日志会输出每个账号的签到结果；若配置了 `GH_PAT`，Sign 完成后工作流会尝试更新仓库变量 `DF_COOKIE`。

## 🔔 通知说明

`notify.py` 封装了多种常见推送方式（Telegram、企业微信、Bark、PushPlus、SMTP 等）。

- 只要设置了对应的环境变量，脚本就会在签到成功或失败时推送消息。
- 推送内容包含签到状态、收益统计以及异常信息，方便在自动化场景中排查问题。
- 若只想在控制台查看结果，可把 `CONSOLE` 变量设为 `true` 即可。

更完整的通知字段请查阅 `notify.py` 开头的 `push_config` 字典。

## ❓ 常见问题

1. **脚本提示 `QLAPI` 未定义怎么办？**
  - 请确认青龙面板版本 ≥ v2.18.1 才支持 `QLAPI` 接口。

3. **Cookie 会保存在哪里？**
  - 青龙面板：通过 `QLAPI` 创建/更新同名变量。
  - GitHub Actions：调用仓库 REST API 更新 Actions Variables。
  - 本地环境：默认仅打印，不会写入文件。

## ⚠️ 免责声明


脚本仅供学习、自动化实验与个人效率提升之用。使用前请确保符合 Deepflood 社区及相关服务的使用协议，风险自负。
