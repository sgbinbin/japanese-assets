# 日语词条 JSON Schema 设计文档

> 设计目标：一个 Schema 支撑 N5~N1 全级别，服务 10+ 种下游产品。
> 设计原则：宁可字段多留空，不要将来推倒重来。

---

## 一、Schema 总览

```
vocab_entry
├── meta          基础信息（必填）
├── jlpt          考试相关（必填）
├── meaning       释义（必填）
├── nuance        语感（核心价值）
├── examples      例句（必填）
├── usage         用法（核心）
├── memory        记忆系统（核心）
├── pitfalls      易错点（核心）
├── culture       文化背景（重要）
├── relations     关联词（重要）
├── ai            AI 扩展（后期）
└── stats         学习数据（后期）
```

---

## 二、字段定义

### 2.1 meta — 基础信息

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | ✅ | 唯一标识，格式 `n2_00001`，用于全系统关联 |
| `word` | string | ✅ | 单词汉字/假名，如「承認」「きれい」 |
| `kana` | string | ✅ | 平假名读音 |
| `romaji` | string | ❌ | 罗马字，给初学者和输入法场景 |
| `word_type` | enum | ✅ | `独立詞` / `接続詞` / `接頭辞` / `接尾辞` / `カタカナ語` |
| `origin` | string | ❌ | 词源，如「承認 ← 漢語」「アルバイト ← ドイツ語」 |
| `kanji_parts` | array | ❌ | 拆字，如「承」→「承る（うけたまわる）」的古义关联 |
| `audio` | object | ❌ | `{ "path": "...", "speaker": "male/female" }` |
| `created_at` | string | ❌ | ISO 8601 |
| `updated_at` | string | ❌ | ISO 8601 |

**为什么需要 `id`：** Anki 卡片、错题本、复习记录都要关联到词。没有 id 就没法做跨系统追踪。

---

### 2.2 jlpt — 考试信息

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `level` | enum | ✅ | `N5` / `N4` / `N3` / `N2` / `N1` |
| `section` | array | ✅ | `["文字・語彙"]` 或 `["文法"]` 或 `["聴解"]`，可多选 |
| `frequency_rank` | int | ❌ | 同级别内的频率排名，1=最高频 |
| `exam_appearances` | array | ❌ | `[{"year": 2023, "month": 7, "section": "語彙", "qnum": 12}]` |
| `trend` | enum | ❌ | `increasing` / `stable` / `decreasing` — 近年出题趋势 |
| `related_grammar` | array | ❌ | 关联语法点 ID，如 `["n2_gram_023"]` |

**为什么需要 `exam_appearances`：** 后续做「真题刷题系统」时，能显示「这个词在 2023 年 7 月出过」，大幅提升学习动力。

---

### 2.3 meaning — 释义

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `meanings` | array | ✅ | 见下方子结构 |
| `meanings[].pos` | string | ✅ | 该义项的词性：「名词」「他动词」「形容动词」等 |
| `meanings[].cn` | string | ✅ | 自然中文，不是字典翻译 |
| `meanings[].en` | string | ❌ | 英文释义，给多语言扩展 |
| `meanings[].priority` | int | ❌ | 义项优先级，1=最常用。用于 Anki 默认显示 |
| `meanings[].register` | string | ❌ | `丁寧` / `普通` / `くだけた` / `改まった` / `俗語` |
| `meanings[].figurative` | boolean | ❌ | 是否为引申义 |

**为什么多义项要拆开：** 「掛かる」有「花费」「挂」「发生」等义，合并在一起会让学习者混乱。拆开后，AI 出题可以针对单个义项出题。

---

