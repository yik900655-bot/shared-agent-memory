# KIE.ai Nano Banana 2 API 接入验收

日期为 2026-11-11。**服务端接入及模拟验收通过。真实 KIE 图片生成、费用和图像质量待验证。**

## 官方依据与请求流程

KIE 将 Nano Banana 2 对应到 Google Gemini 3.1 Flash Image。模型标识是 `nano-banana-2`。选用它是因为用户已确定供应商和模型，且官方支持参考图、正方形 PNG 和异步查询，能满足截图生成图标的接口需要。[KIE 模型说明](https://kie.ai/nano-banana-2) · [Nano Banana 2 请求规范](https://docs.kie.ai/market/google/nanobanana2)

| 步骤 | 实际使用的请求与响应 |
| --- | --- |
| 1. 上传参考图 | Next.js 先检查并解码 PNG、JPEG 或 WebP，再缩放至最长边不超过 1600 px，重新编码为 WebP。服务端以 multipart POST 请求 `https://kieai.redpandaai.co/api/file-stream-upload`，发送 `file`、`uploadPath` 和随机 `fileName`。检查响应的 `code`、`success`（如有）及 `data.downloadUrl`。上传域名取自官方示例命令。官方另有 Base64 上传接口，模型的 `image_input` 要求文件 URL。参考图通过临时 URL 传给模型。 |
| 2. 创建任务 | POST `https://api.kie.ai/api/v1/jobs/createTask`。JSON 包含 `model` 和 `input`，后者为 `prompt`、`image_input: [上传返回的 URL]`、`aspect_ratio: "1:1"`、`resolution: "1K"`、`output_format: "png"`。HTTP 成功后还检查业务 `code: 200`，从 `data.taskId` 取得查询 ID。 |
| 3. 查询状态 | GET `https://api.kie.ai/api/v1/jobs/recordInfo?taskId=…`。继续等待 `waiting`、`queuing`、`generating`。成功时解析 `data.resultJson` 字符串，取 `resultUrls[0]`。失败时识别 `state: "fail"` 和 `failCode`，以经过过滤的错误反馈给浏览器。官方 `failMsg` 和响应 `msg` 可能含请求详情，应用只展示固定错误文案或数字错误码。 |
| 4. 下载与转换 | 只下载允许的 KIE 文档示例 HTTPS 主机上的结果，禁止跳转，下载请求不携带鉴权头。校验图片格式和大小后，生成 512×512 PNG 与 150×150 PNG。浏览器通过同源路由查看缩略图、全尺寸预览并下载 512×512 文件。 |

三类 KIE 调用均由服务端发出。上传、创建和查询使用 Bearer 鉴权。上传时使用用户点击生成后提交的截图，临时上传地址留在服务端。官方说明上传文件在 24 小时后删除。临时地址的访问控制和真实返回的媒体主机仍需实际调用时核对。[文件流上传](https://docs.kie.ai/file-upload-api/upload-file-stream) · [Base64 上传](https://docs.kie.ai/file-upload-api/upload-file-base-64) · [任务查询](https://docs.kie.ai/market/common/get-task-detail) · [鉴权和账户限制](https://docs.kie.ai/getting-started)

## 应用实现与费用控制

- 本地开发副本为 `roblox-kie`，分支 `roblox-kie-api`，应用提交 `38e83923729ea5a84fee1b7b8a08cb9b0c40301c`。基线是 Photobooth 官方提交 `5c18c2abd96f4345e5f0ba2484c71280814f1b16`。原版单独保存。
- 首页沿用原版上传和风格布局，加入游戏名、描述、可选构图和 1～3 张数量控制。默认 3 张。一次上传服务多个任务，独立任务顺序执行，使用近景主角、斜向动作和环境构图三套指令。结果页沿用卡片、全屏预览和下载。
- 浏览器点击生成前保存本地任务 ID，主动 POST 一次。结果页挂载或刷新只 GET 本地状态。同一 ID 的并发重复提交复用已有任务，输入冲突返回 409。首次提交 ID 超过 90 秒失效。一个本地进程同一时间运行一组任务，避免叠加提交。
- 服务端请求超时为 15 秒，每张图任务总等待上限为 120 秒，KIE 查询间隔为 5 秒。查询遇到 429 或服务器错误可继续等待。创建任务无自动重试。仅 KIE 确认失败的图允许用户手动重试一次，保留已完成结果。遇到创建状态不明、超时或结果获取失败，停止该组剩余提交并提示查看 KIE 控制台。
- 本地结果完成后可访问 30 分钟，任务及图片存放于单进程内存。进程重启会清除状态，刷新仍只查询，不自动重建。此方案适用于本地阶段。多实例和重启后的持久幂等、用户授权及生产配额需在部署前设计。

## 代码修改

| 范围 | 应用文件 |
| --- | --- |
| KIE 服务、任务和图片 | `services/kie-imagegen.ts`、`app/api/photobooth/route.ts`、`lib/server/icon-jobs.ts`、`lib/server/image-files.ts` |
| 页面、状态和输入 | `app/page.tsx`、`app/results/page.tsx`、`hooks/use-photobooth-capture.ts`、`hooks/use-photobooth-results.ts`、`lib/photobooth-session.ts`、`lib/constants.ts`、`services/imagegen-api.ts`、`types/photobooth.ts` |
| 现有布局内的控件 | `components/photobooth/icon-inputs.tsx`、`desktop-style-panel.tsx`、`mobile-style-strip.tsx`、`style-options-list.tsx`、`result-image-card.tsx` |
| 替换的旧实现 | 移除原版的 `components/photobooth/model-selector.tsx`、`lib/image-models.ts`、`lib/photobooth-results.ts`、`lib/sse.ts`。 |
| 配置、依赖和测试 | `.env.example`、`README.md`、`package.json`、`package-lock.json`、`tests/photobooth.test.ts` |

`.env.example` 提供空的 `KIE_API_KEY` 和 `KIE_MODEL`。后者为空时使用 `nano-banana-2`。真实密钥仅填产品副本中被 Git 忽略的 `.env.local`。缺少密钥时本地创建路由返回 HTTP 503 和明确提示，且不会发起网络请求。

## 验收证据

| 项目 | 结果 |
| --- | --- |
| TypeScript | `npx --no-install tsc --noEmit` 通过。 |
| ESLint | `npm run lint` 通过，0 错误、0 警告。 |
| 生产构建 | `npm run build` 通过。 |
| 模拟自动测试 | `npm test` 10/10。覆盖输入与格式校验、缺少密钥、异步状态和超时、429、部分成功、重复提交、只重试失败项、输出 PNG 与访问边界。 |
| Chrome 生产构建 | 服务端网络请求全部由 Mock 替换。快速双击产生 1 次本地创建 POST，刷新增加 0 次，点击重试增加 1 次。模拟上游共上传 1 次、创建 4 次，包含初始 3 次和失败项重试 1 次。核对 150×150 缩略图、512×512 全屏图及 PNG 文件头和尺寸。 |
| 密钥隔离 | 浏览器请求只访问本地站点。用虚拟凭据标记构建并检查静态资源、浏览器 HTML 与 API JSON，均未发现该标记。浏览器未收到 KIE 临时 URL。Git 忽略 `.env.local`。 |
| 真实调用 | BLOCKED。未配置真实密钥，也无本轮费用授权。真实上传、任务创建、查询及下载次数均为 0。 |

## 安全审计与下一步

- 请求校验包括截图 4 MiB 上限、JSON 6 MiB 上限、实际图片签名与解码、像素限制、字段长度、风格与数量。结果下载采用 HTTPS 主机白名单、12 MiB 下载上限和解码限制。上传前页面提示用户截图会发送至 KIE。
- 图片转换直接使用 Sharp 0.35.5，对应原版审计中 librsvg 告警的修复版本。Next.js 与 React 保持原版锁定版本。官方 npm registry 审计报告 9 项（1 Critical、2 High、6 Moderate），其中生产依赖 2 项（Next.js Critical，`source-map-js` High）。这些问题未阻塞本地模拟开发，严重依赖问题须在正式部署前处理和复测。[Sharp 公告](https://github.com/advisories/GHSA-wq5f-xc86-pv6w) · [Next.js 公告](https://github.com/advisories/GHSA-2464-fhr4-w9xw) · [source-map-js 公告](https://github.com/advisories/GHSA-68fv-2mgg-jv7q)
- 取得明确费用授权并在本地配置密钥后，先选 **1 张**，使用非敏感测试截图验证临时上传、任务 ID、状态、真实图片、512×512 PNG、150×150 预览和 KIE 账单。单张通过后，再在授权范围内验证三种构图、部分失败和刷新时的实际任务数。KIE 当前账户限额与真实响应仍待确认。
