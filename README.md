# NEXUS Terminal v4.0

一个功能完备的浏览器端模拟终端，以单文件 HTML 实现，零外部依赖（仅 Google Fonts CDN），具备 CRT 复古视觉效果、虚拟文件系统、管道重定向、Git 版本控制模拟、AI 对话模拟等 30+ 命令。

![NEXUS Terminal](https://img.shields.io/badge/NEXUS-Terminal_v4.0-00ff9f?style=for-the-badge&labelColor=0a0e14)
![HTML](https://img.shields.io/badge/纯HTML-单文件架构-orange?style=flat-square)
![零依赖](https://img.shields.io/badge/零依赖-仅Google_Fonts-blue?style=flat-square)

---

## 快速开始

1. 克隆仓库：
   ```bash
   git clone https://github.com/Quitintmian/nexus-terminal.git
   ```
2. 用浏览器打开 `index.html` 即可使用，无需安装任何依赖或启动服务器。

> 也可以直接将 `index.html` 拖入浏览器窗口，或双击文件打开。

---

## 功能概览

| 分类 | 命令 | 说明 |
|------|------|------|
| **文件操作** | `ls`, `cd`, `pwd`, `cat`, `mkdir`, `touch`, `rm`, `tree` | 目录浏览与文件管理 |
| **文件编辑** | `write`, `append`, `edit`, `head`, `tail`, `grep`, `wc`, `find`, `diff`, `chmod` | 内容读写与搜索 |
| **系统信息** | `uname`, `uptime`, `whoami`, `hostname`, `env`, `ps`, `top`, `neofetch` | 模拟系统状态 |
| **网络** | `ping`, `ifconfig`, `curl`, `ssh` | 模拟网络操作 |
| **AI 对话** | `ai <question>`, `ai -c` | 模式匹配 AI 对话系统 |
| **Git** | `git init/add/commit/status/log/diff/branch/checkout` | 完整 Git 工作流模拟 |
| **管道重定向** | `cmd1 \| cmd2`, `cmd > file`, `cmd >> file` | 管道与输出重定向 |
| **工具** | `echo`, `date`, `calc`, `cowsay`, `fortune`, `color`, `man`, `alias` | 实用工具集 |
| **终端控制** | `clear`, `save`, `load`, `reset`, `reboot`, `exit`, `history` | 终端会话管理 |

---

## 架构设计

项目采用**四层分层架构**，所有代码集中在单个 HTML 文件中：

```
┌─────────────────────────────────────────┐
│         表现层 (Presentation)            │  CSS3 CRT 效果 / Matrix 背景 / 主题色系统
├─────────────────────────────────────────┤
│         交互层 (Interaction)             │  键盘事件处理 / Tab 补全 / 行编辑器
├─────────────────────────────────────────┤
│         逻辑层 (Logic)                   │  命令分发 / 管道重定向 / Git 模拟 / AI 对话
├─────────────────────────────────────────┤
│         数据层 (Data)                    │  虚拟文件系统 / localStorage 持久化 / 环境变量
└─────────────────────────────────────────┘
```

---

## 模块详解

### 1. CRT 视觉效果系统

通过纯 CSS3 实现复古 CRT 显示器效果，营造沉浸式终端体验。

**实现原理：**

- **扫描线**：使用 `::before` 伪元素 + `repeating-linear-gradient` 生成 2px 间隔的半透明横线，覆盖在整个终端区域上方
- **屏幕闪烁**：`@keyframes flicker` 动画，在 0.97~1.0 之间随机微调 `opacity`，模拟老式显示器的微弱闪烁
- **磷光发光**：`text-shadow` 多层叠加（0 0 5px / 0 0 10px / 0 0 20px），模拟绿色磷光字符的辉光扩散效果
- **屏幕曲面**：`::after` 伪元素 + `radial-gradient` 模拟 CRT 边缘暗角（vignette effect）
- **Glitch 效果**：reboot 时通过 `clip-path` 随机裁切 + 位移动画模拟信号干扰

```css
/* 扫描线核心实现 */
.terminal-body::before {
  content: '';
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background: repeating-linear-gradient(
    0deg,
    rgba(0, 0, 0, 0.15) 0px,
    rgba(0, 0, 0, 0.15) 1px,
    transparent 1px,
    transparent 2px
  );
  pointer-events: none;
  z-index: 10;
}

/* 磷光发光效果 */
.output-line {
  text-shadow: 0 0 5px rgba(0, 255, 159, 0.3),
               0 0 10px rgba(0, 255, 159, 0.1);
}
```

**主题色系统**：使用 CSS Custom Properties 实现 5 种主题色方案（green/cyan/amber/red/purple），通过 `color` 命令实时切换，所有视觉元素自动响应：

```css
:root {
  --green: #00ff9f;
  --green-dim: #00aa6a;
  --cyan: #00e5ff;
  --amber: #ffb300;
  --red: #ff3d3d;
  --purple: #c77dff;
  --bg: #0a0e14;
}
```

---

### 2. Matrix 数字雨背景

使用 Canvas 2D API 实现经典的 Matrix 数字雨效果，作为终端的动态背景层。

**实现原理：**

- 创建全屏 Canvas 元素，`position: fixed` 置于终端底层（z-index: 0）
- 每列维护独立的下落位置（`drops` 数组），以字符宽度（14px）为列间距
- 每帧用半透明黑色矩形覆盖（`rgba(0,0,0,0.05)`），产生字符渐隐拖尾效果
- 随机选取日文片假名（ア-ン）+ 数字 + 拉丁字符作为雨滴字符
- `requestAnimationFrame` 驱动动画循环，确保 60fps 流畅渲染
- 当列的雨滴超出画布高度时，以 2.5% 的概率重置到顶部，产生随机断续效果

```javascript
function initMatrixBg() {
  const canvas = document.getElementById('matrix-bg');
  const ctx = canvas.getContext('2d');
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;

  const chars = 'アイウエオカキクケコ0123456789ABCDEF';
  const fontSize = 14;
  const columns = Math.floor(canvas.width / fontSize);
  const drops = Array(columns).fill(1);

  function draw() {
    ctx.fillStyle = 'rgba(10, 14, 20, 0.05)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = 'rgba(0, 255, 159, 0.12)'; // 主题色半透明
    ctx.font = fontSize + 'px monospace';
    for (let i = 0; i < drops.length; i++) {
      const char = chars[Math.floor(Math.random() * chars.length)];
      ctx.fillText(char, i * fontSize, drops[i] * fontSize);
      if (drops[i] * fontSize > canvas.height && Math.random() > 0.975)
        drops[i] = 0;
      drops[i]++;
    }
    requestAnimationFrame(draw);
  }
  draw();
}
```

---

### 3. 虚拟文件系统 (VFS)

基于嵌套对象模型实现的内存文件系统，支持目录树、文件读写、路径解析。

**数据结构：**

```javascript
const defaultFileSystem = {
  '~': {
    type: 'dir',
    children: {
      'documents': {
        type: 'dir',
        children: {
          'readme.txt': { type: 'file', content: '欢迎使用 NEXUS Terminal!' },
          'notes.txt':  { type: 'file', content: '这是一个笔记文件' }
        }
      },
      'projects': {
        type: 'dir',
        children: {
          'hello.sh': { type: 'file', content: '#!/bin/bash\necho "Hello, NEXUS!"' }
        }
      },
      '.bashrc': { type: 'file', content: '# NEXUS Terminal Config\nexport PS1="nexus@terminal:~$ "' },
      '.gitconfig': { type: 'file', content: '[user]\n  name = nexus\n  email = nexus@terminal' }
    }
  }
};
```

**路径解析算法：**

采用**栈式路径规范化**算法处理路径导航，支持绝对路径、相对路径和 `..` 回退：

1. 将路径按 `/` 分割为段
2. 遍历每个段：`.` 忽略，`..` 弹栈（回退到上级），其他压栈
3. 将栈中剩余段拼接为规范化路径
4. 通过路径段逐级查找目标目录节点

```javascript
function resolvePath(path) {
  // 处理相对路径：基于 currentDir 拼接
  if (!path.startsWith('~')) {
    path = currentDir + '/' + path;
  }
  const parts = path.split('/').filter(p => p && p !== '.');
  const stack = [];
  for (const part of parts) {
    if (part === '..') stack.pop();
    else stack.push(part);
  }
  return stack.length ? '~/' + stack.join('/') : '~';
}

// 通过路径段逐级查找目录节点
function resolveDir(path) {
  const parts = path.split('/').filter(p => p);
  let current = fileSystem['~'];
  for (const part of parts) {
    if (part === '~') continue;
    if (!current || !current.children || !current.children[part]) return null;
    current = current.children[part];
  }
  return current;
}
```

**支持的路径格式**：绝对路径（`~/documents`）、相对路径（`./file`、`../dir`）、路径简写（`~`）

---

### 4. 命令处理系统

**命令分发流程：**

```
用户输入 → handleKeyDown(Enter)
             ↓
         processCommand(input)
             ↓
         1. 别名替换（alias 展开）
         2. 管道检测（splitPipes 分割）
         3. 重定向检测（> / >> 解析）
         4. processCommandDirect(cmd, args)
             ↓
         switch(cmd) 分发到具体命令函数
```

**异步命令支持**：`processCommand` 为 `async` 函数，支持 `await` 等待异步命令完成（如 `ping`、`top`、`ai`、`reboot`），确保命令按序执行，避免并发输入混乱：

```javascript
async function processCommand(input) {
  // 别名替换
  const firstWord = input.split(/\s+/)[0];
  if (aliases[firstWord]) {
    input = aliases[firstWord] + input.substring(firstWord.length);
  }
  // 管道检测 → 重定向检测 → switch 分发
  await processCommandDirect(input, '');
}
```

**别名系统**：用户可通过 `alias ll="ls -la"` 创建别名，执行时自动替换首词：

```javascript
if (aliases[firstWord]) {
  input = aliases[firstWord] + input.substring(firstWord.length);
}
```

**输入行管理**：每次命令执行后，当前输入行被"冻结"为输出行（显示已执行的命令），然后创建新的输入行。通过 `requestAnimationFrame` 确保 DOM 布局完成后再聚焦输入框：

```javascript
function createInputLine() {
  const oldInput = document.getElementById('command-input');
  if (oldInput) oldInput.removeAttribute('id'); // 防止 id 重复
  // ... 创建新的 input-line ...
  requestAnimationFrame(() => {
    input.focus();
    scrollToBottom();
  });
}
```

---

### 5. 管道与重定向

**管道机制**：使用 `captureBuffer` 输出捕获机制实现命令间数据传递。

核心思路：当 `captureBuffer` 非 `null` 时，`addOutput()` 不渲染到 DOM，而是将文本推入缓冲区。管道链中每个命令的输出成为下一个命令的 `stdin`。

```javascript
// 执行命令并捕获其输出（不渲染到屏幕）
async function executeCaptured(cmdStr, stdin = '') {
  captureBuffer = [];  // 开启捕获模式
  if (stdin) captureBuffer.push(stdin);
  await processCommandDirect(cmdStr, stdin);
  const output = captureBuffer.join('\n');
  captureBuffer = null;  // 关闭捕获模式
  return output;
}

// addOutput 在捕获模式下的行为
function addOutput(text, type = 'result') {
  if (captureBuffer !== null) {
    captureBuffer.push(text);  // 推入缓冲区而非渲染
    return;
  }
  // ... 正常渲染到 DOM ...
}
```

**管道链处理**：将输入按 `|` 分割（考虑引号内的管道符号），依次执行每个命令段：

```javascript
const pipeSegments = splitPipes(input);
if (pipeSegments.length > 1) {
  let pipeData = '';
  for (const seg of pipeSegments) {
    pipeData = await executeCaptured(seg.trim(), pipeData);
  }
  addOutput(pipeData, 'result');  // 最终结果渲染到屏幕
}
```

**重定向**：解析 `>` 和 `>>` 操作符，将命令输出写入虚拟文件：

```javascript
if (redirectFile) {
  const output = await executeCaptured(commandPart);
  const { dir, fileName } = getParentDirAndName(redirectFile);
  if (appendMode && dir.children[fileName]) {
    dir.children[fileName].content += '\n' + output;  // >> 追加
  } else {
    dir.children[fileName] = { type: 'file', content: output };  // > 覆盖
  }
}
```

**使用示例**：
```bash
echo "Hello World" > greeting.txt     # 输出写入文件（覆盖）
echo "New line" >> greeting.txt       # 输出追加到文件
cat readme.txt | grep "NEXUS"         # 管道搜索
ls | wc                              # 统计目录项数
cat notes.txt | head -3 | tail -1    # 管道链
```

---

### 6. 行编辑器 (edit 命令)

模拟 vi 风格的行编辑器，支持逐行编辑文件内容。

**工作模式**：

1. `edit <file>` 进入编辑模式，加载文件内容到 `editLines` 数组
2. 显示当前行号和内容，用户可输入新内容替换当前行
3. 特殊命令：
   - `:wq` — 保存修改并退出，将 `editLines` 写回文件系统
   - `:q` — 放弃修改退出
   - `:d` — 删除当前行
   - `:i` — 在当前行前插入空行
   - `Enter` — 确认当前行编辑，移至下一行
   - `↑` / `↓` — 上下导航浏览行

**状态管理**：编辑模式下使用独立的键盘处理器 `handleEditKeyDown`，终端底部状态栏显示 `EDIT` 标识：

```javascript
function enterEditMode(fileName) {
  const dir = resolveDir(currentDir);
  const file = dir.children[fileName];
  if (!file || file.type !== 'file') {
    addOutput(`edit: ${fileName} 不是文件`, 'error');
    return;
  }
  editMode = true;
  editFile = fileName;
  editLines = file.content.split('\n');
  editLineNum = 0;
  editIndicator.classList.add('active');
  showEditLine();
}

function handleEditKeyDown(e) {
  if (e.key === 'Enter') {
    const val = inputEl.value;
    if (val.trim() === ':wq') { /* 保存退出 */ }
    else if (val.trim() === ':q') { /* 放弃退出 */ }
    else if (val.trim() === ':d') { /* 删除当前行 */ }
    else if (val.trim() === ':i') { /* 插入空行 */ }
    else {
      editLines[editLineNum] = val;  // 替换当前行
      editLineNum++;
    }
    showEditLine();
  }
}
```

---

### 7. Git 版本控制模拟

完整模拟 Git 工作流，包括仓库初始化、暂存、提交、分支管理等核心操作。

**数据模型**：

```javascript
let gitState = {
  initialized: false,   // 是否已初始化仓库
  branch: 'main',       // 当前分支名
  commits: [],          // 提交历史 [{ hash, message, branch, files, timestamp }]
  staged: [],           // 暂存区文件列表
  log: []               // 操作日志
};
```

**核心流程**：

- **`git init`**：设置 `initialized: true`，创建初始提交
- **`git add <file>`**：将文件加入 `staged` 数组，记录文件内容快照
- **`git commit -m "msg"`**：生成 7 位随机 hash，将暂存文件快照存入 `commits`，清空暂存区
- **`git branch <name>`**：创建新分支（基于当前 commits 的快照）
- **`git checkout <branch>`**：切换分支，恢复对应分支的文件系统快照
- **`git diff [file]`**：对比暂存区与工作目录的差异
- **`git status`**：显示当前分支、暂存文件、未跟踪文件

**提交 Hash 生成**：使用时间戳 + 随机数生成模拟 SHA-1 短 hash：

```javascript
const hash = Date.now().toString(16).slice(-7) + Math.random().toString(16).slice(2, 5);
```

**使用示例**：
```bash
git init                              # 初始化仓库
touch main.py                         # 创建文件
git add main.py                       # 暂存文件
git commit -m "Initial commit"        # 提交
git branch feature                    # 创建分支
git checkout feature                  # 切换分支
git status                            # 查看状态
git log                               # 查看提交历史
```

---

### 8. AI 对话模拟

基于模式匹配的对话系统，支持上下文感知的多轮对话。

**实现原理**：

1. 维护 `aiConversationHistory` 数组存储对话历史
2. 对用户输入进行关键词匹配，按优先级匹配预设回复
3. 支持的对话领域：问候、自我介绍、编程、终端操作、哲学、笑话等
4. `ai -c` 清除对话历史，重新开始

```javascript
function generateAIResponse(question) {
  const q = question.toLowerCase();
  // 关键词匹配 + 优先级排序
  if (q.includes('你好') || q.includes('hello'))
    return '你好！我是 NEXUS AI 助手，有什么可以帮你的吗？';
  if (q.includes('你是谁') || q.includes('who are you'))
    return '我是 NEXUS Terminal 内置的 AI 助手...';
  if (q.includes('编程') || q.includes('code'))
    return '编程是一门艺术...';
  // ... 更多模式匹配 ...
  return '这是个有趣的问题...';  // 默认回复
}
```

---

### 9. Tab 自动补全

双层补全机制：命令名补全 + 文件路径补全。

**触发逻辑**：

1. 解析输入，提取光标前的命令名或路径参数
2. 如果是第一个词（命令位置），匹配 `cmdDescriptions` 对象中的命令名
3. 如果是后续参数，匹配当前目录下的文件/目录名
4. 唯一匹配时自动补全，多个匹配时显示候选列表

```javascript
function handleTabComplete() {
  const parts = inputEl.value.split(' ');
  if (parts.length <= 1) {
    // 命令名补全
    const partial = parts[0].toLowerCase();
    const matches = Object.keys(cmdDescriptions).filter(c => c.startsWith(partial));
    if (matches.length === 1) {
      inputEl.value = matches[0] + ' ';  // 唯一匹配，自动补全
    } else if (matches.length > 1) {
      addOutput(matches.join('  '), 'info');  // 多个匹配，显示候选
    }
  } else {
    // 文件路径补全
    const lastPart = parts[parts.length - 1];
    const dir = resolveDir(currentDir);
    const matches = Object.keys(dir.children).filter(n => n.startsWith(lastPart));
    if (matches.length === 1) {
      parts[parts.length - 1] = matches[0];
      inputEl.value = parts.join(' ');
    } else if (matches.length > 1) {
      addOutput(matches.join('  '), 'info');
    }
  }
}
```

---

### 10. localStorage 持久化

通过 `localStorage` 实现终端状态的自动保存与恢复，关闭浏览器后数据不丢失。

**保存内容**：

```javascript
function saveState() {
  const state = {
    fileSystem,        // 虚拟文件系统（完整目录树）
    commandHistory,    // 命令历史记录
    env,               // 环境变量
    aliases,           // 命令别名
    gitState,          // Git 状态（分支、提交、暂存区）
    currentDir         // 当前工作目录
  };
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
}
```

**恢复机制**：

```javascript
function loadState() {
  const data = localStorage.getItem(STORAGE_KEY);
  if (data) {
    const state = JSON.parse(data);
    if (state.fileSystem) fileSystem = state.fileSystem;
    if (state.commandHistory) commandHistory = state.commandHistory;
    if (state.env) env = { ...env, ...state.env };
    if (state.aliases) aliases = state.aliases;
    if (state.gitState) gitState = { ...gitState, ...state.gitState };
    if (state.currentDir) currentDir = state.currentDir;
    historyIndex = commandHistory.length;
    return true;
  }
  return false;
}
```

**相关命令**：
- `save` — 手动保存当前状态到 localStorage
- `load` — 手动从 localStorage 恢复状态
- `reset --confirm` — 清除所有本地数据并重置终端到初始状态（需二次确认）

**自动保存时机**：每次命令执行完成后自动调用 `saveState()`。

---

### 11. 启动动画 (Boot Sequence)

模拟真实系统的启动过程，增强沉浸感。

**流程**：

1. 显示 NEXUS Logo（ASCII Art）
2. 逐行输出启动信息（BIOS 自检、内核加载、安全模块、服务启动等）
3. 进度条动画同步推进（CSS transition 驱动宽度变化）
4. 启动信息淡出（opacity 0.5s 过渡）
5. 显示欢迎 ASCII Art 和使用提示
6. 创建命令输入行，终端就绪

```javascript
async function boot() {
  // 尝试从 localStorage 恢复状态
  let restored = false;
  try { restored = loadState(); } catch (e) {}

  // 播放启动动画
  for (let i = 0; i < bootSequence.length; i++) {
    const step = bootSequence[i];
    const line = document.createElement('div');
    line.textContent = step.text;
    bootText.appendChild(line);
    await sleep(50);
    line.classList.add('visible');
    bootProgress.style.width = `${((i + 1) / bootSequence.length) * 100}%`;
    if (step.delay) await sleep(step.delay);
  }

  // 淡出启动画面
  bootScreen.style.transition = 'opacity 0.5s ease';
  bootScreen.style.opacity = '0';
  await sleep(500);
  bootScreen.remove();

  // 显示欢迎信息
  showWelcome();
  createInputLine();
}
```

---

## 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Enter` | 执行命令 |
| `Tab` | 自动补全（命令名 / 文件路径） |
| `↑` / `↓` | 浏览命令历史 |
| `Ctrl+C` | 中断当前输入 |
| `Ctrl+L` | 清屏 |

---

## 技术栈

- **HTML5** — 单文件架构，所有代码内联
- **CSS3** — CRT 视觉效果、动画、Custom Properties 主题系统
- **JavaScript (ES2020+)** — async/await、模板字符串、解构赋值
- **Canvas 2D API** — Matrix 数字雨背景
- **localStorage API** — 状态持久化
- **Google Fonts** — Fira Code 等宽字体（唯一外部依赖）

---

## 项目结构

```
nexus-terminal/
├── index.html          # 完整项目（HTML + CSS + JS 单文件）
└── README.md           # 项目文档
```

> 项目采用单文件架构，所有 HTML 结构、CSS 样式、JavaScript 逻辑均内联在 `index.html` 中，无需构建工具，浏览器直接打开即可运行。

