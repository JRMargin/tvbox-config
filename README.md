# TVBox / 影视仓 配置 + MiraPlay（猫爪 catPaw）源

本仓库同时提供两件事：

1. **MiraPlay（iPhone）能加载的猫爪源**——`index.js` + `index.config.js` 共四个文件（含两个 `.md5`），已在 iPhone 上实测加载成功。见 [一、MiraPlay / 猫爪源](#一miraplay--猫爪catpaw源)。
2. **TVBox / 影视仓通用点播聚合配置**——`config.json` 系列，107 个站点。见 [二、TVBox / 影视仓 配置](#二tvbox--影视仓-配置)。

---

## 一、MiraPlay / 猫爪（catPaw）源

### 1. 现在能用的地址（按优先级）

| 优先级 | 地址 | 实测结果 |
|---|---|---|
| **首选** | `https://gh-proxy.com/https://github.com/JRMargin/tvbox-config/releases/download/src-v1/index.js.md5` | 6643598 B，3.4 s 拉完，无 `Content-Encoding`，**iPhone 加载成功** |
| 备选（同字节） | `https://github.com/JRMargin/tvbox-config/releases/download/src-v1/index.js.md5` | **能加载**，国内家用宽带 63 s（慢但可用） |
| 调试用（同一 Wi‑Fi） | `http://192.168.1.24:8080/index.js.md5` | 秒开；需本机跑临时 node 服务（`%TEMP%\miraplay_srv2.js`），关机即失效，不是长期方案 |

四个文件经代理均可达、且都不压缩（`index.js.md5` 32 B / `index.js` 6643598 B / `index.config.js.md5` 32 B / `index.config.js` 9809 B），md5 与校验值逐个对得上。

### 2. 决定性结论：必须是两个文件成对

猫爪源不是一个文件，是**一组**：

```
index.js              ← spider 引擎本体（6.6 MB）
index.js.md5          ← 它的校验值（32 B）
index.config.js       ← 站点配置文件（9809 B）
index.config.js.md5   ← 它的校验值（32 B）
```

App 的实际请求顺序（2026-11-23 本机服务日志实录，UA `tvbox/2026062803 CFNetwork/3896.200.61 Darwin/27.2.0`）：

```
GET /index.js.md5          200
GET /index.js              200（6.6 MB，下载完成）
GET /index.config.js.md5   ← 之前这里是 404，App 随即报「源加载失败」
GET /index.config.js
```

**只镜像 `index.js` 而漏掉 `index.config.js`，表现就是「能下载完、md5 也校验过，然后提示源加载失败」**——这正是 2026-11-23 之前反复失败的真正原因。补齐配置后立刻加载成功，与原厂链接行为一致。

其余结论顺带定案：

- 文件名不可改：js 内用 `process.argv[1]` 匹配 `/(?:^|[\\/])index\.js$/`，改名后静默退出、毫无输出。
- 该 spider 是 **CatVodSpider 的 iOS 版**，启动后监听 `127.0.0.1:9988`，方案标识 `address.dynamic = "js2p://_WEB_"`。

### 3. 文件指纹

| 文件 | 大小 | md5 |
|---|---|---|
| `index.js` | 6643598 B | `3bd449f0fe7221060113d785076c73f7` |
| `index.js.md5` | 32 B | 内容即上一行 |
| `index.config.js` | 9809 B | `93f807174a2f253cdcf5ce9ee00f4c18` |
| `index.config.js.md5` | 32 B | 内容即上一行 |

四个文件与原厂（`cat.999888123.xyz`）同字节。原厂把 js 伪装成 `image/jpeg` 经 CDN 分发，本仓库改为 GitHub Release 附件（`application/octet-stream`），内容未改动。

### 4. 为什么有些地址一律加载不了：压缩传输

同一份字节、同样补齐了配置，不同托管结果不同：

| 托管方式 | Content-Encoding | 传输字节 | App 结果 |
|---|---|---|---|
| 原厂 `oss4liview.moji.com/...jpg` | 无 | 6643598 | 能加载 |
| GitHub Release 附件（本仓库） | 无 | 6643598 | **能加载** |
| 本机 node 服务（http） | 无 | 6643598 | **能加载** |
| `raw.githubusercontent.com` | gzip | 1910955 | 源加载失败 / 超时 |
| `ghfast.top` / `gh-proxy.com`（代理 raw） | gzip | 1910955 | 源加载失败 |
| `ghproxy.net` 代理 raw | gzip | 1910955 | 源加载失败 |
| `cdn.jsdelivr.net` | br / gzip | 1873681 | `.md5` 直接 403 |

规律是：**能加载的都不压缩，加载失败的都压缩**。据此判断这个壳取文件后不做解压，把压缩字节直接拿去比对 md5，自然对不上。

（说明：这是在真实托管之间做的对照，没有单独构造「同域名、压缩/不压缩」的受控实验，因此这条判定按「实测归纳」而非严格证明看待；实践上按它执行即可——选不压缩的托管。）

### 5. 报错对照表

| App 提示 | 真正原因 | 处理 |
|---|---|---|
| 格式错误，请输入catPawopen源 | 填了 JSON（如 `miraplay.json`），或地址不是 `.js.md5` 结尾 | 换 `.js.md5` 结尾的地址；JSON 是给 TVBox 系壳用的 |
| 源加载失败（下载完成、校验也过了之后报） | 缺 `index.config.js` / `index.config.js.md5` | 用第 1 节里的成对地址，不要只镜像 js |
| 源加载失败（很快报，几乎没等待） | 服务器压缩传输，或代理不通 | 换成 gh-proxy 代理 Release 附件那条 |
| 长时间卡住然后失败 | 6.6 MB 没下完（raw 直连常见 120 s 超时） | 同上 |
| 对比 md5 之后提示加载失败 | 同上，压缩导致校验失败，或配置侧 404 | 查服务器日志确认是否请求了 `index.config.js.md5` |

### 6. 「专属」到什么程度

- 已独立：spider 引擎、站点配置文件、校验值，全部落在本仓库，原链接失效也不影响加载。
- 未独立：**站点清单与网盘分享仍是它运行时远程拉取的**——`upload.baicanuc.cn/ossfiles/1768320816/` 下的 `ioswex.txt`、`guangya.txt`、`api.txt`、`quarknew.txt`、`yidongnew.txt`、`baidunew.txt`，以及 `oss4liview.moji.com/thd_file/...jpg` 站点配置。这些仍在厂商服务器上，本仓库不改变其归属，也不做二次托管。
- 站点数与原厂一致（97 个，见下方「MiraPlay 配置快照」）。2026-11-23 曾出现站点配置 `08d97aca...jpg` 返回 503、spider 日志 `[RemoteWexConfig] site config failed: timeout of 10000ms exceeded`——那是厂商侧波动，换链接解决不了。

### 7. 已排除的两种猜测（避免以后重复踩）

| 曾经的推断 | 结论 |
|---|---|
| 需要按社区约定包一层 `globalThis.websiteBundle` 才能被 App 执行 | **已证伪**。`catpaw/` 目录即该版本，与失败无关；根目录原样版补齐配置后即可加载。该目录保留作备选，不必使用 |
| 走 Release 附件就能解决「源加载失败」 | **只解决了一半**。附件确实绕开了压缩，但缺 `index.config.js` 时依旧失败；两者同时满足才可加载 |

### 8. MiraPlay 配置快照（`miraplay.json`）

`miraplay.json` 是从 spider 的 `/config` 接口原样抓下来的配置（2026-11-23 快照），与源站输出字节级一致（md5 `59f1ed9ad271e88e2056f838eb141223`，21162 B）：

> **不能填进 MiraPlay 的「猫爪源」输入框**——那个位置只接受 `.js.md5` 形式的 catPaw 源，填 JSON 会报「格式错误，请输入catPawopen源」。此文件仅供能直接吃 TVBox 系 JSON 的壳读取/查看站点清单。

- 走加速：`https://gh-proxy.com/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/miraplay.json`
- 直连：`https://raw.githubusercontent.com/JRMargin/tvbox-config/main/miraplay.json`

内容：`video` 分栏 **97 个站点**（96 个 type3 + 1 个 type4 推送），另有 `read`/`comic`/`music`/`pan` 四个分栏（源站当前为空）和 3 套主题配色 `color`。站点 key 形如 `nodejs_wogg`，`api` 为 `/spider/<key>/3`。

使用前提与限制：

- **必须与同一份 spider 配套**。配置里 `api` 是本地相对路径（`/spider/<key>/3`），由 spider 在本机注册成 `/spider/<key>/<type>/home|category|detail|search|play` 这类 POST 路由；只挂配置、不加载 `index.js`，站点一个都出不来。
- 实测：`POST /spider/wogg/3/home` 返回 `HTTP 200 42180B`，`class` 里是「玩偶电影／玩偶剧集」等真实分类；同批测试里 `douban` 的 home 返回 403（该站上游拒采，非链路问题）。
- 这是**冻结快照**：源站站点会变（其中一条站名带当天日期，快照里固定为 `日期:20261123`），要最新列表就得重新从 `/config` 抓。

### 9. 本仓库根目录文件清单（MiraPlay 相关）

| 文件 | 说明 |
|---|---|
| `index.js` / `index.js.md5` | 猫爪源引擎，必需 |
| `index.config.js` / `index.config.js.md5` | 猫爪源配置，**必需**，漏了必失败 |
| `miraplay.json` | 站点快照，仅供 TVBox 系壳读 |
| `catpaw/index.js` / `.md5` | `websiteBundle` 包装版，备选，可忽略 |

---

## 二、TVBox / 影视仓 配置

一份配置同时适配 **TVBox** 与 **影视仓**（FongMi 系）空壳。仅使用跨壳通用的 csp（type3）与苹果 CMS（type1/type0），已剔除 `.py`、drpy、本地 GoProxy 等非通用源。

当前共 **107 个站点**：type3（csp）97 个、type1 苹果 CMS 9 个、type0 1 个。

### 订阅地址（按网络情况选一条）

| 配置 | 适用情况 | 地址 |
|---|---|---|
| `config.json` | 能直连 GitHub 的网络（raw，spider 也走 raw） | `https://raw.githubusercontent.com/JRMargin/tvbox-config/main/config.json` |
| `config.json`（jsDelivr） | 配置走 CDN 加速，但 spider 仍走 raw | `https://cdn.jsdelivr.net/gh/JRMargin/tvbox-config@main/config.json` |
| `config-cdn.json`（推荐） | 电视直连 raw 不稳；spider 走 gh-proxy 加速 | `https://gh-proxy.com/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/config-cdn.json` |
| `config-cdn.json`（jsDelivr） | 同上，配置文件本身走 CDN | `https://cdn.jsdelivr.net/gh/JRMargin/tvbox-config@main/config-cdn.json` |
| `config-md5.json` | 壳要求 spider 带 md5 校验才加载 | `https://gh-proxy.com/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/config-md5.json` |

> 三个配置文件的 107 个站点完全相同，**唯一区别是 `spider` 字段**。

### 关于 spider.jar 的分发（重要）

97 个 csp 源（网盘 4K、网盘直连、网盘搜索、动漫、短剧、磁力等）全部依赖 `spider.jar` 加载成功；jar 加载失败时，只剩 10 个不依赖 jar 的苹果 CMS 能返回数据。

实测结论：

- **jsDelivr 拒绝分发 `.jar`**：`cdn.jsdelivr.net` / `fastly.jsdelivr.net` / `gcore.jsdelivr.net` 对 `spider.jar` 一律返回 `403`（9 字节 `Forbidden`）。因此 jsDelivr 只能用来加速配置文件，不能用来托管 jar。
- `raw.githubusercontent.com` 直连在国内电视上经常超时或被墙，1.9MB 的 jar 更容易中断。
- 可用的 jar 加速地址（均实测 200、文件完整）：
  - `https://gh-proxy.com/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/spider.jar`
  - `https://ghfast.top/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/spider.jar`
- jar 指纹：大小 **1909012 B**，md5 **`9f7227651023177d2a036830180add15`**，zip 内含 `classes.dex`。`config-md5.json` 即按此 md5 校验。
- jar 走加速（会压缩）对 TVBox 系壳没问题——它们能解压；**这条经验不要套用到 MiraPlay 的猫爪源**，那个壳不吃压缩。

### 导入方法

1. 打开 TVBox / 影视仓，进入「设置」→「配置地址」（影视仓为「订阅」）。
2. 粘贴上面任一地址，确定后等待加载，返回首页即可看到全部站点。
3. 更换地址后必须**重新拉取配置**才会生效；必要时清除 App 数据后再导入。

### 故障排查：首页空白 / 搜索只有几个站点有结果

这两个现象是同一个根因：**配置加载成功了，但 `spider.jar` 没加载上**，于是 97 个 csp 源全哑，只有 10 个苹果 CMS 有返回。

按顺序排查：

1. **先自检网络**：用电视或手机浏览器打开
   `https://gh-proxy.com/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/spider.jar`
   - 能下载出约 1.9MB 的文件 → 网络没问题，问题在壳这边，换 `config-md5.json`。
   - 下载失败或文件很小 → 该加速地址不通，换 `ghfast.top` 那条或改本地 jar。
2. **新文件 404**：jsDelivr 对刚提交的新文件有边缘节点同步延迟，期间会短时返回 404，几分钟后自愈；着急就直接用 gh-proxy 前缀的地址。
3. **粘贴检查**：订阅地址首尾不要带空格或换行。
4. **兜底方案（本地 jar）**：若所有远程地址都不通，把 `spider.jar` 放到电视本地存储，配置里的 `spider` 改成本地路径，可完全绕开网络问题。

### 网盘播放与 Cookie

- 网盘 4K 聚合源（玩偶、快映等）点开即可播放，无需登录。
- 「阿里/夸克/UC/百度/迅雷」等网盘直连源，需在 App 的「网盘及弹幕配置」中填入你自己的凭证：
  - 夸克、UC、百度：粘贴对应网盘网页版 Cookie；
  - 阿里：扫码登录或填 32 位 token。
  Cookie 是你的私人凭证，只保存在本机，本配置不内置任何账号。

### 源清单（共 107 个站点）

| 类别 | 数量 |
|---|---|
| 网盘4K聚合 | 19 |
| 网盘直连 | 8 |
| 网盘搜索 | 5 |
| 影视/其它 | 34 |
| 苹果CMS | 10 |
| 动漫 | 6 |
| 磁力 | 5 |
| 短剧 | 6 |
| 体育 | 5 |
| 音频 | 5 |
| 少儿教育 | 4 |

另含解析接口 29 个、直播 2 条，以及去广告/嗅探规则。

---

## 三、变更记录

**2026-11-23** — MiraPlay 猫爪源修复并定案：

- 查明「下载完成 → md5 校验通过 → 源加载失败」的真因：缺 `index.config.js` 与 `index.config.js.md5`。补齐后 iPhone 实测加载成功。
- 补齐文件到仓库根目录与 Release `src-v1`（四附件：`index.js`、`index.js.md5`、`index.config.js`、`index.config.js.md5`）。
- 修正此前两处误判：`websiteBundle` 包装非必需（`catpaw/` 降级为备选）；Release 附件只能绕开压缩，不能替代配置文件。
- 解决速度问题：raw/ghfast 等压缩托管一律不可用，改由 gh-proxy 代理 Release 附件，6.6 MB 从 63 s 降到 3.4 s。
- 补记一条 TVBox 侧经验：jar 可走压缩加速，MiraPlay 猫爪源不行。

---

## 说明

- 所有接口均经过联网实测，失效域名已剔除；公开资源站域名可能随时变动，后续可重新检测更新。
- 第三方加速（gh-proxy、ghfast、jsDelivr）为公共服务，可用性与稳定性不由本仓库控制；若失效，可回退到 GitHub Release 直链或本地服务。
- 公开资源仅供个人学习，请自行确认当地版权与合规要求，遵守相关法律法规。