### 2.4 nuance — 语感（核心价值字段）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `feeling` | string | ✅ | 一段话描述「这个词给人的感觉」 |
| `formality` | int | ✅ | 1-5，1=最 casual，5=最正式 |
| `emotional_charge` | enum | ✅ | `ポジティブ` / `ネガティブ` / `中立` / `文脈依存` |
| `spoken_vs_written` | string | ❌ | 口语/书面语倾向说明 |
| `softness` | int | ❌ | 1-5，委婉程度。5=非常委婉 |
| `strength` | int | ❌ | 1-5，语气强弱。5=非常强烈 |
| `similar_nuance` | array | ❌ | 语感相近的词，附区别说明 |
| `similar_nuance[].word` | string | ❌ | |
| `similar_nuance[].diff` | string | ❌ | 一句话说清区别 |
| `anti_nuance` | array | ❌ | 语感相反的词 |

**为什么需要语感字段：** 这是传统词典和 AI 产品的最大差距。中国人会「承認」和「認める」都翻译成「承认」，但语感完全不同。这个字段是护城河。

**`feeling` 示例：**
> 「根回し」：不是走后门，是日本职场的润滑剂。想象移植大树前先绕着根松土——正式决定之前，私下把各方意见对齐。不做根回し直接上会，在日本人眼里等于「不会读空气」。

---

### 2.5 examples — 例句

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `sentences` | array | ✅ | 见下方子结构 |
| `sentences[].jp` | string | ✅ | 日语例句 |
| `sentences[].cn` | string | ✅ | 自然中文翻译 |
| `sentences[].audio` | string | ❌ | 音频路径 |
| `sentences[].level` | enum | ❌ | `初級` / `中級` / `上級` — 例句难度 |
| `sentences[].scene` | string | ❌ | 使用场景描述，如「上司对部下说」 |
| `sentences[].style` | enum | ❌ | `会話文` / `書き言葉` / `ニュース` / `メール` |
| `sentences[].blank` | string | ❌ | 挖空版，如「___を確認してください」用于出题 |
| `sentences[].grammar_points` | array | ❌ | 该句包含的语法点 ID |

**为什么每个例句要带场景：** 「確認してください」对上司说是命令，加个「お」变成「ご確認ください」就变成了敬语。场景决定了语感。

---

### 2.6 usage — 用法

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `collocations` | array | ✅ | 常用搭配，如「影響を与える」「確認を取る」 |
| `collocations[].pattern` | string | ✅ | 搭配形式 |
| `collocations[].meaning_cn` | string | ✅ | 搭配意思 |
| `collocations[].frequency` | enum | ❌ | `高` / `中` / `低` |
| `sentence_patterns` | array | ❌ | 句型模板，如「〜に影響を与える」「〜を確認する」 |
| `sentence_patterns[].pattern` | string | ❌ | |
| `sentence_patterns[].meaning_cn` | string | ❌ | |
| `particles` | array | ❌ | 搭配助词，如 `["を","に","が"]` |
| `conjugal_forms` | object | ❌ | 动词/形容词活用形（动词必填） |
| `conjugal_forms.masu` | string | ❌ | ます形 |
| `conjugal_forms.te` | string | ❌ | て形 |
| `conjugal_forms.nai` | string | ❌ | ない形 |
| `conjugal_forms.ta` | string | ❌ | た形 |
| `conjugal_forms.potential` | string | ❌ | 可能形 |
| `conjugal_forms.passive` | string | ❌ | 被动形 |
| `conjugal_forms.causative` | string | ❌ | 使役形 |

**为什么 `collocations` 是必填：** 中国人背单词只记单个词，但日本人说话是按搭配来的。「影響」单独记没用，得记「影響を与える」才能真正会用。

---

### 2.7 memory — 记忆系统

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `chinese_cognate` | object | ❌ | 中文关联分析 |
| `chinese_cognate.similar` | boolean | ❌ | 是否和中文意思相同 |
| `chinese_cognate.diff` | string | ❌ | 和中文的区别（中国人专属字段） |
| `chinese_cognate.trap` | string | ❌ | 中文直觉会踩的坑 |
| `mnemonic` | string | ✅ | 记忆法（必须有帮助，不要废话） |
| `mnemonic_type` | enum | ❌ | `谐音` / `画面` / `故事` / `对比` / `词源` / `拆字` |
| `visual_image` | string | ❌ | 画面感描述，用于记忆宫殿 |
| `word_family` | array | ❌ | 同根词，如「承認」→「認める」「認識」「認可」 |
| `word_family[].word` | string | ❌ | |
| `word_family[].relation` | string | ❌ | 关系描述 |
| `etymology` | string | ❌ | 词源故事，给高阶学习者 |
| `similar_sounding` | array | ❌ | 发音易混词，如「承認(しょうにん)」和「証人(しょうにん)」 |

