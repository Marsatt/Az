# resource-parser.js（加 Hy2 版，2026-11-20）

原版：KOP-XIAO/QuantumultX `Scripts/resource-parser.js`（2026-11-20 拉取，5740 行）
改版：只加了 Hysteria2 支持，其他协议一行没动。原版备份在同目录 `resource-parser-orig.js`（不上传）。

## 能转什么

| 输入 | 输出 |
|---|---|
| `hysteria2://` / `hy2://` 订阅链接 | `hysteria2=host:port, password=..., tag=...` |
| Clash YAML `type: hysteria2` | 同上 |
| 直接写 `hysteria2=` 的 QX 格式行 | 原样透传 |

字段映射：`sni=`/`peer=` → `tls-host=`；`obfs=salamander` + `obfs-password=` 原样带过去；
`?mport=`/`?ports=` 或地址里 `host:443,8443-8450` → `ports=8443-8450`；
`?mhop=` → `hop-interval=`；`?upmbps=`/`?downmbps=` → `upmbps=`/`downmbps=`；
`insecure=1` → `tls-verification=false`（默认也是 false，跟本解析器其他协议一致）。

## 必须满足的条件

1. QX 必须是 **TF 测试版 1.9.0 build 952+**（965 也行）。App Store 1.8.1 不支持 Hy2，
   解析器会把 Hy2 节点跳过并提示升级——这是故意的版本保护，不是 bug。
2. 把这个文件传到你自己的 GitHub 仓库，拿到 raw 链接，例如：
   `https://raw.githubusercontent.com/你的用户名/你的仓库名/main/Scripts/resource-parser.js`
3. QX 配置 `[general]` 里改成你自己的链接：
   `resource_parser_url=https://raw.githubusercontent.com/你的用户名/你的仓库名/main/Scripts/resource-parser.js`
4. 机场订阅那行开 `opt-parser=true`，更新订阅。

## 实测（已在电脑上跑过，不是只改了字）

用 Node 模拟 QX 环境跑完整解析器，build 965：
- 混合订阅（2 条 Hy2 + 1 条 VLESS + 1 条 Trojan）→ 4 条全出，2 条 Hy2 行正确，
  VLESS/Trojan 不受影响。
- Clash YAML 含 1 条 Hy2 + 1 条 VLESS → 2 条全出。
- build 914 跑同一份订阅 → Hy2 两条被跳过、VLESS/Trojan 正常，符合版本保护。

## 唯一不确定的地方（先说清楚）

QX 官方 1.9.0 的 sample.conf 在 GitHub 上还是旧版，没公布 Hy2 各字段的正式名字；
QuanXNews 只公布了打码的基本写法 `hysteria2=example.com:443, password=...`。
所以 `obfs=` / `ports=` / `hop-interval=` / `upmbps=` / `downmbps=` 这几个可选字段名，
是按 Hy2 链接 + Clash + Surge 的通用叫法推断的。

先拿一条 Hy2 节点实测：
- 能连、能上网 → 全通，不用再改。
- QX 报整行无效 → 先删 `ports=` / `hop-interval=` / `upmbps=` / `downmbps=` 再试，
  只留 `host + password + obfs + tls-host + tag`，告诉我报错截图，我再改字段名。
