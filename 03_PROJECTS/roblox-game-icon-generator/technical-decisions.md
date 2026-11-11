# 技术决策

日期为用户确认并录入共享仓库的日期。原决策若被替代，保留正文并补上新决策的链接。

| 编号 | 日期 | 决策 | 理由或依据 | 暂缓或放弃的方案 | 证据来源 | 是否已被取代 |
| --- | --- | --- | --- | --- | --- | --- |
| ROBLOX-001 | 2026-11-11 | 继续寻找开源底座，优先验证 Vercel AI SDK Image Generator。ShipAny 暂不购买。 | 当前阶段的目标是寻找开源底座。AI Wallpaper 和 AI Cover 已完成源码审查。 | 暂缓购买 ShipAny。AI Wallpaper、AI Cover 不列为首选。 | 用户确认及[底座调研](research.md) | 现行，尚未被取代 |
| ROBLOX-002 | 2026-11-11 | 先完成可用的本地网站，再考虑正式部署。 | 将本地可用性作为进入部署评估的条件。 | 在本地网站可用前安排正式部署。 | 用户确认及[项目目标](README.md) | 现行，尚未被取代 |

Vercel 候选的本地验收结果见 [开发状态](development-status.md)。

## ROBLOX-003 · Photobooth 产品副本接入 KIE.ai

2026-11-11，用户指定在已通过本地验收的 Photobooth 上开始 API 改造，并选择 KIE.ai 的 Nano Banana 2。KIE 对外模型标识为 `nano-banana-2`，支持参考图、正方形 PNG 和异步状态查询。对本阶段而言，独立 Photobooth 产品副本提供已运行的上传和结果界面。现有候选调研记录继续保留。[原版验收](local-validation-report.md) · [KIE 模型文档](https://docs.kie.ai/market/google/nanobanana2)

服务端使用 `KIE_API_KEY` 和 `KIE_MODEL`，上传经校验的截图取得临时 URL，再创建 1～3 个顺序任务并查询状态。服务端下载及转码为 512×512 PNG 和 150×150 预览。浏览器只有显式提交才创建任务，结果页刷新只查询。服务端按任务 ID 去重，超时或状态不明时停止继续创建。仅明确失败的项可手动重试一次。密钥、原始上游错误详情和临时参考图地址留在服务端。该决定现行，真实图片和费用待验证。[接入验收](kie-api-integration-report.md)
