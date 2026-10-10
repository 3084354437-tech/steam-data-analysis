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
结果：

排名	游戏名	                        价格
1	Ascent Free-Roaming VR Experience	999,00 €
2	The Leverage Game Business Edition	900,00 €
3	The Leverage Game	900,00 €
4	The Official GamingTaylor Game, Great Job!	800,19 €
5	True Love	460,00 €
结论：Steam 上最贵的不是 3A 大作，而是企业级/专业用途软件（如 VR 体验、视频剪辑、3D 建模工具）。真正的游戏大作通常定价在 60-70 欧元左右。


分析二：货币分布
SELECT
    price_overview ->> '$.currency' AS currency,
    COUNT(*) AS game_count
FROM games
WHERE is_free = 0
  AND JSON_VALID(price_overview)
GROUP BY currency
ORDER BY game_count DESC;
结果：
货币	游戏数	占比
EUR	75,621	99.4%
USD	482	0.6%
GBP	46	0.06%
结论：该数据集几乎全部为欧元区定价，USD 仅占 0.6%，其他货币可忽略不计。

分析三：打折力度最大的游戏
SELECT
    name,
    price_overview ->> '$.discount_percent' AS discount,
    price_overview ->> '$.initial_formatted' AS original_price,
    price_overview ->> '$.final_formatted' AS final_price
FROM games
WHERE is_free = 0
  AND JSON_VALID(price_overview)
  AND CAST(price_overview ->> '$.discount_percent' AS UNSIGNED) > 0
ORDER BY CAST(price_overview ->> '$.discount_percent' AS UNSIGNED) DESC
LIMIT 10;
结果：清一色 95% 折扣，原价 29.99€ → 现价 1.49€。
结论：95% 是 Steam 折扣上限。折扣力度与游戏质量无关，只是发行商的促销策略。

分析四：年度发行量 + Demo 占比
SELECT
    SUBSTRING(release_date, 1, 4) AS release_year,
    SUM(CASE WHEN type = 'game' THEN 1 ELSE 0 END) AS games,
    SUM(CASE WHEN type = 'demo' THEN 1 ELSE 0 END) AS demos,
    ROUND(
        SUM(CASE WHEN type = 'demo' THEN 1 ELSE 0 END) * 100.0 / COUNT(*),
        1
    ) AS demo_pct
FROM games
WHERE release_date IS NOT NULL
  AND release_date != ''
  AND release_date != '\\N'
  AND type IN ('game', 'demo')
GROUP BY release_year
ORDER BY release_year DESC
LIMIT 20;
结果：
年份	正式游戏	Demo	Demo 占比
2024	16,050	5,972	27.1%
2023	15,030	3,187	17.5%
2022	12,506	2,236	15.2%
2021	11,222	1,611	12.6%
2019	7,293	609	7.7%
结论：2024 年 Demo 占比飙升至 27.1%，是 2019 年的 3.5 倍。说明“先出 Demo 试水、再正式发售”已成为 Steam 开发者的主流策略。

分析五：版本类型分布
SELECT
    CASE
        WHEN name LIKE '%Remaster%' THEN 'Remaster'
        WHEN name LIKE '%Remake%' THEN 'Remake'
        WHEN name LIKE '%Definitive%' THEN 'Definitive Edition'
        WHEN name LIKE '%Complete%' THEN 'Complete Edition'
        WHEN name LIKE '%Anniversary%' THEN 'Anniversary Edition'
        ELSE 'Original'
    END AS version_type,
    COUNT(*) AS game_count
FROM games
WHERE type = 'game'
  AND release_date IS NOT NULL
  AND release_date != ''
  AND release_date != '\\N'
GROUP BY version_type
ORDER BY game_count DESC;
结果：
版本类型	数量	占比
Original	94,006	99.6%
Remaster	173	0.18%
Definitive Edition	76	0.08%
Complete Edition	68	0.07%
Remake	43	0.05%
结论：Steam 上 99.6% 都是原版游戏，“炒冷饭”的重制版占比极低。媒体报道造成的“重制版泛滥”印象与数据不符。
