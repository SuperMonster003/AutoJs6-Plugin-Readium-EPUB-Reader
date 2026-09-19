# AutoJs6-Plugin-Readium-EPUB-Reader AGENTS.md

本文件是本仓库的工程约定, 由 `docs/development/repository-standard.md` (AutoJs6 新插件仓库参考规范, 与 `AUTOJS6_PLUGIN_NEW_REPO_AGENTS.md` 同源) 裁剪而来, 只保留对本仓库真实有效的条款. 路线图与阶段性决策见 `ROADMAP.md`; 本文件描述的是 "怎样改仓库", 路线图描述的是 "改什么".

## 1. 规则等级与本仓库的适用范围

- `MUST`: 必须遵循. `SHOULD`: 默认遵循, 偏离时在仓库文档中说明原因. `CONDITIONAL`: 仅在对应能力落地后适用.
- 用户在当前任务中的明确要求优先于本文件.
- 本仓库不包含原生库, 模型, 上游源码快照或 ABI 拆分, 参考规范中对应的 CONDITIONAL 条款不适用 (见第 5.4 节的省略理由).
- 本仓库是 Explorer Action 家族成员 (与 HTML Previewer / Markdown Previewer 同形), 同时承载 Readium Kotlin Toolkit 阅读器; 独立应用形态 (Launcher, `ACTION_VIEW`, 设置页, 发行历史, 手动更新检查), TTS 前台服务, `org.autojs.plugin.EPUB` Binder 服务与宿主 `epub-api` 契约在路线图 P3 / P4 / P5 落地后才适用 (第 8, 14 节).

## 2. 仓库身份

下列值在 Gradle (`resValue`), Manifest, Kotlin 常量 (`ReadiumEpubReaderPlugin`), 资源, 文档 (`.readme/common.json`), 测试和宿主注册信息中 MUST 完全一致. 修改任一值时同步修改全部位置, 并运行 `EpubReaderExplorerCompatibilityTest` 与 `PluginContractInstrumentationTest`.

| 项目 | 值 |
|---|---|
| 仓库与目录名 | `AutoJs6-Plugin-Readium-EPUB-Reader` |
| `rootProject.name` | `autojs6-plugin-readium-epub-reader` |
| 应用标题 (不可翻译) | `Readium EPUB Reader` |
| `applicationId` / namespace | `io.github.supermonster003.autojs6.plugin.readium.epub.reader` |
| 插件 ID / engine / variant | `readium-epub-reader` / `explorer-action` / `default` |
| Explorer Action 动作 ID | `readium-epub-reader.primary` (主按钮, placement 2) 与 `readium-epub-reader` (溢出菜单) |
| 动作标签资源 / 兜底文案 | `action_readium_epub_reader` / `Readium EPUB Reader` |
| 执行 Activity | `EpubReaderActivity`, action `org.autojs.plugin.EXPLORER_ACTION_EXECUTE` |
| 发现服务 | `ExplorerActionService` (`org.autojs.plugin.EXPLORER_ACTION`), `PluginInfoService` (`org.autojs.plugin.INFO`) |
| 声明协议 / 最低宿主 / 审计宿主 | v2 / 5269 / 5282 (协议 v22), 单点定义于 `gradle/explorer-action-compatibility.properties` |
| 脚本全局对象 | `epub` (宿主侧, 路线图 D1 / P6, 尚未落地) |
| 专用 API | `epub-api` (宿主 `plugin-api/epub-api`, 路线图 P5.1 落地后以 AAR 形式进入 `libs/`) |
| 阅读引擎 | Readium Kotlin Toolkit `3.4.0` (`readium-shared` / `readium-streamer` / `readium-navigator` / `readium-navigator-media-tts`, 路线图 D2 / D18) |
| 平台版本插件 | `io.github.supermonster003.autojs6-platform-versions` 1.8.3 |
| 发布文件名 | `autojs6-plugin-readium-epub-reader-v{VERSION_NAME}-{CRC32}.apk` (单 APK) |

## 3. 工作区与提交

### 3.1 会话开始

- MUST 运行 `git status --short`, 检查当前分支, 最近提交和相关文件差异.
- MUST 将已有未提交内容视为用户工作. 不覆盖, 不回滚, 不擅自整理与当前任务无关的改动.
- 禁止使用 `git reset --hard`, `git checkout -- <path>` 或其他可能丢失用户内容的命令, 除非用户明确授权.
- 先阅读 `ROADMAP.md` 的 "阶段总览" 与最后一条 "会话记录", 从路线图建议的起点开始.

### 3.2 开发过程

- 每个行为改动应同时考虑实现, 测试, 10 语言资源, README, changelog, 宿主入口和公共契约.
- 不提交本地缓存, IDE 状态, 调试输出, 真实书籍或无意生成的二进制文件; `docs/fixtures/` 只放生成器产物或许可证明确的样本 (第 15.4 节).
- Gradle 自动修改 `BUILD_TIME` 时, 在确认来源后与相关变更一并处理. 若 Gradle 修改 `VERSION_BUILD`, 必须按第 3.4 节的提交计数规则校正; `VERSION_NAME` 只按语义化版本规则调整.
- 修改第三方依赖时同步记录版本, 来源, 校验值与许可证 (`THIRD_PARTY_NOTICES.md`), 并在 changelog 的 `dependency` 分类记录.
- 路线图条目完成后在 `ROADMAP.md` 勾选并写入证据 (设备, API, 样本, 度量值), 不勾选没有证据的条目.

