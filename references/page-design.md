# 页面设计规范与 PLAN 数据结构

模板 `assets/template.html` 是**数据驱动**的：页面所有模块由
`<script type="application/json" id="plan-data">` 里的一份 JSON 渲染出来。
生成页面时只做三件事：改 `<title>`、填 JSON、把 `~/.travel-planner.json` 的两个值填进
`PLAN.amap`。**不要改模板的 CSS 和渲染 JS**（除非用户点名要改样式，那也先复制模板再改）。

## 填充步骤

1. 复制模板到 `travel-plans/<目的地>-<YYYYMMDD>.html`。
2. 替换 `id="plan-data"` script 块内的整个 JSON（模板里带了一份成都示例数据，全部替换）。
3. `<title>` 改成 `出行手账 · <目的地> <起始日期>`。
4. 用 SKILL.md 里的 python 单行命令校验 JSON 合法性。

## PLAN Schema（字段严格区分大小写）

```jsonc
{
  "meta": {
    "destination": "成都",            // 必填
    "emoji": "🐼",                    // 目的地代表 emoji，hero 区大贴纸
    "year": "2026",                   // 起始年（4位）
    "startDate": "10-01", "endDate": "10-03",  // MM-DD
    "dayCount": 3,
    "travelers": "2 人",
    "theme": "美食 + 人文慢游",        // 一句话旅行主题
    "intro": "≤60字的手账式引言",
    "center": [104.066, 30.66]        // 地图初始中心（市区坐标，[lng,lat]）
  },
  "amap": { "key": "Web端Key", "securityJsCode": "安全密钥" },
  "weather": {
    "mode": "forecast | climate",     // climate 时页面自动显示"气候参考"说明条
    "note": "climate 模式的说明文案；forecast 可留空字符串",
    "days": [
      { "date": "10-01", "label": "周四", "dayweather": "多云",
        "nightweather": "小雨", "daytemp": "25", "nighttemp": "17" }
    ]
  },
  "days": [                           // 逐日行程，顺序即 Day 1..N
    {
      "n": 1, "date": "10-01", "theme": "老成都的烟火气",
      "spots": [
        {
          "type": "sight | meal",     // meal 卡片会换暖色美食样式
          "name": "人民公园", "time": "09:30", "stay": "2h",
          "emoji": "🌳", "lng": 104.06, "lat": 30.662,
          "address": "青羊区少城路12号", "price": "免费", "rating": "4.5",
          "note": "鹤鸣茶社喝盖碗茶，体验掏耳朵（约30元）",
          "photo": "",                // 可选图片 URL，留空用 emoji 封面
          "fromPrev": null            // 第一个节点为 null；其后:
          // { "mode": "walk|metro|bus|taxi|train|ferry|drive",
          //   "minutes": 15, "text": "地铁4号线 宽窄巷子站B口" }
        }
      ]
    }
  ],
  "foods": [                          // 美食推荐区（没排进时间轴的好店）
    { "name": "甘记肥肠粉", "emoji": "🍜", "tags": ["苍蝇馆子","本地人排队"],
      "address": "...", "price": "人均 ¥25", "note": "加节子，配锅盔",
      "lng": 104.05, "lat": 30.66, "photo": "" }
  ],
  "budget": {
    "perPerson": true,
    "items": [ { "icon": "🎫", "category": "门票", "detail": "熊猫基地+武侯祠", "amount": 105 } ],
    "note": "按 2 人同行、舒适型消费估算，仅供参考"
  },
  "lodging": {
    "area": "春熙路-太古里商圈", "emoji": "🏨",
    "reason": "1/2/3号线交汇，去哪都方便，晚上下楼就是夜宵",
    "picks": ["亚朵(春熙路店) 舒适 ¥450/晚", "梦之旅青旅 经济 ¥80/床"],
    "lng": 104.081, "lat": 30.66
  },
  "tips": [ { "icon": "👟", "title": "穿搭", "text": "早晚凉，带薄外套" } ],
  "footer": { "source": "数据与地图：高德地图" }
}
```

## 编排与文案标准

- **时间轴**：`time` 24 小时制 `HH:MM`；`stay` 如 `1.5h`；`fromPrev.minutes` 取整。
- `fromPrev.text` 写人话：「地铁4号线 宽窄巷子站 B口」「沿少城路向北步行」。
- **note** 是手账的灵魂：写"为什么值得/怎么玩/避坑"，每条 ≤40 字，禁空话
  （❌"非常值得一去"；✅"早上8点前入园，熊猫最活跃"）。
- `emoji` 每个节点都给，选最有记忆点的（🐼🍜🏮），美食节点必须给。
- 预算 `amount` 一律填**人均**数值（页面自动求和）；`detail` 写清构成。
- 预览版（无 MCP 数据时）：在 `meta.intro` 前加「⚠️ 预览版：数据未经高德校验，仅供风格确认」。

## 图片（可选增强）

`photo` 留空时页面用 emoji 渐变封面，默认就这么交付，稳定且离线可用。
会话里有图片搜索工具且用户想要真实照片时，搜「目的地+景点名」取横图 URL 填入
`photo`（页面有 onerror 兜底，图挂了自动回退 emoji 封面）。
