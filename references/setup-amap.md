# 高德 Key 申请与 MCP 配置指引

skill 依赖**两把不同的钥匙**，缺一只降级对应能力（互不阻塞）：

| 钥匙 | 类型 | 用途 | 存放位置 |
|---|---|---|---|
| Web服务 Key | 高德控制台「Web服务」 | 供 MCP 调 REST API（天气/POI/路径） | Agent 的 MCP 配置 |
| Web端 Key + 安全密钥 | 高德控制台「Web端(JS API)」 | 生成页面里的交互地图 | `~/.travel-planner.json` |

## 一、申请步骤（一次性）

1. 打开 https://console.amap.com 注册账号并完成个人实名认证。
2. 「应用管理 → 我的应用 → 创建新应用」，应用名随意（如 `travel-planner`）。
3. 在该应用下「添加 Key」两次：
   - 第 1 个：服务平台选 **Web服务** → 得到 `Key A`（给 MCP 用）。
   - 第 2 个：服务平台选 **Web端 (JS API)** → 得到 `Key B` 及配套 **安全密钥 jscode**（给页面用）。域名白名单可留空（本地 file:// 打开需要）。
4. 把 Key B 和它的安全密钥写入 `~/.travel-planner.json`（skill 会自动读取）：
   ```json
   { "amapJsKey": "KeyB的值", "amapSecurityJsCode": "对应安全密钥" }
   ```

## 二、配置高德 MCP 服务器（Windows 实测要点）

在所用 Agent 的 MCP 配置文件 `mcp.servers` 下加一个节点（与已有的其他 MCP 服务器平级；配置文件位置以所用客户端文档为准）：

```json
{
  "mcp": {
    "servers": {
      "amap": {
        "type": "stdio",
        "command": "C:\\path\\to\\npx.cmd",
        "args": ["-y", "@amap/amap-maps-mcp-server"],
        "env": { "AMAP_MAPS_API_KEY": "KeyA的值" },
        "timeoutMs": 60000
      }
    }
  }
}
```

注意：
- Windows 下 `command` 建议用 `npx.cmd` 的绝对路径（终端执行 `where npx` 即可查到，替换上面的占位路径），直接写 `npx` 可能找不到。
- 编辑前备份该文件；只添加 `amap` 节点，不动其他服务器配置。
- 改完后**重启会话**（新会话才会加载 MCP）。

## 三、验证

新会话里应能看到 `maps_` 开头的高德工具（如 `maps_geo`、`maps_weather`、`maps_text_search`）。
让模型调用 `maps_weather` 查任意城市，能返回数据即成功。

仍连不上时依次排查：JSON 语法、`npx` 路径、key 是否填对；可在终端手动跑一次 `npx -y @amap/amap-maps-mcp-server` 看具体报错。

## 四、常见问题

- **页面打开后地图提示 INVALID_USER_SCODE / USERKEY_PLAT_NOMATCH**：Key B 的安全密钥没填或填错，检查 `~/.travel-planner.json`。
- **地图灰屏且提示加载失败**：网络问题或 Key B 类型选成了 Web服务（两种类型不通用）。
- **MCP 工具报 CNNL_ERROR / 配额**：高德个人开发者每日配额（几千次/接口）足够规划使用，超限次日重置。
