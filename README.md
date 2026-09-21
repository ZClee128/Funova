# Funova-A iOS · 提审流水线

通过 GitHub + GitHub Actions（fastlane）把 iOS 包上传并提交 App Store 审核的自动化工程。

> ⚠️ 建议把本仓库设为 **Private（私有）**，避免应用二进制与证书信息外泄。

---

## 一、当前状态核查（已用工具核验）

| 项目 | 结论 |
|------|------|
| IPA 文件 | `FunovaiOSreview-20260921.ipa`（约 8.4MB），bundle id `com.mmhua.com`，显示名 `Funova`，可执行名 `Pro91` |
| IPA 签名类型 | **App Store 分发型**（内嵌 profile：`get-task-allow=false`、无设备列表、团队 `Pham Thi Nhung` / `S88VU4PBFB`、有效期至 2027-08-31）→ **可直接上传，无需重签** |
| 配置描述文件 | `mm.mobileprovision`：分发型，app id `S88VU4PBFB.com.mmhua.com`，与 IPA 一致 |
| 源码 | **无 Xcode 工程**，仅有编译好的 IPA → 当前只能"上传现成 IPA"，无法"从源码打包" |
| `开发证书.p12` | 开发证书（非分发证书），直接上传用不到；且含私钥，**已加入 .gitignore 不提交** |

---

## 二、两种提交方式（见 `fastlane/Fastfile`）

- **方式 A `upload_ipa`（默认，当前可用）**：直接上传仓库里的 `FunovaiOSreview-20260921.ipa`，并**自动带上审核回复文案（APP_REVIEW_NOTES.txt）+ 可选录屏附件 + 自动提审**。注意：该 IPA 的构建号（CFBundleVersion）写死，**每个构建号在 App Store Connect 只能上传一次**。当前包构建号 `4`、版本 `0.1.13`。
  - 录屏附件：设置环境变量 `APP_REVIEW_ATTACHMENT=/path/to/recording.mov` 即作为 App Review Attachment 自动上传；留空则不附视频。
  - 文案路径：默认读仓库内 `APP_REVIEW_NOTES.txt`，可用 `APP_REVIEW_NOTES_FILE` 覆盖。
- **方式 A2 `submit_existing_build`**：IPA 已上传过时，直接提交已上传的构建（默认构建号 `1`）去审核，**不再重传二进制**。工作流支持手动选择 lane 触发（`workflow_dispatch` → lane=`submit_existing_build`）。
- **方式 B `build_and_upload`（有源码后）**：把 Xcode 工程加进仓库，改为 `gym` 从源码打包再上传（需在 Secrets 里配好分发证书与 profile，才能重签新构建号）。

GitHub Actions 工作流默认跑方式 A（`.github/workflows/ios-submit.yml`）。

---

## 三、必须配置的 GitHub Secrets

在本仓库 `Settings → Secrets and variables → Actions` 中添加：

| Secret 名 | 说明 | 获取方式 |
|-----------|------|----------|
| `APPSTORE_CONNECT_API_KEY_KEY_ID` | API Key ID（如 `ABCDE12345`） | App Store Connect → 用户和访问 → 密钥，创建密钥后显示 |
| `APPSTORE_CONNECT_API_KEY_ISSUER_ID` | 签发者 ID（UUID） | 同上页面顶部 "Issuer ID" |
| `APPSTORE_CONNECT_API_KEY_P8` | `.p8` 文件**全文内容**（纯文本） | 创建密钥时下载的 `AuthKey_XXXX.p8` |
| `FASTLANE_USER` | 你的 Apple ID（可选，API Key 方式非必填） | — |

> 创建 API Key 需"账户持有人"或具备"App Manager / Admin"权限。
> 注意：API Key 必须属于 `com.mmhua.com` 对应的团队（`S88VU4PBFB` / Pham Thi Nhung）。

### 关于证书
- 直接上传现成 IPA（方式 A）**不需要**证书 Secret，因为 IPA 已分发签名。
- 若改用方式 B（从源码打包），需额外在 Secrets 放：`DIST_CERT_P12`(base64)、`DIST_CERT_PASSWORD`、`DIST_PROVISIONING_PROFILE`(base64)，或改用 `fastlane match`。

---

## 四、本地 → GitHub → 触发提审

1. **登录 GitHub CLI**（本机尚未登录）：
   ```bash
   gh auth login
   ```
2. **创建私有仓库并推送**（在仓库根目录执行）：
   ```bash
   gh repo create funova-a-ios --private --source=. --remote=origin
   git add .
   git commit -m "chore: init ios submit pipeline"
   git push -u origin main
   # 打 tag 触发流水线
   git tag v1.0.0 && git push origin v1.0.0
   ```
   或手动在 Actions 页面 `Run workflow`。

3. **提审**：流水线会用 `deliver` 上传 IPA 并 `submit_for_review`。
   > 前提：App Store Connect 后台该版本的**元数据（名称/描述/截图/分级等）已填写完整**，否则提审会失败（属正常校验）。

---

## 五、合规提醒（务必确认）

- 提审账号（`Pham Thi Nhung` / `S88VU4PBFB`）必须对 `com.mmhua.com` 这个 App **拥有合法权利**，且 App 内容需符合《App Store 审核指南》。
- 该 IPA 内部显示名/可执行名存在 `Funova` / `Pro91` / `manhua` 等多处不一致，请确认对外展示信息与提交信息一致，避免审核被拒。
- 请勿在流水线中做任何隐藏、伪造 bundle id、规避审核的行为；本工程仅做标准分发上传。

---

## 目录结构

```
.
├── FunovaiOSreview-20260921.ipa      # 待提审的安装包（已分发签名，构建号 4 / 版本 0.1.13）
├── mm.mobileprovision             # 分发型描述文件
├── 开发证书.p12                   # 开发证书（不提交，直接上传用不到）
├── Gemfile                        # fastlane 依赖
├── fastlane/
│   ├── Appfile                    # app_identifier / team_id
│   └── Fastfile                   # upload_ipa / build_and_upload
└── .github/workflows/
    └── ios-submit.yml             # GitHub Actions 提审工作流
```
