# TVBox / 影视仓 通用点播聚合源

一份配置同时适配 **TVBox** 与 **影视仓**（FongMi 系）空壳。仅使用跨壳通用的 csp（type3）与苹果 CMS（type1/type0），已剔除 `.py`、drpy、本地 GoProxy 等非通用源。

当前共 **107 个站点**：type3（csp）97 个、type1 苹果 CMS 9 个、type0 1 个。

## 订阅地址（按网络情况选一条）

| 配置 | 适用情况 | 地址 |
|---|---|---|
| `config.json` | 能直连 GitHub 的网络（raw，spider 也走 raw） | `https://raw.githubusercontent.com/JRMargin/tvbox-config/main/config.json` |
| `config.json`（jsDelivr） | 配置走 CDN 加速，但 spider 仍走 raw | `https://cdn.jsdelivr.net/gh/JRMargin/tvbox-config@main/config.json` |
| `config-cdn.json`（推荐） | 电视直连 raw 不稳；spider 走 gh-proxy 加速 | `https://gh-proxy.com/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/config-cdn.json` |
| `config-cdn.json`（jsDelivr） | 同上，配置文件本身走 CDN | `https://cdn.jsdelivr.net/gh/JRMargin/tvbox-config@main/config-cdn.json` |
| `config-md5.json` | 壳要求 spider 带 md5 校验才加载 | `https://gh-proxy.com/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/config-md5.json` |

> 三个配置文件的 107 个站点完全相同，**唯一区别是 `spider` 字段**。

## iOS（MiraPlay）专用 spider 镜像

原链接 `http://wexfnw:wexfnw@cat.999888123.xyz/index.js.md5` 的 `.md5` 只是 32 字节校验值（`3bd449f0fe7221060113d785076c73f7`），真正的资源是同目录的 `index.js`——**CatVodSpider 的 iOS 版 spider**（6.6 MB，启动后监听 `127.0.0.1:9988`，方案标识 `js2p://_WEB_`）。已原样镜像到本仓库根目录，文件名与校验方式保持不变，可直接替换原链接：

| 用途 | 地址 |
|---|---|
| **猫爪（catPaw）源（MiraPlay 输入框只认这个）** | `https://raw.githubusercontent.com/JRMargin/tvbox-config/main/index.js.md5` |

- **不要把加速地址填进去**：`https://gh-proxy.com/https://raw.githubusercontent.com/.../index.js.md5` 这种路径里又套一层完整 URL 的写法，会被 MiraPlay 的格式校验判为非法，提示「格式错误，请输入catPawopen源」。猫爪源只接受形如 `https://域名/.../index.js.md5` 的地址（或 `github://<PAT>@github.com/<user>/<repo>/<branch>/index.js.md5`）。
- jsDelivr 对该文件也不可用：`.md5` 返回 **403 Forbidden**（jsDelivr 会拦 `.md5`、`.jar` 这类后缀），只有 `index.js` 本体能取到。
- 校验值 `3bd449f0fe7221060113d785076c73f7`，体积 6643598 B，镜像文件与源站 md5 完全一致。
- 实测（云主机 Node 22 跑镜像文件）：`CatVodSpiderios listening on http://127.0.0.1:9988`。

注意事项：

- 这份 js 只是 **spider 引擎**，本身不含站点清单；站点与网盘分享是它运行时远程拉取的（`upload.baicanuc.cn/ossfiles/1768320816/` 下的 `ioswex.txt`、`guangya.txt`、`api.txt`、`quarknew.txt`、`yidongnew.txt`、`baidunew.txt`，以及 `oss4liview.moji.com/...jpg` 站点配置）。这些文件仍在原厂商服务器上，本镜像不改变其归属，所以“专属”只到 spider 这一层。
- 2026-11-23 实测：站点配置 `oss4liview.moji.com/thd_file/2026/10/27/08d97aca7cc67966a19d05d5ba3ff725.jpg` 返回 503，spider 日志为 `[RemoteWexConfig] site config failed: timeout of 10000ms exceeded`——**原链接当前也拿不到站点数据**，不是本地网络问题。
- 该 js 只有在文件名以 `index.js` 结尾时才会自启（内部用 `process.argv[1]` 做正则匹配），改名会静默退出、无任何输出。
- jsDelivr 拉这个 6.6 MB 文件多次连接中断，未列入推荐地址。

### MiraPlay 配置快照（`miraplay.json`）

`miraplay.json` 是从上面那份 spider 的 `/config` 接口原样抓下来的配置（2026-11-23 快照），与源站输出**字节级一致**（md5 `59f1ed9ad271e88e2056f838eb141223`，21162 B），未做任何改写：

> **注意：这份 JSON 不能填进 MiraPlay 的「猫爪源」输入框**——那个位置只接受 `.js.md5` 形式的 catPaw 源，填 JSON 会报「格式错误，请输入catPawopen源」。`miraplay.json` 仅供能直接吃 TVBox 系 JSON 的壳读取/查看站点清单用。

