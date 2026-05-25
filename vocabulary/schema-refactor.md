# 日语词条 Schema 减法优化

> 基于现有 schema + quality-spec + sample entries 做减法。
> 不推翻，只精简。目标：降低复杂度，提高可维护性。

---

## 第一部分：必须保留的核心字段

### 保留原则

> 一个字段如果删掉会导致「学习者理解不了这个词」或「AI 没法出题」，就必须保留。

| 字段 | 保留原因 |
|------|----------|
| `id` | 全局关联的唯一标识，Anki/错题本/复习记录都要引用 |
| `basic.word` | 没有词就没有一切 |
| `basic.kana` | 读音是日语学习的基础 |
| `basic.jlpt_level` | 分级是词库的核心组织方式 |
| `basic.part_of_speech` | 词性决定了用法和活用 |
| `basic.meanings_cn` | 中文释义，但必须结构化（多义项拆分） |
| `basic.core_meaning` | 一句话说清核心意思，给 AI 教学和快速理解用 |
| `nuance.core_nuance` | **核心价值字段**，区别于字典的关键 |
| `nuance.formality` | 1-5 量化，决定用在什么场合，AI 推荐排序用 |
| `nuance.emotion` | 决定语感温度，学习者理解词的情绪色彩 |
| `examples.sentences` | 例句是学习的核心载体，必须有场景和语体 |
| `usage.is_spoken/is_written/is_formal/is_casual/is_business` | 布尔值，筛选和推荐的基础设施 |
| `mistakes.common_mistakes` | **核心价值字段**，中国人的专属痛点 |
| `memory.memory_hint` | 记忆法，帮助学习者记住 |
| `memory.chinese_memory_trick` | 中国人专属记忆法 |
| `tags.scene_tags` | 场景标签，筛选和推荐用 |
| `metadata.created_at` | 追踪创建时间 |
| `metadata.reviewed` | 质量管控，人工审核标记 |

**总计：18 个字段（含子字段）**

### 为什么这些是核心

1. **没有 nuance** → 和字典没区别
2. **没有 mistakes** → 对中国人没价值
3. **没有 sentences** → 学习者没法理解用法
4. **没有 usage 布尔值** → 没法做筛选和推荐
5. **没有 tags** → 没法做场景化学习

---

## 第二部分：建议删除的字段

### 删除清单

