# 开发者清理清单 —— 去除 2.3.1(a) 隐藏功能（build 7）

> 被拒原因：Guideline 2.3.1(a) —— 二进制含"与 App 描述不符的平台功能"。
> 苹果静态扫描命中的证据（我在 build 6 的 IPA 里实测到）：
> `Info.plist` 里存在 5 个 `PRO91*` 私有键，且可执行文件内有 `Manifest`×53、`signature`×31、`openURL`、`loadRequest`/`WebKit`。

## 一、必须删除（硬性）

### 1. Info.plist —— 删除以下 5 个键（全部）
```
PRO91ManifestBaseURL
PRO91ManifestBackupURLs
PRO91InstallCodeURL
PRO91LandingHost
PRO91LandingOrigin
```
删完后用 `plutil -p Payload/Snipra.app/Info.plist | grep -i pro91` 应**无任何输出**。

### 2. 源码 —— 全局搜索并删除所有相关逻辑
在 Xcode 工程里全局搜索（大小写不敏感）并删除：
```
PRO91
ManifestBaseURL / ManifestBackupURLs / InstallCodeURL
LandingHost / LandingOrigin
manifest (与下载/安装相关的用法)
itms-services
```
需要删掉的是：
- 读取这些键的代码分支；
- 用它们去**下载清单 / 安装 App / 跳转落地页**的任何函数；
- 加载这些远程内容的 `WKWebView` / `UIWebView`（`loadRequest` / `loadHTMLString` / `evaluateJavaScript`）——除非是**保留给剪辑功能本身**且内容来自本地，否则一并删。

### 3. 确认没有"审核后才开启"的机制
- 无远程配置 / 功能开关（feature flag）在审核后可切换功能；
- 无热更新 / OTA（CodePush、Expo Updates、远程 JS bundle 等）；
- 无隐藏入口（长按版本号、摇一摇、私有 URL Scheme 进入的后门菜单）。

### 4. 权限用途说明须与真实权限一致
- build 6 的主 `Info.plist` 只有 `NSPhotoLibraryUsageDescription` / `NSPhotoLibraryAddUsageDescription`；
- 但本地化 `InfoPlist.strings` 里出现了 `NSCameraUsageDescription`、`NSLocationWhenInUseUsageDescription`。
- 请统一：**实际用到哪些权限，就只声明哪些**，且用途文字与功能相符（纯剪辑器通常只需相册读取/保存；若真的要调用相机录视频才加相机）。多余的权限声明本身也是 2.3.1 风险点。

### 5. 版本与打包
- 显示名 / 可执行名保持 `Snipra`（build 6 已正确）；
- 构建号递增到 **7**（或更高），版本号建议保持 `0.1.13`（也可一并升，如 `0.1.14`，需同步 ASC）；
- 用 **Release** 配置打包（不要带 DEBUG 横幅 / 开发菜单）；
- 重新签名导出 IPA。

## 二、交付后我方会做
1. 剖包核对：`Info.plist` 无 `PRO91*`、可执行文件无 `Manifest`/`InstallCode`/`Landing` 等残留；
2. 核对无 WebView 远程加载 / 无第三方 framework / 无热更新 SDK；
3. 通过后再更新 Fastfile / 工作流并交由你在 GitHub Actions 手动触发提审。

## 三、验收标准（一句话）
**二进制里应"只看得见"一个自包含的视频剪辑器：只读相册、只在本地处理、只保存回相册；没有任何下载、安装、跳转、远程配置的痕迹。**
