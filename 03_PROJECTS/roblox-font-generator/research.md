# Roblox Font Generator 调研记录

记录日期 2026-11-12。当前假设是 Roblox 用户希望把一段文字转换成可复制的 Unicode 样式，并用于自己选择的目标字段。首批获客候选为 Google 英文自然搜索，承接页为单页工具。搜索意图、可争位置与字段兼容性待实测。

## 已收到的数据

| 项目 | 记录 | 来源与验证状态 |
| --- | --- | --- |
| 核心词 | `roblox font generator` | 用户指定的调研词。 |
| 美国月搜索量 | 1600 | 哥飞 Agent 报告，2026-09，美国。原始面板、匹配口径未独立核验。 |
| 哥飞版 KD | 36.6 | 同一转述。指标口径及当前值待核验。 |
| 趋势 | 相对平稳 | Agent 转述，观察窗口与原图待核验。 |
| 技术 | A 类，纯前端 Unicode 映射 | 项目初判。映射边界、复制及目标字段行为待预检。 |
| MVP 预计 | 约一天 | 以竞品体验和字段预检通过为前提。 |

## 已发现竞品

| 页面 | 2026-11-12 可读页面的观察 |
| --- | --- |
| [robloxfonts.com](https://robloxfonts.com/) | 页面展示专门的 Roblox 文本转换工具、样式和复制控件。兼容标签为站方陈述。 |
| [iTextKit Roblox Font Generator](https://itextkit.com/tools/roblox-font-generator) | 工具集中有专门页面，公开内容展示输入、样式和逐条复制。 |
| [J-Kit Roblox Font Generator](https://jkit.tools/en/tools/roblox-font-generator) | 专门页面可读，抓取内容显示工具加载占位，交互待浏览器验证。 |
| [gamefonttools.com](https://gamefonttools.com/) | 页面展示 Unicode 复制与 Roblox Studio 字体代码两种用途。本项目比较其中的复制任务。 |

上述发现用于选择实测对象。排名、流量、付费节点、复制结果及 Roblox 兼容性尚未实测。

## Roblox 官方限制

Roblox 区分用户名与显示名称。显示名称适用平台文字过滤，通常每七天可改一次。官方说明日文字符支持仍为部分用户可用的测试功能。用户名是独立的账号标识，修改需要 Robux，且受平台过滤。[Roblox 显示名称说明](https://en.help.roblox.com/hc/en-us/articles/4401938870292-Changing-Your-Display-Name) · [Roblox 用户名说明](https://en.help.roblox.com/hc/en-us/articles/203313130-Changing-Your-Username)

按字段分别记录粘贴、平台校验、实际保存和重新打开后的显示。粘贴成功只能证明输入可进入字段。没有实测的 Unicode 样式不得承诺可用于用户名或显示名称，也不得把单次结果泛化到其他账号、字段或设备。

## 45～60 分钟预检

1. 从四个候选中选三个，以同一组含大小写、数字和空格的样本文字体验输入 → 展示 → 复制。记录核心卖点、价格或付费节点、卡点和输出。
2. 用代表样式在实际 Roblox 目标字段做验证，区分显示名称、用户名与其他展示文字字段。实际保存、账号变更和花费按授权执行，无法执行的步骤标为未验证。
3. 分别判断技术能否转换复制、样式是否可读、结果能否在目标字段完成任务。若核心场景在预检中无法成立，缩小到已有证据的场景或暂缓。确认可用字段和最低质量后才进入单页 MVP。

按 [立项与交付方法](../../01_PRINCIPLES/tool-site-validation-playbook.md) 记录反证、来源、实测日期和完成标准。[当前开发状态](development-status.md)