### 3.3 提交

- 除非用户明确要求本次会话不要提交, 会话结束前 MUST 将本次范围内的全部文件按逻辑提交, 一个路线图子项一个提交.
- 使用 Conventional Commits 风格: `feat:`, `fix:`, `docs:`, `build:`, `test:`, `ci:`, `chore:`, 可加作用域, 例如 `feat(reader): ...`, `feat(explorer): ...`, `feat(binder): ...`.
- 一个提交表达一个完整意图; 行为实现, 对应测试和对应 changelog 通常放在同一提交.
- 提交前 MUST 审阅 `git diff --check`, `git diff --cached`, `git status --short`, 确认没有密钥, 密码, 令牌, 本地路径, 临时 APK, 真实书籍或无关改动.
- 本仓库的提交作者邮箱 MUST 为 `30370009+SuperMonster003@users.noreply.github.com` (仓库级 `git config user.email`), 不使用个人邮箱.
- 会话结束时最终 `git status --short` 无输出; 若发现无法纳入本次提交的用户改动, 停止自动提交并向用户说明.

### 3.4 提交计数

- `VERSION_BUILD` MUST 与当前分支 `HEAD` 可达的 Git 提交数一致.
- 每次准备新提交时, 先用当前提交数加 1 得到即将产生的 build number, 写入 `version.properties`, 再把该文件与本次逻辑改动一并提交. 不要先写成当前提交数再提交.

```bash
next=$(( $(git rev-list --count HEAD 2>/dev/null || echo 0) + 1 ))
sed -i "s/^VERSION_BUILD=.*/VERSION_BUILD=$next/" version.properties
```

最后一笔提交完成后 MUST 验证 `VERSION_BUILD == git rev-list --count HEAD` 且 `git status --short` 无输出. 若发现不一致, 将 `VERSION_BUILD` 设置为 "当前提交数 + 1" 并创建一笔有明确含义的校正提交.

### 3.5 版本名称

- `VERSION_NAME` 从 1.0.0 开始, 按语义化版本管理, 与提交数量不绑定; 1.0.0 在路线图 P8 发布, 之前的提交都属于 1.0.0 的开发构建.
- 修改 `VERSION_NAME` 时同步更新全部 changelog JSON 的版本 key, README, 发布文件名断言与测试夹具, 再运行文档生成器.

## 4. 仓库结构