| 字段 | 删除原因 |
|------|----------|
| `basic.romaji` | 中国人不需要罗马字，假名就够了。浪费字段。 |
| `basic.meanings_en` | 目标用户是中国人，英文释义没用。以后做多语言再加。 |
| `examples.natural_situations` | 和 `sentences.scene` 重复。一个字段解决两个问题。 |
| `examples.spoken_examples` | 和 `sentences` 中 `style=spoken` 的条目重复。 |
| `examples.written_examples` | 和 `sentences` 中 `style=written` 的条目重复。 |
| `nuance.tone` | 和 `nuance.emotion` 高度重叠。两个词描述同一件事。 |
| `nuance.native_feeling` | 太主观，AI 容易编，人工审核难判断。删掉。 |
| `nuance.similar_chinese_but_different` | 和 `mistakes.chinese_interference` 重复。 |
| `nuance.usage_image` | 和 `memory.image_scene` 重复。 |
| `exam.is_frequently_tested` | 用 `exam.importance_score` 一个数字就够了。布尔值太粗。 |
| `exam.common_patterns` | 和 `exam.common_collocations` 重复。 |
| `exam.grammar_links` | 需要真实语法数据库支撑，现在没有。硬编就是假数据。 |
| `exam.reading_usage` | 太模糊。「阅读中出现在职场文章」——这种描述 AI 随便编，没有信息量。 |
| `exam.listening_usage` | 同上，太模糊。 |
| `exam.common_question_types` | 「选择题」「填空题」——每种题都可能出，这个字段没有区分度。 |
| `exam.similar_words` | 和 `relations.synonyms` 重复。 |
| `memory.association_method` | 和 `memory_hint` 重复。写好 hint 自然就知道方法。 |
| `memory.story_method` | 太创意，很难批量生成。100 个词写 100 个故事，维护成本爆炸。 |
| `memory.keyword_memory` | 和 `memory_hint` 重复。 |
| `usage.common_age_group` | 太模糊。「20-50岁」——几乎所有词都是这个范围。 |
| `usage.frequency_score` | 很难客观评分。AI 打分没有依据，人工打分主观性强。 |
| `usage.common_in_anime` | 太细分。以后做专题内容再加。 |
| `culture.hidden_meaning` | 和 `nuance.core_nuance` 重复。语感本身就包含隐含意思。 |
| `culture.air_reading_note` | 太细分。只有少数词和「读空气」相关。 |
| `mistakes.wrong_usage_examples` | 和 `common_mistakes` 中的 `wrong` 字段重复。 |
| `mistakes.wrong_translations` | 和 `common_mistakes` 重复。 |
| `mistakes.chinese_interference` | 和 `common_mistakes` 中的类型标注重复。 |
| `mistakes.native_speaker_corrections` | AI 容易编造「母语者纠正」。没有真实语料支撑就是假数据。 |
| `relations.related_words` | 太模糊。「相关」没有明确定义，和 synonyms/antonyms 重复。 |
| `relations.related_expressions` | 和 `related_words` 重复。 |
| `ai_learning.*` | 全部延期。见第三部分。 |
| `tags.emotion_tags` | 和 `nuance.emotion` 重复。 |
| `tags.topic_tags` | 和 `scene_tags` 重叠严重。 |
| `tags.difficulty_tags` | 用 `exam.importance_score` 替代。 |
| `metadata.source` | 现在都是 AI 生成的，标注 source 没意义。 |

**总计删除：35 个字段**

### 为什么这些该删

1. **重复字段**（18 个）→ 同一个信息存两遍，维护时不知道改哪个
2. **太模糊的字段**（7 个）→ AI 随便填，没有信息量，浪费 token
3. **太创意的字段**（3 个）→ 批量生成时每个都要写不同内容，token 爆炸
4. **需要真实数据的字段**（3 个）→ 没有数据源，硬编就是假数据
5. **太细分的字段**（4 个）→ 90% 的词用不上

---

## 第三部分：建议延期实现的字段

### 阶段化路线图

| 阶段 | 触发条件 | 新增字段 | 原因 |
|------|----------|----------|------|
| **Phase 1** | 现在 | 无 | 先用核心字段跑通 100 词 |
| **Phase 2** | 100 词完成 | `ai_learning.quiz_templates` | 100 词 × 3 种题型 = 300 道题，验证出题系统 |
| **Phase 3** | 300 词完成 | `pronunciation.*` | 需要 native speaker 数据，AI 声调不准 |
| **Phase 4** | 300 词完成 | `basic.etymology` | 词源故事需要准确性，AI 容易编 |
| **Phase 5** | 1000 词完成 | `ai_learning.shadowing/conversation/roleplay` | 有了足够词库再做 AI 对话 |
| **Phase 6** | 产品化 | `metadata.quality_score` | 有了用户反馈再做质量评分 |
| **Phase 7** | 产品化 | `stats.*` | 需要真实用户数据 |

### 为什么延期

| 字段 | 原因 |
|------|------|
| `pronunciation.*` | AI 声调标注准确率很低（尤其特殊读法），需要人工核对。300 词以后请 native speaker 批量校对。 |
| `basic.etymology` | AI 编词源故事很容易出错。100 词以后再做，且需要参考权威词典。 |
| `ai_learning.quiz_templates` | Phase 2 做，因为 100 词足够验证模板是否好用。 |
| `ai_learning.shadowing` | 需要配合 TTS 和语音识别，产品化阶段才有意义。 |
| `metadata.quality_score` | 没有用户反馈数据时，评分没有依据。 |