**为什么需要 `chinese_cognate`：** 中国人学日语最大的优势（汉字）也是最大的陷阱。「手紙」中文是 toilet paper，日语是 letter。这个字段让 AI 能专门针对中国人的弱点出题。

---

### 2.8 pitfalls — 易错点

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `common_mistakes` | array | ✅ | 中国人常犯的错误 |
| `common_mistakes[].type` | enum | ✅ | `读音` / `意思` / `用法` / `搭配` / `语感` / `助词` / `活用` |
| `common_mistakes[].wrong` | string | ✅ | 错误示例 |
| `common_mistakes[].correct` | string | ✅ | 正确示例 |
| `common_mistakes[].explanation` | string | ✅ | 为什么错 |
| `confused_with` | array | ❌ | 易混词对比 |
| `confused_with[].word` | string | ❌ | |
| `confused_with[].diff` | string | ❌ | 一句话说清区别 |
| `confused_with[].test_tip` | string | ❌ | 考试时怎么区分 |
| `reading_traps` | array | ❌ | 容易读错的读音 |
| `reading_traps[].wrong` | string | ❌ | |
| `reading_traps[].correct` | string | ❌ | |

**为什么 `common_mistakes` 要分类：** 后续 AI 错题分析系统可以统计「这个学生读音类错误最多」，然后自动推送读音专项训练。

---

### 2.9 culture — 文化背景

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `cultural_note` | string | ❌ | 文化背景说明 |
| `social_context` | string | ❌ | 社会使用场景，如「对长辈不能用」 |
| `workplace_usage` | string | ❌ | 职场使用要点 |
| `gender_tendency` | enum | ❌ | `男性多用` / `女性多用` / `中性` / `儿童用语` |
| `age_tendency` | enum | ❌ | `年配` / `若者` / `全年齢` |
| `regional` | string | ❌ | 方言/地域倾向，如「关西常用」 |
| `keigo_relation` | object | ❌ | 敬语关联 |
| `keigo_relation.humble` | string | ❌ | 谦让语形式 |
| `keigo_relation.honorific` | string | ❌ | 尊敬语形式 |
| `keigo_relation.polite` | string | ❌ | 丁寧语形式 |

**为什么需要 `keigo_relation`：** 日本职场日语是刚需。「言う」的谦让语是「申す」、尊敬语是「おっしゃる」——这个信息对职场学习者至关重要。

---

### 2.10 relations — 关联词

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `synonyms` | array | ❌ | 同义词（附区别） |
| `synonyms[].word` | string | ❌ | |
| `synonyms[].diff` | string | ❌ | 区别说明 |
| `antonyms` | array | ❌ | 反义词 |
| `antonyms[].word` | string | ❌ | |
| `compound` | array | ❌ | 包含该词的复合词 |
| `derivative` | array | ❌ | 派生词 |
| `same_kanji_diff_reading` | array | ❌ | 同汉字不同读法（中国人的痛点） |

**为什么需要 `same_kanji_diff_reading`：** 「人」可以读「じん」「にん」「ひと」——这个字段让 AI 能做专门的读音辨析训练。

---

