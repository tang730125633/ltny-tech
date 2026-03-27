# QMD 记忆系统安装与配置指南

> 适用于：OpenClaw v2026.2.x + macOS / Linux
> 编写：Claude 导师 | 日期：2026-03-28
> 验证环境：macOS Darwin 25.0.0, Apple M4, Node.js v24.11.1

---

## 一、QMD 是什么

QMD（Query-Memory-Document）是 OpenClaw 的**高级记忆检索后端**。

OpenClaw 默认的记忆检索是简单的文本匹配，效果差、容易找不到相关内容。QMD 替换了这个检索层，提供三种搜索策略：

| 搜索模式 | 说明 | 速度 | 适用场景 |
|:---|:---|:---|:---|
| `search` | BM25 关键词匹配 | ⚡ 即时 | 日常使用（推荐默认） |
| `vsearch` | 向量语义搜索 | 🕐 几秒 | 中文语义理解 |
| `query` | 混合搜索 + LLM 重排 | 🕐 较慢 | 最高质量检索 |

**核心特点**：
- 全部本地运行，数据不离开机器
- 使用 3 个小型 GGUF 模型（总计约 2GB）
- 记忆文件（Markdown）不需要任何修改
- 支持 GPU 加速（Metal / CUDA）

---

## 二、安装前检查

### 必要条件

```bash
# 1. 确认 Node.js 已安装（v18+）
node --version

# 2. 确认 npm 可用
npm --version

# 3. 确认 OpenClaw 已安装
openclaw --version

# 4. 确认磁盘空间（需要约 2GB 用于模型文件）
df -h ~
```

### 可选（推荐）

```bash
# bun 包管理器（更快的安装速度）
bun --version
```

---

## 三、安装 QMD

### 方式一：npm 安装（推荐，兼容性最好）

```bash
npm install -g @tobilu/qmd
```

### 方式二：bun 安装（更快）

```bash
bun install -g https://github.com/tobi/qmd
export PATH="$HOME/.bun/bin:$PATH"
```

### 验证安装

```bash
qmd --help
```

应该看到 `qmd — Quick Markdown Search` 帮助信息。

如果提示 `command not found`：
```bash
# npm 安装的情况
which qmd
# 如果没有，检查 npm 全局 bin 目录是否在 PATH 中
npm bin -g

# bun 安装的情况，确保 PATH 包含 bun bin
export PATH="$HOME/.bun/bin:$PATH"
# 写入 shell 配置使其永久生效
echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.zshrc
```

---

## 四、初始化索引

### 1. 更新索引（扫描记忆文件）

```bash
qmd update
```

QMD 会自动扫描 OpenClaw workspace 下的记忆文件：
- `MEMORY.md`（长期记忆）
- `memory/*.md`（日志记忆）

### 2. 生成向量嵌入

```bash
qmd embed
```

首次运行会自动下载嵌入模型（约 313MB），请等待下载完成。

### 3. 验证索引状态

```bash
qmd status
```

确认：
- `Documents > Total` 显示已索引的文件数量
- `Documents > Vectors` 显示已生成的向量数量
- `Models > Embedding` 显示模型已就绪

### 4. 测试搜索

```bash
# BM25 关键词搜索
qmd search "你的关键词"

# 向量语义搜索（首次会下载额外模型约 1.2GB）
qmd vsearch "语义查询"
```

---

## 五、配置 OpenClaw 使用 QMD

### 找到配置文件

OpenClaw 的主配置文件是 `openclaw.json`，通常在：

```
~/.openclaw/openclaw.json
```

> ⚠️ 注意：不是 `config.json`（那个是频道/渠道配置）

### 添加 memory 配置

在 `openclaw.json` 中添加 `memory` 字段：

```json
{
  "memory": {
    "backend": "qmd",
    "citations": "auto",
    "qmd": {
      "searchMode": "search",
      "includeDefaultMemory": true,
      "update": {
        "onBoot": true,
        "interval": "10m"
      }
    }
  }
}
```

### 完整示例

假设你的 `openclaw.json` 原本长这样：

```json
{
  "agents": {
    "defaults": {
      "model": { "primary": "kimi-coding/k2p5" },
      "workspace": "/Users/mf/.openclaw/workspace"
    }
  },
  "gateway": {
    "port": 18789
  },
  "skills": {
    "install": { "nodeManager": "npm" }
  }
}
```

添加 memory 配置后：