---

## 第四部分：最终推荐 Production Schema（精简版）

```json
{
  "id": "n2_00001",

  "basic": {
    "word": "根回し",
    "kana": "ねまわし",
    "jlpt_level": "N2",
    "part_of_speech": ["名词"],
    "meanings_cn": [
      {"cn": "提前打招呼", "priority": 1},
      {"cn": "私下铺垫", "priority": 2}
    ],
    "core_meaning": "正式场合之前私下沟通达成共识"
  },

  "nuance": {
    "core_nuance": "日本职场的核心生存技能。不是走后门，是正式场合前私下达成共识。",
    "formality": 3,
    "emotion": "中性"
  },

  "examples": {
    "sentences": [
      {
        "jp": "会議の前に根回ししておいたから、大丈夫だよ。",
        "cn": "开会之前我提前打过招呼了，没问题的。",
        "scene": "同事之间",
        "style": "spoken"
      },
      {
        "jp": "日本では根回しをしないと物事が進まない。",
        "cn": "在日本不提前打招呼的话，事情推不动。",
        "scene": "解释文化",
        "style": "written"
      }
    ]
  },

  "usage": {
    "is_spoken": true,
    "is_written": true,
    "is_formal": false,
    "is_casual": true,
    "is_business": true,
    "common_in_daily_life": false,
    "common_in_news": true,
    "collocations": [
      {"pattern": "根回しをする", "meaning_cn": "做提前沟通", "frequency": "高"}
    ],
    "conjugal_forms": null
  },

  "mistakes": {
    "common_mistakes": [
      {
        "type": "语感",
        "wrong": "根回し就是走后门。",
        "correct": "根回し是正式场合前的私下沟通，中性偏正面。",
        "explanation": "中文走后门是贬义，但日语根回し是职场必备技能。"
      }
    ],
    "confused_with": [
      {
        "word": "事前調整",
        "diff": "事前調整更书面，根回し更口语更有日本味"
      }
    ]
  },

  "memory": {
    "memory_hint": "移植大树——先绕着根松土，再移植。日本公司做决策也一样。",
    "image_scene": "一棵大树，有人蹲在树根旁边松土，远处会议室门口站着一群人",
    "chinese_memory_trick": "中文没有对应概念。不要翻译成走后门。"
  },

  "culture": {
    "japan_culture_note": "根回し是日本「空気を読む」文化的具体体现。",
    "social_context": "职场常用，也可评价某人的沟通能力",
    "japanese_workplace_note": "提案前必须根回し，否则会议上直接被否决。"
  },

  "relations": {
    "synonyms": [{"word": "事前調整", "diff": "更正式更书面"}],
    "antonyms": [{"word": "ぶっつけ本番", "diff": "毫无准备直接上"}]
  },

  "tags": {
    "scene_tags": ["职场", "商务", "会议"]
  },

  "metadata": {
    "created_at": "2026-05-25T00:00:00Z",
    "reviewed": false
  }
}
```

### 字段统计

| 类别 | 精简前 | 精简后 |
|------|--------|--------|
| 顶层字段 | 12 | 9 |
| 总子字段 | 80+ | 30 |
| 冗余字段 | 18 | 0 |
| AI 易编字段 | 7 | 0 |

### 精简后的优势

1. **批量生成快**：每个词 token 消耗降低约 60%
2. **人工审核快**：30 个字段 vs 80+ 个字段
3. **不会互相打架**：没有重复字段，改一处就行
4. **AI 不容易编**：删掉了所有「太模糊」的字段

---

## 第五部分：S/A/B 级词分级系统

### 设计原则

> 不是所有词都写百科全书。
> 重点词深做，普通词轻做。
> 用最少的人力覆盖最多的词。

### 分级标准