```text
AutoJs6-Plugin-Readium-EPUB-Reader/
|-- .changelog/                 lang_*.json x 10 + template_changelog.md (文案源)
|-- .github/workflows/          build.yml, markdown.yml
|-- .python/                    generate_markdown.py (+ .bat), check_markdown.bat, generate_launcher_icons.py, generate_fixtures.py, tests/
|-- .readme/                    common.json, lang_*.json x 10, template_readme.md, README-*.md (生成)
|-- app/                        Android 插件 (Explorer Action 服务, 阅读器, 资源, JVM 与 instrumentation 测试)
|   |-- sm003.jks               本地签名密钥, Git 忽略
|   `-- src/{main,test,androidTest}
|       `-- main/java/.../reader/          插件根包: 服务, Activity, 策略, 兼容记录
|           `-- book/                      PfdResource, BookOpener, BookFingerprint, ByteRanges (Readium 接入层)
|-- build-logic/                org.autojs.build.{utils,versions,signs,jvm-convention,...} 约定插件
|-- docs/                       16kb.md, explorer-action-compatibility.md, development/repository-standard.md
|   |-- dev/                    阶段证据 (p0-readium-spike.md 等)
|   `-- fixtures/               生成的 EPUB 样本 + SHA256SUMS.txt + README.md (androidTest assets 来源)
|-- gradle/                     libs.versions.toml, explorer-action-compatibility.properties, wrapper/
|-- libs/                       宿主 API AAR (哈希锁定, 见 libs/README.md)
|-- locks/                      host-api-aars.lock
|-- AGENTS.md, ROADMAP.md, README.md (生成, 简体中文), LICENSE (MPL-2.0), THIRD_PARTY_NOTICES.md
|-- build.gradle.kts, settings.gradle.kts, gradle.properties, version.properties
`-- sign.properties             本地签名配置, Git 忽略
```

不要仅为目录整齐创建空模块. 宿主侧的契约模块, 脚本 API 与文档位于各自仓库 (第 10 节), 不放入本仓库. `tools/` 被 `.gitignore` 忽略, 可复用的脚本一律放 `.python/`.

## 5. Gradle 与版本平台

### 5.1 在线平台版本插件

- MUST 使用在线仓库中的 `io.github.supermonster003.autojs6-platform-versions` (当前 1.8.3). 升级时先确认新版本已能从公共仓库解析, 并与其他官方插件仓库统一升级.
- 禁止使用 `mavenLocal()`, 禁止本地平台版本实现, 禁止提交 `gradle/data` 消费端覆盖.
- 平台插件只在根 `settings.gradle.kts` 应用一次, 且整个 `plugins` 块位于 `includeBuild("build-logic")` 之前; `build-logic/settings.gradle.kts` 不应用它.
- 根 `build.gradle.kts` 用 `System.getProperty("gradle.agp.version")` 声明 `com.android.application` 并 `apply false`; 模块只应用插件, 不硬编码版本. 版本逃生门只用 `version.properties` 的 `OVERRIDDEN_*`, 常规构建保持 `NONE`.
- `app` 模块从 `version.properties` 和 `org.autojs.build.versions` 读取 compileSdk, minSdk, targetSdk, versionCode, versionName.
- 不声明 `org.jetbrains.kotlin.android`; Kotlin 支持由 AGP 内置能力与约定插件提供. `gradle/libs.versions.toml` 的 AndroidX 版本与 Readium 3.4.0 传递依赖保持一致, 不低于 Readium 的要求.
- `isCoreLibraryDesugaringEnabled = true` MUST 保留 (Readium 经 kotlinx-datetime 使用 `java.time`, minSdk 24).

验收命令 (模拟 GitHub Actions 的 Temurin 环境, 日志 MUST 只有一段 `Version information for IDE platform and Gradle plugins`):

```powershell
.\gradlew.bat --no-daemon '-Djava.vendor=Eclipse Adoptium' '-Djava.vendor.version=Temurin-21.0.12.1+1' :app:assembleDebug :app:testDebugUnitTest
```

### 5.2 仓库边界

- Gradle 构建 MUST 自包含. 禁止引用仓库外部的 JAR, AAR, `flatDir` 或兄弟项目路径 (例如 `../AutoJs6/...`).
- 宿主 API AAR MUST 复制到 `libs/` 并在 `locks/host-api-aars.lock` 记录小写 SHA-256; `app/build.gradle.kts` 在配置期校验文件存在, 非 debug 命名, 哈希匹配, 锁文件键集合精确, 且 `explorer-action-api.sha256` 与 `gradle/explorer-action-compatibility.properties` 的 `explorerActionApiSha256` 一致. 更新 AAR 时同步更新锁文件, 兼容属性, `docs/explorer-action-compatibility.md`, `THIRD_PARTY_NOTICES.md` 与契约测试.
- `explorer-action-api.aar` 是冻结的 v1 描述符 (协议 v2 复用); 升级到 v4+ 需要不同的 host session 资源模型, 只改目录版本号是禁止的 (见兼容矩阵 "Upgrade procedure").
- 宿主与插件需要同步更新时分别修改各仓库 (宿主 `D:/idea-projects/AutoJs6`), 不通过跨仓库相对路径制造隐式耦合.

### 5.3 签名与发布构建

- `sign.properties` 与 `app/sm003.jks` 从宿主复制到相同相对路径, MUST 保持被 Git 忽略 (`git check-ignore` 验证). 仓库中不得出现密码, token, 私钥或开发者绝对路径; `local.properties` 同样忽略.
- 保留 `org.autojs.build.signs`, `signingConfigs` 与 release 签名选择逻辑.
- `appendDigestToReleasedFiles` 任务 MUST 保留该名称, 依赖 `assembleRelease` 与 `verifySignedReleaseArtifacts`, 在签名缺失时失败, 校验实际 APK 集合恰为 `autojs6-plugin-readium-epub-reader-v{VERSION_NAME}.apk`, 并追加 CRC32 生成 `autojs6-plugin-readium-epub-reader-v{VERSION_NAME}-{CRC32}.apk` 到 `app/releases/` (不入库).

### 5.4 不启用 ABI 拆分的理由

插件完全由 Kotlin / Java 字节码与普通资源构成 (Readium 及其传递依赖 media3, jsoup, kotlinx 均为纯 JVM 库), 拆分包内容实质相同, 不会带来下载或兼容性收益. 因此:

- 不配置 `splits.abi`, 不配置 `ndk.abiFilters`, 每次发布只有一个 APK; `nativeAlignment { expectNoNativeLibraries }` 在构建期拒绝意外引入的原生库.
- `getInfo()` MUST 显式写有 `supportedAbis = emptyArray()`, 测试断言其为显式空数组.
- 16 KB page size 检查不适用 (`NATIVE_PAGE_ALIGNMENT` meta-data 为 0); 若未来引入含原生库的依赖, 本节作废并需补齐 ABI 与 16 KB 验证.

### 5.5 R8 与 Readium

- `app/proguard-rules.pro` 只保留插件包, `PluginInfo` 与 Explorer Action API; Readium 通过各 AAR 自带的 consumer 规则保活, 不在本仓库复制其规则. 任何 R8 缺失类告警 MUST 以显式规则或依赖调整解决, 不用 `-dontwarn` 一刀切.
- 引入或升级运行时依赖后 MUST 执行 `:app:assembleRelease` (缺失类只会在这里暴露), 并把 release APK 体积记入 changelog 或 `docs/dev/`; Readium 升级需重跑路线图 P0.2 的完整清单 (D18).
- `isShrinkResources = true` 保持开启; 由 Readium 导航器按名称加载的资源 (Readium CSS, 脚本资产) 由其 consumer 规则保护, 若出现运行时缺资源, 先查 R8 报告再决定是否 `keep`.

## 6. Manifest 与激活协议

- Manifest MUST 声明 `org.autojs.permission.PLUGIN`, `<queries>` 宿主包名, `org.autojs.plugin.WAKE_ACTIVITY` 与 `org.autojs.plugin.info.AUTHOR` meta-data, `NATIVE_PAGE_ALIGNMENT=0`.
- `WakeActivity` MUST 为 `exported=true`, `Theme.NoDisplay`, `excludeFromRecents`, `finishOnTaskLaunch`, 受 PLUGIN 权限保护, 响应 `org.autojs.plugin.action.WAKE` + DEFAULT category, 启动后立即结束, 不做任何副作用.
- `PluginInfoService`, `ExplorerActionService` 与 `EpubReaderActivity` MUST `exported=true` 且受 PLUGIN 权限保护; `EpubReaderActivity` 只响应 `org.autojs.plugin.EXPLORER_ACTION_EXECUTE`, 独立入口 (路线图 P4) 落地时以另外的 Activity 承载 Launcher 与 `ACTION_VIEW application/epub+zip` (D27), 不把执行 Activity 直接导出给任意应用.
- 所有对外组件逐项审查 `android:exported`; 除契约入口外不得导出其他组件.
- `android:usesCleartextTraffic="true"` 是维护者决定 (路线图 D31: 书内 `http://` 资源照常加载), Manifest 注释 MUST 保留该说明; 更新检查 (D28) 仍只走 HTTPS.
- 权限清单当前只包含 `INTERNET` 与 PLUGIN; 路线图 P3 (TTS 前台服务) 落地时追加 `FOREGROUND_SERVICE`, `FOREGROUND_SERVICE_MEDIA_PLAYBACK`, `POST_NOTIFICATIONS` (仅开启朗读时请求), 并同步 `PluginContractInstrumentationTest` 的权限集合断言, README 安全章节与 changelog. 不申请存储, 媒体, 无障碍或悬浮窗权限 (D19).

