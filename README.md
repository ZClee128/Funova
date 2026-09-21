# Funova-A iOS · 提审流水线

通过 GitHub + GitHub Actions（fastlane）把 iOS 包上传并提交 App Store 审核的自动化工程。

> ⚠️ 建议把本仓库设为 **Private（私有）**，避免应用二进制与证书信息外泄。

---

## 一、当前状态核查（已用工具核验）

| 项目 | 结论 |
|------|------|
| IPA 文件 | `FunovaiOSreview-20260921.ipa`（约 8.4MB），bundle id `com.mmhua.com`，显示名 `Funova`，可执行名 `Funova`（名称一致） |
| IPA 签名类型 | **App Store 分发型**（内嵌 profile：`get-task-allow=false`、无设备列表、团队 `Pham Thi Nhung` / `S88VU4PBFB`、有效期至 2027-08-31）→ **可直接上传，无需重签** |
| 配置描述文件 | `mm.mobileprovision`：分发型，app id `S88VU4PBFB.com.mmhua.com`，与 IPA 一致 |
| 源码 | **无 Xcode 工程**，仅有编译好的 IPA → 当前只能"上传现成 IPA"，无法"从源码打包" |
| `开发证书.p12` | 开发证书（非分发证书），直接上传用不到；且含私钥，**已加入 .gitignore 不提交** |

---

## 二、两种提交方式（见 `fastlane/Fastfile`）

- **方式 A `upload_ipa`（默认，当前可用）**：直接上传仓库里的 `FunovaiOSreview-20260921.ipa`，并**自动带上审核回复文案（APP_REVIEW_NOTES.txt）+ 录屏附件（app_review_recording.mp4）+ 自动提审**。注意：该 IPA 的构建号（CFBundleVersion）写死，**每个构建号在 App Store Connect 只能上传一次**。当前包构建号 `5`、版本 `0.1.13`。
  - 录屏附件：工作流已固定读仓库内 `app_review_recording.mp4` 作为 App Review Attachment 自动上传。要更新录屏，直接替换该文件即可。
  - 文案路径：工作流已固定读仓库内 `APP_REVIEW_NOTES.txt`，可用 `APP_REVIEW_NOTES_FILE` 覆盖。
- **方式 A2 `submit_existing_build`**：IPA 已上传过时，直接提交已上传的构建（默认构建号 `5`）去审核，**不再重传二进制**。工作流支持手动选择 lane 触发（`workflow_dispatch` → lane=`submit_existing_build`）。
- **方式 A3 `resubmit_with_info`（被拒后补资料重提）**：不重传二进制，只把"审核备注 + 录屏附件"提交并重新提审，默认构建号 `5`。适用于被拒后不想换包、只补说明的场景。
- **方式 B `build_and_upload`（有源码后）**：把 Xcode 工程加进仓库，改为 `gym` 从源码打包再上传（需在 Secrets 里配好分发证书与 profile，才能重签新构建号）。

GitHub Actions 工作流**仅手动触发**（`workflow_dispatch`），默认跑方式 A（`.github/workflows/ios-submit.yml`）。

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

## 四、手动触发提审（被拒后"自动回复 + 重提"也走这里）

工作流已改为**仅手动触发**。你只需在 GitHub 网页点一下即可，无需打 tag、无需手动进 App Store Connect 回复。

### 操作步骤
1. 打开 `https://github.com/ZClee128/Funova/actions`
2. 左侧选 **iOS Build & App Store Submit** → 点右上角 **Run workflow**
3. 分支选 `main`，`lane` 下拉选要跑的：
   - **`upload_ipa`（默认）** —— 上传当前 IPA（build 5）+ 附上 `APP_REVIEW_NOTES.txt` 回复文案 + 附上 `app_review_recording.mp4` 录屏 + 自动提审。
     👉 **被拒后想"自动回复苹果并重新提审"，就选这个**（前提是你换了新的 IPA，构建号要比被拒的高）。
   - **`submit_existing_build`** —— 二进制已经传过、不想重传，只提交已上传的构建去审核。
   - **`resubmit_with_info`** —— 被拒后**不换包**，只补"回复文案 + 录屏"并重新提审（同一构建号）。
4. 点 **Run workflow**，等运行结束（约 1–3 分钟上传 + 提审）。日志里出现 `Successfully submitted the app for review!` 即成功。

> 前提：App Store Connect 后台该版本的**元数据（名称/描述/截图/分级等）已填写完整**，否则提审会失败（属正常校验）。

### 换包 / 更新录屏时
- **换 IPA**：把新包命名为 `FunovaiOSreview-20260921.ipa` 覆盖原文件（构建号必须 > 已上传的最高值），再手动触发 `upload_ipa`。
- **只换录屏**：直接替换 `app_review_recording.mp4`，再触发 `resubmit_with_info`（不重传二进制）。
- **只改回复文案**：编辑 `APP_REVIEW_NOTES.txt` 后重新触发对应 lane。

---

## 五、合规提醒（务必确认）

- 提审账号（`Pham Thi Nhung` / `S88VU4PBFB`）必须对 `com.mmhua.com` 这个 App **拥有合法权利**，且 App 内容需符合《App Store 审核指南》。
- 该 IPA 显示名/可执行名均为 `Funova`，对外展示信息一致；但 bundle id 含 `com.mmhua.com`、团队为 `Pham Thi Nhung`，请确认这些与你在 App Store Connect 后台登记的信息一致，避免审核被拒。
- 请勿在流水线中做任何隐藏、伪造 bundle id、规避审核的行为；本工程仅做标准分发上传。

---

## 目录结构

```
.
├── FunovaiOSreview-20260921.ipa      # 待提审的安装包（已分发签名，构建号 5 / 版本 0.1.13）
├── app_review_recording.mp4         # 审核录屏附件（自动附给审核员）
├── APP_REVIEW_NOTES.txt             # 被拒后自动回复苹果的说明文案
├── APP_REVIEW_RESPONSE.md           # 同一份文案的 Markdown 备份
├── mm.mobileprovision             # 分发型描述文件
├── 开发证书.p12                   # 开发证书（不提交，直接上传用不到）
├── Gemfile                        # fastlane 依赖
├── fastlane/
│   ├── Appfile                    # app_identifier / team_id
│   └── Fastfile                   # upload_ipa / submit_existing_build / resubmit_with_info / build_and_upload
└── .github/workflows/
    └── ios-submit.yml             # GitHub Actions 提审工作流（仅手动触发）
```