### 2.11 ai — AI 扩展（后期实现）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `difficulty_score` | float | ❌ | 0.0-1.0，AI 计算的综合难度 |
| `confusion_cluster` | string | ❌ | 易混词分组 ID，如 `cluster_023` |
| `teaching_script` | string | ❌ | AI 语感教学的脚本模板 |
| `quiz_templates` | array | ❌ | 自动生成题目的模板 |
| `quiz_templates[].type` | enum | ❌ | `选择` / `填空` / `排序` / `听力` / `造句` |
| `quiz_templates[].template` | string | ❌ | 题目模板，含 `{word}` `{kana}` 等占位符 |
| `quiz_templates[].distractors` | array | ❌ | 干扰项生成规则 |
| `review_strategy` | string | ❌ | 复习策略建议，如「和根回し一起复习」 |

**为什么需要 `quiz_templates`：** 100 个词可以自动变出 500+ 道题。不用手写出题，AI 按模板批量生成。

---

### 2.12 stats — 学习数据（后期实现）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `global_stats` | object | ❌ | 全局统计 |
| `global_stats.total_learners` | int | ❌ | 学过这个词的人数 |
| `global_stats.avg_wrong_rate` | float | ❌ | 平均错误率 |
| `global_stats.hard_vote` | int | ❌ | 被标记为「难」的次数 |
| `per_user` | object | ❌ | 单用户数据（存在用户端，不存词库） |
| `per_user.correct_count` | int | ❌ | |
| `per_user.wrong_count` | int | ❌ | |
| `per_user.last_review` | string | ❌ | |
| `per_user.next_review` | string | ❌ | Anki SRS 下次复习时间 |
| `per_user.ease_factor` | float | ❌ | Anki 难度系数 |

---

## 三、字段优先级

### 🔴 P0 — 核心字段（必须有值，否则词库不可用）

```
meta.id, meta.word, meta.kana
jlpt.level, jlpt.section
meaning.meanings[].cn, meaning.meanings[].pos
nuance.feeling, nuance.formality, nuance.emotional_charge
examples.sentences[至少2条].jp, .cn
usage.collocations[至少1条]
memory.mnemonic
pitfalls.common_mistakes[至少1条]
```

### 🟡 P1 — 重要字段（有值质量大幅提升）

```
nuance.similar_nuance, nuance.spoken_vs_written
examples.sentences[].scene, .style, .blank
usage.sentence_patterns, usage.conjugal_forms
memory.chinese_cognate, memory.visual_image
pitfalls.confused_with, pitfalls.reading_traps
culture.cultural_note, culture.keigo_relation
relations.synonyms, relations.antonyms
```

### 🟢 P2 — 扩展字段（后期再做）

```
meta.romaji, meta.origin, meta.kanji_parts, meta.audio
jlpt.frequency_rank, jlpt.exam_appearances, jlpt.trend
meaning.meanings[].en, .register, .figurative
ai.*
stats.*
```

---

## 四、完整 JSON 示例