## 7. PluginInfo 与能力协商

- `PluginRuntimeInfo.kt` 的 `readiumEpubReaderPluginInfo()` 负责 Android 侧读取 (包版本, 本地化描述, `@raw/plugin_instruction`, 构建日期); 身份常量集中在 `ReadiumEpubReaderPlugin`, 审计数值集中在 `EpubReaderExplorerCompatibility` (来自 BuildConfig).
- `name` 与不可翻译的 `app_name` 一致; `description` 来自当前 locale 的 `plugin_description`; `versionName` / `versionCode` 来自 `PackageInfo`; `versionDate` 来自 `plugin_version_date` (`MMM d, yyyy`, `GMT+08:00`); `id` / `engine` / `variant` 与第 2 节一致.
- `capabilities` 包含 `PluginCapabilityKeys.REQUIRES_HOST_VERSION` (Long) 与 `ExplorerActionCapabilityKeys.PROTOCOL_VERSION` (Int); 路线图 P5 的 `epub` 服务另有 `EpubCapabilityKeys`, 两个服务的 `id` / `variant` / 版本字段 MUST 一致, 只有 `engine` 与 `capabilities` 不同 (D10).

## 8. Explorer Action 契约与 Binder

- 公共常量, key, 动作 ID 与 placement MUST 来自 `explorer-action-api` AAR 与 `ReadiumEpubReaderPlugin`, 禁止散落字符串字面量.
- 目录 (`readiumEpubReaderActionCatalog()`) 声明恰好两个动作 (主按钮 + 溢出), `TARGET_FILE`, `ACCESS_READ_ONLY`, MIME `application/epub+zip`, 扩展名 `epub`; 目录键集合由 instrumentation 测试精确断言.
- `EpubReaderIntentPolicy.resolve()` 是唯一的入口校验: 动作 ID, 协议版本恰为 v2, 宿主版本 >= 5269, 主界面来源, 读 + 前缀授权, 两条 `content://` URI (无 query / fragment), 父子关系 (`EpubReaderPathPolicy.isDescendant`), 两项 ClipData, 显示名清洗与 EPUB 格式门 (`isSupportedEpub`). 任何一项不满足即拒绝, 不猜测.
- 父目录 URI 只做校验, 永不访问 (EPUB 自包含, D9).
- CONDITIONAL (路线图 P5): `org.autojs.plugin.EPUB` 服务与 `epub-api` 契约落地后, 已发布 AIDL 演进只在末尾追加方法并通过 `CONTRACT_VERSION` 协商; 请求 / 响应为 `Bundle` 固定 key 下的 JSON, 所有 Binder 输入做边界校验, 上限常量与路线图附录 B.5 一致; 阅读器会话经宿主两步启动 (D12), 插件服务不从后台启动 Activity.

