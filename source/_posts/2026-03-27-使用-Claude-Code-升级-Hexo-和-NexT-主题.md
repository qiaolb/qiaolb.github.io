---
title: 使用 Claude Code 升级 Hexo 和 NexT 主题
date: 2026-03-27 09:40:18
tags:
  - Hexo
  - NexT
  - Claude Code
  - 博客
categories: [工具]
---

# 概述

今天做了一件很酷的事情 —— 使用 **Claude Code** 辅助升级了我的博客。

先说说背景：我的博客已经运行了很多年，但 Hexo 框架和 NexT 主题的版本一直停留在好几年前：

- **Hexo**: 6.2.0 → 8.1.1
- **NexT 主题**: 7.8.0 → 8.27.0

这可不是个小工程，涉及到依赖升级、配置迁移、兼容性测试等等。但有了 Claude Code 的帮忙，整个过程出奇地顺利。

---

# 什么是 Claude Code？

Claude Code 是 Anthropic 推出的命令行 AI 助手，专门用于帮助开发者完成编程任务。它可以：

- 理解并执行复杂的开发任务
- 自动执行 shell 命令
- 读写和修改代码文件
- 执行 git 操作
- 进行代码审查

简单说，就是一个懂编程的 AI 助手，可以直接在你的项目里工作。

---

# 升级过程

## 1. 项目分析

首先，我让 Claude Code 分析了当前项目：

```bash
# Claude Code 自动执行的分析命令
ls -la
cat package.json
cat themes/next/package.json
git log --oneline -5
```

通过分析，AI 快速了解了项目结构和技术栈。

## 2. 版本调研

接下来，让 AI 查询最新版本的依赖：

```bash
npm show hexo version
npm show hexo-theme-next version
```

查询结果：
- Hexo 最新版：**8.1.1**
- NexT 主题最新版：**8.27.0**

## 3. 执行升级

这是最复杂的部分，涉及到多个步骤：

### 3.1 更新 package.json

AI 帮我修改了 `package.json` 中的所有依赖版本：

```json
{
  "dependencies": {
    "hexo": "^8.1.1",
    "hexo-deployer-git": "^4.0.0",
    "hexo-generator-archive": "^2.0.0",
    "hexo-generator-category": "^2.0.0",
    "hexo-generator-index": "^3.0.0",
    "hexo-generator-searchdb": "^1.4.1",
    "hexo-generator-tag": "^2.0.0",
    "hexo-renderer-marked": "^6.0.0",
    "hexo-renderer-stylus": "^3.0.0",
    "hexo-theme-next": "^8.27.0"
  }
}
```

### 3.2 安装依赖

```bash
npm install
```

### 3.3 配置迁移

这是最繁琐的部分。NexT 8.x 的配置方式有所变化，需要将我原来的配置迁移到新的格式。

AI 帮我：
1. 读取了旧的 `themes/next/_config.yml` 配置
2. 读取了新版本的默认配置
3. 对比差异并生成兼容配置
4. 将配置写入 `_config.yml` 的 `theme_config` 部分

保留的配置包括：
- 社交链接（GitHub、Weibo、Twitter）
- 打赏功能
- 百度统计和推送
- 字体自定义
- 动画效果
- 本地搜索

### 3.4 测试构建

```bash
npm run build
```

构建成功，生成了 94 个文件！

### 3.5 本地预览

```bash
npm run server
```

访问 `http://localhost:4000` 查看效果，一切正常！

## 4. 提交和推送

最后一步，让 AI 帮我完成 git 操作：

```bash
git add package.json package-lock.json yarn.lock _config.yml
git commit -m "升级：Hexo 8.1.1 和 NexT 主题 8.27.0"
git push origin master
```

---

# 使用体验

## 优点

1. **效率高**：整个升级过程只用了不到半小时，如果手动操作可能需要几个小时甚至更久。

2. **自动化**：不需要我一条条执行命令，AI 会自动分析、决策、执行。

3. **减少错误**：配置文件的手动修改很容易出错，AI 可以仔细对比和迁移配置。

4. **上下文理解**：AI 能理解项目的整体结构，做出的决策更加合理。

5. **交互式协作**：我可以随时提问、调整方向，AI 会即时响应。

## 不足

1. **需要人工确认**：关键操作（如 git push）还是需要人工确认，不能完全放手。

2. **复杂决策仍需人工**：对于一些架构层面的决策（比如要不要迁移到 Next.js），AI 只能给出建议，最终决定权在人。

---

# 升级结果

| 项目 | 原版本 | 新版本 |
|------|--------|--------|
| Hexo | 6.2.0 | 8.1.1 |
| NexT | 7.8.0 | 8.27.0 |
| 依赖包 | 多个旧版本 | 全部最新 |

升级后变化：
- FontAwesome 图标升级到 v6（`fa` → `fab/fa/fa-solid`）
- 新增 Light-Dark 模式支持（可选）
- 改进的移动端适配
- 更好的性能

---

# 总结

这次升级体验让我印象深刻。Claude Code 这样的 AI 助手正在改变我们写代码的方式 —— 它不是替代我们，而是成为得力的助手。

对于一些重复性、繁琐的任务（比如依赖升级、配置迁移），AI 可以高效完成；而我们则可以把精力集中在更有价值的事情上，比如内容创作、架构设计等。

**推荐尝试**：如果你也有老旧项目需要升级，不妨试试用 AI 助手来帮忙。

---

# 相关链接

- [Hexo 官网](https://hexo.io/)
- [NexT 主题文档](https://theme-next.js.org/)
- [Claude Code 官网](https://claude.ai/code)