```json
{
  "agents": {
    "defaults": {
      "model": { "primary": "kimi-coding/k2p5" },
      "workspace": "/Users/mf/.openclaw/workspace"
    }
  },
  "gateway": {
    "port": 18789
  },
  "memory": {
    "backend": "qmd",
    "citations": "auto",
    "qmd": {
      "searchMode": "search",
      "includeDefaultMemory": true,
      "update": {
        "onBoot": true,
        "interval": "10m"
      }
    }
  },
  "skills": {
    "install": { "nodeManager": "npm" }
  }
}
```

### 配置项说明

| 字段 | 值 | 说明 |
|:---|:---|:---|
| `backend` | `"qmd"` | 启用 QMD 作为记忆后端 |
| `citations` | `"auto"` | 回答时自动引用记忆来源 |
| `searchMode` | `"search"` | BM25 模式（最快，推荐默认） |
| `includeDefaultMemory` | `true` | 包含 MEMORY.md 和 memory/ 目录 |
| `update.onBoot` | `true` | 网关启动时自动重建索引 |
| `update.interval` | `"10m"` | 每 10 分钟自动更新索引 |

### 可选：添加额外的索引路径

如果有其他需要索引的文档目录，可以在 `qmd.paths` 中添加：

```json
{
  "memory": {
    "backend": "qmd",
    "qmd": {
      "searchMode": "search",
      "includeDefaultMemory": true,
      "paths": [
        {
          "name": "knowledge-base",
          "path": "/Users/mf/.openclaw/workspace/archive/08_文档资料/知识库",
          "pattern": "**/*.md"
        }
      ]
    }
  }
}
```

---

## 六、重启生效

配置修改后，需要重启 OpenClaw 网关：

### 方式一：TUI 内重启

在 OpenClaw TUI 中输入：

```
/restart
```

### 方式二：命令行重启

```bash
# 停止网关
openclaw gateway stop
# 或者强制停止
pkill -f "openclaw-gateway"

# 等待 2-3 秒
sleep 3

# 重新启动
openclaw gateway
```

### 验证 QMD 已加载

查看网关日志，确认出现以下信息：

```
qmd memory startup initialization armed for agent "main"
```

日志位置：`/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log`

---

## 七、故障排查

### 问题 1：`qmd: command not found`

```bash
# 检查安装位置
npm list -g @tobilu/qmd
# 确认全局 bin 在 PATH 中
echo $PATH
npm bin -g
```

### 问题 2：索引为空（0 files）

```bash
# 检查 workspace 路径是否正确
ls ~/.openclaw/workspace/MEMORY.md
ls ~/.openclaw/workspace/memory/

# 手动指定路径重建
qmd collection remove memory-root
qmd collection add memory-root ~/.openclaw/workspace --pattern "MEMORY.md"
qmd update
qmd embed
```

### 问题 3：向量搜索很慢

首次使用 `vsearch` 或 `query` 模式会下载额外模型（约 1.2GB + 0.6GB），这是正常的。后续使用会很快。

```bash
# 预热：提前下载所有模型
qmd embed          # 下载嵌入模型
qmd vsearch "test" # 触发下载查询扩展模型
qmd query "test"   # 触发下载重排模型
```

### 问题 4：网关未识别 QMD 配置

确认 memory 配置写在了正确的文件中：
- ✅ `~/.openclaw/openclaw.json`（主配置）
- ❌ `~/.openclaw/config.json`（频道配置，写这里无效）

### 问题 5：`device_token_mismatch` 报错

如果修改配置后 TUI 连不上，需要清除设备配对数据重新配对：

```bash
# 停止所有 OpenClaw 进程
pkill -f "openclaw-gateway"
pkill -f "openclaw-tui"
pkill -f "openclaw$"

# 备份并清除配对数据
cp ~/.openclaw/devices/paired.json ~/.openclaw/devices/paired.json.bak
cp ~/.openclaw/devices/pending.json ~/.openclaw/devices/pending.json.bak
echo '{}' > ~/.openclaw/devices/paired.json
echo '{}' > ~/.openclaw/devices/pending.json

# 重新启动网关
openclaw gateway

# 在新终端启动 TUI（会自动重新配对）
openclaw
```

---

## 八、高性能配置（M4 Pro / 64GB 专用）

> 适用于戴总的 Mac mini：M4 Pro + 64GB RAM + 2TB SSD

默认配置使用最轻量的模型和最快的搜索模式，适合普通机器。
戴总的机器性能强劲，可以解锁全部能力。

### 升级嵌入模型

默认的 embeddinggemma-300M 只有 3 亿参数。可以升级到 Qwen3-Embedding-0.6B（6 亿参数），检索质量显著提升：