## 9. 阅读器核心与网络约束

- 书籍 MUST 只经只读 `ParcelFileDescriptor` 或 `content://` URI 进入 (D11): `book/PfdResource` 以 `FileChannel` 定位读实现 Readium `Resource`, `sourceUrl` 为 null 以走 `StreamingZipArchiveProvider`; 任何路径都不把 EPUB 复制到缓存或解压到磁盘. `PfdResource.close()` 关闭描述符, 调用方不得重复关闭.
- `book/BookOpener` 是唯一的 Readium 打开入口 (`AssetRetriever` + `DefaultPublicationParser` + `PublicationOpener`, `pdfFactory = null`), 先确认 `Format.conformsTo(Specification.Epub)` 再解析; 错误映射为 `BookOpenError`, 其 `message` 可直接展示. LCP 标记的书籍由 Readium fallback content protection 拒绝, 不做解密.
- `book/BookFingerprint` (D23): `quickKey` (大小 + 首 1 MiB + 末 64 KiB) 为临时键, `fullKey` (全文件 SHA-256) 为正式键; 二者与 `ByteRanges` 保持 Android-free 以便 JUnit 覆盖.
- 书内脚本与远程资源保持 Readium 默认行为 (D6): 不剥离 `<script>`, 不拦截请求, 不注入 Readium 之外的 JavaScript 接口, 不开放 `file://`. 外部链接只放行 `http` / `https`, 默认先确认再交给系统浏览器 (D25).
- 阅读器状态 (`Publication`, `EpubNavigatorFactory`, 最近 `Locator`) 只驻留 `EpubReaderViewModel`; 进程被杀后从 Intent 重新打开, 不恢复导航器片段 (`createDummyFactory`).
- 程序化跳转 (目录, 从头开始, 后续的书签 / 搜索结果) MUST 经 `EpubReaderActivity.jumpTo` 排队到导航器就绪 (`PaginationListener.onPageChanged` 首次触发) 之后再调用 `go()`: Readium 3.4.0 在初始资源加载完成前收到 `go()` 会永久停止 `currentLocator` 更新 (路线图 2026-09-19 会话记录).
- 插件不上报遥测, 不发起书籍之外的网络请求; 手动更新检查 (P4, D28) 只访问 GitHub Releases 且只走 HTTPS.
- 纯逻辑 (Intent 策略, 路径策略, 指纹, 范围裁剪, 目录扁平化, 版本比较, 偏好编解码与主题配色 (`prefs/`), 后续的文本分块) 保持 Android-free, 由 JUnit4 覆盖.

## 10. 主项目职责

若改动同时需要修改 `D:/idea-projects/AutoJs6`, MUST 遵循:

- 宿主只保留入口 (`FileUtils.TYPE.EPUB` / `PreviewerType.EPUB` / `ExplorerDocumentPreviewerPluginUi.specFor` 引导, 归档上限按类型 (D24), 契约模块, Binder 客户端, 脚本 API `epub`), 插件拥有解析与渲染的真实实现 (D20); 插件未安装或被禁用时宿主不得提供重复实现.
- 宿主先区分 `未安装`, `已安装但未激活或禁用`, `版本不兼容`, `调用失败`, `可用`, 各状态有对应提示与引导 (安装来源, 激活按钮, 所需版本).
- 更新包名, action, category, ID 或 API 时同步检查宿主注册表 (`InstalledPluginRepository`, `PluginCenterViewModel.SERVICE_ACTION_BY_ENGINE`, `PluginDefaultEnabledPolicy`), ProGuard/R8, 安装 URL, 启用状态缓存和测试夹具.
- 涉及公开脚本 API 时再同步 `AutoJs6-Documentation`, `AutoJs6-TypeScript-Declarations`, `AutoJs6-Plugin-Offline-Docs`, `AutoJs6-Plugin-Ace-Editor`.

## 11. 应用标题与字符串资源

- 用户可见字符串 MUST 覆盖 `values`, `values-en`, `values-ar`, `values-es`, `values-fr`, `values-ja`, `values-ko`, `values-ru`, `values-zh`, `values-zh-rHK`, `values-zh-rTW`; `values` 与 `values-en` 共有条目内容一致, 各语言占位符与转义一致 (`.python/tests/test_repository_contract.py` 与 `generate_markdown.py --check` 共同校验).
- `app_name` 位于 `strings_donottranslate.xml` 且 `translatable="false"`; `plugin_author`, `plugin_id`, `plugin_engine`, `plugin_variant`, `plugin_requires_host_version`, `plugin_version_date` 由 Gradle `resValue` 生成.
- 每个 locale MUST 有 `plugin_description`, 与 `.readme/lang_*.json` 的 `text_plugin_synopsis` 逐字一致: 简洁说明能力, 句尾不加终止标点, 不写 "EPUB 插件" 前缀, 不写 "适用于 AutoJs6" 等限定表述.
- `<string>` 按 `name` 升序; plurals 与数组放入各自文件.
- 所有资源与文档字符串使用 ASCII 标点 (`, . : ; ! ? ( ) [ ] / -`), 省略号用 `...` 并加 `tools:ignore="TypographyEllipsis"`; 禁止全角标点, 顿号, 弯引号 (生成器与仓库契约测试会扫描).