```json
{
  "meta": {
    "id": "n2_00001",
    "word": "根回し",
    "kana": "ねまわし",
    "romaji": "nemawashi",
    "word_type": "独立詞",
    "origin": "和語 — 移植树木前处理根部",
    "kanji_parts": null,
    "audio": null,
    "created_at": "2026-05-24T00:00:00Z",
    "updated_at": "2026-05-24T00:00:00Z"
  },

  "jlpt": {
    "level": "N2",
    "section": ["文字・語彙"],
    "frequency_rank": 45,
    "exam_appearances": [
      {"year": 2019, "month": 12, "section": "語彙", "qnum": 8}
    ],
    "trend": "stable",
    "related_grammar": []
  },

  "meaning": {
    "meanings": [
      {
        "pos": "名词",
        "cn": "提前打招呼，私下铺垫",
        "en": "informal prior consultation",
        "priority": 1,
        "register": "普通",
        "figurative": true
      },
      {
        "pos": "サ变动词",
        "cn": "提前沟通，事前疏通",
        "en": "to lay groundwork",
        "priority": 2,
        "register": "普通",
        "figurative": true
      }
    ]
  },

  "nuance": {
    "feeling": "日本职场的核心生存技能。字面是「绕着树根转」——移植树木前先处理根部。不是贬义的走后门，而是正式场合前私下达成共识。日本人开会基本走形式，真正的决定在根回し阶段就做完了。",
    "formality": 3,
    "emotional_charge": "中立",
    "spoken_vs_written": "口语书面都常用，职场高频词",
    "softness": 3,
    "strength": 2,
    "similar_nuance": [
      {
        "word": "事前調整",
        "diff": "事前調整更正式，根回し更口语化、更有日本味"
      }
    ],
    "anti_nuance": [
      {
        "word": "ぶっつけ本番",
        "diff": "不做任何准备直接上，根回し的反面"
      }
    ]
  },

  "examples": {
    "sentences": [
      {
        "jp": "会議の前に根回ししておいたから、大丈夫だよ。",
        "cn": "开会之前我提前打过招呼了，没问题的。",
        "audio": null,
        "level": "中級",
        "scene": "同事之间，轻松语气",
        "style": "会話文",
        "blank": "会議の前に___しておいたから、大丈夫だよ。",
        "grammar_points": ["n3_gram_teoku"]
      },
      {
        "jp": "彼、いきなり反対意見言うけど、根回しが下手なんだよね。",
        "cn": "他突然就提反对意见，但其实私下沟通能力很差。",
        "audio": null,
        "level": "中級",
        "scene": "评价同事，背后议论",
        "style": "会話文",
        "blank": "彼、いきなり反対意見言うけど、___が下手なんだよね。",
        "grammar_points": []
      },
      {
        "jp": "日本では根回しをしないと物事が進まないことが多い。",
        "cn": "在日本不提前打招呼的话，事情往往推不动。",
        "audio": null,
        "level": "中級",
        "scene": "解释日本文化",
        "style": "書き言葉",
        "blank": null,
        "grammar_points": ["n3_gram_naito"]
      }
    ]
  },

  "usage": {
    "collocations": [
      {"pattern": "根回しをする", "meaning_cn": "做提前沟通", "frequency": "高"},
      {"pattern": "根回しが必要", "meaning_cn": "需要提前铺垫", "frequency": "高"},
      {"pattern": "根回しが下手/上手", "meaning_cn": "沟通能力差/好", "frequency": "中"}
    ],
    "sentence_patterns": [
      {"pattern": "〜を根回しする", "meaning_cn": "就…提前打招呼"},
      {"pattern": "〜の根回しが必要", "meaning_cn": "…需要提前铺垫"}
    ],
    "particles": ["を", "が"],
    "conjugal_forms": null
  },

  "memory": {
    "chinese_cognate": {
      "similar": false,
      "diff": "中文没有对应概念。中国人容易理解成「走后门」但语感完全不同。",
      "trap": "不要翻译成「搞关系」，它在日语里是中性偏正面的职场技能"
    },
    "mnemonic": "移植大树——你不能直接拔，得先绕着根松土。日本公司做决策也一样，先「松土」（私下沟通），再「移植」（正式开会决定）。",
    "mnemonic_type": "画面",
    "visual_image": "一棵大树，有人蹲在树根旁边慢慢松土，远处一群人站在会议室门口等着",
    "word_family": [
      {"word": "根回す", "relation": "动词原形（较少用）"},
      {"word": "根回し的", "relation": "形容词化用法（口语）"}
    ],
    "etymology": "园艺术语。移植树木前，提前一年在树冠投影范围切断侧根，促发须根，提高存活率。",
    "similar_sounding": []
  },

  "pitfalls": {
    "common_mistakes": [
      {
        "type": "语感",
        "wrong": "根回し就是走后门。",
        "correct": "根回し是正式场合前的私下沟通，是中性偏正面的。",
        "explanation": "中文「走后门」是贬义，但日语「根回し」是职场必备技能，不做反而会被认为不会读空气。"
      },
      {
        "type": "用法",
        "wrong": "彼は根回しをした（想表达「他走了后门」）",
        "correct": "彼は根回しをした（实际意思是「他提前沟通过了」，中性）",
        "explanation": "如果想表达负面含义，应该用「裏工作」或「根回しを使った」加上下文。"
      }
    ],
    "confused_with": [
      {
        "word": "事前調整",
        "diff": "事前調整更书面、更正式；根回し更有日本文化味、更口语",
        "test_tip": "考试中两者常互换，但根回し更常考"
      }
    ],
    "reading_traps": []
  },

  "culture": {
    "cultural_note": "根回し是日本「空気を読む」文化的具体体现。它反映了日本社会重视和谐、避免正面冲突的价值观。理解根回し，就理解了一半日本职场。",
    "social_context": "职场常用，日常对话中也可用来评价某人的沟通能力",
    "workplace_usage": "提案前必须根回し，否则会议上直接被否决。这是新人入职第一个要学的潜规则。",
    "gender_tendency": "中性",
    "age_tendency": "全年齢",
    "regional": null,
    "keigo_relation": null
  },

  "relations": {
    "synonyms": [
      {"word": "事前調整", "diff": "更正式的书面表达"},
      {"word": "裏工作", "diff": "偏负面，暗地里操作"}
    ],
    "antonyms": [
      {"word": "ぶっつけ本番", "diff": "毫无准备直接上"}
    ],
    "compound": [],
    "derivative": [],
    "same_kanji_diff_reading": []
  },

  "ai": {
    "difficulty_score": 0.6,
    "confusion_cluster": "cluster_职场沟通",
    "teaching_script": "今天我们来学一个理解日本职场的关键概念——{word}。{feeling}我们来看几个真实场景…",
    "quiz_templates": [
      {
        "type": "填空",
        "template": "会議の前に{blank}しておいたから、大丈夫だよ。",
        "distractors": ["事前調整", "裏工作", "準備"]
      },
      {
        "type": "选择",
        "template": "「{word}」の正しい意味は？",
        "distractors": ["走后门", "当面对质", "临时抱佛脚"]
      }
    ],
    "review_strategy": "和「事前調整」「空気を読む」一起复习，构成职场沟通词群"
  },

  "stats": {
    "global_stats": {
      "total_learners": 0,
      "avg_wrong_rate": 0.0,
      "hard_vote": 0
    },
    "per_user": null
  }
}
```

