---
{"dg-publish":true,"dg-path":"share/obsidian/笔记还在吃灰？用AI+Git管理Obsidian.md","permalink":"/share/obsidian/笔记还在吃灰？用AI+Git管理Obsidian/","dg-note-properties":{}}
---

# 笔记还在吃灰？用AI+Git管理Obsidian

Hi，这里是奇妙堡。最近折腾 Obsidian 折腾出了点心得，来跟大家聊聊。

你是不是也这样：收藏夹存了几百篇文章，笔记软件里记了一堆东西，过仨月回头看，大部分都成了"死文件"。找不到在哪，也想不起来写了啥。

我就是。直到前阵子看到 Karpathy 说了句话，给我整明白了。

他说：**把知识库当代码仓库管，LLM 当编译器。**

Twitter 上 @yanhua1010 根据这个思路写了篇文章，把落地方案捋了一遍，传播挺广（402K 浏览）。他用了张表做类比：

| 软件工程 | 知识库工程 |
|---------|-----------| 
| `src/` | `raw/`（原始资料） |
| `build/` | `wiki/`（知识条目） |
| `logs/` | `outputs/`（问答归档） |
| 编译器 | LLM |
| IDE | Obsidian |
| Lint / CI | 健康检查 |

我看完就想：道理是这个道理，但地基呢？写代码之前得先建 Git 仓库，搞知识库之前，也得先把"仓库"搭起来。

所以这篇不聊理念（Yanhua 那篇讲得够清楚了），就说一件事：**怎么把 Obsidian 变成一个真的能用 Git 管的仓库。**

## 为什么选 Obsidian + Git

Obsidian 的好处很简单：笔记就是 Markdown 文件，存在本地。不锁定、不加密、不靠云服务。你哪天不想用了，文件还是你的。后面要接 AI，AI 能直接读写这些文件，这点很关键。

Git 的好处也直接。既然要"当代码仓库管"，那就用代码仓库那套工具：

- 版本历史：每次改动都有记录，知识怎么长出来的，翻翻 log 就知道
- 后悔药：AI 帮你改笔记改砸了？`git revert` 一下就回去了
- 免费同步：GitHub 私有仓库，不花钱，多设备都能用

你可能想问：Obsidian 自己不是有同步方案吗？

有。但都有毛病。Obsidian Sync 收费（$8/月）；iCloud 同步老冲突，丢过文件的人知道那种心情；坚果云 + Remotely Save 能凑合用，但多一层东西就多一层出问题的可能。

Git 是程序员用了几十年的工具，靠不靠谱不用讨论。而且它本来就是干"管理文件变更"这活儿的，跟知识库的需求天然对得上。

## 实操：从零搭建

### 第一步：初始化仓库

打开终端，进你的 Obsidian vault 目录，三条命令：

```bash
cd ~/Documents/Obsidian\ Vault
git init
git remote add origin git@github.com:你的用户名/obsidian-vault.git
```

配个 `.gitignore`，排除不该同步的文件：

```
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/plugins/obsidian-git/data.json
.trash/
.DS_Store
```

首次提交推送：

```bash
git add -A
git commit -m "init: obsidian vault"
git push -u origin main
```

完事了。你的笔记库现在是个 Git 仓库了。

### 第二步：装 obsidian-git 插件

每次手动 `git push` 肯定不现实，得自动化。obsidian-git 插件（作者 Vinzent03）就干这个的。

装法：Obsidian 设置 → 第三方插件 → 社区插件 → 搜 `Git` → 装上启用。

重点配这几个：

| 配置项 | 推荐值 | 说明 |
|-------|--------|------|
| Auto commit-and-sync interval | `10` 分钟 | 每 10 分钟自动提交同步 |
| Auto commit-and-sync after stopping file edits | 开启 | 你停下来才触发，不打断写作 |
| Pull on startup | 开启 | 打开 Obsidian 时拉最新内容 |
| Push on commit-and-sync | 开启 | 提交完自动推到 GitHub |
| Pull on commit-and-sync | 开启 | 提交前先拉一下，少冲突 |
| Disable informative notifications | 开启 | 关掉弹窗，看状态栏就行 |
| Hide notifications for no changes | 开启 | 没变化别烦我 |

其他默认就好。

配完之后效果：每 10 分钟自动备份到 GitHub，开 Obsidian 自动拉取，关了自动推送。你基本感觉不到它在跑。

### 第三步：图片怎么处理

笔记里肯定有图片。很多人担心图片把 Git 仓库撑爆。

说实话，GitHub 的限制是：单文件 < 100MB，仓库建议 < 1GB。普通笔记够用了。

保险起见可以装个 Image Converter 插件，粘贴图片时自动压缩转 WebP，体积能小好几倍。

图片特别多的话（比如每天截几十张图），可以把附件目录塞进 `.gitignore`，图片走 iCloud 或别的方式同步。不过大多数人用不上这招。

## 搭好了，AI 怎么接进来

地基打好了，接下来是正事：让 AI "编译"你的知识库。

你可能觉得得装 API 插件、搭什么服务。不用。Obsidian vault 就是个普通文件夹，AI 工具（比如 Claude Code）直接读写里面的 Markdown 就行。没有中间层。

所以你可以让 AI：

- 读你剪藏的文章，整理成结构化摘要
- 扫一遍笔记库，提取概念，建索引
- 回答基于你笔记的问题，把 Q&A 存下来
- 做"健康检查"：找定义冲突、孤立笔记、断链

这就是 Yanhua 说的三层结构：`raw/` 放原始素材，`wiki/` 放整理后的内容，`outputs/` 放问答记录。AI 负责把 raw 加工成 wiki，你负责往 raw 里丢东西。

Git 在这里的作用是后悔药。AI 动了什么，`git diff` 看一眼。改错了，`git revert`。有这个兜底，才敢放心让 AI 动手。

还有一点：别一上来就搞 RAG。笔记不多的时候（几百篇），维护几个索引文件就够了。AI 先看索引找方向，再去读具体内容。简单、靠谱、不花钱。等笔记真过了一万条，再上向量检索也不迟。

## 我的目录结构

贴一下我现在的 vault 结构：

```
Obsidian Vault/
├── 工作/          # 工作相关笔记
├── 开发/          # 技术笔记
├── 工具/          # 工具使用记录
├── 生活/          # 生活记录
├── 随思/          # 随手记的想法
├── 附件/          # 图片等附件统一放这
└── .gitignore     # Git 忽略配置
```

结构挺朴素的，按工作和生活分。后面打算慢慢往 Karpathy 的 `raw/wiki/outputs` 三层结构靠，让 AI 更好介入。

不用一步到位。先把版本化跑通，结构慢慢调。

## 最后

先把笔记"版本化"。有了 Git 托底，后面接 AI 也好、做自动化也好、多端同步也好，都有根基了。

Yanhua 说得对：**两周跑通最小闭环。** 建仨目录，装好插件，开始往里扔东西。别等体系完美了再动手，先跑起来。

下一期聊个相关的事：这篇文章本身就是 Markdown 写的，怎么发公众号？有个叫 md2wechat 的命令行工具，能把 Markdown 转成微信公众号格式，还能直接建草稿。写完到发出去，不用离开终端。

---

> 参考资料：
> - Yanhua (@yanhua1010)：[用 LLM + Obsidian 构建个人知识库](https://x.com/yanhua1010/status/2039966047378583815)
> - obsidian-git 插件：[GitHub - Vinzent03/obsidian-git](https://github.com/Vinzent03/obsidian-git)