### 11.1 启动器图标

- `app/src/main/res/mipmap/ic_launcher*.png` (legacy 圆角 / 圆形, adaptive 前景 / 单色) 由 `.python/generate_launcher_icons.py` 确定性生成; 修改图标时修改脚本并重新生成, 不手工改 PNG. `mipmap-anydpi-v26/` 的 adaptive XML 引用这些图层与 `@color/ic_launcher_background`.
- 图标语义为翻开的书 (两页 + 书脊 + 文本行), 不沿用其他插件的图案或颜色身份; 背景色与 `values/colors.xml` 的 `ic_launcher_background` 保持一致.

## 12. README 与多语言生成

- README 与 changelog MUST 由 `.readme/*.json`, `.changelog/*.json` 与模板通过 `.python/generate_markdown.py` 生成; 生成产物不得手工编辑. `raw*/plugin_instruction.md` 手工维护, 生成器只校验存在与字符卫生.
- 修改 JSON 或模板后先运行 `py .python/generate_markdown.py`, 再运行 `py .python/generate_markdown.py --check` (CI `markdown.yml` 也会执行). 生成器校验语言集合, JSON 键与列表形状, 全角符号, 未替换占位符, 版本对齐, Explorer Action 兼容属性与矩阵, 孤儿产物与漂移.
- 根 `README.md` 是简体中文版本, 与 `.readme/README-zh-Hans.md` 同源; 语言导航必须出现 `简体中文`.
- README 先说明用户能完成什么, 再说明安装与使用; `p_status` 段 MUST 如实标明当前构建已交付与尚未交付的能力 (未落地的路线图能力不得写成已提供). 不写 Android Studio 或 IntelliJ IDEA 版本信息, 不向普通用户解释 `supportedAbis`, 签名过程等实现细节.
- README 链接必须指向本仓库的真实 release, issue, license 与生成 changelog.

## 13. Changelog

- `.changelog/` 只存放 10 个 `lang_*.json` 与模板; 生成的多语言 changelog 位于 `app/src/main/assets/doc/`.
- 涉及 `feature`, `fix`, `improvement`, `dependency` 的提交 MUST 更新当前 `VERSION_NAME` 对应 `vX.Y.Z` 条目的全部语言 JSON, `released_date` 更新为当日 `YYYY/MM/DD`.
- 分类 key 只用 `hint`, `feature`, `fix`, `improvement`, `dependency`; 标签沿用既有固定翻译 (简体中文 `提示`, `新增`, `修复`, `优化`, `依赖`; 英文 `Hint`, `Feature`, `Fix`, `Improvement`, `Dependency`; 其他语言见现有 JSON).
- `feature` 条目不以 `新增` 开头, `fix` 条目不以 `修复` 开头; `dependency` 只记录 Gradle 依赖变化, 使用 `附加`, `升级`, `移除` 等固定动作词.
- 与 AutoJs6 GitHub Issue 有关时按既有格式写明 Issue 引用.

## 14. 数据存储, 独立界面与发行历史 (CONDITIONAL, 路线图 P1.3 / P4)

- 进度与书签 (D13) 存于插件私有目录 `files/books/<指纹>/`, 键为内容指纹, 不落盘明文路径; 原子写用纯 JVM 的 `store/AtomicFiles` (临时文件 + fsync + 重命名, 不用 `android.util.AtomicFile`, 便于 JUnit 覆盖); 临时键迁移到正式指纹后在 `files/books/aliases/<临时键>` 记录别名, 打开时先经 `BookDataStore.resolveKey` 解析; 每本书书签上限 500, 书目上限 500 (LRU, 淘汰时清理悬空别名). 用户导入的字体是唯一的其它落盘内容.
- 全局阅读偏好 (D14) 存于 `files/reader-preferences.json` (`store/ReaderPreferencesStore`, 信封 `format` / `themeMode` / `preferences`, `preferences` 为 Readium `EpubPreferencesSerializer` 的 JSON); 读取经 `prefs/PreferencesCodec` 白名单 + 钳制 + 枚举校验, 损坏回退默认; 文件只存 `themeMode`, `theme` 在提交导航器时由 `ThemeMapping.resolve(themeMode, hostDarkMode)` 派生; 旧 `reader_settings.scroll_mode` 只在文件不存在时迁移一次; 写入 400 ms 去抖, `onPause` / `onCleared` 冲刷.
- 设置页与 Launcher 入口 (P4) SHOULD 跟随宿主的语言, 夜间模式和主题色 (`HostAppearanceActivity`), 宿主配置不可用时安全回退; 最近书籍只保存用户经系统文档选择器明确授予的持久 URI.
- 设置页 MUST 提供独立的 `发行历史` 入口 (`ReleaseHistory.kt`, 按当前 locale 读取 `doc/CHANGELOG-{LANGUAGE_TAG}.md`, 找不到时回退英语); 更新检查仅手动 (D28).
- 所有界面覆盖无障碍标签, RTL, 大字体, 夜间模式与进程恢复.

