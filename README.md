# Claude Code + Remotion 接入指南

> 用 Claude Code 驱动 Remotion 生成产品宣传视频的完整教程。包含从环境准备到导出 MP4 的全流程，以及常见踩坑的 Q&A。

![Remotion](https://img.shields.io/badge/Remotion-4.x-6366f1?style=flat-square)
![Claude Code](https://img.shields.io/badge/Claude_Code-Anthropic-d97757?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-22d3ee?style=flat-square)
![Maintenance](https://img.shields.io/badge/Maintained-yes-22c55e?style=flat-square)

---

## 🎯 这个组合解决什么问题

Remotion 是用 React 代码写视频的框架——JSX 组件直接渲染成 MP4。配合 Claude Code 描述需求自动生成代码，特别适合 **SaaS 产品 demo、Feature 介绍、数据增长展示、批量周报视频**。

| 维度 | Remotion + Claude Code | AI 视频生成（Runway/Kling） |
|---|---|---|
| UI 文字清晰度 | ✅ 像素级精确 | ❌ 容易模糊/漂移 |
| 精确控制 | ✅ 完全可控 | ❌ 不可控 |
| 迭代速度 | ✅ 改代码秒级生效 | ❌ 重新生成 1-3 分钟 |
| 批量生产 | ✅ 模板化输入数据 | ❌ 每次都要重生 |
| 物理质感 | ⚠️ 弱 | ✅ 强 |
| 复杂自然运镜 | ⚠️ 弱 | ✅ 强 |

---

## 📚 目录

- [快速开始](#-快速开始)
- [1. 环境准备](#1-环境准备)
- [2. 创建 Remotion 项目](#2-创建-remotion-项目)
- [3. Claude Code 协作工作流](#3-claude-code-协作工作流)
- [4. Remotion 核心 API 速查](#4-remotion-核心-api-速查)
- [5. 导出 MP4](#5-导出-mp4)
- [6. Q&A 常见问题](#6-qa-常见问题)
- [7. 完整工作流图](#7-完整工作流图)
- [推荐资源](#推荐资源)

---

## 🚀 快速开始

如果你已经熟悉环境，5 步直达：

```bash
# 1. 装 Claude Code（已装跳过）
npm install -g @anthropic-ai/claude-code

# 2. 创建 Remotion 项目
npx create-video@latest my-product-video

# 3. 进入项目并启动
cd my-product-video && npm start

# 4. 在项目目录开 Claude Code
claude

# 5. 告诉它要做什么视频，看 §3.2 指令模板
```

> 第一次跑遇到坑？跳到 [Q&A](#6-qa-常见问题) 找答案。

---

## 1. 环境准备

### 1.1 前置依赖

- **Node.js 18+**：Remotion 最低要求
- **Claude Code**：`npm install -g @anthropic-ai/claude-code`
- **代理工具**（中国大陆 / 香港用户必备）：Clash / Surge / V2Ray 等

### 1.2 验证 Node 版本

```bash
node -v   # 应该 >= v18
npm -v
```

### 1.3 给终端配置代理（境外用户跳过）

Anthropic 服务和 GitHub 在部分地区受限。代理软件默认只代理浏览器，**必须手动给终端配置代理**。

```bash
# 临时（当前窗口有效）
export https_proxy=http://127.0.0.1:7890
export http_proxy=http://127.0.0.1:7890
export all_proxy=socks5://127.0.0.1:7890

# 永久（zsh）
echo 'export https_proxy=http://127.0.0.1:7890' >> ~/.zshrc
echo 'export http_proxy=http://127.0.0.1:7890' >> ~/.zshrc
echo 'export all_proxy=socks5://127.0.0.1:7890' >> ~/.zshrc
source ~/.zshrc
```

> 端口号换成你代理软件的实际端口：Clash 默认 `7890`、V2Ray 默认 `10809`、Surge 默认 `6152`。

**给 Git 也配代理**（Remotion 脚手架内部要 git clone GitHub）：

```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

### 1.4 验证网络

```bash
curl -I https://api.anthropic.com
curl -I https://github.com
```

两个都应该返回 HTTP 状态码（200/401/403 都行，能连上就好）。如果超时或 DNS 失败，回去检查代理。

**特别注意**：看 `cf-ray` 响应头最后 3 位字母，确认 IP 落地国家：

- ✅ `LAX` / `SJC` / `IAD`（美国）、`NRT` / `KIX`（日本）、`SIN`（新加坡）
- ❌ `HKG`（香港）、`PVG` / `PEK`（中国大陆）

香港和中国大陆的 IP 访问 Anthropic 会返回 403。

### 1.5 启动 Claude Code

```bash
claude
```

首次会引导登录。登录成功后会进入交互界面。

---

## 2. 创建 Remotion 项目

### 2.1 方式 A：官方脚手架（网络好时优先）

```bash
npx create-video@latest my-product-video
```

交互提示按这个顺序选：
- **Template**：`Blank`（最干净）
- **Tailwind CSS?**：`Yes`
- **Agent skills?**：`Yes`
- **Package manager**：`npm`

成功后跑：

```bash
cd my-product-video
npm start
```

浏览器自动打开 `http://localhost:3000`，进入 Remotion Studio。

### 2.2 方式 B：手动创建（网络差时推荐）

如果脚手架反复失败（git clone 超时、TLS 中断等），直接让 Claude Code 手写文件。

**给 Claude Code 发的指令模板：**

> 帮我手动创建一个最小可用的 Remotion 项目在 ~/my-product-video，不要用 create-video 脚手架。
>
> 需要的文件：
> 1. `package.json`（依赖：remotion、@remotion/cli、@remotion/bundler、react@18、react-dom@18、typescript、@types/react、@types/react-dom；scripts 包含 start、build）
> 2. `tsconfig.json`（React JSX + ESNext + strict）
> 3. `remotion.config.ts`
> 4. `src/index.ts`（调用 registerRoot）
> 5. `src/Root.tsx`（注册 Composition，1080x1920，30fps，durationInFrames 240）
> 6. `src/Composition.tsx`（深色背景 + 文字淡入占位动画）
> 7. `.gitignore`（忽略 node_modules、out）
> 8. `public/` 空目录
>
> 写完后告诉我跑 npm install。

**关键文件参考内容：**

<details>
<summary><b>📄 package.json</b></summary>

```json
{
  "name": "my-product-video",
  "version": "1.0.0",
  "scripts": {
    "start": "remotion studio",
    "build": "remotion render"
  },
  "dependencies": {
    "@remotion/cli": "^4.0.200",
    "@remotion/bundler": "^4.0.200",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "remotion": "^4.0.200"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "typescript": "^5.3.0"
  }
}
```
</details>

<details>
<summary><b>📄 src/index.ts</b></summary>

```ts
import { registerRoot } from 'remotion';
import { Root } from './Root';
registerRoot(Root);
```
</details>

<details>
<summary><b>📄 src/Root.tsx</b></summary>

```tsx
import { Composition } from 'remotion';
import { MyVideo } from './Composition';

export const Root: React.FC = () => {
  return (
    <Composition
      id="MyVideo"
      component={MyVideo}
      durationInFrames={240}
      fps={30}
      width={1080}
      height={1920}
    />
  );
};
```
</details>

### 2.3 安装依赖

```bash
cd ~/my-product-video
npm install
```

国内慢的话用淘宝镜像：

```bash
npm install --registry=https://registry.npmmirror.com
```

### 2.4 启动 Remotion Studio

```bash
npm start
```

浏览器打开 `http://localhost:3000`，能看到默认动画 = 环境完全跑通 ✅

---

## 3. Claude Code 协作工作流

### 3.1 项目结构组织

让 Claude Code 把大视频拆成多个 scene 组件：

```
src/
├── index.ts
├── Root.tsx              # 注册 Composition
├── Composition.tsx       # 主入口，用 Sequence 串联 scenes
├── scenes/
│   ├── Scene1Intro.tsx
│   ├── Scene2Dashboard.tsx
│   ├── Scene3Highlight.tsx
│   ├── Scene4Growth.tsx
│   └── Scene5Outro.tsx
└── components/
    ├── Particles.tsx
    └── DashboardMock.tsx
public/
├── dashboard.png         # 产品截图
└── logo.svg
```

### 3.2 高质量指令模板

通用结构：**整体规格 + 设计规范 + 逐场景分镜 + 技术要求**。

```
帮我把 src/Composition.tsx 改成一个 [X] 秒的 [产品类型] 社媒宣传视频，[风格描述]。

整体规格：[分辨率]，[帧率]，durationInFrames [总帧数]。

设计规范：
- 背景：[描述]
- 主色：[hex 色值]
- 文字：[字体、字号]
- 动画原则：[spring/interpolate]

分镜（用 Sequence 拆分）：

Scene 1 (frame 0-60, 2秒)：[内容描述]
Scene 2 (frame 60-180, 4秒)：[内容描述]
...

技术要求：
- 拆成独立组件文件：src/scenes/...
- 所有动画用 spring (damping 12-15) 或 interpolate + Easing
- 图片用 staticFile() 包装
```

### 3.3 使用 CLAUDE.md 沉淀规范

在项目根目录建 `CLAUDE.md`，Claude Code 每次对话都会自动读取：

```markdown
# 项目规范

## 视频规格
- 分辨率：1080x1920（竖屏）
- 帧率：30fps
- 时长：15 秒（450 frames）

## 视觉规范
- 主背景：#0a0a14 → #1a1a2e 渐变
- 强调色：#6366f1（紫蓝）+ #22d3ee（霓虹青）
- 文字：白色 + #a1a1aa 灰色
- 字体：-apple-system, "PingFang SC", sans-serif

## 动效规范
- 入场用 spring damping 12-15
- 过渡用 interpolate + Easing.out(Easing.cubic)
- 避免线性动画
- 所有元素入场带轻微缩放（0.8 → 1）
```

### 3.4 迭代技巧

- **要解释再动手**：复杂动效前加一句"先告诉我你打算怎么实现，确认后再写代码"
- **小步迭代**：一次改一个 scene，不要一次改全部
- **善用反馈**：直接说"太快了/太慢了/颜色不对"，Claude Code 改起来很快
- **保留 git 历史**：每个能跑的版本提交一次，方便回滚

---

## 4. Remotion 核心 API 速查

让 Claude Code 用这些 API 输出会更稳：

```tsx
import {
  useCurrentFrame,     // 获取当前帧
  useVideoConfig,      // 获取 fps/width/height
  interpolate,         // 线性插值动画
  spring,              // 弹性动画
  Sequence,            // 时间段控制
  AbsoluteFill,        // 全屏容器
  Img,                 // 图片
  Audio,               // 音频
  staticFile,          // 引用 public/ 资源
  Easing,              // 缓动函数
} from 'remotion';

// 基础用法
const frame = useCurrentFrame();
const opacity = interpolate(frame, [0, 30], [0, 1], { extrapolateRight: 'clamp' });
const scale = spring({ frame, fps: 30, config: { damping: 12 } });

// 时间段
<Sequence from={60} durationInFrames={120}>
  <YourScene />
</Sequence>

// 资源
<Img src={staticFile('dashboard.png')} />
<Audio src={staticFile('bgm.mp3')} />
```

---

## 5. 导出 MP4

```bash
# 默认渲染
npx remotion render

# 指定 Composition 和输出路径
npx remotion render src/index.ts MyVideo out/video.mp4

# 加速渲染
npx remotion render --concurrency=4

# 指定起止帧（导出片段）
npx remotion render --frames=0-90
```

输出后直接上传抖音 / 小红书 / Reels / X。

---

## 6. Q&A 常见问题

<details>
<summary><b>Q1: <code>curl -I https://api.anthropic.com</code> 返回 403，cf-ray 是 HKG？</b></summary>

Anthropic 屏蔽香港、中国大陆等地区的访问。需要切换代理节点到 **美国 / 日本 / 新加坡 / 英国 / 台湾**，避开 HKG / PVG / PEK。切换后再 curl 验证 cf-ray 不再是 HKG。
</details>

<details>
<summary><b>Q2: 提示 "Unable to connect to Anthropic services" / ERR_BAD_REQUEST？</b></summary>

终端没走代理。运行：

```bash
export https_proxy=http://127.0.0.1:7890
export http_proxy=http://127.0.0.1:7890
```

端口换成你代理软件的实际端口。然后重新 `claude`。
</details>

<details>
<summary><b>Q3: <code>npm install</code> 报 EACCES / EEXIST 权限错误？</b></summary>

之前用过 `sudo npm`，缓存目录有 root 文件。修复：

```bash
sudo chown -R $(whoami) ~/.npm
```

输 Mac 开机密码（不显示是正常的）。验证：

```bash
ls -la ~/.npm | head -5
```

所有行开头应该是你的用户名，没有 `root`。

**永远不要再用 `sudo npm install`**，全局包遇到权限问题应该配置 npm 前缀到用户目录而非 sudo。
</details>

<details>
<summary><b>Q4: 在 Claude Code 里跑 <code>! sudo xxx</code> 报 "a terminal is required"？</b></summary>

Claude Code 的 `!` shell 执行环境没有 TTY，sudo 没法读密码。**任何需要密码 / 交互确认的命令，必须在你自己的真实终端里跑**。

经验法则：
- 非交互命令（构建、读文件、写代码）→ Claude Code 的 `!` 跑
- 需要密码 / [Y/n] / 选项菜单的命令 → 你自己的终端跑
</details>

<details>
<summary><b>Q5: <code>create-video</code> 脚手架卡在 "Imagining..." 很久？</b></summary>

实际是在跑 `npm install`，下载依赖很慢。正常需要 1-3 分钟，最多 5 分钟。耐心等。如果超过 5 分钟：

1. 让它继续等
2. 在 Claude Code 里按 `Ctrl+B` 放后台
3. 换淘宝镜像重装：`npm install --registry=https://registry.npmmirror.com`
</details>

<details>
<summary><b>Q6: <code>create-video</code> 报 TLS 中断 / ETIMEDOUT？</b></summary>

脚手架内部要 `git clone` GitHub 拉模板，但 git 没走代理。配置 git 代理：

```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
curl -I https://github.com   # 应该返回 200
```

如果还是不行，**放弃脚手架，改用手动创建（见 §2.2）**。Claude Code 直接写文件不需要 git。
</details>

<details>
<summary><b>Q7: 文件夹建好了但是空的 / 没有 package.json？</b></summary>

脚手架在 git clone 阶段失败，建了空目录就退出了。两种处理：

1. 修好网络重跑（git 代理 + 验证 github 可达）
2. 让 Claude Code 手动建项目（见 §2.2 方式 B）
</details>

<details>
<summary><b>Q8: <code>npm install</code> 报 ENOENT: no such file or directory, open package.json？</b></summary>

当前目录没有 `package.json`。两种可能：

1. 走错目录了 → `cd` 到正确的项目目录
2. 项目根本没建好 → 让 Claude Code 手动建项目文件
</details>

<details>
<summary><b>Q9: Remotion Studio 启动了但浏览器不自动打开？</b></summary>

手动访问 `http://localhost:3000`。如果端口被占用，会显示别的端口（比如 3001），看终端输出的实际地址。
</details>

<details>
<summary><b>Q10: 视频里的中文显示成方框 / 乱码？</b></summary>

Remotion 默认用浏览器字体，中文需要显式指定。

**方法 1：用系统字体**（最简单）
```tsx
style={{ fontFamily: '-apple-system, "PingFang SC", "Microsoft YaHei", sans-serif' }}
```

**方法 2：用 Google Fonts 中文字体**
```bash
npm install @remotion/google-fonts
```

```tsx
import { loadFont } from '@remotion/google-fonts/NotoSansSC';
const { fontFamily } = loadFont();
```
</details>

<details>
<summary><b>Q11: 渲染 MP4 很慢？</b></summary>

```bash
# 增加并发
npx remotion render --concurrency=8

# 降低质量加速测试
npx remotion render --jpeg-quality=70

# 只渲染部分帧测试
npx remotion render --frames=0-60
```

终稿渲染再用默认参数。
</details>

<details>
<summary><b>Q12: AbsoluteFill 不占满屏幕？</b></summary>

检查父容器。`AbsoluteFill` 是 `position: absolute` + 100% 撑满，要求父级 `position: relative` 或本身是 root。Remotion 的 `<Composition>` 已经处理好了，自己嵌套时注意。
</details>

<details>
<summary><b>Q13: 想用真实产品截图，怎么放进项目？</b></summary>

```bash
cp ~/Downloads/dashboard.png ~/my-product-video/public/
```

代码里用：

```tsx
import { Img, staticFile } from 'remotion';

<Img src={staticFile('dashboard.png')} />
```

**不要用 `import image from './dashboard.png'`**，Remotion 推荐用 `staticFile()`。
</details>

<details>
<summary><b>Q14: 怎么加背景音乐？</b></summary>

```bash
cp ~/Music/bgm.mp3 ~/my-product-video/public/
```

```tsx
import { Audio, staticFile } from 'remotion';

<Audio src={staticFile('bgm.mp3')} volume={0.5} />
```

音乐时长不够 = 自动重复；超过视频 = 自动截断。
</details>

<details>
<summary><b>Q15: 改了代码但预览没更新？</b></summary>

1. 浏览器硬刷新：Cmd/Ctrl + Shift + R
2. 检查终端有没有报错（语法错误会卡住编译）
3. 关掉 `npm start` 重启
4. 极端情况清理：`rm -rf node_modules/.cache && npm start`
</details>

<details>
<summary><b>Q16: 怎么导出竖屏 + 横屏两个版本？</b></summary>

在 `Root.tsx` 注册多个 Composition：

```tsx
<Composition id="Vertical" width={1080} height={1920} ... />
<Composition id="Horizontal" width={1920} height={1080} ... />
```

渲染时指定：

```bash
npx remotion render src/index.ts Vertical out/v.mp4
npx remotion render src/index.ts Horizontal out/h.mp4
```
</details>

<details>
<summary><b>Q17: Claude Code 改代码改坏了怎么办？</b></summary>

```bash
git diff              # 看修改了什么
git checkout .        # 撤销最后一次修改
```

建议每完成一个 scene 就 commit 一次，方便回滚。
</details>

<details>
<summary><b>Q18: 想批量生成视频（比如每周数据周报）？</b></summary>

用 Remotion 的 props 参数化：

```tsx
<Composition
  id="WeeklyReport"
  component={WeeklyReport}
  defaultProps={{ revenue: 12847, growth: 23.5, week: 'W42' }}
/>
```

```bash
npx remotion render WeeklyReport out/w42.mp4 --props='{"revenue":15203,"week":"W43"}'
```

写个脚本读数据库 → 调用 render 命令 → 自动生成每周视频。
</details>

---

## 7. 完整工作流图

```
┌─────────────────────────────────────────────┐
│  1. 配代理（境外用户跳过）                    │
│     - 终端 export https_proxy                │
│     - git config --global http.proxy        │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│  2. 创建项目                                  │
│     A. create-video 脚手架(网络好)            │
│     B. Claude Code 手动建(更稳)               │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│  3. npm install + npm start                  │
│     验证 localhost:3000 能看到默认动画         │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│  4. 写 CLAUDE.md 沉淀规范                    │
│     放产品截图到 public/                      │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│  5. 给 Claude Code 完整分镜指令               │
│     → 它创建 scene 组件                      │
│     → 浏览器自动热更新预览                    │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│  6. 迭代调整（颜色/时长/动效）                │
│     每个稳定版本 git commit                   │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│  7. npx remotion render 导出 MP4              │
│     上传社媒                                  │
└─────────────────────────────────────────────┘
```

---

## 推荐资源

- 📘 [Remotion 官方文档](https://www.remotion.dev/docs/)
- 📦 [Remotion 示例库](https://github.com/remotion-dev/remotion/tree/main/packages/example)
- 🤖 [Claude Code 文档](https://docs.claude.com)
- 🎵 免版税音乐：[Pixabay Music](https://pixabay.com/music/)、[Mixkit](https://mixkit.co/)
- 🔤 中文字体：`@remotion/google-fonts/NotoSansSC`

---

## 🤝 贡献

欢迎提交 Issue 或 PR 分享你踩过的新坑和解决方案！

## 📝 License

MIT

---

*Last updated: 2026-05-26*
