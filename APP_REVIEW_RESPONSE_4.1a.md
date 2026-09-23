# App Store 回复文案 —— Guideline 4.1(a) Copycats 拒信

> 适用场景：你选择「我没有 Funova 的权利 → 改名/去除第三方引用」路线。
> 占位符已填入：**新名 = Snipra**、**版本 0.1.13**、**构建号 6**。对应的纯文本回复在 `APP_REVIEW_NOTES_4.1a.txt`，已被工作流（ios-submit.yml）默认读取并写入 App Review Information。
> 语言用英语（Apple 审核团队读英文）。

> ℹ️ **关于新名 Snipra**：`Snipra` 是生造的原创品牌词（"snip" 意为"剪"，贴合视频剪辑），经检索 App Store 无精确重名，也不含任何真实品牌词。相对之前考虑过的含真实品牌词的方案，改用 Snipra 可显著降低再次触发 4.1(a)「与第三方品牌相似」的风险。最终可用性以 App Store Connect 名称框的实时校验为准。

---

## 在 Resolution Center 回复的文案（或写进 App Review Information → Notes）

Hello App Review Team,

Thank you for the feedback regarding Guideline 4.1(a).

To clarify: we do not have, and do not claim, any rights to, affiliation with, or authorization from the third-party "Funova" brand. We apologize for the confusing metadata.

To resolve this, we have:

1. Removed all references to "Funova" from the app's metadata — including the app name, subtitle, description, keywords, and screenshots.
2. Renamed the app to **Snipra**. The submitted build (version 0.1.13, build 6) uses the new app name and contains no "Funova" references.
3. The new name "Snipra" is an original brand name we created for this app; it does not reference or incorporate any third-party trademark or brand.

We have resubmitted the updated build for your review. Please let us know if any further information is needed.

Thank you.

---

## 关键事实（写进 ASC 前先确认）
- "Funova" 是真实存在的第三方日本品牌（Funova, Inc.，东京，健身/美容/健康，funova.co.jp）。
- 你的 app 是视频剪辑类，与真实 Funova 无关，故按 4.1(a) 必须去除引用或改名。
- 不得伪造授权证明 / 不得写不实声明（属商标侵权 + 欺骗审核，会连累账号）。我们走的是"无权利 → 主动改名去除引用"的合规路线。

## 改名的硬约束
- 手机桌面上显示的名字来自二进制 `CFBundleDisplayName`，无源码改不了 → **必须让开发者用新名重打 IPA**。
- 新 IPA 构建号必须 > 已上传最高值（当前 5），例如 6。
- App Store Connect 元数据（名称/副标题/描述/关键词/截图）也要同步去掉 "Funova" 并改成新名 `Snipra`。
- bundle id `com.mmhua.com` 可不变，只改显示名。
