# Japanese Language Learning Assets

日语学习数据资产库 — 从 N5 到 N1 的结构化词库。

## 用途

- JLPT 应试
- Anki 卡组生成
- AI 出题系统
- AI 语感教学
- 日语学习 App
- 错题分析
- 记忆宫殿

## 目录结构

```
/vocabulary/
├── n2_001.json              # N2 第一批 100 词
├── n2_product_samples.json  # N2 产品级样例 10 词
├── schema.md                # JSON Schema 设计文档
└── quality-spec.md          # 内容质量规范
```

## Schema 设计

每个词条包含：
- `meta` — 基础信息
- `jlpt` — 考试信息
- `meaning` — 释义（多义项拆分）
- `nuance` — 语感（核心价值字段）
- `examples` — 例句（带场景/情绪/挖空版）
- `usage` — 用法（搭配/句型/活用形）
- `memory` — 记忆系统（中文对比/画面感/词源）
- `pitfalls` — 易错点（按类型分类）
- `culture` — 文化背景
- `relations` — 关联词
- `ai` — AI 扩展（出题模板/教学脚本）
- `stats` — 学习数据

## 质量标准

详见 `quality-spec.md`。

核心原则：
- 例句要像真人说话，不要字典味
- 语感解释要用比喻和画面，不要空洞形容词
- 记忆法要真正有帮助，不要废话
- 易错点要针对中国人的实际错误

## 计划

- [x] N2 产品级样例 10 词
- [ ] N2 完整词库 800+ 词
- [ ] N2 语法库
- [ ] N2 真题结构化
- [ ] Anki 卡组自动生成
- [ ] N3/N4/N5 词库
- [ ] N1 词库
