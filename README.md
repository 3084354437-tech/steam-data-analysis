# steam-data-analysis
Steam 游戏数据分析项目（MySQL + DBeaver）
# Steam 游戏数据分析（2024年10月）

## 📌 项目背景

Steam 是全球最大的 PC 游戏平台，每天有大量游戏上架。本项目基于 GitHub 开源数据集 `steam-insights`（2024年10月导出），使用 **MySQL + DBeaver** 对 Steam 游戏数据进行多维度分析，回答以下业务问题：

1. 最贵的游戏是什么？价格分布如何？
2. 各货币定价的分布情况？
3. 哪些游戏打折力度最大？
4. 每年发行量趋势如何？Demo 占比有何变化？
5. Steam 上“重制版/完全版”游戏占比如何？

## 🛠️ 工具与技术

- **数据库**：MySQL
- **客户端**：DBeaver
- **数据源**：GitHub - NewbieIndieGameDev/steam-insights
- **核心技能**：JSON 解析、条件聚合、窗口函数、类型转换、日期处理

## 📊 数据概览

- 总记录数：140,082 条
- 正式游戏（game）：122,191 条
- Demo：17,891 条
- 主要货币：EUR（99.4%）、USD（0.6%）

## 🔍 分析一：最贵的游戏

**SQL 代码**：
```sql
SELECT
    name,
    price_overview ->> '$.final_formatted' AS price,
    price_overview ->> '$.currency' AS currency
FROM games
WHERE is_free = 0
  AND JSON_VALID(price_overview)
  AND price_overview ->> '$.currency' = 'EUR'
ORDER BY CAST(price_overview ->> '$.final' AS UNSIGNED) DESC
LIMIT 10;
