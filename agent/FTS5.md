# 通俗解释

想象你有一个巨大的图书馆，里面有 10 万本书。如果你要找一本「讲 Python 编程的书」，传统的方式是一本一本地翻看——这显然太慢了。

于是你做了一个「目录卡片」系统：每本书你都在卡片上记录书名、作者、关键词。所有卡片按字母顺序排列在一个巨大的抽屉柜里。当你要找 Python 的书时，直接去 "P" 开头的抽屉里找就行了。这就是「索引」。

但 FTS5 比这个更强大。它不仅记录「这本书里有 Python 这个词」，还记录「Python 这个词在书的第 3 页第 2 段出现了」。更进一步，它能理解「Python 编程」「Python 开发」「Python 语言」都是相关的，甚至能处理拼写错误（比如你把 Python 拼成了 Pthon）。

在 Hermes 的语境中，FTS5 做的事情是：

1. 把每一条历史消息、每一个工具调用结果、每一个文件操作记录，都「拆开」成单词
2. 建立一个巨大的「倒排索引」——记录每个词出现在哪些记录中
3. 当你搜索时，瞬间找到包含这些词的所有记录
4. 按照相关度排序，把最相关的结果返回给你

# 索引的建立过程

```
CREATE VIRTUAL TABLE messages_fts USING fts5(
    content,                    -- 要索引的文本内容
    conversation_id UNINDEXED,  -- 不索引，仅存储关联ID
    timestamp UNINDEXED         -- 不索引，仅存储
);

-- 当新消息写入时，自动添加到索引（触发器自动完成）
INSERT INTO messages_fts(content, conversation_id, timestamp)
VALUES ('用户输入的内容...', 123, '2026-04-21 10:30:00');
```

索引建立的过程：

1.  **分词** ：将文本拆分成单词。对于中文，需要使用专门的分词器 （如jieba或SQLite的icu分词器）。Hermes默认使用porter分词器处理英文， simple分词器处理中文。
2. **标准化（Normalization）** 
    - 统一大小写
    - 去除标点符号
    - 词干提取（running和runs都归为run）
  3. **去停用词：** 过滤掉【的】【是】【the】【and】等无意义的词，减少索引体积
  4. **构建倒排索引** ：记录每个词在哪些文档中出现，以及出现的位置

## 检索速度与效果

FTS5的检索速度非常快。在普通笔记本电脑上，从100万条记录中检索关键词，通常只需要几毫秒到几十毫秒。

实际的检索场景例子：
假设你在半年前的一个会话中讨论过「如何用 Docker 部署 PostgreSQL」。现在你忘了具体的命令，只记得「Docker」和「PostgreSQL」两个关键词。你开启新会话，输入：
```Shell
User: 上次部署 PostgreSQL 的 Docker 命令是什么来着？
```

Hermes的内部检索过程：

```SQL
-- 1. 提取关键词
-- 分析输入："部署 PostgreSQL 的 Docker 命令"
-- 提取关键词：PostgreSQL, Docker, 部署, 命令
-- 过滤停用词：保留 PostgreSQL, Docker, 命令

-- 2. FTS5 检索
SELECT 
    content,
    conversation_id,
    rank  -- FTS5 内置的相关度评分
FROM messages_fts
WHERE messages_fts MATCH 'PostgreSQL AND Docker AND 命令'
ORDER BY rank
LIMIT 5;

```
