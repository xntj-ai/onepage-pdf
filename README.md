# onepage-pdf

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-d97757.svg)](https://claude.com/claude-code)

**One HTML page → one tall, continuous-page PDF — no pagination breaks, desktop layout preserved, text still selectable.**

**一个 HTML 页面 → 一张超长的连续单页 PDF —— 不分页、不断裂,桌面版排版原样保留,文字仍可选中。**

onepage-pdf is a [Claude Code](https://claude.com/claude-code) / Agent skill. Hand it a long scrolling HTML page — a proposal, a report, a landing page — and it produces a single-page PDF: one continuous sheet instead of a stack of A4 pages with content sliced across the cuts. Backgrounds, gradients and glass effects survive; the text stays selectable; and you can optionally redact strings before export with a built-in leak check.

onepage-pdf 是一个 [Claude Code](https://claude.com/claude-code) / Agent 技能。给它一个长滚动式 HTML 页面 —— 提案、报告、落地页 —— 它就产出一张单页 PDF:一整张连续的长纸,而不是被切成一摞 A4、内容卡在分页线上断开。背景、渐变、毛玻璃都保留,文字依然可以选中;导出前还能可选地脱敏字符串,并自带泄漏校验。

It is built on one quiet but stubborn fact: **you cannot know a page's printed height up front, so this tool never tries to.** It renders onto an oversized page, measures the real content from the rendered PDF, and crops to fit. The height is cropped, not calculated — so it is exact by construction.

它建立在一个不起眼却很顽固的事实上:**你无法事先得知一个页面打印后的真实高度,所以本工具干脆不去预测。** 它先渲染到一张超大的页面上,再从渲染好的 PDF 里量出内容的真实底边,然后裁切到刚好贴合。高度是裁出来的,不是算出来的 —— 因此天然精确。

## Why onepage-pdf · 为什么用 onepage-pdf

The browser's own "Print to PDF" cannot give you a single continuous page, and naïve HTML-to-PDF exports break in predictable ways: Chrome's print pipeline evaluates media queries against the default paper width (~741px) instead of your `@page` size, viewport units drift, web fonts shift line heights, and the px→inch→pt rounding is just sloppy enough to spawn a phantom second page. A long page gets chopped into A4 slices with text severed mid-line.

浏览器自带的"打印成 PDF"给不了你一张连续的长页,而粗暴的 HTML 转 PDF 会以几种可预测的方式翻车:Chrome 的打印管线按默认纸宽(约 741px)而非你的 `@page` 尺寸来判定媒体查询,视口单位会漂移,网页字体会改变行高,而 px→英寸→pt 的取整误差又恰好大到能凭空多出一页。一个长页面被切成一片片 A4,文字从行中间被拦腰截断。

onepage-pdf takes the opposite stance from "predict then print": it renders first, measures the rendered result, and crops. The output is one exact-height page that keeps the desktop layout, keeps the backgrounds, and keeps the text selectable — the kind of PDF you can send a client in a chat app and have it just open, cleanly, on a phone.

onepage-pdf 反其道而行,不走"先预测再打印":它先渲染,再量出渲染结果,然后裁切。产物是一张高度精确的单页,保住桌面版排版、保住背景、保住可选中的文字 —— 正是那种你能在聊天软件里发给客户、对方在手机上点开就干净打开的 PDF。

## Features · 功能特性

- **One continuous page, never paginated** — renders onto an oversized "bedrock" page, then crops the MediaBox + CropBox down to the real content height. No A4 cuts, nothing sliced mid-line. **一张连续长页,永不分页** —— 先渲染到一张超大的「基岩」页上,再把 MediaBox + CropBox 裁到内容真实高度。没有 A4 切口,不会从行中间截断。
- **Height cropped, not calculated** — the printed height is measured from the rendered PDF (PyMuPDF render-level bbox log), never guessed from screen `scrollHeight`. Exact by construction. **高度是裁出来的,不是算出来的** —— 打印高度从渲染好的 PDF 里量出(PyMuPDF 渲染级 bbox 日志),绝不靠屏幕 `scrollHeight` 去猜。天然精确。
- **Desktop layout & backgrounds preserved** — `print-color-adjust: exact` keeps colors and gradients; `background-attachment: fixed` is normalized so it doesn't paint on page 1 only; scroll-reveal animations are forced visible. **保住桌面排版与背景** —— `print-color-adjust: exact` 留住颜色与渐变;`background-attachment: fixed` 被归一化,避免只画在第一页;滚动渐显动画被强制显形。
- **Selectable text** — output is real vector PDF text, not a screenshot, so it stays searchable and copy-pasteable. **文字可选中** —— 产物是真正的矢量 PDF 文字,不是截图,因此可搜索、可复制粘贴。
- **Optional redaction with leak check** — substitute strings before rendering (`--replace`) and assert a forbidden word list never survives in the HTML *or* the final PDF text (`--forbid`), aborting if it does. **可选脱敏 + 泄漏校验** —— 渲染前替换字符串(`--replace`),并断言一份禁用词清单在 HTML 与最终 PDF 文字里都不残留(`--forbid`),一旦残留就中止。
- **CJK-safe** — stages files through TEMP to dodge Chrome's flaky non-ASCII path handling, and the references cover the Noto-CJK weight / tofu pitfalls. **中文友好** —— 把文件中转到 TEMP 目录,绕开 Chrome 对非 ASCII 路径的不稳定处理,参考文档也覆盖了 Noto CJK 字重 / 豆腐块等坑。
- **Zero service, two dependencies** — a single Python script driving headless Chrome (or Edge) plus PyMuPDF. No server, no SaaS, no print microservice. **零服务,两个依赖** —— 一个 Python 脚本驱动无头 Chrome(或 Edge)加 PyMuPDF。没有服务器、没有 SaaS、没有打印微服务。

## When to use · 适用场景

Reach for onepage-pdf when an HTML report / proposal / landing page has to leave the browser as **one** clean PDF — to send in a chat app, attach to an email, or archive — and a normal print-to-PDF either paginates it into A4 slices or cuts content at the page break. It is ideal for AI-generated deliverable pages (PPVI-style proposals, data reports) that look right on a desktop and must stay that way on a phone.

当一个 HTML 报告 / 提案 / 落地页必须以**一张**干净的 PDF 离开浏览器 —— 用聊天软件发出去、当邮件附件、或归档 —— 而普通的打印成 PDF 要么把它切成 A4 片段、要么在分页处截断内容时,就用 onepage-pdf。它特别适合 AI 生成的交付页(PPVI 风格的提案、数据报告):桌面上看着对,到手机上也得保持对。

It is **not** the tool for a print job that genuinely wants paginated A4 (just print normally), nor for pixel-faithful capture of WebGL / canvas / perspective-transform scenes that the print path rasterizes away.

它**不适合**真的需要分页 A4 的打印任务(正常打印即可),也不适合像素级还原 WebGL / canvas / 透视变换那类会被打印路径栅格化掉的场景。

## Install · 安装

Clone this repository into your Claude Code skills directory:

把本仓库克隆到你的 Claude Code 技能目录:

```bash
git clone https://github.com/xntj-ai/onepage-pdf.git ~/.claude/skills/onepage-pdf
pip install pymupdf
```

Two runtime requirements: **Python 3.10+ with [`pymupdf`](https://pypi.org/project/PyMuPDF/)**, and **a local Chrome or Edge** (auto-detected on Windows / macOS / Linux, or pass `--chrome PATH`). There is no Playwright, no Node, and no service to run.

两个运行依赖:**Python 3.10+ 加 [`pymupdf`](https://pypi.org/project/PyMuPDF/)**,以及**本地的 Chrome 或 Edge**(在 Windows / macOS / Linux 上自动探测,或用 `--chrome PATH` 指定)。不需要 Playwright,不需要 Node,也没有要常驻的服务。

## Usage · 用法

**With Claude** — point Claude Code at an HTML file and ask for a single-page PDF, e.g. *"use onepage-pdf to turn this proposal into a single continuous-page PDF."* Claude inspects the source HTML, writes the breakpoint locks and glass fallbacks itself (see Options), runs the conversion, and renders a thumbnail to eyeball the result.

**对 Claude 说** —— 把 Claude Code 指向一个 HTML 文件,要它出一张单页 PDF,比如"用 onepage-pdf 把这份提案转成不分页的连续单页 PDF"。Claude 会先检查源 HTML,自己写好断点锁定与毛玻璃兜底(见「参数」),跑转换,再渲染一张缩略图来肉眼核对结果。

**On the command line** — call the script directly:

**命令行直接调** —— 直接调用脚本:

```bash
python scripts/onepage_pdf.py input.html -o output.pdf --width 1280
```

Or run it on the bundled demo (gradient background, 3-column grid, glassmorphism, scroll-reveal animations, CJK typography), passing the matching fix-CSS:

或者拿仓库自带的 demo 跑一遍(渐变背景、三列网格、毛玻璃、滚动渐显动画、中文排版),带上配套的修正 CSS:

```bash
python scripts/onepage_pdf.py examples/demo.html -o demo.pdf \
    --extra-css examples/demo-fixes.css
```

## Options · 参数

| Flag 参数 | Default 默认 | Purpose 用途 |
|---|---|---|
| `--width` | `1280` | Page width in CSS px, snapped to a multiple of 8 (8px = 6pt exactly) to dodge MediaBox rounding. Match the page's design width. 页宽(CSS px),对齐到 8 的倍数(8px 恰好 6pt)以避开 MediaBox 取整误差。设为页面的设计宽度。 |
| `--bedrock` | `18000` | Initial oversized page height in px; auto-doubles (up to `--max-retries`) if content still overflows. 初始的超大页高(px);若内容仍溢出会自动翻倍(上限由 `--max-retries` 定)。 |
| `--padding` | `32` | Whitespace kept below the content, px. 内容下方保留的留白,px。 |
| `--extra-css` | — | CSS file appended to the injected print styles — breakpoint locks, glassmorphism fallbacks, vh/vw overrides. 追加到注入打印样式后的 CSS 文件 —— 断点锁定、毛玻璃兜底、vh/vw 覆盖。 |
| `--replace` | — | JSON file `[["old","new"],...]` applied to the HTML before rendering, in order (longest/most specific first). For redaction. JSON 文件 `[["old","new"],...]`,渲染前按顺序作用于 HTML(越长越具体的放前面)。用于脱敏。 |
| `--forbid` | — | Text file, one word per line; aborts if any word survives in the HTML **or** the final PDF text. Pair with `--replace`. 文本文件,每行一个词;若任一词在 HTML **或**最终 PDF 文字里残留则中止。与 `--replace` 搭配。 |
| `--crop` | `vector` | Content-bottom detection: `vector` (render bbox log) or `pixel` (raster row scan); vector auto-falls back to pixel. 内容底边探测:`vector`(渲染 bbox 日志)或 `pixel`(栅格逐行扫描);vector 会自动回退到 pixel。 |
| `--virtual-time` | `10000` | Chrome `--virtual-time-budget` in ms; fast-forwards timer-driven JS. Raise for slow JS-built pages. Chrome `--virtual-time-budget`(ms);快进定时器驱动的 JS。JS 构建慢的页面调大。 |
| `--max-retries` | `2` | Bedrock doublings allowed when content overflows. 内容溢出时允许的基岩翻倍次数。 |
| `--chrome` | auto | Explicit browser executable path. 显式指定浏览器可执行文件路径。 |
| `--keep-temp` | — | Keep the TEMP work directory for debugging. 保留 TEMP 工作目录以便调试。 |

## Examples · 示例

[`examples/demo.html`](./examples/demo.html) — a sample proposal page (gradient background, glassmorphism cards, a 3-column grid, scroll-reveal sections, Chinese typography) that exercises every tricky case at once. Paired with [`examples/demo-fixes.css`](./examples/demo-fixes.css), which locks the 900px grid back to 3 columns and solidifies the glass cards for print.

[`examples/demo.html`](./examples/demo.html) —— 一个示例提案页(渐变背景、毛玻璃卡片、三列网格、滚动渐显分区、中文排版),把每一个棘手情况一次性都触发了一遍。配套 [`examples/demo-fixes.css`](./examples/demo-fixes.css),把 900px 处塌掉的网格锁回三列,并把毛玻璃卡片在打印时实底化。

```bash
# Redact two strings, then prove neither survives in the PDF
# 脱敏两个字符串,再证明它们都没残留进 PDF
python scripts/onepage_pdf.py input.html -o public.pdf \
    --replace subs.json --forbid words.txt
```

A successful run prints `OK 1 page, WxHpt (...px), N KB -> output.pdf`. If redaction ran, it also prints `forbid check (pdf): clean`.

一次成功的运行会打印 `OK 1 page, WxHpt (...px), N KB -> output.pdf`。若做了脱敏,还会打印 `forbid check (pdf): clean`。

## How it works · 技术原理

The screen and the printed page are two different layouts: `--print-to-pdf` renders with **print** media CSS, media queries evaluate against ~741px (not your `@page` width), late web fonts shift line heights, and the px→pt conversion rounds non-monotonically. So a measured `scrollHeight` is the wrong number, and any predicted height risks a phantom page.

屏幕和打印页是两套不同的排版:`--print-to-pdf` 用**打印**媒体 CSS 渲染,媒体查询按约 741px(而非你的 `@page` 宽度)判定,迟到的网页字体会改变行高,px→pt 的换算还会非单调取整。所以量到的 `scrollHeight` 是个错的数,任何预测出来的高度都可能凭空多出一页。

onepage-pdf sidesteps prediction entirely:

onepage-pdf 干脆完全绕开预测:

1. **Render onto a bedrock page.** Inject an `@page` rule sized `width × 18000px` (snapped to 8px) and print with headless Chrome — deliberately so tall that nothing paginates and rounding never matters. If content still overflows, the bedrock auto-doubles and re-renders. **渲染到一张基岩页上。** 注入一条尺寸为 `宽 × 18000px`(对齐到 8px)的 `@page` 规则,用无头 Chrome 打印 —— 故意做得这么高,让任何东西都分不了页、取整也无所谓。若内容仍溢出,基岩自动翻倍重渲。
2. **Measure the real content bottom.** Read PyMuPDF's render-level bbox log (text + images + vector paths + Form XObject contents), skip invisible text and the full-height body-background fill, and take the lowest real bottom edge. **量出内容的真实底边。** 读取 PyMuPDF 的渲染级 bbox 日志(文字 + 图片 + 矢量路径 + Form XObject 内容),跳过隐形文字和占满整页的 body 背景填充,取最低的真实底边。
3. **Crop to fit.** Rewrite both MediaBox and CropBox to the same exact-height rect (in raw y-up PDF coordinates), so viewers that honor MediaBox and viewers that honor CropBox agree. **裁切贴合。** 把 MediaBox 与 CropBox 重写成同一个精确高度的矩形(用 PDF 原生的 y-up 坐标),让认 MediaBox 的查看器和认 CropBox 的查看器达成一致。

If the vector bbox log keeps decorative geometry that hangs below the real content, `--crop pixel` rasterizes at low DPI and scans rows bottom-up for horizontal variance instead (a vertical gradient is horizontally uniform, so it doesn't count as content). The full rationale — the two-viewport media-query behavior, the hidden 1.5× print shrink, PDF size limits, CJK font pitfalls, and the CDP-measured-height alternative — lives in [references/mechanics.md](./references/mechanics.md).

如果矢量 bbox 日志把垂在真实内容下方的装饰性图形也算了进来,`--crop pixel` 改为低 DPI 栅格化,自底向上逐行扫描水平方差(垂直渐变是水平均匀的,因此不算内容)。完整原理 —— 两套视口的媒体查询行为、隐藏的 1.5× 打印缩放、PDF 尺寸上限、中文字体坑,以及 CDP 量高的替代路线 —— 都在 [references/mechanics.md](./references/mechanics.md) 里。

## FAQ · 常见问题

**Does it need Playwright or a browser automation framework?** No. It shells out to your local Chrome or Edge with `--print-to-pdf` and crops the result with PyMuPDF — that's the whole stack. Just Python + a browser.

**它需要 Playwright 或浏览器自动化框架吗?** 不需要。它用 `--print-to-pdf` 直接调起你本地的 Chrome 或 Edge,再用 PyMuPDF 裁切结果 —— 整个技术栈就这些。只要 Python + 一个浏览器。

**Why does my desktop layout collapse in the PDF?** Print media queries fire at ~741px, so any `@media (max-width: N)` with N ≥ 741 triggers and collapses your grid. Lock those breakpoints back to their desktop values with `!important` rules in `--extra-css` (see `examples/demo-fixes.css`).

**为什么我的桌面排版在 PDF 里塌了?** 打印媒体查询在约 741px 处生效,所以任何 N ≥ 741 的 `@media (max-width: N)` 都会触发并把网格压塌。用 `--extra-css` 里的 `!important` 规则把这些断点锁回桌面值(见 `examples/demo-fixes.css`)。

**My glassmorphism / blur disappeared.** `backdrop-filter` blur is silently dropped in PDF output ([crbug 40895818](https://issues.chromium.org/issues/40895818)). Add a print fallback that turns blur off and uses a near-opaque background: `.glass{backdrop-filter:none!important;background:rgba(255,255,255,.88)!important}`.

**我的毛玻璃 / 模糊不见了。** `backdrop-filter` 模糊在 PDF 输出里被静默丢弃([crbug 40895818](https://issues.chromium.org/issues/40895818))。加一段打印兜底,关掉模糊并改用近乎不透明的背景:`.glass{backdrop-filter:none!important;background:rgba(255,255,255,.88)!important}`。

**The PDF height looks huge — is that a problem?** A single continuous page is naturally tall. 14400pt (≈200in) is Acrobat's implementation limit, not a format limit; Chrome, Firefox, PDFium and WeChat preview render larger pages fine. The script warns past 14400pt as a compatibility note, not an error.

**PDF 高度看起来很大 —— 有问题吗?** 一张连续的长页自然就高。14400pt(约 200 英寸)是 Acrobat 的实现上限,不是格式上限;Chrome、Firefox、PDFium 和微信预览都能正常渲染更高的页。脚本超过 14400pt 时给的是兼容性提醒,不是错误。

**Chinese text renders as blank boxes (tofu).** Declare `<html lang="zh-CN">` and a CJK fallback in `font-family`; on Docker/CI install `fonts-noto-cjk`. Details in [references/mechanics.md](./references/mechanics.md#cjk-pitfalls).

**中文显示成空白方块(豆腐块)。** 声明 `<html lang="zh-CN">` 并在 `font-family` 里给一个中文兜底字体;Docker/CI 上安装 `fonts-noto-cjk`。详见 [references/mechanics.md](./references/mechanics.md#cjk-pitfalls)。

## Related · 相关

- [ppvi](https://github.com/xntj-ai/ppvi) — the visual identity for building the kind of restrained, light-glass HTML pages onepage-pdf is made to export. Make the page, then make the PDF. 用来构建那种克制的浅色玻璃风 HTML 页的视觉体系 —— 正是 onepage-pdf 要导出的那类页面。先做页面,再做 PDF。
- [flowmaker](https://github.com/xntj-ai/flowmaker) — a sibling Claude Code skill that also outputs a single self-contained HTML file. 同门的 Claude Code 技能,产物同样是一个自包含的 HTML 文件。
- [xntj.tv](https://xntj.tv) — more Claude Code workflows and skills from 张拼拼 · XNTJ. 更多来自张拼拼·XNTJ 的 Claude Code 工作流与技能。

## License · 许可证

[MIT](./LICENSE) © [张拼拼 · XNTJ](https://xntj.tv)
