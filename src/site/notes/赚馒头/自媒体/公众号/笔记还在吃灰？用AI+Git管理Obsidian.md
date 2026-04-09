---
{"dg-publish":true,"permalink":"/赚馒头/自媒体/公众号/笔记还在吃灰？用AI+Git管理Obsidian/","tags":["gardenEntry"],"dg-note-properties":{}}
---

你有没有过这种感觉：收藏夹里存了几百篇文章，笔记软件里记了一堆想法，但三个月后回头看，大部分都变成了"死文件"。你记不住在哪，也记不住讲了什么。

我也是。直到最近看到 Karpathy 提出的一个思路，一下子把我点醒了。

他说：**把知识库当代码仓库管，LLM 当编译器。**

Twitter 上 @yanhua1010 写了一篇很棒的文章，把这个理念做了完整的落地方案梳理，引起了很大的共鸣（402K 浏览）。他用了一张表来类比：

| 软件工程 | 知识库工程 |
|---------|-----------|
| `src/` | `raw/`（原始资料） |
| `build/` | `wiki/`（知识条目） |
| `logs/` | `outputs/`（问答归档） |
| 编译器 | LLM |
| IDE | Obsidian |
| Lint / CI | 健康检查 |

看到这张表的时候我就想：理念有了，但基础设施呢？就像写代码之前得先把 Git 仓库建好一样，搞知识库之前，得先把"仓库"搭起来。

所以这篇文章不讲理念（Yanhua 那篇已经讲得很好了），我只聊一件事：**怎么把 Obsidian 变成一个真正的"代码仓库"。**

## 为什么是 Obsidian + Git

先说 Obsidian。它的笔记就是普通的 Markdown 文件，存在你自己的电脑上。不锁定、不加密、不依赖云服务。哪天你不想用了，文件还是你的。这一点很重要，因为后面接 AI 的时候，AI 需要直接读写这些文件。

再说 Git。既然要"当代码仓库管"，那就真的用代码仓库的工具链。Git 给你的是：

- **版本历史**：每一次修改都有记录，知识怎么演进的，一目了然
- **安全网**：AI 帮你改笔记改错了？`git revert` 一键回滚
- **免费同步**：GitHub 私有仓库，免费、稳定、多端可用

你可能会问：Obsidian 不是有自己的同步方案吗？

有。但各有各的问题。Obsidian Sync 要付费（每月 $8）；iCloud 同步经常冲突，丢过文件的人都懂；坚果云 + Remotely Save 插件能用，但多了一层依赖。

而 Git 是程序员用了几十年的东西，稳定性不用怀疑。更关键的是，它天然就是为"管理文件变更"设计的，和知识库的需求完美契合。

## 实操：从零搭建

### 第一步：初始化仓库

打开终端，进入你的 Obsidian vault 目录，三条命令搞定：

```bash
cd ~/Documents/Obsidian\ Vault
git init
git remote add origin git@github.com:你的用户名/obsidian-vault.git
```

然后配一个 `.gitignore`，把不需要同步的文件排除掉：

```
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/plugins/obsidian-git/data.json
.trash/
.DS_Store
```

首次提交并推送：

```bash
git add -A
git commit -m "init: obsidian vault"
git push -u origin main
```

就这样，你的笔记库已经是一个 Git 仓库了。

### 第二步：装 obsidian-git 插件

手动 `git push` 太麻烦了，我们需要自动化。obsidian-git 插件（作者 Vinzent03）就是干这个的。

安装方式：Obsidian 设置 → 第三方插件 → 浏览社区插件 → 搜索 `Git` → 安装启用。

装好之后，重点配这几项：

| 配置项 | 推荐值 | 说明 |
|-------|--------|------|
| Auto commit-and-sync interval | `10` 分钟 | 每 10 分钟自动提交并同步 |
| Auto commit-and-sync after stopping file edits | 开启 | 停止编辑后才触发，不会打断你写作 |
| Pull on startup | 开启 | 打开 Obsidian 时自动拉取最新内容 |
| Push on commit-and-sync | 开启 | 提交后自动推送到 GitHub |
| Pull on commit-and-sync | 开启 | 提交前先拉取，减少冲突 |
| Disable informative notifications | 开启 | 关掉频繁的提示，看状态栏就够了 |
| Hide notifications for no changes | 开启 | 没变化时不弹通知 |

