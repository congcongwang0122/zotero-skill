<div align="center">

# zotero-skill

文献管理自动化工具箱：连接 Zotero 本地 API，一键搜索、导出 BibTeX、插入引用、读取 PDF 全文。

不只是让文献「躺在那里」，而是让每一篇都能被真正「用起来」。

<br>

![](https://img.shields.io/badge/type-Skill-black)
![](https://img.shields.io/badge/license-MIT-blue)
![](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-green)
![](https://img.shields.io/badge/language-Python%203.10+-yellow)
![](https://img.shields.io/badge/status-stable-brightgreen)

<br>

[30 秒看懂](#30-秒看懂) · [快速开始](#快速开始) · [功能地图](#功能地图) · [使用示例](#使用示例)

</div>

---

## 30 秒看懂

| 你给它什么 | 它帮你做什么 |
|:---|:---|
| 一个 Zotero 收藏夹 | 🔍 遍历全部文献，输出标题、作者、年份、摘要 |
| 一个关键词 | 📝 全文搜索，快速定位相关论文 |
| 一份 LaTeX / Markdown 草稿 | 📎 自动查找文献并插入 `\cite{}` 或 `[@key]` |
| 一篇本地 PDF | 📖 提取全文文本，生成摘要与关键结论 |
| 一个 BibTeX / RIS 文件 | 📥 批量导入 Zotero，自动去重 |
| 整个 Zotero 库 | 📤 一键导出 `references.bib`，条目数自动统计 |

---

## 为什么需要它

做科研的你，一定经历过这些痛苦时刻：

- 某个文件夹里攒了几十上百篇论文，永远没有时间系统整理；
- 写综述时，对着一堆 PDF 发呆，不知道从何下手；
- Zotero 里的文献越积越多，但它们只是「躺在那里」，从未被真正「读过」。

**zotero-skill 就是为了解决这些问题而生。** 它直接连接你的 Zotero 本地数据库，让 AI 替你梳理文献、总结脉络、输出综述。

---

## 快速开始
<img width="1293" height="1317" alt="8a8083ce873159c3e3ffadb2d4fba694" src="https://github.com/user-attachments/assets/90091051-6046-4b40-aced-0636d18a2beb" />

```bash
# 1. 确保 Zotero 桌面客户端已打开
# 2. 开启本地 API：编辑 → 设置 → 高级 → 允许其他应用程序与 Zotero 通信

# 3. 检查连接状态
py scripts/zotero.py status --json

# 4. 搜索文献
py scripts/zotero.py search "transformer" --json

# 5. 导出 BibTeX
py scripts/zotero.py export-bibtex --out references.bib

# 6. 插入引用到 LaTeX 文件
py scripts/zotero.py cite --query "transformer" --tex main.tex --bib references.bib
```

> **Note:** macOS / Linux 用户请将 `py` 替换为 `python3`。

---

## 功能地图

| 能力 | 命令示例 | 说明 |
|:---|:---|:---|
| **连接检测** | `status` | 检查 Zotero 本地 API 和 Connector 是否就绪 |
| **API 启用** | `enable --restart` | 自动修改 `prefs.js` 并重启 Zotero |
| **搜索文献** | `search "关键词"` | 全文搜索，返回标题、作者、年份 |
| **列出收藏夹** | `collections` | 查看所有 Collection 及其文献数量 |
| **列出标签** | `tags` | 查看所有标签及对应文献数 |
| **导出 BibTeX** | `export-bibtex --out refs.bib` | 支持分页导出，自动去重 |
| **同步 .bib** | `sync-bib --out references.bib` | 一键覆盖式导出整个库 |
| **获取引用** | `citations --style apa` | 按指定格式生成格式化引用 |
| **读取全文** | `fulltext <attachmentKey>` | 提取 Zotero 已索引的 PDF 全文 |
| **PDF 定位** | `file-url <attachmentKey>` | 获取 PDF 本地文件路径 |
| **插入引用** | `cite --query "xxx" --tex file.tex` | 自动查文献、写 .bib、插 `\cite{}` |
| **批量导入** | `import-bibtex --file refs.bib --yes` | 通过 Connector 导入到当前选中收藏夹 |

---

## 技术亮点

| 特性 | 说明 |
|:---|:---|
| **纯标准库** | `zotero.py` 仅依赖 Python 3 stdlib，无需 pip 安装任何包 |
| **HTTP 直连** | 通过 `localhost:23119` 调用 Zotero Local API，零配置 |
| **PDF 本地读取** | 自动定位 `%USERPROFILE%\Zotero\storage\` 下的 PDF 文件 |
| **跨平台兼容** | Windows、macOS、Linux 均适配，自动检测平台差异 |
| **隐私安全** | 所有数据读取在本地完成，文献不上传云端 |
| **防御性编程** | 分页安全上限、空路径校验、进程精确匹配，避免误杀与死循环 |

---

## 使用示例

### 场景一：批量总结一个收藏夹的文献

```bash
# 1. 获取收藏夹 key
py scripts/zotero.py collections --json

# 2. 获取该收藏夹下的所有文献
py scripts/zotero.py search "collection:XXX" --json

# 3. 读取每篇文献的 PDF 全文，交给 AI 总结
```

### 场景二：写论文时插入引用

```bash
# 一句话自动完成：查文献 → 写 .bib → 插 \cite{}
py scripts/zotero.py cite --query "attention is all you need" --tex main.tex --bib references.bib
```

### 场景三：导出完整参考文献

```bash
# 一键导出整个库到 references.bib
py scripts/zotero.py sync-bib --out references.bib
```

---

## 项目结构

```
zotero-skill/
├── SKILL.md                    # Claude Code Skill 主文件
├── README.md                   # 本文件
├── scripts/
│   └── zotero.py               # 纯 stdlib 的 Zotero 操作脚本
├── references/
│   └── local-api-routes.md     # Zotero Local API 完整路由文档
├── agents/
│   └── openai.yaml             # OpenAI 平台适配配置
└── assets/
    └── icon.png                # Skill 图标
```

---

## 开源信息

- **GitHub**: https://github.com/congcongwang0122/zotero-skill
- **License**: MIT
- **适用平台**: 任何支持 Skill 扩展的 AI 助手平台（Claude Code、Codex、Cursor 等）

---

<div align="center">

> **「你负责读论文的灵感，我负责读论文的体力活。」**
>
> —— zotero-skill，让文献综述从一周缩短到一小时。

<br>

*Star ⭐ 一下，让更多人看到这个工具！*

</div>
