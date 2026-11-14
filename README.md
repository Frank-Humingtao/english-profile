# 英语词汇量报告

一位中国成年英语学习者的英语词汇量报告，每次测试后自动重算并更新。本仓库的文件由程序生成，请勿手改。

## 当前快照

- 词族：约 **3,814** 个（2,765–4,863），括号内为 95% 区间
  - 前 14k 词族（BNC/COCA 1–14 级）**3,276** 个（2,654–3,898），14k 以外 **538** 个（84–992）
- lemma：约 **7,806** 个（6,069–9,543）
- 各频段掌握率：1k 96% · 2k 64% · 3k 41% · 4k 27% · 5k 20%
- 最近测试日期：2026-11-14

```mermaid
xychart-beta
  title "词族数"
  x-axis ["11-13", "11-14"]
  y-axis "词族" 0 --> 4885
  line [3832, 3814]
```

最新的 Progress Report（简短摘要）：[打开且带 `progress-report` label 的 issue](https://github.com/Frank-Humingtao/english-profile/issues?q=is%3Aissue+is%3Aopen+label%3Aprogress-report)，始终只有一份。

## 参数说明

所有区间都是 **95% 区间**。`p` 是"认识这个词的意思"的概率，已扣除猜测（见测量方法）。

### `summary.json`

| 键 | 含义 |
| --- | --- |
| `schema_version` | 格式版本，当前为 3 |
| `updated_at` | 最近一次测试的日期（YYYY-MM-DD） |
| `lexicon_version` | 所用词表的版本；变化时逐词数据可能整体移动 |
| `vocab_size.families` | 认识的词族数：`{estimate, low, high}`，即估计值和 95% 区间。一个词族的 `p` 取其成员 lemma 中最大的 `p`，再对全部 25,000 个词族求和 |
| `vocab_size.lemmas` | 认识的 lemma 数：全部 43,028 个 lemma 的 `p` 之和，格式同上 |
| `vocab_size.core` | 只算 BNC/COCA 1–14 级（最常用的 14,000 个词族）：`families`、`lemmas` 两项，格式同上 |
| `vocab_size.tail` | 只算 15–25 级（14k 以外的长尾），格式同上；主要靠外推，区间宽 |
| `band_counts` | 三类词（Word Band）各有多少个 lemma：`mastered`、`target`、`distant` |
| `level_mastery` | 每个 BNC/COCA 千词级一项：`level`（第几个 1000 词族）、`families`（该级词族数）、`mastery`（该级词族 `p` 的平均值，即该级认识比例的估计，0–1） |

三类词（按 lemma 划分）：

- **Mastered**：`p` ≥ 0.9，已经可靠地认识；
- **Target**：`p` < 0.9，且词频排名 ≤ 估计的 lemma 数 + 2000，处在已知范围的边缘；
- **Distant**：其余，目前离已知范围较远。

### `words.tsv`

每个 lemma 一行，按词频排名排序，制表符分隔：

| 列 | 含义 |
| --- | --- |
| `lemma` | 词的基本形式（如 *run* 代表 runs、ran、running） |
| `band` | 三类词之一：`Mastered` / `Target` / `Distant` |
| `p` | 认识这个词意思的概率，0–1 |
| `se` | `p` 的标准误。测过的词小；没测过、靠模型外推的词大 |
| `rank` | 词频排名，1 为最常用 |

### 其他文件

- `history.json`：每天一条 `{date, families, low, high}`，即当天的词族数及 95% 区间（上面的曲线）。
- `recent.json`：`newly_mastered` 是近 7 天新进入 Mastered 的词 `{lemma, rank, date}`，`date` 为发现的日期。

### 版本与读取地址

- `schema_version` 在文件格式不兼容地变化时加一（3：`words.tsv` 增加 `se`，`vocab_size` 增加 `core` / `tail`）。
- `lexicon_version` 标识词表；同一版本下的 `rank` 和词族划分不变。
- 固定地址（始终指向最新版本）：
  - `https://raw.githubusercontent.com/Frank-Humingtao/english-profile/main/summary.json`
  - `https://raw.githubusercontent.com/Frank-Humingtao/english-profile/main/words.tsv`
  - `https://raw.githubusercontent.com/Frank-Humingtao/english-profile/main/history.json`
  - `https://raw.githubusercontent.com/Frank-Humingtao/english-profile/main/recent.json`

## 测量方法

这里的做法不是常见的标准化测试，读数字前需要知道它怎么来的：

- **题型**：Yes/No 自报。屏幕上出现一个词，回答认识或不认识。题目中混入约 15% 的伪词（拼写像英语、但不存在的词）。答"认识"的真词，约 10% 会当场追问一道中文释义四选一，带"不认识"选项。
- **选题**：按词频段分层抽样，覆盖到 25 级的长尾，并多选当前最不确定的词。多次测试的证据累积起来一起计算，越测区间越窄；旧证据随时间慢慢降权，异常的场次（伪词误认或释义答错过多）降权。
- **模型（两层）**：
  - 知识层：每个词有一个难度，由词频排名（取对数）和考纲级别（中考、高考、四级、六级、考研、托福/雅思、GRE 中最早收录的一级）回归得出，并允许单个词偏离回归值；学习者有一个能力值。两者之差给出认识的概率。
  - 作答层：单独刻画"认识却答不认识"（漏答）、"不认识却答认识"（误认，由伪词估计，真词略高）和四选一的猜中。
  - 所以 `p` 是"认识"本身的概率，猜中和误认已扣除。测过的词，`p` 结合了它自己的作答；没测过的词，`p` 来自回归。
- **`se` 与区间**：由拟合参数的不确定性传播得到。测过的词 `se` 小；远离测过区域、靠外推的词 `se` 大。总数的区间还包含"哪些词恰好认识"的随机性。
- **词表**：BNC/COCA 词族表（Paul Nation，25 级、每级 1000 个词族），加上从词族成员到 lemma 的映射，共 43,028 个 lemma。

## 和常见测试怎么对照

下面只给大致方向，不能精确换算。

- **计数口径差别很大**：同一个人，词族数 < lemma 数 < 单词形式数。本报告里 lemma 数约为词族数的 2.0 倍；把屈折变化和派生词都算作不同的词，数字还会更大。比较前先确认对方用的是哪种口径。
- **词族口径偏宽**：一个词族里只要认识任一成员就算认识整个词族，所以词族数会高于"整个词族都会"的真实数量。
- **VST**（Vocabulary Size Test，四选一，按词族计，测到 14k 或 20k）：四选一会因猜测而高估，文献中回忆水平约 2,000 词的学习者可被估到 9,000 以上。本报告扣除了猜测、只算认识意思，所以同一个人的 `vocab_size.families` 通常**低于** VST 分数；与 14k 版对照时用 `vocab_size.core.families`。
- **LexTALE**（60 项 Yes/No，含伪词）：给出的是正确率百分比，用来粗分总体水平，不是词汇量，无法与本报告的数字换算；它对中国学习者的效度也偏弱。
- **testyourvocab**（自报勾选，无核验，按词典词条计）：口径接近 lemma 或更宽，应对照 `vocab_size.lemmas`。它不校正虚报，本报告用伪词和释义核验校正，所以本报告的数字通常**更低**。
- **14k 以外**（`vocab_size.tail`）直接测到的词很少，主要靠外推，误差大；跨测试比较时以 `core` 为主。

依据见调研：[自适应词汇测试](https://github.com/Frank-Humingtao/learn-english/blob/main/docs/research/02-adaptive-vocab-test.md)、[准确的词汇量测量](https://github.com/Frank-Humingtao/learn-english/blob/main/docs/research/07-accurate-vocab-measurement.md)。

## 隐私

公开数据只有日期，没有作答时间，也没有原始作答记录。

## 数据来源与许可

本仓库的派生数据以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 发布。词表与词频排名来自 BNC/COCA word family lists（Paul Nation）和 ECDICT（MIT）。