## 15. 测试要求

### 15.1 JVM 单元测试 (`app/src/test`)

- `EpubReaderExplorerCompatibilityTest`: 审计数值 (v2 / 5269 / 5282 / v22), 检查点单调, 宿主版本分类, 身份常量.
- `EpubReaderIntentPolicyTest`: `isSupportedEpub` 的扩展名 / MIME / 冲突容器规则, `sanitizeDisplayName`.
- `book/ByteRangesTest`, `book/BookFingerprintTest`: 范围裁剪边界, 指纹的确定性 / 覆盖范围 / 与文件名无关.
- `ReleaseHistoryTest`: locale 到 changelog 资产的映射与回退.
- `PluginRuntimeInfoTest`: 两条动作规格 (ID / 位置 / 优先级 / 目标 / 访问 / MIME / 扩展名), 共享标签与 Activity.
- `store/ProgressCodecTest`, `store/BookDataStoreTest`, `store/ProgressThrottleTest`: 进度 JSON 往返与损坏输入, 原子写 / 迁移合并 / LRU / 别名 / 非法键, 节流与冲刷.
- `book/TocFlattenerTest`, `reader/PageTurnPolicyTest`, `reader/ReaderProgressTest`: 目录扁平化上限与当前章节匹配, 点按区与音量键, 进度快照.
- `prefs/PreferencesCodecTest`, `prefs/ThemeMappingTest`, `prefs/ReaderThemeColorsTest`, `prefs/PreferenceRangesTest`, `store/ReaderPreferencesStoreTest`: 偏好信封往返 / 白名单 / 钳制 / 损坏输入, 主题模式映射, 对比度与系统栏亮度, 步进吸附, 文件读写与清除.
- 后续阶段按路线图补充: 书签编解码, 文本分块, 错误映射, 上限.

### 15.2 Android instrumentation (`app/src/androidTest`)

- `PluginContractInstrumentationTest` MUST 覆盖: 两个服务的显式绑定, `getInfo()` 身份与能力, `resValue` 身份与 Kotlin 常量一致, 目录形状与键集合, Manifest 导出 / 权限 / intent-filter (发现, INFO, 执行, Wake), 权限集合精确.
- `EpubReaderIntentPolicyInstrumentationTest`: 完整 v2 信封被接受, 协议 / 身份 / 宿主版本 / 来源 / 授权 / ClipData / 父子关系 / 格式门的每条拒绝路径.
- `book/PfdResourceInstrumentationTest`: 描述符资源的长度与定位读 (含并发), Readium 经描述符打开 EPUB 2 / EPUB 3 样本 (标题, 阅读顺序, 目录, 章节内容), 损坏样本以错误结束而不崩溃.
- `EpubReaderUiInstrumentationTest`, `EpubReaderProgressInstrumentationTest`: 经 debug `EpubReaderTestContentProvider` 用完整 v2 信封启动阅读器; 翻页, 目录跳转, 重建恢复, 错误态, 进度落盘与重开恢复, 别名, 音量键与滚动模式, 就绪前跳转重放; 证据写入 `files/p0-spike/` 与 `files/p1-evidence/`, 用 `adb exec-out run-as <包名> cat` 拉取.
- `EpubReaderPreferencesInstrumentationTest`: 偏好到达导航器 (`EpubSettings` 与 Readium CSS `--USER__fontSize`), chrome 配色, 落盘与重启恢复, 面板控件, 旧 `scroll_mode` 迁移; 证据写入 `files/p2-evidence/`.
- `book/BookFingerprintInstrumentationTest`: 大样本 (200 MiB, 空间不足时 64 MiB) 经描述符的临时键与全量哈希耗时.
- 有设备或模拟器时执行 `:app:connectedDebugAndroidTest` (API 24 与 API 35 各一次); 性能度量与正确性测试分开.

### 15.3 设备冒烟

- 阅读器界面改动后 SHOULD 在 API 24 模拟器与一台 API 33+ 真机上用 `docs/fixtures/minimal-epub3.epub` 走一次: 从宿主文件管理器打开, 翻页, 目录跳转, 旋转屏幕, 返回; 在 ColorOS 等会保持新装应用停止状态的设备上 SHOULD 做真实激活验收; 未执行时在路线图如实记录.

### 15.4 样本

- `docs/fixtures/` 由 `.python/generate_fixtures.py` 生成, `SHA256SUMS.txt` 与 `README.md` 同步; 外部样本 (IDPF / W3C) 加入前 MUST 确认许可证并在 README 记录来源与 SHA-256; 真实书籍永不入库.

## 16. CI 基线

