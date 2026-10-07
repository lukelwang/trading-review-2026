# Artifact parity 验证报告

验证日期：2026-10-06。原始文件：`source/ranking-artifact.html`；交付页面：`index.html`。
全程离线，未安装软件，未创建远程仓库或推送。页面可按 GitHub Pages 的 `main` 分支、`/` 根目录方式发布。

## 文字、源码与 HTML

| 检查 | 结果 | 证据 |
| --- | --- | --- |
| 可见文字 | 通过 | Python `html.parser` 排除 head、style、script 和新增主题按钮后，合并文本并规范空白；两份报告均为 5,305 个字符，完全相同。 |
| 数字与表格 | 通过 | 按顺序比较的 377 个数字项和 158 个表格单元格完全相同；未修改任何数字、句子或单元格。 |
| 原报告正文 | 通过 | 从 index body 中仅移除新增按钮后，17,702 字节的原报告正文逐字节相同。 |
| 原 CSS | 通过 | 源文件的完整 style 内容在 index 中原样保留，逐字节相同。新增 reset 和按钮样式位于独立 style 中。 |
| 原图表脚本 | 通过 | 原 script 内容逐字节相同，diff 为空；主题逻辑放在单独的 head script 中。 |
| JavaScript 语法 | 通过 | `node --check` 检查源文件的 1 个 script 和 index 的 2 个 script，全部通过。 |
| HTML 结构 | 通过 | 非 void 标签显式闭合且嵌套平衡；各有一个 html、head、body；唯一 title 位于 head；16 个 id 无重复。 |
| 章节链接 | 通过 | 导航的 8 个 `href="#id"` 均能找到目标元素。 |
| 外壳与依赖 | 通过 | doctype、`lang="zh-CN"`、UTF-8、指定 viewport、robots noindex、safe-area reset 均存在；唯一外部依赖为原 Google Fonts stylesheet，无外部 JS。 |
| 发布文件 | 通过 | `.nojekyll` 为空；`.gitignore` 包含 `BRIEF.md` 和 `.codex-*`，验证辅助文件及截图目录均被忽略，`PARITY.md` 可提交。 |

SHA-256：

```text
完整原始文件
6ce800563e5a07de57dfdd6e2cd9326fe611b22237480c42ae3f4dd03414d3e9
原图表 script 内容（source 与 index 相同）
cf1de17ec531956c9c906181b7b23ff847f55553c1f0f294ce0550ce15f9ff70
原 style 内容（source 与 index 相同）
2bef769598642cc22c3a7ba3015a1017ffedabf77a8b3d95044cfa07f4448c01
```

## 主题与浏览器验证边界

主题按钮位于 sticky nav 内，状态为系统 → 浅色 → 深色 → 系统。
head 中的独立脚本在解析正文和原图表脚本前读取保存的偏好并设置或移除 html 的 `data-theme`；localStorage 读写均使用 try/catch。
原 MutationObserver 和 OS 配色监听代码保持不变，图表仍在绘制时读取 CSS tokens。

主题逻辑的 18 项 Node `vm` 模拟 DOM/storage 检查通过：已保存浅色/深色在 DOMContentLoaded 前生效；缺失、系统及无效保存值使用系统状态；按钮文字与循环顺序正确；浅色/深色在模拟重载中保留；回到系统时移除保存项；分别或同时拒绝 storage 的 get/set/remove 时仍可正常切换。这些检查未使用真实浏览器，不能验证首次绘制是否闪烁或图表实际重绘。

仅根据原 CSS tokens 计算的对比度如下；这是颜色数值检查，不代表已在浏览器中观察到配色或图表重绘：

| 配色 | 正文 ink / bg | 次级正文 ink-2 / bg | 按钮 ink-2 / surface |
| --- | --- | --- | --- |
| 浅色 | 16.31:1 | 7.11:1 | 7.75:1 |
| 深色 | 16.15:1 | 10.74:1 | 9.83:1 |

实际浏览器检查受环境阻止：

- 本机存在 `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`，也存在已安装的 Python Playwright。
- Playwright 使用该 Chrome 启动时，进程立即以 SIGABRT 终止，报 TargetClosedError；清理进程另报 EPERM。
- 使用新 profile 直接启动 headless Chrome 的一次替代检查也立即终止，退出码 134，无 DOM 或截图输出。
- Computer Use 浏览器清单为空；内置浏览器入口返回 `Browser is not available: iab`。

因此以下项目**尚未实测**：

- 通过 `prefers-color-scheme` 模拟和按钮切换观察浅色/深色的实际背景、文字、三张 SVG 重绘及 ink 颜色。
- 1200px 和 400px 下的实际渲染、全页横向滚动、nav 内部横向滚动及 sticky 行为、按钮与标题的实际边界。
- 浏览器中的 localStorage 重载、首次绘制、鼠标/键盘 tooltip 和章节跳转。
- 1200px/400px 的浅色、深色四张截图及视觉检查；`.codex-shots/` 已创建但没有可用截图。

原 CSS 的 `body { background:var(--bg) }`、表格 `overflow-x:auto` 包装、nav 的 `position:sticky` 与 `overflow-x:auto` 均原样保留；按钮仅参与 nav 布局，未改动标题样式。上述事实不能替代移动端实测。

原 Google Fonts link 原样保留，未联网下载字体。离线渲染预期回退到系统字体；浏览器未成功启动，尚未实际观察此回退。未为离线检查替换或修改字体定义。

发布前仍需在可运行的浏览器中完成上述渲染检查。本报告验证源码复制的一致性，不重新审计私人交易数据库或报告中的统计结论。

## 本地提交限制

尚未创建要求的初始提交。仓库仍在 `main`，没有提交。尝试将六个交付文件按明确列表加入暂存区时，沙箱拒绝写入 `.git/index.lock`：

```text
fatal: Unable to create '/Users/lukewang/code/trading-review-2026/.git/index.lock': Operation not permitted
```

当前环境将 `.git` 设为只读，且不允许权限提升；不能在此环境完成暂存或提交。六个交付文件均已准备在工作区，未创建任何 remote 或执行 push。运营者可在有 Git 写权限的本地终端执行：

```sh
git add -- index.html .nojekyll README.md .gitignore PARITY.md source/ranking-artifact.html
git commit -m "Parity copy of the 2026 trading-review artifact"
```
