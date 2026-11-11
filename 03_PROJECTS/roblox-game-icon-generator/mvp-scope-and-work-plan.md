# Roblox Game Icon Generator｜MVP 范围与工作量

审查日期 2026-11-11。以应用分支 `roblox-kie-api` 的实际提交 `38e83923729ea5a84fee1b7b8a08cb9b0c40301c` 为基线。最新决定是先上线一个最小英文工具，观察 30 天的搜索与使用数据，再决定是否继续投入。本文的首发范围优先于历史文档里的多风格、多图计划。

**本地纯文字单张闭环约 2.5～3.5 小时。达到 Vercel 公网上线验收约 11～17 个工程小时。** 按每天 8 小时约 1.5～2 个工作日，上限略超过两天。估算以 KIE 真单验证顺利、合适的 Vercel 计划、站点 URL、部署权限和共享状态资源可用为前提。账号开通、权限审批、DNS 生效和 KIE 供应商排障的等待时间另计。

## 首发范围

- 英文首页保留游戏名称、游戏描述和一个 Generate 按钮。提交后等待，查看单张预览并下载 512×512 PNG。
- 继续使用 KIE.ai Nano Banana 2，服务端固定一次一张和一份默认提示词。默认用文字生成。参考图入口建议首发隐藏，保留现有上传组件、校验和服务端上传代码。若选择首发同时开放参考图，预计多 1～2 小时做可选分支、提交大小限制与模拟验证，真实上传鉴权排障另计。
- 隐藏风格选择、张数选择、构图参数、摄像头及批量重试入口。结果卡、全屏预览和下载继续复用。复杂功能和已做的原型代码保留供后续判断。

## A｜已完成，可复用

