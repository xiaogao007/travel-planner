# 高德 MCP 工具清单与取数策略

> 工具名以会话中实际可用的为准（常见前缀 `mcp__amap__` 或 `maps_`）。下表是官方
> `@amap/amap-maps-mcp-server` 的标准工具集，找不到完全同名的就按功能找最接近的。

| 工具 | 功能 | 本 skill 用法 |
|---|---|---|
| `maps_geo` | 地理编码（城市名→adcode/中心坐标） | 第一步定位目的地，取 `city` 与中心坐标填 `PLAN.meta.center` |
| `maps_weather` | 天气（实况 + 近 3~4 天预报） | 传 adcode；出发日在窗口内→`mode:"forecast"`，否则→`mode:"climate"` |
| `maps_text_search` | POI 关键词搜索 | 搜「风景名胜/公园/博物馆」拿景点，搜「美食/小吃/火锅」拿美食 |
| `maps_around_search` | 周边 POI 搜索 | 备用：围绕住宿点/某景点找吃的 |
| `maps_direction_walking` | 步行路径 | 相邻景点间距离 ≤1.5km 时取步行耗时 |
| `maps_direction_transit_integrated` | 公交/地铁路径 | 步行超阈值时取公交方案（线路名+耗时） |
| `maps_direction_driving` | 驾车路径 | 打车/自驾场景估耗时与费用 |
| `maps_distance` | 批量测距 | 快速判断两个点该步行还是坐车 |

## 取数顺序

1. **定位**：`maps_geo(目的地城市名)` → adcode、市中心坐标。
2. **天气**：`maps_weather(adcode)`。高德只有实况+3~4 天预报：
   - 出发日 ≤3 天后：逐日填真实预报（`dayweather/daytemp/nighttemp` 来自返回的 `casts`）。
   - 更远：`mode:"climate"`，用常识写往年同期气候（如"10 月成都 15~23°C 多夜雨"），
     `note` 里写明"出发前 3 天内重新生成可获取实时预报"。
3. **景点池**：`maps_text_search` 按「风景名胜」+ 城市名搜（size 取 20），
   需要更多再按区县/商圈补搜。记录每个 POI 的 `name/location(经,纬)/address/biz_ext.rating`。
4. **美食池**：同样搜美食。目标 6~10 家，覆盖：本地代表菜、街头小吃、夜宵/火锅类。
5. **交通衔接**：对编排好的一天内相邻节点调 `maps_direction_walking` 或
   `maps_direction_transit_integrated`，取 `duration`（秒→分钟）与首条线路名，
   填进下一节点的 `fromPrev`。

## 选点与编排原则

- 评分（`biz_ext.rating`）≥4.0 优先；同类景点去重；每个 POI 必须拿到**经纬度**（地图要用）。
- **按地理聚类分天**：把景点池按坐标聚成 N 团（一天一团），核心地标优先排白天，
  夜市/江边夜景排晚上。
- 步行 ≤25 分钟（约 1.5km）选 walk；否则公交（metro/bus）；带老人小孩或末班车后选 taxi。
- 城际交通（高铁/机场大巴）作为当天第一个或最后一个节点，用 `fromPrev` 描述。
- 每天节点数 4~6（含 2 正餐）；宁可少排，不排折返路线。
- 美食 POI 落在当天活动区域附近才排进时间轴，其余进「美食推荐」分区。

## 预算取数

- 门票：POI 详情或常识（`cost` 字段不一定有），没有就按常识估算并标注"参考价"。
- 餐饮：按店铺人均（POI `biz_ext.cost`）× 天数汇总。
- 交通：步行 0；地铁按趟次 2~5 元估；打车按 driving 距离 × 当地起步价常识。
- 住宿：按目的地常见档位给 2~3 档（经济/舒适/轻奢）单晚价 ×（天数-1）。