| 等级 | 占比 | 标准 | 示例 |
|------|------|------|------|
| **S 级** | 10-15% | 文化概念词 / 中国人易错 / 高频考试 / 语感复杂 | 根回し、渋い、もったいない、手前、空気を読む |
| **A 级** | 50-60% | 常用词 / 有明确搭配 / 需要例句和记忆法 | 確認、影響、提供、管理 |
| **B 级** | 30-40% | 意思明确 / 中文直觉不会错 / 低频 | 程度、表情、主張 |

### 各级字段要求

#### S 级词（全字段）

```json
{
  "id": "n2_00001",
  "basic": {
    "word": "根回し",
    "kana": "ねまわし",
    "jlpt_level": "N2",
    "part_of_speech": ["名词"],
    "meanings_cn": [
      {"cn": "提前打招呼", "priority": 1},
      {"cn": "私下铺垫", "priority": 2}
    ],
    "core_meaning": "正式场合之前私下沟通达成共识"
  },
  "nuance": {
    "core_nuance": "日本职场的核心生存技能。不是走后门，是正式场合前私下达成共识。",
    "formality": 3,
    "emotion": "中性"
  },
  "examples": {
    "sentences": [
      {"jp": "...", "cn": "...", "scene": "...", "style": "spoken"},
      {"jp": "...", "cn": "...", "scene": "...", "style": "spoken"},
      {"jp": "...", "cn": "...", "scene": "...", "style": "written"}
    ]
  },
  "usage": {
    "is_spoken": true,
    "is_written": true,
    "is_formal": false,
    "is_casual": true,
    "is_business": true,
    "common_in_daily_life": false,
    "common_in_news": true,
    "collocations": [
      {"pattern": "根回しをする", "meaning_cn": "做提前沟通", "frequency": "高"}
    ],
    "conjugal_forms": null
  },
  "mistakes": {
    "common_mistakes": [
      {"type": "语感", "wrong": "...", "correct": "...", "explanation": "..."}
    ],
    "confused_with": [
      {"word": "事前調整", "diff": "..."}
    ]
  },
  "memory": {
    "memory_hint": "移植大树——先松土再移植。",
    "image_scene": "一棵大树，有人蹲在树根旁边松土",
    "chinese_memory_trick": "中文没有对应概念。"
  },
  "culture": {
    "japan_culture_note": "根回し是日本「空気を読む」文化的具体体现。",
    "social_context": "职场常用",
    "japanese_workplace_note": "提案前必须根回し。"
  },
  "relations": {
    "synonyms": [{"word": "事前調整", "diff": "更正式"}],
    "antonyms": [{"word": "ぶっつけ本番", "diff": "毫无准备"}]
  },
  "tags": {"scene_tags": ["职场", "商务"]},
  "metadata": {"created_at": "2026-05-25", "reviewed": false}
}
```

**S 级要求：**
- 例句 3 条（2 spoken + 1 written）
- nuance 必须有深度解释
- mistakes 至少 1 条
- memory 三个字段全填
- culture 三个字段全填
- relations 至少 1 对
- collocations 至少 2 个

---

#### A 级词（标准字段）

```json
{
  "id": "n2_00100",
  "basic": {
    "word": "確認",
    "kana": "かくにん",
    "jlpt_level": "N2",
    "part_of_speech": ["名词", "サ变动词"],
    "meanings_cn": [{"cn": "确认", "priority": 1}],
    "core_meaning": "确认某事是否正确"
  },
  "nuance": {
    "core_nuance": "日常和职场都常用，比チェック更正式。",
    "formality": 3,
    "emotion": "中性"
  },
  "examples": {
    "sentences": [
      {"jp": "メールを確認してください。", "cn": "请确认邮件。", "scene": "职场", "style": "spoken"},
      {"jp": "間違いがないか確認しておいて。", "cn": "确认一下有没有错。", "scene": "日常", "style": "spoken"}
    ]
  },
  "usage": {
    "is_spoken": true,
    "is_written": true,
    "is_formal": true,
    "is_casual": true,
    "is_business": true,
    "common_in_daily_life": true,
    "common_in_news": true,
    "collocations": [
      {"pattern": "確認する", "meaning_cn": "确认", "frequency": "高"}
    ],
    "conjugal_forms": null
  },
  "mistakes": {
    "common_mistakes": [],
    "confused_with": []
  },
  "memory": {
    "memory_hint": "确→确定，认→认。和中文一样。",
    "image_scene": null,
    "chinese_memory_trick": null
  },
  "culture": {
    "japan_culture_note": null,
    "social_context": null,
    "japanese_workplace_note": null
  },
  "relations": {
    "synonyms": [],
    "antonyms": []
  },
  "tags": {"scene_tags": ["职场", "日常"]},
  "metadata": {"created_at": "2026-05-25", "reviewed": false}
}
```