其他配置保持默认就好。

配完之后，你的笔记会每 10 分钟自动备份到 GitHub。打开 Obsidian 时自动拉取最新内容，关掉时自动推送。整个过程无感知。

### 第三步：图片怎么办

笔记里肯定会插图片，这是很多人担心的点：图片会不会把 Git 仓库撑爆？

实际上，GitHub 对仓库的限制是：单个文件 < 100MB，仓库总大小建议 < 1GB。对于普通笔记来说，完全够用。

但为了保险，建议装一个 Image Converter 插件。它能在你粘贴图片时自动压缩、转 WebP 格式，体积能小好几倍。

如果你的笔记图片特别多（比如每天几十张截图），可以考虑把附件目录加到 `.gitignore` 里，图片单独用 iCloud 或其他方式同步。但大多数人不需要这么做。

## 搭好之后，AI 怎么接入

基础设施搭好了，接下来才是重头戏：让 AI 来"编译"你的知识库。

你可能以为需要装什么 API 插件、搭什么服务。其实不用。Obsidian vault 就是一个普通文件夹，AI 工具（比如 Claude Code）可以直接读写里面的 Markdown 文件。不需要中间层，不需要额外依赖。

这意味着你可以让 AI 做这些事：

- 读取你最近剪藏的文章，生成结构化摘要
- 扫描笔记库，提取关键概念，自动建索引
- 回答基于你笔记内容的问题，并把 Q&A 结果存下来
- 定期做"健康检查"：找出定义冲突、孤立笔记、缺失链接

这就是 Yanhua 推文里说的三层结构：`raw/` 放原始资料，`wiki/` 放编译产物，`outputs/` 放运行时输出。AI 负责把 raw 编译成 wiki，你负责往 raw 里喂素材。

而 Git 在这里扮演的角色是**安全网**。AI 改了什么，`git diff` 一看就知道。改错了，`git revert` 回滚。心里有底，才敢放手让 AI 干活。

还有一点很重要：**别上来就搞 RAG。** 知识库规模不大的时候（几百篇笔记），维护几个索引文件就够了。AI 先读索引定位，再直接阅读相关内容。简单、可靠、零成本。等笔记真的过了一万条，再考虑向量检索不迟。

## 我的目录结构

分享一下我当前的 vault 结构，供参考：

```
Obsidian Vault/
├── 工作/          # 工作相关笔记
├── 开发/          # 技术笔记
├── 工具/          # 工具使用记录
├── 生活/          # 生活记录
├── 随思/          # 随手记录的想法
├── 附件/          # 统一存放图片等附件
└── .gitignore     # Git 忽略配置
```

这是一个比较朴素的结构，按生活和工作的维度来分。后续我会逐步引入 Karpathy 的 `raw/wiki/outputs` 三层思路，让 AI 能更好地参与进来。

知识管理是个渐进的过程，不用一步到位。先把版本化跑通，再慢慢优化结构。

## 最后

搭基础设施是第一步。先让笔记"版本化"，有了 Git 的保障，后面不管是接 AI、做自动化、还是多端同步，都有了稳固的地基。

Yanhua 在推文里说得好：**两周跑通最小闭环。** 建三个目录，装好插件，开始往里面喂素材。不需要完美的体系，先跑起来再说。

下一期，我会聊另一个有意思的话题：这篇文章本身就是用 Markdown 写的，那怎么发到公众号？有一个叫 **md2wechat** 的命令行工具，可以把 Markdown 一键转成微信公众号格式，甚至直接创建草稿。从写作到发布，全程不离开终端。敬请期待。

---

> 参考资料：
> - Yanhua (@yanhua1010)：[用 LLM + Obsidian 构建个人知识库](https://x.com/yanhua1010/status/2039966047378583815)
> - obsidian-git 插件：[GitHub - Vinzent03/obsidian-git](https://github.com/Vinzent03/obsidian-git)