- `build.yml`: push, pull request 与手动触发; `contents: read`; JDK 21 Temurin; 运行 AAR 门禁, JVM 测试, lint, debug / androidTest APK 与原生对齐检查, 上传产物; 在 API 24 (x86) 与 API 35 (x86_64) 模拟器上执行 instrumentation 测试.
- `markdown.yml`: Windows 环境运行 `.python\check_markdown.bat`, 阻止生成文档漂移.
- CI action 使用固定大版本并定期更新; timeout 与真实构建时长匹配.

## 17. 验证顺序

按变更范围执行最小但充分的验证:

```powershell
py .python/generate_markdown.py --check
py -3 -m unittest discover -s .python/tests
.\gradlew.bat :app:verifyExplorerActionApiCompatibility
.\gradlew.bat :app:testDebugUnitTest
.\gradlew.bat :app:assembleDebug :app:assembleDebugAndroidTest
.\gradlew.bat :app:lintDebug
.\gradlew.bat :app:assembleRelease             # 引入或升级运行时依赖后 MUST 执行: R8 缺失类只会在这里以构建失败暴露
.\gradlew.bat :app:connectedDebugAndroidTest   # 有设备或模拟器时
```

Release 前额外执行 `.\gradlew.bat :app:appendDigestToReleasedFiles`, 检查 `app/releases/` 中只出现预期的已签名单 APK 且 CRC32 与文件内容一致. 任何未执行的验证都在最终说明中明确列出原因; 构建耗时较长时给予足够时间, 不用过短 timeout 误判失败.

## 18. 许可证, 安全, 隐私与第三方内容

- 根目录 `LICENSE` 为 Mozilla Public License 2.0, README 徽章与源码头保持一致; Readium 为 BSD-3-Clause, 记入 `THIRD_PARTY_NOTICES.md`.
- `android:allowBackup="false"` 保持不变.
- 不记录书籍内容, 文件路径或用户书签到普通日志; 诊断只保留格式, 耗时, 大小与错误分类.
- 联网行为限定为书籍自身引用的远程资源 (含明文 HTTP, D31) 与手动更新检查; README 安全章节如实说明.
- 第三方代码与 AAR 必须记录来源, 版本, 校验值与许可证 (`THIRD_PARTY_NOTICES.md`); 引入或升级依赖时同一提交更新该文件.

## 19. 完成检查清单

- [ ] 第 2 节身份值在 Gradle, Manifest, Kotlin 常量, 资源, 文档与测试中一致; `PluginContractInstrumentationTest` 通过.
- [ ] 平台插件只在根 settings 应用一次, 无 `mavenLocal()`, 无外部路径引用, 无 `gradle/data`.
- [ ] `libs/` AAR 与 `locks/host-api-aars.lock`, `gradle/explorer-action-compatibility.properties` 哈希匹配, `THIRD_PARTY_NOTICES.md` 已更新.
- [ ] `sign.properties`, `app/sm003.jks`, `local.properties` 被 Git 忽略; `appendDigestToReleasedFiles` 可用.
- [ ] Wake Activity, INFO 服务, Explorer Action 服务契约完整; `getInfo()` 显式 `supportedAbis = emptyArray()`.
- [ ] 10 语言资源与文档完整, ASCII 标点, `plugin_description` 无句尾点号; 图标与样本由脚本生成.
- [ ] JSON 文案源已生成产物且 `--check` 通过; 当前版本全部语言 changelog 已更新.
- [ ] 单元测试, assemble, lint, release 通过; 有设备时 instrumentation 通过, 否则明确记录.
- [ ] `ROADMAP.md` 已勾选完成条目并写入证据; `VERSION_BUILD` 与提交数一致; `git status --short` 无输出.

## 20. 参考项目路由

只读取完成当前任务所需的参考, 不复制项目专属内容:

- Explorer Action v2 骨架, 兼容审计, 宿主外观跟随, 发行历史对话框, 文档生成器: `D:/idea-projects/AutoJs6-Plugin-HTML-Previewer`, `D:/idea-projects/AutoJs6-Plugin-Markdown-Previewer`
- 宿主 AAR 哈希锁定, `THIRD_PARTY_NOTICES.md`, 裁剪版 AGENTS, 图标生成脚本: `D:/idea-projects/AutoJs6-Plugin-Angus-Mail`, `D:/idea-projects/AutoJs6-Plugin-MCP-Server`
- 独立设置页, 跟随宿主主题与内置发行历史: `D:/idea-projects/AutoJs6-Plugin-Three-Stone-AI`
- 进度记忆与前台播放服务形态: `D:/idea-projects/AutoJs6-Plugin-3-Ember-Player`
- Readium 上游 (3.4.0 tag 的 test-app 是导航器接入的权威示例): <https://github.com/readium/kotlin-toolkit>
- 宿主入口, 插件发现, 安装和启用引导, 文档预览器模板: `D:/idea-projects/AutoJs6`

参考时以这些仓库的当前代码为准, 不以历史 README 或旧 release 中已经淘汰的写法为准.