---

## 五、设计决策说明

### 为什么不用 `definitions` 而用 `meanings`？
因为「definitions」是词典思维，「meanings」是学习者思维。学习者需要知道「这个词在我嘴里怎么用」，不是「它在字典里怎么定义」。

### 为什么 `nuance` 是独立字段而不是塞进 `meaning`？
因为语感是独立于释义存在的。「承認」和「認める」释义相同，但语感完全不同。如果塞进 meaning 里，会被义项淹没。

### 为什么 `examples` 里要有 `blank`？
因为这是 Anki「填空卡」的直接数据源。有 blank 就能自动生成 Cloze Deletion 卡片，零成本扩展卡组。

### 为什么 `ai.quiz_templates` 不是后期才做？
因为越早设计好模板，批量生成时质量越高。100 个词 × 3 种题型 = 300 道题，全靠模板驱动。

### 为什么 `stats.per_user` 设计成 null？
词库里不存用户数据。per_user 结构定义好，但实际数据存在用户端（App/Anki）。词库是共享资产，用户数据是私有的。

---

## 六、实施建议

### Phase 1（现在）
填写 P0 字段 → 产出可用的词库 + Anki 卡组

### Phase 2（词库到 500 词时）
补充 P1 字段 → 产出语感教学 + 错题分析

### Phase 3（产品化时）
启用 `ai.*` 和 `stats.*` → 产出 AI 对话学习 + 智能复习

---

## 七、文件命名规范

```
/vocabulary/
├── n5_001.json          # N5 第1批
├── n5_002.json          # N5 第2批
├── n4_001.json
├── n3_001.json
├── n2_001.json
├── n1_001.json
├── schema.json          # 这份 Schema
├── CHANGELOG.md         # 变更记录
└── README.md            # 使用说明
```

每批 100 词，文件编号递增。
