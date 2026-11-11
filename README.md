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
