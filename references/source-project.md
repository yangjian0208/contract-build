# 源项目参考:AI-Resonance

本技能的方法论实证来源。后续学习/跟进上游更新时,先读这份地图再深入。

## 链接

- 仓库:https://github.com/WZZNNE/AI-Resonance (MIT, TypeScript, pnpm monorepo)
- 线上:https://wzznne.github.io/AI-Resonance/
- 全量剥离文档快照(2026-10-12):`references/ai-resonance-design-system.md`
  (三部分合集:五层解剖+材质配方 / 同类方案四流派全景 / 施工方法 git 考证)
- 上游是每日 AI 情报雷达:零服务器(GitHub Actions+Pages)、五榜单透明评分、183 天记忆

## 方法论 ↔ 仓库物证对照(后续更新看哪里)

| 技能阶段/规则 | 仓库证据位置 |
|---|---|
| P0 契约(值=代码值、取代条款、Do Not) | `docs/DESIGN.md`("v2 条款取代下文不一致处")、`docs/VISUAL.md`(值级契约范本) |
| P0 VERIFIED 证据台账(✔/? + 日期 + 失败模式) | `docs/VERIFIED.md`(244KB,逐外部系统记录实测/反例) |
| P1 爆发 + 测试当缰绳 | 首提交 445 文件带 975 项测试;`packages/*/test/` fixture 录制回放 |
| P2 独立审查(双会话) | `docs/IMPLEMENTATION-*.md` 开头"按 review 文档审查意见实现" |
| P4 验证回执 + 诚实边界 | `docs/IMPLEMENTATION-2026-09-19.md` 验证节(分模块测试数、双视口手测、明确列出未验证项) |
| P5 减法 | git 史:09-25 03:25 视觉层生成 → 04:29 "calmer" → 04:54 "declutter";VISUAL.md 辉光预算即剪枝成文残留 |
| P6 取代式沉淀 | DESIGN.md vN 演进、`docs/DELIVERY-CHANGES.md` |
| 契约之外的工程模式 | `packages/pipeline/src/http.ts`(HTTP 罔层+fixture 脱敏)、`packages/web/src/core/storage.ts`(存储降级链)、`packages/channels/src/report/`(三端共享 DOM-free 渲染器)、`packages/web/src/theme/`(token 派生+boot 防闪白) |

## 跟进上游更新的固定路线

1. **网络**:github.com 直连常超时,走 api.github.com(Python urllib + base64 解码 contents 接口;
   无 token 限 60 次/小时,易 403);线上产物可直接 curl
   `https://wzznne.github.io/AI-Resonance/api/v1/manifest.json`
2. **查变更**:比对仓库 `pushed_at` 与本文档快照日期;main 分支新提交看作者署名与波次
   (该项目人/AI 混合署名,波次即施工轮)
3. **挖新模式**:docs/ 新文件优先读 → `config.yaml` diff(评分/信源变更)→ packages/ 结构变化 → git log
4. **新发现归档**:按主题写进剥离文档对应部分,并在上方对照表追加一行;技能规则改动走取代条款

## 快照声明

本文档与剥离文档快照记录的是 2026-10-12 时点;上游演进后以仓库为准,更新时先复验再采信。
