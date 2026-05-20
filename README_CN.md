# oh-my-playwright

**[English](README.md)**

一个 [opencode](https://github.com/anomalyco/opencode) 技能插件，用于浏览器自动化和网站探索，支持**自动检测登录墙、引导认证、零配置首次安装**。

## 功能

- **探索网站**：自动识别核心功能并生成交互记录
- **自动检测登录墙**：识别 7+ 种认证类型（密码表单、OAuth、SSO、MFA、二维码、验证码、短信验证）
- **引导认证**：在可见浏览器中引导用户手动登录，或连接用户已登录的真实浏览器
- **一次性配置**：只问 3 个问题，生成最终配置，重启一次即可
- **生成测试用例**：基于探索结果输出结构化测试用例

## 工作原理

### 首次安装（Phase 0）

首次使用时，技能会检测你当前的 Playwright MCP 配置，通过 3 个问题完成设置：

1. **连接方式** — Extension 模式（推荐）、Persistent Profile 模式、CDP 模式
2. **浏览器** — Edge（推荐）、Chrome、内置 Chromium
3. **无头模式** — 可见（推荐）或不可见

然后根据你的选择生成正确的配置，检查前置条件（扩展装了没、调试开了没），一次性写入。你只需重启 opencode 一次，以后不用再改配置。

### 登录检测（Phase 1）

导航到网站时，自动检测当前页面是否为登录墙：
- URL 模式匹配（`login`、`signin`、`auth`、`sso`、`oauth` 等）
- 密码输入框、OAuth 按钮、验证码组件
- 二维码图片、短信验证提示
- 页面标题和内容分析

### 认证处理（Phase 2）

检测到登录墙后：
1. 告知用户检测到了什么类型的认证
2. 用户在可见浏览器中手动登录（支持所有方式：密码、二维码、MFA、SSO、验证码）
3. 技能验证登录成功后继续操作
4. 可选保存会话状态以备后续使用

如果登录失败（如网站阻止自动化浏览器），可以切换到连接用户真实浏览器的模式。

### 探索与测试生成（Phase 3-4）

- 识别 3-5 个核心功能
- 逐个交互并记录 UI 元素
- 生成结构化测试用例

## 环境要求

- [opencode](https://github.com/anomalyco/opencode) CLI
- Node.js 18+
- Playwright MCP（`npx @playwright/mcp@latest`）
- 以下任一：
  - **Extension 模式**：Playwright MCP Bridge 浏览器扩展
  - **Persistent Profile 模式**：无额外依赖
  - **CDP 模式**：浏览器中开启远程调试

## 安装

### 1. 安装技能

```bash
# 克隆到 opencode 技能目录
git clone https://github.com/<your-username>/oh-my-playwright ~/.opencode/skills/oh-my-playwright
```

或手动安装：
1. 创建目录 `~/.opencode/skills/oh-my-playwright/`
2. 将 `SKILL.md` 复制进去

### 2. 安装 Playwright MCP Bridge 扩展（Extension 模式需要）

如果使用 Extension 模式（推荐），需要安装浏览器扩展：

**扩展地址**：[Playwright MCP Bridge - Chrome Web Store](https://chromewebstore.google.com/detail/playwright-mcp-bridge/mmlmfjhmonkocbjadbfplnigmagldckm)

**安装步骤：**
1. 在浏览器（Edge 或 Chrome）中打开上面的链接
2. 点击"添加到 Chrome"或"获取"
3. 确认安装
4. 浏览器工具栏中应出现扩展图标

**Edge 用户注意**：Edge 支持直接安装 Chrome Web Store 的扩展，在 Edge 中打开链接正常安装即可。

### 3. （可选）开启远程调试（仅 CDP 模式需要）

如果使用 CDP 模式：
- **Edge**：在地址栏输入 `edge://inspect/#remote-debugging`，开启"Allow remote debugging for this browser instance"
- **Chrome**：在地址栏输入 `chrome://inspect/#remote-debugging`，开启"Allow remote debugging for this browser instance"

## 配置

首次使用时技能会自动引导配置。如需手动配置 Playwright MCP，编辑 `~/.config/opencode/opencode.json`：

**Extension 模式（推荐）：**
```json
"playwright": {
    "type": "local",
    "command": ["npx", "@playwright/mcp@latest", "--extension", "--browser", "msedge"],
    "enabled": true
}
```

**Persistent Profile 模式：**
```json
"playwright": {
    "type": "local",
    "command": ["npx", "@playwright/mcp@latest", "--channel", "msedge", "--user-data-dir", "C:\\Users\\<you>\\.playwright-profile"],
    "enabled": true
}
```

**CDP 模式：**
```json
"playwright": {
    "type": "local",
    "command": ["npx", "@playwright/mcp@latest", "--cdp-endpoint=msedge"],
    "enabled": true
}
```

## 致谢

本技能最初基于 opencode 内置的 `playwright-explore-website` 模板技能。原始模板仅包含 17 行指令文本，本技能已完全重写（300+ 行），新增：

- 完整的登录检测和认证处理（Phase 1-2）
- 一次性配置向导与配置生成（Phase 0）
- 多浏览器支持（Edge、Chrome、Chromium）
- 防挂机弹窗检测
- 中文检测模式（登录、验证）

## 许可证

MIT
