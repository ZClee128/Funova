# App Store 审核回复草稿（Guideline 2.1 – Information Needed）

> 被拒类型：账号 App Review 历史较少，Apple 要求补充资料才能继续审。
> 解决方式：**不需要重新传包**。在 App Store Connect 的 Resolution Center 回复 + App Review Information 的 Notes 字段贴下面内容，并附上录屏，然后重新「Submit for Review」（仍是构建 3 / 版本 0.1.13）即可。
> 下面英文部分可直接粘贴（Apple 审核读英文）。`[方括号]` 为需你确认/补充的占位。

---

## 英文回复正文（贴进 Notes 字段 + Resolution Center 回复，两者一致）

**1. Screen recording**
A screen recording captured on a physical iPhone running the latest iOS is attached to this submission. It demonstrates the complete user flow: launching the app → tapping to access the device Photo Library → selecting a video → previewing and editing it on-device → saving the edited video back to the Photo Library. The app requires no account registration or login, so the full flow is shown from a cold launch.

**2. App purpose and target audience**
Funova is a lightweight, on-device video editing and preview tool for iPhone. It lets users open videos stored in their Photo Library, preview and edit them locally on the device, and save the finished video back to the Photo Library. It solves the need for quick, private, no-account video trimming/preview without uploading anything to a server. Target audience: general iPhone users who want simple local video editing.

**3. Setup and access instructions**
- No registration or login is required. All features are available immediately after launching the app.
- On first launch the app requests Photo Library access; grant it to load videos.
- Main flow: open the app → select a video from the Photo Library → preview/edit on device → tap save to write the result back to the Photo Library.
- No demo account, credentials, or sample files are needed. `[若实际上有登录/后台账号，请在此补充账号密码]`

**4. External services, tools, or platforms**
The app operates fully on-device. It only reads from and writes to the device Photo Library using standard iOS system frameworks. No third-party analytics, advertising, authentication, or payment SDKs are bundled (verified: the app contains 0 embedded third-party frameworks). `[若 App 实际连接了自有/第三方服务器（上传、AI、登录等），请在此描述域名/用途]`

**5. Regional differences**
The app functions consistently across all regions. There are no region-specific features, content, or restrictions.

**6. Regulated industry / protected material**
Not applicable. The app is not in a highly regulated industry and does not include protected third-party material.

---

## 你这边要做的 3 件事
1. **录屏**（必需，最关键）：在真机 iPhone + 最新 iOS 上录一段，从打开 App 开始，走完「选视频 → 预览/编辑 → 保存到相册」全流程。格式 .mov/.mp4。
2. **确认两处占位**：
   - 第 3 条：App 是否真的**没有**登录/后台账号？（包里无任何登录/网络权限痕迹，我按「无登录」写了）
   - 第 4 条：App 是否**真的没有**连接任何服务器/云/AI/支付？（包内 0 个第三方 framework，我按「纯本地」写了）
   若其中任一项其实「有」，把实际情况发我，我改文案，否则 Apple 可能再拒。
3. **提交**：把上面英文正文贴进 App Store Connect 的 Notes 字段和 Resolution Center 回复，上传录屏附件，然后重新 Submit for Review（构建 3）。

> 提示：包内对外显示名是 `Funova`，但 `CFBundleName`/`Executable` 是 `Pro91`，名字不一致。本次 2.1 主要查功能，但建议后续统一对外名称，避免被额外质疑。