- 走加速：`https://gh-proxy.com/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/miraplay.json`
- 直连：`https://raw.githubusercontent.com/JRMargin/tvbox-config/main/miraplay.json`

内容：`video` 分栏 **97 个站点**（96 个 type3 + 1 个 type4 推送），另有 `read`/`comic`/`music`/`pan` 四个分栏（源站当前为空）和 3 套主题配色 `color`。站点 key 形如 `nodejs_wogg`，`api` 为 `/spider/<key>/3`。

使用前提与限制：

- **必须与同一份 spider 配套**。配置里 `api` 是本地相对路径（`/spider/<key>/3`），由 spider 在本机注册成 `/spider/<key>/<type>/home|category|detail|search|play` 这类 POST 路由；只挂配置、不加载 `index.js`，站点一个都出不来。
- 实测：`POST /spider/wogg/3/home` 返回 `HTTP 200 42180B`，`class` 里是「玩偶电影／玩偶剧集」等真实分类；同批测试里 `douban` 的 home 返回 403（该站上游拒采，非链路问题）。
- 这是**冻结快照**：源站站点会变（其中一条站名带当天日期，快照里固定为 `日期:20261123`），要最新列表就得重新从 `/config` 抓。
- 站点内容仍依赖厂商远程资源（`upload.baicanuc.cn` 系列 txt 与 `oss4liview.moji.com` 的站点配置，后者 2026-11-23 为 503），本仓库不做二次托管。

## 关于 spider.jar 的分发（重要）

97 个 csp 源（网盘 4K、网盘直连、网盘搜索、动漫、短剧、磁力等）全部依赖 `spider.jar` 加载成功；jar 加载失败时，只剩 10 个不依赖 jar 的苹果 CMS 能返回数据。

实测结论：

- **jsDelivr 拒绝分发 `.jar`**：`cdn.jsdelivr.net` / `fastly.jsdelivr.net` / `gcore.jsdelivr.net` 对 `spider.jar` 一律返回 `403`（9 字节 `Forbidden`）。因此 jsDelivr 只能用来加速配置文件，不能用来托管 jar。
- `raw.githubusercontent.com` 直连在国内电视上经常超时或被墙，1.9MB 的 jar 更容易中断。
- 可用的 jar 加速地址（均实测 200、文件完整）：
  - `https://gh-proxy.com/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/spider.jar`
  - `https://ghfast.top/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/spider.jar`
- jar 指纹：大小 **1909012 B**，md5 **`9f7227651023177d2a036830180add15`**，zip 内含 `classes.dex`。`config-md5.json` 即按此 md5 校验。

## 导入方法

1. 打开 TVBox / 影视仓，进入「设置」→「配置地址」（影视仓为「订阅」）。
2. 粘贴上面任一地址，确定后等待加载，返回首页即可看到全部站点。
3. 更换地址后必须**重新拉取配置**才会生效；必要时清除 App 数据后再导入。

## 故障排查：首页空白 / 搜索只有几个站点有结果

这两个现象是同一个根因：**配置加载成功了，但 `spider.jar` 没加载上**，于是 97 个 csp 源全哑，只有 10 个苹果 CMS 有返回。

按顺序排查：

1. **先自检网络**：用电视或手机浏览器打开
   `https://gh-proxy.com/https://raw.githubusercontent.com/JRMargin/tvbox-config/main/spider.jar`
   - 能下载出约 1.9MB 的文件 → 网络没问题，问题在壳这边，换 `config-md5.json`。
   - 下载失败或文件很小 → 该加速地址不通，换 `ghfast.top` 那条或改本地 jar。
2. **新文件 404**：jsDelivr 对刚提交的新文件有边缘节点同步延迟，期间会短时返回 404，几分钟后自愈；着急就直接用 gh-proxy 前缀的地址。
3. **粘贴检查**：订阅地址首尾不要带空格或换行。
4. **兜底方案（本地 jar）**：若所有远程地址都不通，把 `spider.jar` 放到电视本地存储，配置里的 `spider` 改成本地路径，可完全绕开网络问题。

## 网盘播放与 Cookie

- 网盘 4K 聚合源（玩偶、快映等）点开即可播放，无需登录。
- 「阿里/夸克/UC/百度/迅雷」等网盘直连源，需在 App 的「网盘及弹幕配置」中填入你自己的凭证：
  - 夸克、UC、百度：粘贴对应网盘网页版 Cookie；
  - 阿里：扫码登录或填 32 位 token。
  Cookie 是你的私人凭证，只保存在本机，本配置不内置任何账号。

## 源清单（共 107 个站点）

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

## 说明

- 所有接口均经过联网实测，失效域名已剔除；公开资源站域名可能随时变动，后续可重新检测更新。
- 第三方加速（gh-proxy、ghfast、jsDelivr）为公共服务，可用性与稳定性不由本仓库控制；若失效，可回退到 raw 地址或本地 jar。
- 公开资源仅供个人学习，请自行确认当地版权与合规要求，遵守相关法律法规。
