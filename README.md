# Web Reader Pro

[中文](#中文) | [English](#english)

---

## 中文

### 介绍

Web Reader Pro 是一个为 OpenClaw 设计的增强型网页内容提取技能。它采用多层级降级策略，配备智能路由学习、缓存机制和质量评估系统。

### 核心特性

| 特性 | 描述 |
|------|------|
| **三级降级策略** | Jina API → Scrapling → WebFetch，自动选择最优方案 |
| **Jina 配额监控** | 计数器追踪 API 调用，配额接近时自动告警并降级 |
| **智能缓存层** | 同一 URL 短期重复抓取直接返回缓存，节省配额 |
| **质量评分系统** | 字数统计 + 标题检测，不达标自动尝试下一级 |
| **域名路由学习** | 记录每个域名的最优提取方式，持续优化 |
| **指数退避重试** | 每个层级配备重试机制，失败后自动指数退避 |

### 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/0xcjl/web-reader-pro.git
cd web-reader-pro

# 2. 安装依赖
pip install -r requirements.txt

# 3. 安装 Scrapling（用于 Tier 2，需要 Node.js）
./scripts/install_scrapling.sh

# 4. 配置环境变量
export JINA_API_KEY="your-jina-api-key"

# 5. 使用示例
python -c "
from scripts.web_reader_pro import WebReaderPro
reader = WebReaderPro()
result = reader.fetch('https://example.com')
print(f'标题: {result[\"title\"]}')
print(f'内容: {result[\"content\"][:500]}...')
"
```

### 安装为 OpenClaw Skill

将 `SKILL.md` 和 `scripts/` 目录复制到 OpenClaw skills 目录：

```bash
cp -r SKILL.md scripts/ ~/.openclaw/skills/web-reader-pro/
```

### 配置说明

#### 环境变量

| 变量名 | 必填 | 默认值 | 说明 |
|--------|------|--------|------|
| `JINA_API_KEY` | 是 | - | Jina Reader API 密钥 |
| `WEB_READER_CACHE_DIR` | 否 | `~/.openclaw/cache/web-reader-pro/` | 缓存目录 |
| `WEB_READER_LEARNING_DB` | 否 | `~/.openclaw/data/web-reader-pro/routes.json` | 学习数据库路径 |
| `WEB_READER_JINA_QUOTA` | 否 | `100000` | Jina API 配额上限 |

#### 代码配置

```python
from scripts.web_reader_pro import WebReaderPro

reader = WebReaderPro(
    jina_api_key="your-key",      # 环境变量或直接传入
    cache_ttl=3600,                # 缓存有效期（秒）
    quality_threshold=200,         # 最低字数阈值
    max_retries=3,                 # 每层最大重试次数
    enable_learning=True,          # 启用域名学习
    scrapling_path="/path/to/scrapling"  # Scrapling 路径
)
```

### API 返回格式

```python
{
    "title": "页面标题",
    "content": "提取的内容（Markdown格式）",
    "url": "https://example.com",
    "tier_used": "jina",           # 实际使用的层级
    "quality_score": 85,           # 质量评分 (0-100)
    "cached": False,              # 是否来自缓存
    "domain_learned_tier": "jina", # 该域名的最优层级
    "extracted_at": "2024-01-01T00:00:00Z"
}
```

### 层级说明

| 层级 | 速度 | JS 渲染 | 适用场景 | 成本 |
|------|------|---------|----------|------|
| Tier 1: Jina | 快 | 否 | 静态页面、文章 | API 调用 |
| Tier 2: Scrapling | 中 | 是 | SPA、动态内容 | CPU |
| Tier 3: WebFetch | 最快 | 否 | 简单页面、降级 | 免费 |

### 故障排除

**Q: Jina API 返回 403 或 429？**
A: 配额可能耗尽或触发限流。检查 `reader.get_jina_status()`，技能会自动降级到其他层级。

**Q: Scrapling 安装失败？**
A: 确保已安装 Node.js (>=14) 和 npm，然后运行 `./scripts/install_scrapling.sh`

**Q: 缓存没有生效？**
A: 检查缓存目录写入权限，确保 `cache_ttl` 设置正确。

### 贡献

欢迎提交 Issue 和 Pull Request！

### 许可证

MIT License

---

## English

### Introduction

Web Reader Pro is an enhanced web content extraction skill designed for OpenClaw. It uses a multi-tier fallback strategy with intelligent routing, caching, and quality assessment systems.

### Core Features

| Feature | Description |
|---------|-------------|
| **Three-Tier Fallback** | Jina API → Scrapling → WebFetch, auto-select optimal method |
| **Jina Quota Monitoring** | Counter tracks API calls, auto-warn and fallback when quota approaches |
| **Smart Cache Layer** | Same URL short-term repeat fetches return cached, saving quota |
| **Quality Scoring** | Word count + title detection, auto-escalate if below threshold |
| **Domain Routing Learning** | Records optimal extraction method per domain, continuous optimization |
| **Exponential Backoff Retry** | Each tier has retry mechanism with automatic exponential backoff |

### Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/0xcjl/web-reader-pro.git
cd web-reader-pro

# 2. Install dependencies
pip install -r requirements.txt

# 3. Install Scrapling (for Tier 2, requires Node.js)
./scripts/install_scrapling.sh

# 4. Set environment variable
export JINA_API_KEY="your-jina-api-key"

# 5. Example usage
python -c "
from scripts.web_reader_pro import WebReaderPro
reader = WebReaderPro()
result = reader.fetch('https://example.com')
print(f'Title: {result[\"title\"]}')
print(f'Content: {result[\"content\"][:500]}...')
"
```

### Installation as OpenClaw Skill

Copy `SKILL.md` and `scripts/` to OpenClaw skills directory:

```bash
cp -r SKILL.md scripts/ ~/.openclaw/skills/web-reader-pro/
```

### Configuration

#### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `JINA_API_KEY` | Yes | - | Jina Reader API key |
| `WEB_READER_CACHE_DIR` | No | `~/.openclaw/cache/web-reader-pro/` | Cache directory |
| `WEB_READER_LEARNING_DB` | No | `~/.openclaw/data/web-reader-pro/routes.json` | Learning database path |
| `WEB_READER_JINA_QUOTA` | No | `100000` | Jina API quota limit |

#### Code Configuration

```python
from scripts.web_reader_pro import WebReaderPro

reader = WebReaderPro(
    jina_api_key="your-key",      # From env or direct
    cache_ttl=3600,                # Cache TTL in seconds
    quality_threshold=200,         # Min word count threshold
    max_retries=3,                 # Max retries per tier
    enable_learning=True,          # Enable domain learning
    scrapling_path="/path/to/scrapling"  # Scrapling binary path
)
```

### API Response Format

```python
{
    "title": "Page Title",
    "content": "Extracted content (Markdown)",
    "url": "https://example.com",
    "tier_used": "jina",           # Actual tier used
    "quality_score": 85,           # Quality score (0-100)
    "cached": False,              # From cache or not
    "domain_learned_tier": "jina", # Optimal tier for this domain
    "extracted_at": "2024-01-01T00:00:00Z"
}
```

### Tier Comparison

| Tier | Speed | JS Rendering | Best For | Cost |
|------|-------|--------------|----------|------|
| Tier 1: Jina | Fast | No | Static pages, articles | API calls |
| Tier 2: Scrapling | Medium | Yes | SPAs, dynamic content | CPU |
| Tier 3: WebFetch | Fastest | No | Simple pages, fallback | Free |

### Troubleshooting

**Q: Jina API returns 403 or 429?**
A: Quota may be exhausted or rate limited. Check `reader.get_jina_status()`, skill will auto-fallback.

**Q: Scrapling installation failed?**
A: Ensure Node.js (>=14) and npm are installed, then run `./scripts/install_scrapling.sh`

**Q: Cache not working?**
A: Check cache directory write permissions and `cache_ttl` setting.

### Contributing

Issues and Pull Requests welcome!

### License

MIT License

---

## Links

- [GitHub Repository](https://github.com/0xcjl/web-reader-pro)
- [Report Issues](https://github.com/0xcjl/web-reader-pro/issues)
