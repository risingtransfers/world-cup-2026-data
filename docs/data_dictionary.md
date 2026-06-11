# Rising Transfers — 数据字典 (Data Dictionary)

> **版本:** 2.0 | **更新日期:** 2026-02-11  
> **覆盖范围:** 所有 Supabase 表、JSONB 字段结构、RPC 函数、Views  
> **生成方式:** 基于 60+ migration 文件 + TypeScript 类型定义 + Python dataclass 反向梳理

---

## 目录

1. [维度表 (Dimensions)](#1-维度表-dimensions)
2. [事实表 (Facts)](#2-事实表-facts)
3. [向量/AI 表](#3-向量ai-表)
4. [话题管道 (Topics Pipeline)](#4-话题管道-topics-pipeline)
5. [聊天系统 (Chat)](#5-聊天系统-chat)
6. [实体映射 (Mapping)](#6-实体映射-mapping)
7. [RPC 函数](#7-rpc-函数)
8. [视图 (Views)](#8-视图-views)
9. [JSONB 字段详细结构](#9-jsonb-字段详细结构)

---

## 1. 维度表 (Dimensions)

### `dim_players` — 球员主表
| 字段 | 类型 | 说明 |
|------|------|------|
| `player_id` | INT PK | Sportmonks 球员 ID |
| `team_id` | INT FK → dim_teams | 当前球队 |
| `name` | TEXT NOT NULL | 全名 (如 "Erling Braut Haaland") |
| `common_name` | TEXT | 常用名 (如 "E. Haaland") |
| `position_name` | TEXT | 位置 (如 "Attacker") |
| `detailed_position_id` | INT | 细粒度位置 FK |
| `date_of_birth` | DATE | 出生日期 |
| `nationality_id` | INT | 国籍 ID |
| `nationality_name` | TEXT | 国籍名称 |
| `market_value` | NUMERIC | 身价 (EUR) |
| `contract_expires` | DATE | 合同到期 |
| `contract_start` | DATE | 合同开始 |
| `contract_option` | TEXT | 合同选项 |
| `joined_date` | DATE | 加盟日期 |
| `height` | TEXT | 身高 |
| `foot` | TEXT | 惯用脚 |
| `image_url` | TEXT | 头像 URL |
| `agent_name` | TEXT | 经纪人姓名 |
| `agent_id` | INT FK → dim_agents | 经纪人 ID |
| `agency_id` | TEXT | 经纪公司 ID |
| `transfermarkt_id` | TEXT | TM ID |
| `transfermarkt_url` | TEXT | TM 链接 |
| `curated_data` | JSONB | 人工标注数据 (含 aliases 数组) |
| `tm_verified` | BOOLEAN | 是否已 TM 验证 |
| `name_zh` | TEXT | 中文名 |
| `updated_at` | TIMESTAMPTZ | 最后更新 |

**关键索引:** `name` (trigram), `common_name` (trigram), `market_value`, `contract_expires`, `agent_id`

### `dim_teams` — 球队主表
| 字段 | 类型 | 说明 |
|------|------|------|
| `team_id` | INT PK | Sportmonks 球队 ID |
| `name` | TEXT NOT NULL | 球队名 |
| `short_code` | TEXT | 缩写 |
| `league_id` | INT FK | 联赛 |
| `country_id` | INT | 国家 |
| `country_name` | TEXT | 国家名 |
| `aliases` | JSONB | 别名数组 |
| `logo_url` | TEXT | 队徽 |
| `tm_team_id` | TEXT | TM 球队 ID |
| `tm_verified` | BOOLEAN | TM 验证状态 |
| `last_deep_sync_at` | TIMESTAMPTZ | 最后深度同步 |
| `name_zh` | TEXT | 中文名 |

### `dim_leagues` — 联赛
| 字段 | 类型 | 说明 |
|------|------|------|
| `league_id` | INT PK | 联赛 ID |
| `name` | TEXT NOT NULL | 联赛名 |
| `country_id` | INT | 国家 |
| `is_active` | BOOLEAN | 是否活跃 |
| `name_zh` | TEXT | 中文名 |

### `dim_coaches` — 教练
| 字段 | 类型 | 说明 |
|------|------|------|
| `coach_id` | BIGINT PK | 教练 ID |
| `team_id` | BIGINT FK | 当前球队 |
| `name` | TEXT NOT NULL | 全名 |
| `common_name` | TEXT | 常用名 |
| `current_club` | TEXT | 当前执教俱乐部 |
| `ppm` | NUMERIC(4,2) | 场均积分 |
| `preferred_formation` | TEXT | 偏好阵型 |
| `coaching_history` | JSONB | 执教历史 |
| `tm_id` | TEXT UNIQUE | TM ID |

### `dim_agencies` — 经纪公司
| 字段 | 类型 | 说明 |
|------|------|------|
| `agency_id` | TEXT PK | 经纪公司 ID |
| `agency_name` | TEXT NOT NULL | 公司名 |
| `total_value` | BIGINT | 总价值 |
| `player_count` | INT | 球员数 |
| `top_players` | JSONB | 旗下明星球员 |

### `dim_agents` — 经纪人
| 字段 | 类型 | 说明 |
|------|------|------|
| `agent_id` | SERIAL PK | 经纪人 ID |
| `name` | TEXT NOT NULL | 姓名 |
| `agency_id` | TEXT FK | 所属公司 |
| `tm_agent_id` | TEXT UNIQUE | TM ID |
| `notable_clients` | JSONB | 知名客户 |
| `total_portfolio_value` | BIGINT | 组合总价值 |

### `dim_team_tier` — 球队分级
| 字段 | 类型 | 说明 |
|------|------|------|
| `team_id` | INT PK | 球队 ID |
| `team_name` | TEXT | 球队名 |
| `tier` | CHAR(1) | 等级: S/A/B/C/D |
| `squad_market_value` | NUMERIC | 阵容总价值 |
| `team_type` | TEXT | club / national_team / women / youth |

### `dim_tm_players` — Transfermarkt 球员 (独立源)
| 字段 | 类型 | 说明 |
|------|------|------|
| `tm_id` | TEXT UNIQUE | TM 球员 ID |
| `name` | TEXT NOT NULL | 球员名 |
| `position` | TEXT | 位置 |
| `current_club` | TEXT | 当前俱乐部 |
| `market_value` | BIGINT | 身价 |
| `contract_expires` | DATE | 合同到期 |
| `raw_data` | JSONB | TM 原始数据 |

### `dim_countries` / `dim_seasons` / `dim_detailed_positions`
标准维度表，不再赘述。

---

## 2. 事实表 (Facts)

### `match_player_stats` — 比赛级球员统计 (3.2M+ 行)
| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | UUID PK | |
| `fixture_id` | BIGINT NOT NULL | 比赛 ID |
| `player_id` | BIGINT FK | 球员 |
| `team_id` | BIGINT FK | 球队 |
| `season_id` | BIGINT NOT NULL | 赛季 |
| `minutes` | INTEGER | 出场时间（外层便捷列，由触发器自动与 `stats->>'minutes'` 同步）|
| `rating` | NUMERIC(4,2) | 评分 |
| `stats` | JSONB | 详细统计（含 `minutes`、`goals`、`assists` 等原始数据）|

> ⚠️ **`minutes` 字段重要说明（2026-02-21 更新）**
> 
> 该外层列在 2026-02-21 前存在与 `stats->>'minutes'` 长期不同步的问题（外层列历史上全为 0，实际上场时间仅存于 JSONB 中）。经过全量数据回填（428,000 条记录修复）及触发器 `trg_sync_minutes` 部署后，问题已彻底解决。
>
> **读取建议**：任何依赖 `minutes` 的查询或 ETL，应使用防御性写法：
> ```sql
> COALESCE(NULLIF(minutes, 0), (stats->>'minutes')::int, 0)
> ```
> **写入保障**：触发器 `trg_sync_minutes`（BEFORE INSERT OR UPDATE）已确保新写入数据两列永远一致，无需手动同步。

**stats JSONB 结构:**
```json
{
  "minutes": 90,
  "goals": 1, "assists": 0,
  "shots_total": 3, "key_passes": 2,
  "duels_won": 5, "dribbles_success": 2,
  "interceptions": 1, "rating": 7.5,
  "details": { "xg": 0.65 }
}
```

### `fact_transfer_rumours` — SM 转会传闻
| 字段 | 类型 | 说明 |
|------|------|------|
| `rumour_id` | UUID PK | |
| `player_id` | INT FK | 球员 |
| `from_team_id` / `to_team_id` | INT FK | 转会方向 |
| `probability_label` | TEXT | 概率标签 |
| `monetary_value` | NUMERIC | 金额 |

### `fact_rumours` — TM 转会传闻
| 字段 | 类型 | 说明 |
|------|------|------|
| `rumour_id` | TEXT PK | TM 传闻 ID |
| `player_name` | TEXT | 球员名 |
| `current_club` | TEXT | 当前俱乐部 |
| `interested_clubs` | JSONB | 意向俱乐部列表 |
| `probability` | TEXT | 概率 |
| `scraped_at` | TIMESTAMPTZ | 抓取时间 |

### `fact_transfer_history` — 转会历史 (TM)
| 字段 | 类型 | 说明 |
|------|------|------|
| `player_id` | INT FK | 球员 |
| `value_date` | DATE | 转会日期 |
| `market_value` | NUMERIC | 身价 |
| `transfer_fee` | NUMERIC | 转会费 |
| `club_from` / `club_to` | TEXT | 转会方向 |
| `season` | TEXT | 赛季 |

### `fact_market_value_history` — 身价历史 (TM)
| 字段 | 类型 | 说明 |
|------|------|------|
| `tm_player_id` | TEXT NOT NULL | TM 球员 ID |
| `value_date` | DATE NOT NULL | 估值日期 |
| `market_value` | BIGINT | 身价 |
| `club_name` | TEXT | 所在俱乐部 |

### 其他事实表
- `fact_transfers` — SM 转会记录
- `fact_injuries` / `fact_injury_history` — 伤病记录
- `fact_player_national_team` — 国家队数据
- `fact_player_achievements` — 荣誉/奖项
- `fact_player_agent_history` — 经纪人变更历史
- `fact_contracts` — 合同详情
- `fact_social_mentions` — 社媒提及
- `fact_player_stats_seasonal` — 赛季级统计聚合
- `fact_squad_depth_snapshot` — 阵容深度快照
- `metrics_valuation_gap` — 身价偏差分析
- `graph_agent_affinity` — 经纪人-球队关系图谱

---

## 3. 向量/AI 表

### `vec_player_dna` — 球员战术 DNA (~6K 行)
| 字段 | 类型 | 说明 |
|------|------|------|
| `player_id` | INT FK | 球员 |
| `season_id` | INT | 赛季 |
| `narrative` | TEXT | AI 生成战术叙事 |
| `embedding` | vector(768) | 叙事向量 (Gemini) |
| `style_tags` | TEXT[] | 风格标签 (如 ['Clinical Finisher', 'Aerial Threat']) |
| `position_group` | TEXT | 位置组 (FWD/MID/DEF/GK) |
| `league_name` / `league_tier` | TEXT/INT | 联赛信息 |
| `data_quality` | TEXT | FULL / PARTIAL / SPARSE |
| `is_curated` | BOOLEAN | 人工审核标记 |
| `is_latest_season` | BOOLEAN | 是否最新赛季 |
| Per90 统计 | NUMERIC | goals, assists, shots, key_passes, tackles, interceptions, passes, duels_won, dribbles, clearances, touches, aerials_won, saves 等 |

**唯一约束:** `(player_id, season_id)`

### `raw_tweets` — 原始推文
| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | TEXT PK | 推文 ID |
| `text` | TEXT | 推文内容 |
| `author_username` | TEXT | 作者 |
| `embedding` | vector(768) | 内容向量 |
| `metrics` | JSONB | `{like_count, reply_count, retweet_count}` |
| `is_processed` | BOOLEAN | 是否已处理 |

### `player_mentions_cache` — 球员提及缓存
| 字段 | 类型 | 说明 |
|------|------|------|
| `player_id` | INT PK | 球员 |
| `mention_count_1h` / `mention_count_24h` | INT | 提及次数 |
| `tier1_count_1h` / `tier1_count_24h` | INT | Tier1 记者提及 |
| `heat_score` | DECIMAL | 热度分 |

### `ghost_topic_cache` — 幽灵话题缓存
| 字段 | 类型 | 说明 |
|------|------|------|
| `player_id` | INT PK | 球员 |
| `assembled_data` | JSONB | 预装配数据 |
| `quality_level` | TEXT | basic / lite / full |
| `expires_at` | TIMESTAMPTZ | 过期时间 (24h) |

---

## 4. 话题管道 (Topics Pipeline)

### `rumour_topics` — 话题主表 (456 行)
| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | UUID PK | 话题 ID |
| `player_id` | INT FK | 球员 |
| `player_name` | VARCHAR | 球员名 |
| `from_team_name` | VARCHAR | 来源球队 |
| `interested_teams` | JSONB | 目标球队列表 |
| `pipeline_stage` | VARCHAR | DISCOVERED → SELECTED → ASSEMBLING → READY → PUBLISHED → ARCHIVED |
| `heat_score` | FLOAT | 热度分 |
| `assembled_data` | JSONB | **装配后完整数据** (见 [§9](#9-jsonb-字段详细结构)) |
| `last_assembled_at` | TIMESTAMP | 最后装配时间 |
| `data_version` | INT | 数据版本 (当前 v3) |
| `ai_prediction` | JSONB | AI 预测结果 |
| `match_confidence` | DECIMAL | 球员匹配置信度 |
| `selected_at` / `ready_at` / `archived_at` | TIMESTAMPTZ | 各阶段时间戳 |

### `topic_assembly_queue` — 装配队列
任务型队列，管理 topic 装配流程。状态: `pending → processing → completed/failed`

### `topic_ai_content` — AI 生成内容
| 字段 | 类型 | 说明 |
|------|------|------|
| `topic_id` | UUID FK | 话题 |
| `content_type` | VARCHAR | dna / analysis / briefing |
| `content` | JSONB | AI 内容 |
| `version` | INT | 版本 (自动递增) |
| `model_used` | VARCHAR | 使用的模型 |

### `topic_events` — 话题事件日志
| 字段 | 类型 | 说明 |
|------|------|------|
| `topic_id` | UUID FK | 话题 |
| `event_type` | VARCHAR | DISCOVERED / SELECTED / ASSEMBLED / PUBLISHED 等 |
| `event_data` | JSONB | 事件详情 |

### `rumour_candidates` — 传闻候选
| 字段 | 类型 | 说明 |
|------|------|------|
| `tweet_id` | VARCHAR UNIQUE | 推文 ID |
| `player_name` | VARCHAR | 球员名 |
| `from_club` / `to_club` | VARCHAR | 转会方向 |
| `confidence` | INT | 置信度 |
| `topic_id` | UUID FK | 关联话题 |

---

## 5. 聊天系统 (Chat)

### `chat_sessions` — 会话
| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | UUID PK | 会话 ID |
| `user_id` | UUID | 用户 |
| `topic_id` | UUID FK | 关联话题 (Topic 模式) |
| `title` | TEXT | 标题 |
| `mode` | TEXT | free / topic |

### `chat_messages` — 消息
| 字段 | 类型 | 说明 |
|------|------|------|
| `session_id` | UUID FK | 会话 |
| `role` | TEXT | user / assistant / system |
| `content` | TEXT | 消息内容 |
| `tool_calls` | JSONB | 工具调用记录 |

---

## 6. 实体映射 (Mapping)

### `entity_player_aliases` — 球员别名 (~68 行 → 扩充中)
| 字段 | 类型 | 说明 |
|------|------|------|
| `player_id` | INT FK | 球员 |
| `alias` | TEXT NOT NULL | 别名文本 |
| `alias_type` | TEXT | nickname / short_name / local_name / twitter_name |
| `language` | TEXT | 语言 |
| `is_primary` | BOOLEAN | 是否主要别名 |

**使用方:**
- `lib/entity-resolver.ts` (ILIKE 模糊匹配)
- `fn_resolve_player_id` (精确匹配)

### `entity_team_aliases` — 球队别名
| 字段 | 类型 | 说明 |
|------|------|------|
| `sm_team_id` | INT FK | 球队 ID |
| `alias` | TEXT NOT NULL | 别名 |
| `alias_type` | TEXT | alias / short / local / nickname |

### `integration_tm_sm_map` — TM↔SM 球员映射
| 字段 | 类型 | 说明 |
|------|------|------|
| `sm_player_id` | INT UNIQUE FK | SM 球员 ID |
| `tm_player_id` | TEXT | TM 球员 ID |
| `match_method` | TEXT | 匹配方式 |
| `match_confidence` | NUMERIC | 匹配置信度 |
| `tm_market_value_current` | BIGINT | TM 当前身价 |

### `entity_unmatched` — 未匹配实体
| 字段 | 类型 | 说明 |
|------|------|------|
| `entity_type` | TEXT | player / team / coach |
| `tm_id` | TEXT | TM ID |
| `tm_name` | TEXT | TM 名称 |
| `best_candidates` | JSONB | 最佳候选 |
| `status` | TEXT | pending / matched / ignored / no_match |

---

## 7. RPC 函数

| 函数 | 用途 | 调用方 |
|------|------|--------|
| `fn_resolve_player_id(name)` | 球员名称解析 → player_id | SQL 触发器 |
| `match_tweets(embedding, threshold, count)` | 推文向量相似度搜索 | topic_assembler.py |
| `search_player_dna(query, position, count)` | DNA 语义搜索 | API |
| `get_player_capability_metrics(id, n)` | 球员能力指标 | intelligence-layer |
| `get_player_form_guide(id, n)` | 近期状态趋势 | intelligence-layer |
| `enqueue_topic_assembly(id, priority, by)` | 入队装配任务 | Admin API |
| `get_next_assembly_task()` | 获取下一个装配任务 | 调度器 |
| `refresh_player_mentions_cache()` | 刷新球员提及缓存 | Cron |
| `sync_agent_names_to_dim_agents()` | 同步经纪人数据 | ETL |
| `search_hot_topics(min_heat, count)` | 热门话题搜索 | Chat API |

---

## 8. 视图 (Views)

| 视图 | 用途 |
|------|------|
| `v_topics_enriched` | 话题 + 球员/球队完整信息 |
| `v_topics_pipeline` | 按 pipeline_stage 统计 |
| `v_topic_data_quality` | 话题数据匹配率 |
| `v_assembly_queue_stats` | 装配队列统计 |
| `view_scouting_report` | 球探报告 (含 xG, 影子估值) |
| `view_transfer_feed` | 转会动态 Feed |
| `v_agency_portfolio` | 经纪人组合总览 |
| `v_data_quality_summary` | 数据质量指标 |
| `v_player_injury_summary` | 伤病概览 |
| `v_player_international_summary` | 国际赛事统计 |

---

## 9. JSONB 字段详细结构

### `rumour_topics.assembled_data` (v3.0)

```typescript
{
  // 基础信息
  player_id: number | null
  player_name: string
  market_value: number | null
  position: string | null
  
  // 6 大模块
  player_profile: {
    player_id, name, common_name, image_url, age, height, foot,
    nationality, position, current_team_id/name/logo, target_team_id/name/logo
  }
  player_dna: {
    style_tags: string[], narrative: string,
    position_group, league_name, data_quality,
    goals_per90, assists_per90, shots_per90, key_passes_per90,
    tackles_per90, interceptions_per90, passes_per90, duels_won_per90,
    dribbles_per90, clearances_per90
  }
  transfer_value: {
    market_value, contract_expires/start/option,
    sm_rumours: [], tm_rumours: [],
    probability_label, monetary_amount
  }
  agent_network: {
    agent_name, agency_id/name,
    agency_total_value, agency_player_count,
    agency_top_players: string[]
  }
  coach_connection: {
    current_coach_id/name, target_coach_id/name,
    preferred_formation, coaching_history: [],
    has_prior_connection
  }
  social_sentiment: {
    tweet_count, total_likes, total_retweets,
    sample_tweets: [], sentiment_score, trending_keywords
  }
  
  // AI 生成内容
  ai_dna: object | null
  ai_analysis: object | null
  ai_briefing: object | null
  
  // 实时补丁
  live_patch: {
    breaking_news_alert: boolean,
    live_tweet_count: number,
    latest_tweet: object,
    heat_spike_ratio: number
  } | null
  
  // 元数据
  meta: {
    is_ghost_topic: boolean,
    assembled_at: string,
    data_version: number,
    ai_status: string
  }
  
  assembled_at: string  // ISO timestamp
  data_version: 3
}
```

### `dim_players.curated_data`

```json
{
  "aliases": ["Haaland", "哈兰德", "E. Haaland"],
  "notes": "人工备注",
  "verified_at": "2026-02-01T00:00:00Z"
}
```

### `rumour_topics.interested_teams`

```json
[
  {
    "team_id": 9,
    "team_name": "Manchester City",
    "likelihood": "high",
    "mention_count": 15
  }
]
```

### `rumour_topics.ai_prediction`

```json
{
  "score": 72,
  "tier": "PROBABLE",
  "recommendation": "MONITOR",
  "reasoning": "Multiple Tier-1 sources...",
  "factors": ["journalist_tier", "contract_situation"],
  "predicted_at": "2026-02-10T12:00:00Z"
}
```

---

## 附录: 数据流关系图

```
Sportmonks API → dim_players, dim_teams, match_player_stats, fact_transfers
                  ↓
Transfermarkt  → dim_tm_players, fact_rumours, fact_transfer_history
                  ↓                              ↓
                integration_tm_sm_map ←→ entity_player_aliases
                  ↓
Twitter API    → raw_tweets → rumour_candidates → rumour_topics
                  ↓                                    ↓
                vec_player_dna (ETL)          topic_assembler.py → assembled_data
                  ↓                                    ↓
                embedding (Gemini)            AI content (dna/analysis/briefing)
                  ↓                                    ↓
                Chat API ← entity-resolver ← fetchData tool
```