**A 级要求：**
- 例句 2 条（至少 1 spoken）
- nuance 简短但有信息量
- mistakes 可以为空（没有明显易错点就不写）
- memory 只填 memory_hint
- culture 可以为空
- relations 可以为空
- collocations 至少 1 个

---

#### B 级词（最小字段）

```json
{
  "id": "n2_00500",
  "basic": {
    "word": "程度",
    "kana": "ていど",
    "jlpt_level": "N2",
    "part_of_speech": ["名词"],
    "meanings_cn": [{"cn": "程度", "priority": 1}],
    "core_meaning": "和中文程度一样"
  },
  "nuance": {
    "core_nuance": "和中文完全相同，可以直接用中文理解。",
    "formality": 3,
    "emotion": "中性"
  },
  "examples": {
    "sentences": [
      {"jp": "どの程度まで許されるのか。", "cn": "被允许到什么程度？", "scene": "日常", "style": "spoken"}
    ]
  },
  "usage": {
    "is_spoken": true,
    "is_written": true,
    "is_formal": true,
    "is_casual": true,
    "is_business": false,
    "common_in_daily_life": true,
    "common_in_news": false,
    "collocations": [],
    "conjugal_forms": null
  },
  "mistakes": {
    "common_mistakes": [],
    "confused_with": []
  },
  "memory": {
    "memory_hint": null,
    "image_scene": null,
    "chinese_memory_trick": null
  },
  "culture": {
    "japan_culture_note": null,
    "social_context": null,
    "japanese_workplace_note": null
  },
  "relations": {
    "synonyms": [],
    "antonyms": []
  },
  "tags": {"scene_tags": ["日常"]},
  "metadata": {"created_at": "2026-05-25", "reviewed": false}
}
```

**B 级要求：**
- 例句 1 条
- nuance 一句话（甚至可以写「和中文一样」）
- mistakes/memory/culture/relations 全部可空
- collocations 可以为空

---

### 分级的好处

| 好处 | 说明 |
|------|------|
| **Token 节省** | S 级约 800 token/词，A 级约 400 token/词，B 级约 200 token/词。100 词省 40%+ |
| **审核聚焦** | 人工只审核 S 级和 A 级，B 级抽查即可 |
| **批量生产快** | B 级词 AI 一次生成 50 个，几乎不用改 |
| **质量可控** | 精力集中在最重要的词上 |

### 分级判断流程

```
这个词和中文意思完全不同？ → S 级
这个词中国人经常用错？     → S 级
这个词有文化背景？         → S 级
这个词高频考试？           → A 级
这个词有明确搭配？         → A 级
这个词中文直觉不会错？     → B 级
这个词低频？               → B 级
```

---

## 总结

| 维度 | 精简前 | 精简后 |
|------|--------|--------|
| 顶层字段 | 12 | 9 |
| 总子字段 | 80+ | 30 |
| 每词平均 token | ~800 | ~400 |
| 人工审核时间 | 10 min/词 | 3 min/词 |
| 重复字段 | 18 | 0 |
| AI 易编字段 | 7 | 0 |

**核心原则：**
- 同一个信息只存一处
- 没有数据源的字段不编
- 太模糊的字段不写
- 重点词深做，普通词轻做