| 现有能力 | 源码位置 | 证据、改动影响与剩余风险 |
| --- | --- | --- |
| 英文表单和响应式布局 | `app/page.tsx`、`components/photobooth/icon-inputs.tsx`、`desktop-style-panel.tsx`、`mobile-style-strip.tsx` | 已有名称、描述、构图、1～3 张选择、六种风格及图片上传与摄像头。默认 3 张，生成必须先选图及风格。复用名称、描述、按钮与现有容器。 |
| 现有单张与参考图生成链路 | `app/api/photobooth/route.ts`、`services/kie-imagegen.ts`、`lib/server/icon-jobs.ts`、`types/photobooth.ts` | 选 1 张时服务端只创建一个任务，参考图会先校验、压缩并上传。当前实现始终要求上传，纯文字路径尚未接通。KIE 的同一个模型接口支持纯文字，官方请求示例使用 `image_input: []`。这是应用表单、校验和提示词的小改动，无需新供应商。[KIE 模型 API](https://docs.kie.ai/market/google/nanobanana2) |
| 异步状态、超时与失败处理 | `services/kie-imagegen.ts`、`services/imagegen-api.ts`、`hooks/use-photobooth-results.ts`、`lib/photobooth-session.ts` | 已有 KIE 状态查询、15 秒单次请求超时、120 秒任务超时，浏览器刷新只查状态，创建超时无自动重提。单机内去重已通过模拟。保留查询、状态转换和等待 UI。公网需处理跨实例状态、待确认任务和用量限制。 |
| 预览和真实 PNG 处理 | `lib/server/image-files.ts`、`app/results/page.tsx`、`components/photobooth/result-image-card.tsx`、`result-preview-modal.tsx`、`lib/browser/file-utils.ts` | Sharp 实际编码 512×512 和 150×150 PNG，API 返回 `image/png`，结果页已有预览、全屏和下载。本轮重跑的模拟测试验证了格式和尺寸，历史浏览器模拟也验证了下载字节。真实 KIE 图片仍待验收。 |
| 构建与基础输入防护 | `package.json`、`package-lock.json`、`middleware.ts`、`tests/photobooth.test.ts` | 同一提交在隔离副本中 `npm run build` 通过，包含类型与 ESLint 检查。现有模拟测试 10/10。密钥在服务端，已有限长、文件签名、图片解码和下载主机限制。Origin/Referer 检查可限制跨站浏览器请求，服务端脚本仍可伪造请求头。 |

当前默认 Prompt 能表达正方形、清楚的视觉焦点、对比度和小尺寸辨识度。它同时要求以截图为依据，拼接风格和三种构图。首发只需把它收敛为一份文字生成提示，保留名称和描述，补上清晰光影、单一主体与可辨识轮廓的要求。可选参考图若开放，只在有图时添加保留原主体的句子。真实画面质量和游戏元素保真度待验证。

## B｜小改动，合计 7～11 小时

| 工作 | 涉及文件 | 影响与验收标准 | 估算 |
| --- | --- | --- | --- |
| 1. 首页收敛为名称、描述、生成按钮 | `app/page.tsx`、`components/photobooth/icon-inputs.tsx`、`desktop-style-panel.tsx`、`mobile-style-strip.tsx`、`app/results/page.tsx` | 复用输入与结果组件。隐藏拍摄、上传、风格、构图和数量选择，界面一次只提交一张。320/390 px 宽度下主表单、结果和下载可操作。 | 1～1.5 小时 |
| 2. 接通纯文字单张并保留一个默认 Prompt | `types/photobooth.ts`、`app/api/photobooth/route.ts`、`lib/server/icon-jobs.ts`、`services/kie-imagegen.ts`、`tests/photobooth.test.ts` | 服务端固定 1 张，关闭公网多图和付费重试入口。无参考图时跳过文件上传，按官方结构传空图数组。模拟验证每个提交只创建一个任务，错误能区分创建、查询和下载。 | 1.5～2 小时 |
| 3. 定向修补依赖 | `package.json`、`package-lock.json` | 对 Next.js 15.5 分支和受影响的传递包做最小升级。复跑构建、模拟测试和生产审计。失败时核查依赖树与版本覆盖。 | 1～2 小时 |
| 4. 补齐首页 SEO 与市场验证事件 | `app/layout.tsx`、`app/page.tsx`、`lib/constants.ts`、`app/results/layout.tsx`（新增）、`app/robots.ts`、`app/sitemap.ts`（新增）、生成路由与结果页 | 首页的英文 Title、Description、H1 和 canonical 指向同一个产品。结果页加 `noindex`，不进 sitemap。配置 Search Console，并记录生成受理、生成成功和下载动作，不采集游戏描述或图片。 | 1.5～2.5 小时 |
| 5. 真单、预览环境与生产验收 | `README.md`、`.env.example`、相关测试及 Vercel 环境变量 | 单张真实调用需另获授权或由用户操作。核对 KIE 任务记录和实际 PNG，模拟或受控复测重复请求、失败恢复、限额与移动端。验证生产 URL、搜索可抓取和事件回传。 | 2～3 小时 |

第 1、2 项约 2.5～3.5 小时，完成后可在本地验收纯文字流程。第 5 项工时包含一次正常真单和常规部署验证。若此前 401 在纯文字路径复现，需要按出错阶段追加排查，完成日期随供应商或账号处理时间变化。

## C｜非小改项与暂缓范围

| 工作 | 涉及文件 | 影响、风险与决策 | 估算 |
| --- | --- | --- | --- |
| Vercel 跨实例状态和公网额度控制 | `app/api/photobooth/route.ts`、`lib/server/icon-jobs.ts`、`services/kie-imagegen.ts`、`hooks/use-photobooth-results.ts`、`tests/photobooth.test.ts`，以及一个服务端共享存储模块 | 选 Vercel 时是上线必要项。建议用最小共享 KV 保存原子提交记录、任务 ID、状态和用量计数，附到期时间。POST 在返回前持久化 KIE 任务 ID，GET 查状态，结果按需在服务端取回并转码。未知是否已创建时保持待确认，防止自动再扣费。跨实例同一提交只创建一次，限流或预算超限在创建前阻断。 | 4～6 小时，计入总量 |
| 参考图公开上线 | 既有 `capture-stage.tsx`、`hooks/use-photobooth-capture.ts`、`lib/server/image-files.ts`、生成路由、KIE 服务及测试 | 暂缓入口，保留代码。若开放，需压低客户端大小上限或调整传输方式，并独立验收上传链路。当前 4 MiB 原图经 Base64 后约 5.33 MiB，超出 Vercel Functions 4.5 MB 请求限制。 | 首发计 0 小时。若纳入，基础改造另加 1～2 小时 |
| 多风格、批量、图片理解、增强、抠图、Roblox ID、素材库、登录、支付、图库与后台 | 现有风格组件和多任务原型保留。未来新增能力涉及表单、任务服务、数据与权限 | 本轮暂缓。共享 KV 只承载短期任务和费用控制。用户数据库、永久图片存储与后台均无首发需求。 | 首发计 0 小时 |

## 公网上线的阻塞和配置

1. **任务状态跨请求。** 当前在 `globalThis Map` 中保存任务与 PNG Buffer，POST 启动 `void run(...)` 后立刻返回 202。重启会丢状态，Vercel 请求可能落到不同实例并随流量缩到零。仅延长超时或使用后台生命周期机制无法解决跨实例共享。把 KIE 创建和状态查询拆成明确的服务端请求，并保存最少状态，可以保留现有结果轮询交互。还需验证用户刷新、部署重启以及创建响应丢失时的行为。[Vercel Functions 生命周期](https://vercel.com/docs/functions) · [Vercel 载荷和时长限制](https://vercel.com/docs/functions/limitations)
2. **公共入口的费用。** 当前「同时只运行一组、内存最多 20 组」是本地限制，每实例独立。公网应在发请求前执行服务端限流和全站预算上限，并配置独立 KIE Key 的日／总额度等可用上限。对空额度、401、429、超时和已完成结果分别返回稳定状态。统计生成与下载时只记录事件和结果状态。[KIE 鉴权与用量上限](https://docs.kie.ai/getting-started)
3. **真实 KIE 仍未打通。** 上一轮用户提交时页面出现 401，同一 Key 后续通过了余额查询及无文件上传鉴权检查。失败发生在上传、创建或查询的哪一步仍未知。先验收纯文字单张可跳过参考图上传这一步，实际请求能否成功需真单验证。本轮仅使用模拟测试。
4. **部署输入。** 当前服务端需要 `KIE_API_KEY`，`KIE_MODEL` 可省略并默认 `nano-banana-2`。现有中间件可读 `NEXT_PUBLIC_APP_URL`，建议设置最终站点完整 origin，供同源校验及 SEO 使用。Vercel 的 `VERCEL_URL` 是平台变量。共享状态凭据和预算上限是拟新增配置，具体名称随实现确认。应用仓库当前无 Git remote，部署需准备独立应用源码入口。商业用途适用的 Vercel 计划、共享状态资源、域名和部署权限均未验证。[Vercel 环境变量](https://vercel.com/docs/environment-variables/managing-environment-variables) · [Vercel Hobby 使用范围](https://vercel.com/docs/plans/hobby)

## 当前 SEO 和移动端实测

- 隔离生产构建的首页及 `/results` 均返回 200。首屏 HTML 自带名称、描述和生成按钮，语言为 `en`。首页 Title 还是 `Photobooth ImageGen Demo`，Description 仍描述照片、多风格和流式结果。无 H1、canonical、robots 元标签，`/robots.txt` 与 `/sitemap.xml` 都返回 404。结果页当前也无 `noindex`。补一个明确的产品 H1、相符的简短英文说明和自指 canonical，生成一页 sitemap 并让结果页可被读取其 `noindex`。Google 收录结果通过 Search Console 核对。[Google JavaScript SEO 基础](https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics)
- 320 和 390 CSS px 的浏览器视口下，当前首页输入、上传和按钮可见，页面无整体横向溢出。390 px 的结果页空状态正常。真实结果图的手机端预览和下载仍待上线前验收。

## 当前依赖安全结论

本轮从官方 npm registry 重新审计，完整依赖 **9 项告警**，包括 1 Critical、2 High、6 Moderate。生产依赖 **2 项**。告警没有阻塞本机隔离构建和测试，实际漏洞可达性没有验证。

| 依赖 | 实测版本与风险 | 最小处理 |
| --- | --- | --- |
| Next.js | 15.5.25，Critical。npm 还报告同版本存在 High 和 Moderate 公告。已确认 Next 实际共用 Sharp 0.35.5，符合 Critical 公告给出的 Sharp 缓解条件。 | 公网前定向升同分支 15.5.28 或经复审的 15.5 安全补丁，并复跑回归和审计。[Next.js Critical 公告](https://github.com/advisories/GHSA-2464-fhr4-w9xw) · [Next.js High 公告](https://github.com/advisories/GHSA-p3m5-59mp-jv5r) |
| `source-map-js` | 1.2.1，生产依赖 High，经 PostCSS 等引入。 | 刷新兼容锁定版本至至少 1.2.2，必要时用定向 override，确认生产审计通过。[source-map-js 公告](https://github.com/advisories/GHSA-68fv-2mgg-jv7q) |
| `brace-expansion` | 1.1.18 和 2.1.4，High，只在当前开发依赖树。Moderate 的 `minimatch` 与相关 TypeScript ESLint 条目也在开发树。 | 同批定向修补开发链，保持各分支兼容。已确认 1.1.21 与 2.1.7 可从 npm 获取。剩余开发链告警按可达性记录和复查。[brace-expansion 公告](https://github.com/advisories/GHSA-q2hr-2g5m-vwhr) |

原版的 Sharp High 在当前审计中已消失。未修改依赖。公网发布验收以生产依赖无 Critical/High 为准。

## 上线验收与 30 天观察

1. **产品闭环。** 英文首页只需名称和描述即可生成。授权真单返回一张可预览和下载的 512×512 PNG，验证文件头、尺寸和主体可辨识度。桌面和移动端都能完成任务。
2. **可靠与费用。** 双击、刷新、并发重复提交与跨实例查询只对应一次创建。创建超时、401/429 和额度耗尽时有明确反馈，待确认的请求不会自动创建第二个任务。实际 KIE 任务数可核对。
3. **公开验收。** 首页 HTTPS 200，初始 HTML 有表单和产品信息，canonical 对应正式 URL。robots、sitemap 和结果页索引规则正确。生产审计 Critical/High 为 0，检查密钥隔离和应用只读查询。
4. **观察数据。** 从上线日起记录 30 天。Search Console 看收录、展示与自然搜索点击，工具侧看生成受理、生成成功与下载动作。下载按钮事件若无法确认文件交付，只报下载点击。无明显反馈时按最新决定暂停。如果因收录或埋点失效缺少数据，先在复盘中标明无法据此判断需求。

最推荐的执行顺序是先走通纯文字单张并验证一次真实 KIE，再完成 Vercel 状态和费用护栏、依赖补丁、最小 SEO 与数据验收。本轮在无真实密钥的隔离副本中构建并执行模拟测试，只更新共享记忆文档。

历史证据见 [原版验收](local-validation-report.md) 与 [KIE 接入验收](kie-api-integration-report.md)，当前阶段见 [开发状态](development-status.md)。
