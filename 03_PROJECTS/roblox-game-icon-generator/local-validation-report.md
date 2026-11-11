# OpenAI ImageGen Photobooth 原版本地验收

执行日期为 2026-11-11（太平洋时间）。**本地安装、启动和基础交互已验收。真实图片生成 BLOCKED，第二阶段尚未全部完成。**

## A. 项目来源和环境

| 项目 | 实际核验结果 |
| --- | --- |
| 官方仓库 | [openai/openai-imagegen-demo](https://github.com/openai/openai-imagegen-demo) |
| 分支和提交 | `main` · [`5c18c2abd96f4345e5f0ba2484c71280814f1b16`](https://github.com/openai/openai-imagegen-demo/commit/5c18c2abd96f4345e5f0ba2484c71280814f1b16)。克隆后用 `git ls-remote origin refs/heads/main` 核对 SHA 一致。 |
| 许可证 | MIT。[该提交的 LICENSE](https://github.com/openai/openai-imagegen-demo/blob/5c18c2abd96f4345e5f0ba2484c71280814f1b16/LICENSE) |
| 本地路径 | `/Users/mac/Documents/Obsidian Vault/热词快站/Roblox-Game-Icon-Generator/photobooth-upstream/` |
| 环境 | macOS 26.5.1（arm64），Node.js `v24.15.0`，npm `11.12.1`，Chrome `157.0.8103.50`，Playwright `1.65.0`。 |
| 清单和结构 | 已读取原版 README、`package.json`、`package-lock.json`、`.env.example` 和许可证。锁文件格式为 v3，锁定 Next.js `15.5.25`、React `19.1.0`。已核对 `app/`、`components/`、`hooks/`、`lib/`、`public/`、`services/`、`tests/` 和 `types/`。 |

项目目录沿用共享仓库已存在的 `03_PROJECTS/roblox-game-icon-generator/`。原版克隆独立存放。验收结束时原版 Git 跟踪文件干净，`HEAD` 与 `origin/main` 一致。

## B. 安装情况

`npm ci` 退出码 0，耗时 7.880 秒，安装 381 个包。安装输出包含 139 个包的 funding 提示，未出现安装警告或错误。安装使用原版锁文件。

从 `.env.example` 复制出被 Git 忽略的 `.env.local`，生成密钥字段保留空值。开发和生产预览均在回环地址运行。

## C. 运行情况

`npm run dev -- --hostname 127.0.0.1 --port 3000` 在 2.6 秒后就绪。访问地址为 `http://127.0.0.1:3000/`。首次冷访问返回 HTTP 200，耗时约 7.53 秒，包含开发编译时间。Chrome 中实际打开首页，标题为 `Photobooth ImageGen Demo`。检查到 4 个脚本、1 个样式表和 2 个字体资源成功加载，首页本地资源请求失败和浏览器错误均为 0。

构建后用 `npm start -- --hostname 127.0.0.1 --port 3000` 额外启动生产构建，`/`、`/results` 和 `/icon.png` 均返回 HTTP 200。

## D. 功能验收

Chrome 使用桌面窗口和移动触控模拟。结果页的模拟验收通过浏览器测试环境给原版前端提供分段的 SSE 测试响应，普通几何图形作为测试图。真实本地 API 验证使用空密钥，服务端在调用上游前返回错误。真实 OpenAI 生成请求为 0。

| 功能 | 状态 | 实测证据 |
| --- | --- | --- |
| 开发启动、首页及静态资源 | PASS | 开发服务器就绪，Chrome 首页 HTTP 200，脚本、CSS、字体加载成功。 |
| 文件选择上传、预览和清空 | PASS | 用页面的 Upload Photo 选择 PNG、JPEG、WebP，均可解码预览为 128×192。关闭 PNG 预览后回到初始状态。 |
| PNG 拖拽上传 | PASS | 拖入时显示投放提示，放下后预览实际解码为 128×192。 |
| 摄像头权限、预览和拍摄 | PASS | Chrome 允许访问后，在本机摄像头及独立的虚拟摄像头测试中均拍出 640×480 预览，拍摄后关闭视频流。模拟拒绝权限时显示可继续上传的提示。真实摄像头画面仅在当前测试页内验收，随后清空。 |
| 非图片格式拒绝 | PASS | 上传 `text/plain` 文件显示 `Please drop or upload an image file.`，生成按钮保持禁用。 |
| 浏览器无法解码的 TIFF | FAIL | 有效 TIFF 文件以 `image/tiff` 上传后预览解码报 `EncodingError`，宽度为 0，无错误提示，生成按钮仍可点击。 |
| 损坏 PNG 的拒绝与提示 | FAIL | 标记为 `image/png` 的损坏文件复现同一问题。 |
| 文件读取错误与恢复 | PASS | 在浏览器中模拟 FileReader 错误后显示 `Unable to process uploaded image.`。再次选择有效 PNG 可正常预览。 |
| 大图处理 | PASS | 1500×1500、6,753,623 字节的 PNG 可预览。点击生成时页面提示图片太大并留在首页。 |
| 风格列表、选择、取消和多选 | PASS | 六种风格均可见，默认选中 Knitted 和 Digital Art。取消后可重选，选满四项时其他风格禁用，取消一项后恢复。 |
| 模型与复杂度 | PASS | 五个模型可见，可切换模型。GPT Image 2.6 默认 High，可选 Low 和 Medium。切到 GPT Image 2 后隐藏复杂度控件。刷新后保留模型，复杂度恢复 High。 |
| 首页响应式布局 | PASS | 测了 320、375、390、768、1024、1440 CSS px。输入、模型、生成控件可见，页面无整体横向溢出。移动端风格条可横向滑动。 |
| 原版 API 本地校验与缺少密钥的界面 | PASS | 风格目录 HTTP 200 且返回 6 项，外部 Origin HTTP 403，非法模型 HTTP 400。用有效输入点击生成，真实本地路由返回 HTTP 500，结果页显示 `OPENAI_API_KEY is not configured` 和重试按钮。 |
| 真实生成请求、流式结果、多风格、真实失败重试与保留结果 | BLOCKED | 当前无可用的环境密钥和费用授权，真实调用及输出质量待验收。 |
| 模拟流式结果、多风格和失败重试 | PASS | 在浏览器中先发送 partial，再让 Knitted 完成、Digital Art 失败。点重试后仅请求 Digital Art，已完成的 Knitted 图片保持不变，最后两张均可预览。还核对了请求中的两种风格、模型和 Medium 复杂度。 |
| 结果展示、全屏预览和 PNG 下载 | PASS | 在模拟数据下点击结果可打开覆盖整个视口的预览。卡片和预览均可下载，核对文件名与 PNG 字节内容。样本 `knitted-portrait.png` 为 632 字节，`digital-art-portrait.png` 为 631 字节，均为 128×192。 |
| 刷新与 Create another set | PASS | 模拟完成后刷新 `/results` 会重发原风格请求，旧结果图不会立即恢复。Create another set 返回首页、清除会话。无会话直接打开结果页有明确提示，且不会发起生成。 |
| 移动端上传、风格、结果和下载交互 | PASS | Chrome 触控模拟在 320 和 390 CSS px 走通上传、触控选风格、选模型、模拟结果、卡片下载和预览下载。两个宽度页面均无整体横向溢出。 |
| 320 CSS px 结果卡片完整显示 | FAIL | 卡片内 Download 文案被裁切。Digital Art 操作行可视宽 136 px、内容宽 164 px，按钮右边界超出卡片约 27 px。390 CSS px 下同一卡片未裁切。 |
| iOS Safari 和 Android 实体设备 | NOT TESTED | 本次移动端证据来自 Chrome 触控模拟。 |

## E. 自动化检查

| 命令 | 退出码和耗时 | 错误数与结果 | 对本地使用的影响 |
| --- | --- | --- | --- |
| `npm run lint` | 0 · 8.422 秒 | 0 错误，0 警告。 | 可继续使用。 |
| `npm test` | 0 · 2.606 秒 | 10 通过，0 失败，0 跳过。原版测试使用模拟请求，覆盖来源检查、输入校验、模型与复杂度、SSE、结果保留和会话。 | 可继续使用。 |
| `npm run build` | 0 · 18.520 秒 | 0 个最终构建错误。字体下载出现 1 次 `getaddrinfo ENOTFOUND fonts.gstatic.com`，Next.js 内置重试后编译、类型检查和静态页面生成成功。 | 生产构建可本地启动。若再次遇到 DNS 错误，先确认本机能解析并访问字体服务，再重跑相同命令。 |
| `npm audit` | 1 · 0.727 秒 | 默认 `registry.npmmirror.com` 返回 HTTP 404 和 `[NOT_IMPLEMENTED]`，1 次审计接口失败。 | 已改用官方 registry 对相同锁文件执行只读审计。 |
| `npm audit --registry=https://registry.npmjs.org` | 1 · 34.531 秒 | 官方审计完成。10 项依赖告警（1 Critical、3 High、6 Moderate）。退出码由发现告警产生。 | 本机回环验证可继续。进入部署评估前应核对受影响路径、更新依赖并重新审计。 |
| `npm audit --omit=dev --registry=https://registry.npmjs.org` | 1 · 2.413 秒 | 补充扫描生产依赖。3 项告警（1 Critical、2 High），来自 `next`、`sharp` 和 `source-map-js`。 | 生产依赖仍有需要处理的告警。 |

官方审计报告的包与风险如下。严重级别来自扫描结果，具体触发条件以各项目公告为准。

| 依赖 | 风险等级 | 报告发现 |
| --- | --- | --- |
| `next` | Critical | 已安装 15.5.25。扫描报告包含图像优化相关远程代码执行公告、自托管缓存投毒及拒绝服务公告。[Next.js 安全公告](https://github.com/advisories/GHSA-2464-fhr4-w9xw)、[缓存公告](https://github.com/advisories/GHSA-mcj8-r9mp-w47p) |
| `sharp` | High | 已安装 0.35.4。上游 librsvg 依赖风险。[sharp 安全公告](https://github.com/advisories/GHSA-wq5f-xc86-pv6w) |
| `brace-expansion` | High | 已安装 1.1.18 和 2.1.4。模式扩展可能消耗过多 CPU 或耗尽调用栈。[GitHub 安全公告](https://github.com/advisories/GHSA-q2hr-2g5m-vwhr) |
| `source-map-js` | High | 已安装 1.2.1。特定 source map 偏移值可能导致事件循环阻塞。[GitHub 安全公告](https://github.com/advisories/GHSA-68fv-2mgg-jv7q) |
| `minimatch`、`@typescript-eslint/typescript-estree`、`@typescript-eslint/parser`、`@typescript-eslint/eslint-plugin`、`@typescript-eslint/type-utils`、`@typescript-eslint/utils` | Moderate | 扫描发现 glob 模式放大可能造成拒绝服务，并报告受影响的传递依赖链。[minimatch 安全公告](https://github.com/advisories/GHSA-g3hv-75wg-m249) |

本轮使用锁文件版本完成原版验收。应用漏洞的可达性尚未验证。

## F. 问题清单

| 问题 | 影响 | 如何复现 | 建议后续处理 |
| --- | --- | --- | --- |
| 真实生成待验证 | 阻塞核心生成验收。 | 空密钥状态下上传有效 PNG，点击 Generate Styles。结果页显示未配置提示。 | 在获得费用授权并安全配置密钥后，使用普通测试图验证一次小规模生成，再补验多风格、流式输出与失败重试。 |
| 无法解码的图片仍可提交 | 中。有效 PNG、JPEG、WebP 仍能使用。 | 用选择器上传有效 TIFF 或损坏 PNG。预览破图、宽度为 0，生成按钮仍启用。 | 改造阶段在接受文件后实际解码，遇到不支持或损坏时提示并禁用生成。 |
| 窄屏结果卡片裁切下载文案 | 中。320 CSS px 下仍可使用全屏预览下载。 | 用 320 CSS px 宽度打开带两张图片的结果页，查看卡片底部。 | 改造阶段让卡片底部换行或重新布局，在 320 CSS px 复测。 |
| 刷新结果页会重新请求 | 中。模拟时观察到重复发出原风格请求。推断在配置有效密钥后，刷新可能再次消耗 API 用量。 | 在模拟完成页刷新，观察原风格请求重发且旧结果未立即恢复。 | 改造阶段验证结果恢复或请求去重，验收时对照每次请求次数。 |
| 依赖审计有 Critical 和 High 告警 | 高。依赖修订和复核应进入正式上线前的评估。 | 用官方 registry 对原锁文件执行 `npm audit`。 | 在单独授权的依赖维护中，先检查 Next.js 和 sharp 的受影响范围及修复版本，再运行相同测试和审计。 |

## G. 最终结论

1. **下载成功。** 已克隆官方 `main`，本地 SHA 与远程分支一致。
2. **安装成功。** `npm ci` 退出码 0，安装 381 个包。
3. **启动成功。** 开发服务器与生产预览均可在回环地址打开。
4. **图片输入基本可用。** PNG、JPEG、WebP、拖拽、摄像头及预览已通过。TIFF 和损坏 PNG 的处理失败。
5. **真实生成 BLOCKED。** 本地缺少密钥的提示、模拟流式结果和模拟下载已验收，真实接口与输出质量仍待验证。
6. **底座具备继续评估的本地条件。** 进入 Roblox 产品改造的建议判断仍待真实生成验收，并需确认严重依赖风险的处理计划。
7. **待解决事项**包括真实生成、无效图片提示、320 CSS px 下载文案裁切、刷新重发的成本风险、依赖风险及实体手机验证。

项目进度见 [development-status.md](development-status.md)。
