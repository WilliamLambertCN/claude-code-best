# 替换本地 Claude Code 为 CCB 完整指南

本文档详细说明如何将本地环境的官方 Claude Code 替换为开源版本 CCB (Claude Code Best)，包括 CLI 工具安装、VS Code 插件配置等内容。

## 目录

- [前置检查](#前置检查)
- [场景一：本地未安装官方 Claude Code](#场景一本地未安装官方-claude-code)
- [场景二：本地已安装官方 Claude Code](#场景二本地已安装官方-claude-code)
- [VS Code 插件配置](#vs-code-插件配置)
- [验证与测试](#验证与测试)
- [常见问题](#常见问题)

---

## 前置检查

### 1. 检查当前环境

```bash
# 检查是否已安装官方 Claude Code
which claude
npm list -g @anthropic-ai/claude-code

# 检查是否已安装 CCB
which ccb
bun pm ls -g claude-code-best

# 检查 Bun 版本（需要 >= 1.3.11）
bun --version
```

### 2. 系统要求

- **Bun** >= 1.3.11（必须，用于运行 CCB）
- **Node.js** >= 18（可选，用于运行官方 Claude Code）
- **操作系统**：Linux / macOS / Windows (WSL)

### 3. 网络环境（国内用户）

如果在国内，需要先配置 GitHub 代理：

```bash
export DEFAULT_RELEASE_BASE=https://ghproxy.net/https://github.com/microsoft/ripgrep/prebuilt/releases/download/v15.0.1
```

---

## 场景一：本地未安装官方 Claude Code

### 方案 A：直接安装 CCB（推荐）

#### 1. 全局安装

```bash
# 从 NPM 安装
bun install -g claude-code-best

# 信任包（避免每次运行都提示）
bun pm -g trust claude-code-best
```

#### 2. 验证安装

```bash
# 检查命令是否可用
which ccb
# 输出：/home/developer/.bun/bin/ccb

# 检查版本
ccb --version

# 测试运行
ccb
```

#### 3. 创建 `claude` 命令别名（可选）

如果习惯使用 `claude` 命令，可以创建别名：

**方式 1：Shell 别名（临时）**
```bash
# 在 ~/.bashrc 或 ~/.zshrc 中添加
alias claude='ccb'

# 生效配置
source ~/.bashrc  # 或 source ~/.zshrc
```

**方式 2：符号链接（永久）**
```bash
# 创建 claude -> ccb 的符号链接
ln -s /home/developer/.bun/bin/ccb /home/developer/.bun/bin/claude

# 确保 Bun 路径在 PATH 中
export PATH="/home/developer/.bun/bin:$PATH"
```

---

### 方案 B：从源码安装

#### 1. 克隆仓库

```bash
git clone https://github.com/claude-code-best/claude-code.git
cd claude-code
```

#### 2. 安装依赖

```bash
bun install
```

#### 3. 构建生产版本

```bash
bun run build
```

构建产物位于 `dist/cli.js`，约 450+ chunk 文件。

#### 4. 全局链接

```bash
# 方式 1：使用 Bun 链接
bun link

# 方式 2：手动创建符号链接
sudo ln -s $(pwd)/dist/cli.js /usr/local/bin/ccb

# 方式 3：添加到 PATH
echo "export PATH=\"$(pwd)/dist:\$PATH\"" >> ~/.bashrc
```

#### 5. 开发模式（可选）

如果需要调试或修改源码：

```bash
# 开发模式运行（启用 BUDDY、TRANSCRIPT_CLASSIFIER 等功能）
bun run dev

# 带 debugger 运行
bun run dev:inspect
# 然后在 Chrome DevTools 中连接 localhost:9229
```

---

## 场景二：本地已安装官方 Claude Code

### 现状分析

假设你的环境如下：

```bash
# 官方版本
which claude
# /home/developer/.nvm/versions/node/v20.20.1/bin/claude

npm list -g @anthropic-ai/claude-code
# @anthropic-ai/claude-code@2.1.92

# CCB 版本（已安装）
which ccb
# /home/developer/.bun/bin/ccb
```

### 方案 A：保留两者，通过配置切换（推荐）

#### 1. 安装 CCB（如果未安装）

```bash
bun install -g claude-code-best
bun pm -g trust claude-code-best
```

#### 2. 验证两个命令都可用

```bash
# 官方版本
claude --version
# 2.1.92

# 开源版本
ccb --version
# 1.1.0
```

#### 3. 配置 VS Code 插件使用 CCB

见 [VS Code 插件配置](#vs-code-插件配置) 章节。

---

### 方案 B：完全替换官方版本

#### 1. 备份官方版本

```bash
# 备份官方命令
sudo mv /home/developer/.nvm/versions/node/v20.20.1/bin/claude \
        /home/developer/.nvm/versions/node/v20.20.1/bin/claude-official

# 或者保留软链接信息
ls -la /home/developer/.nvm/versions/node/v20.20.1/bin/claude
```

#### 2. 创建符号链接指向 CCB

```bash
# 方式 1：系统级链接（需要 sudo）
sudo ln -s /home/developer/.bun/bin/ccb \
           /home/developer/.nvm/versions/node/v20.20.1/bin/claude

# 方式 2：用户级链接（推荐）
ln -s /home/developer/.bun/bin/ccb /home/developer/.bun/bin/claude
export PATH="/home/developer/.bun/bin:$PATH"
```

#### 3. 验证替换

```bash
which claude
# /home/developer/.bun/bin/claude -> /home/developer/.bun/bin/ccb

claude --version
# 1.1.0 (CCB)
```

#### 4. 卸载官方版本（可选）

```bash
# 如果确定不再需要官方版本
npm uninstall -g @anthropic-ai/claude-code
```

---

### 方案 C：通过 Shell 别名动态切换

#### 1. 编辑 Shell 配置

```bash
# 编辑 ~/.bashrc 或 ~/.zshrc
nano ~/.bashrc
```

#### 2. 添加别名函数

```bash
# 默认使用 CCB
alias claude='ccb'

# 保留官方版本访问路径
alias claude-official='/home/developer/.nvm/versions/node/v20.20.1/bin/claude'

# 或者创建切换函数
use-official-claude() {
    unalias claude
    alias claude='/home/developer/.nvm/versions/node/v20.20.1/bin/claude'
    echo "已切换到官方 Claude Code"
}

use-ccb() {
    unalias claude
    alias claude='ccb'
    echo "已切换到 CCB"
}
```

#### 3. 生效配置

```bash
source ~/.bashrc
```

---

## VS Code 插件配置

### 插件概览

VS Code 中可能存在的 Claude 相关插件：

| 插件 ID | 名称 | 当前版本 | 是否支持自定义 CLI |
|---------|------|---------|------------------|
| `anthropic.claude-code` | Claude Code for VS Code | 2.1.92 | ✅ 支持（通过 `claudeProcessWrapper`） |
| `andrepimenta.claude-code-chat` | Chat for Claude Code | 1.1.0 | ❌ 仅 WSL 环境支持 |
| `saoudrizwan.claude-dev` | Claude Dev | - | 查看插件文档 |

### 1. 官方插件配置 (`anthropic.claude-code`)

#### 方法 A：通过 VS Code UI 配置

1. 打开 VS Code 设置：`Ctrl + ,` (Windows/Linux) 或 `Cmd + ,` (macOS)
2. 搜索 "Claude Code"
3. 找到 **"Claude Process Wrapper"** 配置项
4. 填入以下值之一：
   - `ccb`（如果 ccb 在 PATH 中）
   - `/home/developer/.bun/bin/ccb`（完整路径）
   - `/usr/local/bin/ccb`（如果全局安装）

#### 方法 B：直接编辑 settings.json

```bash
# 打开 VS Code 配置文件
code ~/.config/Code/User/settings.json
```

添加或修改配置：

```json
{
  "claudeCode.claudeProcessWrapper": "ccb"
}
```

或者使用完整路径：

```json
{
  "claudeCode.claudeProcessWrapper": "/home/developer/.bun/bin/ccb"
}
```

#### 方法 C：工作区级别配置

在项目根目录创建 `.vscode/settings.json`：

```json
{
  "claudeCode.claudeProcessWrapper": "ccb"
}
```

这样不同项目可以使用不同的 CLI 版本。

#### 验证配置生效

```bash
# 打开 VS Code 输出面板
# View -> Output -> 选择 "Claude Code"

# 查看启动日志，应该看到类似：
# [INFO] Using custom Claude process wrapper: ccb
```

---

### 2. Claude Code Chat 插件配置 (`andrepimenta.claude-code-chat`)

**限制**：该插件在非 WSL 环境下硬编码调用 `claude` 命令，不支持自定义 CLI 路径。

#### 方案 A：使用符号链接（推荐）

```bash
# 1. 备份官方 claude 命令（如果存在）
if [ -f /home/developer/.nvm/versions/node/v20.20.1/bin/claude ]; then
    mv /home/developer/.nvm/versions/node/v20.20.1/bin/claude \
       /home/developer/.nvm/versions/node/v20.20.1/bin/claude-official
fi

# 2. 创建符号链接
ln -s /home/developer/.bun/bin/ccb /home/developer/.bun/bin/claude

# 3. 确保 Bun 路径优先级最高
export PATH="/home/developer/.bun/bin:$PATH"

# 4. 添加到 Shell 配置文件
echo 'export PATH="/home/developer/.bun/bin:$PATH"' >> ~/.bashrc
```

#### 方案 B：创建 wrapper 脚本

```bash
# 创建 claude wrapper 脚本
cat > /home/developer/.bun/bin/claude << 'EOF'
#!/bin/bash
# Claude Code wrapper - redirects to CCB
exec /home/developer/.bun/bin/ccb "$@"
EOF

# 赋予执行权限
chmod +x /home/developer/.bun/bin/claude

# 确保 PATH 优先级
export PATH="/home/developer/.bun/bin:$PATH"
```

#### 方案 C：修改 PATH 优先级

```bash
# 在 ~/.bashrc 或 ~/.zshrc 中添加
export PATH="/home/developer/.bun/bin:$PATH"

# 重新加载配置
source ~/.bashrc

# 验证 which claude 指向 ccb 的符号链接
which claude
# /home/developer/.bun/bin/claude

ls -la /home/developer/.bun/bin/claude
# lrwxrwxrwx 1 developer developer 28 ... claude -> /home/developer/.bun/bin/ccb
```

#### WSL 环境配置（仅限 WSL 用户）

如果在 WSL 环境下使用：

```json
// VS Code settings.json
{
  "claudeCodeChat.wsl.enabled": true,
  "claudeCodeChat.wsl.distro": "Ubuntu",
  "claudeCodeChat.wsl.nodePath": "/usr/bin/node",
  "claudeCodeChat.wsl.claudePath": "/home/developer/.bun/bin/ccb"
}
```

---

### 3. 同时配置两个插件

如果同时安装了两个插件，推荐配置：

```json
// ~/.config/Code/User/settings.json
{
  // 官方插件使用 CCB
  "claudeCode.claudeProcessWrapper": "ccb",
  
  // Claude Code Chat 插件（需要符号链接支持）
  "claudeCodeChat.wsl.enabled": false,
  
  // 其他常用配置
  "claudeCode.useTerminal": false,
  "claudeCode.preferredLocation": "panel",
  "claudeCode.autosave": true
}
```

配合符号链接：

```bash
# 创建 claude -> ccb 链接
ln -s /home/developer/.bun/bin/ccb /home/developer/.bun/bin/claude

# 确保 PATH 优先级
export PATH="/home/developer/.bun/bin:$PATH"
```

---

## 验证与测试

### 1. 验证 CLI 安装

```bash
# 检查命令可用性
which ccb
which claude

# 检查版本
ccb --version
claude --version  # 如果创建了链接

# 测试基本功能
echo "say hello" | ccb -p
```

### 2. 验证 VS Code 插件

#### 官方插件验证

1. 打开 VS Code
2. 按 `Ctrl+Shift+P` (Windows/Linux) 或 `Cmd+Shift+P` (macOS)
3. 输入 "Claude Code: Open"
4. 在 Claude Chat 中输入：`/version`
5. 检查输出日志：
   - 打开 Output 面板：`View -> Output`
   - 选择 "Claude Code" 频道
   - 查找 "Using custom Claude process wrapper: ccb"

#### Claude Code Chat 插件验证

1. 按 `Ctrl+Shift+C` 打开聊天面板
2. 输入测试消息
3. 检查调用的命令：
   ```bash
   # 在终端中查看进程
   ps aux | grep claude
   # 应该看到 /home/developer/.bun/bin/ccb 或 /home/developer/.bun/bin/claude
   ```

### 3. 功能测试清单

- [ ] CLI 基本命令正常（`ccb --version`, `ccb --help`）
- [ ] Pipe 模式工作（`echo "say hello" | ccb -p`）
- [ ] 交互模式工作（`ccb`）
- [ ] VS Code 官方插件可以启动 Claude
- [ ] VS Code Chat 插件可以启动 Claude
- [ ] 文件读写功能正常
- [ ] 工具调用正常（Bash、Grep、Read、Edit 等）

### 4. 环境变量检查

```bash
# 检查 PATH 优先级
echo $PATH | tr ':' '\n' | grep -n bun

# 检查 CCB 相关环境变量
env | grep -E "CLAUDE|OPENAI|GEMINI"

# 检查 Feature Flags（可选）
echo $FEATURE_BUDDY
```

---

## 常见问题

### Q1: 如何恢复使用官方版本？

**A:** 根据你的安装方式：

**方式 1：如果只是 VS Code 配置**
```json
// 删除或注释掉 settings.json 中的配置
// "claudeCode.claudeProcessWrapper": "ccb"
```

**方式 2：如果创建了符号链接**
```bash
# 删除链接
rm /home/developer/.bun/bin/claude

# 恢复官方命令（如果备份过）
mv /home/developer/.nvm/versions/node/v20.20.1/bin/claude-official \
   /home/developer/.nvm/versions/node/v20.20.1/bin/claude
```

**方式 3：如果使用 Shell 别名**
```bash
# 删除别名
unalias claude

# 或重新加载配置
source ~/.bashrc
```

---

### Q2: VS Code 插件报错 "command not found: claude"

**A:** 检查以下几点：

1. **确认 ccb 已安装**
   ```bash
   which ccb
   ccb --version
   ```

2. **检查符号链接是否存在**
   ```bash
   ls -la /home/developer/.bun/bin/claude
   ```

3. **检查 PATH 环境变量**
   ```bash
   echo $PATH | grep bun
   # 如果没有输出，添加到配置文件：
   echo 'export PATH="/home/developer/.bun/bin:$PATH"' >> ~/.bashrc
   source ~/.bashrc
   ```

4. **重启 VS Code**
   ```bash
   # 完全退出 VS Code
   killall code
   
   # 重新打开
   code
   ```

---

### Q3: 如何同时使用官方版本和 CCB？

**A:** 使用不同的命令：

```bash
# 使用 CCB
ccb "explain this code"

# 使用官方版本（需要完整路径）
/home/developer/.nvm/versions/node/v20.20.1/bin/claude "explain this code"

# 或者创建别名
alias claude-official='/home/developer/.nvm/versions/node/v20.20.1/bin/claude'
```

在 VS Code 中：
- 使用工作区级别配置，不同项目使用不同的 `settings.json`
- 或者使用 `claudeProcessWrapper` 切换

---

### Q4: CCB 和官方版本有什么区别？

**A:** 主要区别：

| 特性 | 官方版本 | CCB |
|------|---------|-----|
| 开源 | ❌ 闭源 | ✅ 开源 (逆向工程) |
| 功能 | 完整功能 | 大部分功能（部分模块 stub） |
| 费用 | 需要订阅 | 免费（需要 API key） |
| 自定义 | 有限 | 完全可控 |
| 更新 | 官方维护 | 社区维护 |
| 兼容性 | 稳定 | 可能有 bug |

CCB 的优势：
- 支持自定义 API endpoint（OpenAI 兼容、Gemini）
- 支持 Computer Use、Voice Mode 等功能
- 完全透明的代码
- 可本地调试和修改

---

### Q5: 升级 CCB 后插件不工作？

**A:** 重新安装和链接：

```bash
# 1. 升级 CCB
bun update -g claude-code-best

# 2. 重新信任
bun pm -g trust claude-code-best

# 3. 重新创建符号链接（如果有）
rm /home/developer/.bun/bin/claude
ln -s /home/developer/.bun/bin/ccb /home/developer/.bun/bin/claude

# 4. 重启 VS Code
```

---

### Q6: 如何检查当前使用的是哪个版本？

**A:** 使用以下命令：

```bash
# 检查 ccb 版本
ccb --version

# 检查 claude 指向的实际命令
ls -la $(which claude)

# 检查进程调用的实际程序
ps aux | grep claude | grep -v grep
```

在 VS Code 中：
1. 打开 Output 面板
2. 选择 "Claude Code" 频道
3. 查看启动日志中的版本信息

---

### Q7: 配置后某些功能不工作？

**A:** 排查步骤：

1. **检查环境变量**
   ```bash
   env | grep CLAUDE
   ```

2. **检查 API Key**
   ```bash
   # 确保设置了 API Key
   echo $ANTHROPIC_API_KEY
   ```

3. **检查权限**
   ```bash
   # 检查 ccb 可执行权限
   ls -la /home/developer/.bun/bin/ccb
   ```

4. **查看详细日志**
   ```bash
   # 使用 debug 模式运行
   DEBUG=* ccb
   ```

5. **检查 Feature Flags**
   ```bash
   # 某些功能需要启用 feature flag
   FEATURE_BUDDY=1 ccb
   ```

---

### Q8: Windows / macOS 有什么不同？

**A:** 平台差异：

**Linux**（本文档主要针对）：
- 路径：`/home/developer/.bun/bin/`
- Shell：`bash` / `zsh`
- VS Code 配置：`~/.config/Code/User/settings.json`

**macOS**：
- 路径：`/Users/username/.bun/bin/`
- Shell：`zsh` (默认)
- VS Code 配置：`~/Library/Application Support/Code/User/settings.json`

**Windows**：
- 路径：`C:\Users\username\.bun\bin\`
- Shell：PowerShell / Git Bash
- VS Code 配置：`%APPDATA%\Code\User\settings.json`
- 建议使用 WSL 运行 CCB

**通用配置**（跨平台）：
```bash
# 添加 Bun 到 PATH（根据平台选择）
export PATH="$HOME/.bun/bin:$PATH"  # Linux/macOS
$env:Path = "$env:USERPROFILE\.bun\bin;$env:Path"  # Windows PowerShell
```

---

## 附录

### 相关文档链接

- [CCB 项目主页](https://github.com/claude-code-best/claude-code)
- [CCB 在线文档](https://ccb.agent-aura.top/)
- [官方 Claude Code 文档](https://docs.anthropic.com/en/docs/claude-code)
- [Bun 官方文档](https://bun.sh/docs)

### 快速命令参考

```bash
# 安装 CCB
bun install -g claude-code-best

# 创建符号链接
ln -s /home/developer/.bun/bin/ccb /home/developer/.bun/bin/claude

# 验证安装
ccb --version
which claude

# 配置 VS Code
code ~/.config/Code/User/settings.json
# 添加: "claudeCode.claudeProcessWrapper": "ccb"

# 重启 VS Code 使配置生效
```

### 环境变量速查

```bash
# CCB 相关
CLAUDE_CODE_USE_OPENAI=1          # 启用 OpenAI 兼容模式
OPENAI_API_KEY="sk-..."           # OpenAI API Key
OPENAI_BASE_URL="http://..."      # OpenAI endpoint

# Gemini 相关
CLAUDE_CODE_USE_GEMINI=1          # 启用 Gemini 模式
GEMINI_API_KEY="..."              # Gemini API Key

# Feature Flags
FEATURE_BUDDY=1                   # 启用 Buddy 功能
FEATURE_VOICE_MODE=1              # 启用 Voice Mode

# 网络代理（国内用户）
DEFAULT_RELEASE_BASE=https://ghproxy.net/...
```

---

**文档版本**: v1.0  
**最后更新**: 2026-04-07  
**作者**: Claude Code Best 社区  
**许可**: MIT