```bash
# 在 openclaw.json 的 agents.defaults 中添加嵌入模型配置
# 或通过 CLI 配置：
openclaw configure --section agents.defaults.embeddingModel \
  'hf:Qwen/Qwen3-Embedding-0.6B-GGUF/Qwen3-Embedding-0.6B-Q8_0.gguf?provider=local&hybrid=true&cpu_threads=8'
```

### 升级搜索模式

将 `searchMode` 从 `"search"` 改为 `"query"`，启用完整的混合搜索 + LLM 重排：

### 完整高性能 memory 配置

```json
{
  "memory": {
    "backend": "qmd",
    "citations": "auto",
    "qmd": {
      "searchMode": "query",
      "includeDefaultMemory": true,
      "paths": [
        {
          "name": "knowledge-base",
          "path": "/Users/mf/.openclaw/workspace/archive/08_文档资料/知识库",
          "pattern": "**/*.md"
        }
      ],
      "update": {
        "onBoot": true,
        "interval": "5m"
      }
    },
    "memorySearch": {
      "query": {
        "hybrid": {
          "enabled": true,
          "vectorWeight": 0.6,
          "textWeight": 0.4,
          "candidateMultiplier": 3
        }
      }
    },
    "mmr": {
      "enabled": true,
      "lambda": 0.7
    },
    "temporalDecay": {
      "enabled": true,
      "halfLifeDays": 30
    },
    "limits": {
      "maxResults": 10,
      "maxSnippetChars": 2000,
      "maxInjectedChars": 8000
    }
  }
}
```

### 高性能 vs 标准配置对比

| 配置项 | 标准配置（普通机器） | 高性能配置（M4 Pro 64GB） |
|:---|:---|:---|
| searchMode | `search`（BM25） | `query`（混合 + LLM 重排） |
| 嵌入模型 | embeddinggemma-300M | Qwen3-Embedding-0.6B |
| 混合搜索 | ❌ 关闭 | ✅ 开启（向量 0.6 + 文本 0.4） |
| MMR 多样性重排 | ❌ 关闭 | ✅ 开启（lambda=0.7） |
| 时间衰减 | ❌ 关闭 | ✅ 开启（半衰期 30 天） |
| 索引更新间隔 | 10 分钟 | 5 分钟 |
| 返回结果数 | 默认 | 最多 10 条 |
| 注入上下文量 | 默认 | 最多 8000 字符 |
| 额外索引路径 | 无 | 知识库目录 |
| 预估额外内存占用 | ~500MB | ~2GB |
| 单次搜索延迟 | <100ms | 1-3 秒（可接受） |

### 预热所有模型（首次安装后执行）

```bash
# 下载并预热全部 3 个模型，避免首次对话时等待
qmd update
qmd embed -f
qmd vsearch "电力行业政策"   # 触发查询扩展模型下载
qmd query "碳核算方法"       # 触发重排模型下载
echo "✅ 所有模型已就绪"
```

---

## 九、模型文件说明（全部模型）

QMD 使用 3 个本地 GGUF 模型，存储在 `~/.cache/qmd/models/`：

| 模型 | 大小 | 用途 | 何时下载 |
|:---|:---|:---|:---|
| embeddinggemma-300M | ~313MB | 文本向量化 | `qmd embed` 时 |
| qmd-query-expansion-1.7B | ~1.2GB | 查询扩展 | 首次 `vsearch` 时 |
| Qwen3-Reranker-0.6B | ~0.6GB | 结果重排 | 首次 `query` 时 |

全部模型总计约 2.1GB，均在本地运行，不需要 API 密钥。

---

## 九、Codex / Claude 协作说明

本文档由 Claude 导师编写，供 Codex 或其他 AI 协作者参考执行。

**Codex 执行清单**：

1. [ ] 在目标机器上运行 `npm install -g @tobilu/qmd`
2. [ ] 运行 `qmd update && qmd embed` 建立初始索引
3. [ ] 在 `~/.openclaw/openclaw.json` 中添加 memory 配置
4. [ ] 重启网关：`openclaw gateway stop && sleep 3 && openclaw gateway`
5. [ ] 检查日志确认 `qmd memory startup initialization armed`
6. [ ] 在 TUI 中测试对话，验证记忆检索是否生效

**注意事项**：
- 确保磁盘空间 ≥ 3GB（模型 + 索引）
- 首次安装建议预热所有模型（见故障排查第 3 点）
- 如果机器没有 GPU，QMD 会使用 CPU 运行，速度稍慢但功能不受影响

---

*由 Claude 导师编写 | tang730125633/ltny-tech*
