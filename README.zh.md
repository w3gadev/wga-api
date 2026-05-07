<!-- TOC -->

* [WGA API 文档](#wga-api-文档)
* [开发服务器信息](#开发服务器信息)
    * [URL](#url)
    * [API 密钥](#api-密钥)
* [API 列表](#api-列表)
    * [注册验证事件 API](#注册验证事件-api)
        * [认证](#认证)
        * [请求体](#请求体)
        * [字段](#字段)
        * [property 对象](#property-对象)
            * [mission](#mission)
            * [social](#social)
            * [game](#game)
            * [asset](#asset)
            * [device](#device)
        * [响应](#响应)

<!-- TOC -->

# WGA API 文档

# 开发服务器信息

## URL

### 生产服务器
`https://api.wga.xyz/`

### 开发服务器
`https://dev-api.wga.xyz/`

## API 密钥

请在请求的 `X-WGA-API-Key` 请求头中包含 API 密钥。

# API 列表

## 注册验证事件 API

`POST /api/verification-events`

此 API 供客户端（合作伙伴服务）将特定用户行为（社交、游戏、任务、资产等）注册为验证事件。WGA 服务器存储这些事件，并在后续的验证、批处理和链上锚定流程中加以利用。

### 认证

此 API 使用 `WgaProjectContext.projectId()`。因此，请求必须包含项目认证信息（例如 API 密钥或 JWT）。`projectId` 从认证凭据中提取，并在服务器内部自动注入。

### 请求体

```json
{
  "verificationEventType": "SOCIAL",
  "verificationPlatformType": "MOBILE",
  "verificationActionType": "POST_CREATE",
  "clientTimestamp": "2025-12-09T04:12:34Z",
  "clientUserId": "user_12345",
  "ip": "203.0.113.10",
  "userAgent": "Mozilla/5.0 ...",
  "country": "KR",
  "property": {
    "social": {
      "contentId": "post_abc",
      "contentType": "TEXT",
      "parentContentId": null,
      "textLength": 120,
      "mediaCount": 1
    },
    "device": {
      "os": "iOS",
      "osVersion": "17.1",
      "deviceModel": "iPhone15,3",
      "networkType": "WIFI"
    }
  }
}
```

### 字段

| 字段                     | 类型            | 必填 | 说明                                          |
|--------------------------|-----------------|------|-----------------------------------------------|
| verificationEventType    | enum            | ✅   | 事件领域类别                                  |
| verificationPlatformType | enum            | ✅   | 事件发生的平台                                |
| verificationActionType   | enum            | ✅   | 用户行为类型                                  |
| clientTimestamp          | ISO-8601 string | ✅   | 基于客户端的事件发生时间 (Instant)            |
| clientUserId             | string          | ✅   | 合作伙伴服务的用户标识符                      |
| ip                       | string          | ❌   | 客户端 IP 地址                                |
| userAgent                | string          | ❌   | User-Agent 字符串                             |
| country                  | string          | ❌   | 国家代码 (ISO 3166-1 alpha-2) <br/>例：KR, US |
| property                 | object          | ❌   | 行为专属的详细元数据                          |

| 枚举                  | 值                   |
|-----------------------|----------------------|
| VerificationEventType | SOCIAL, GAME |

| 枚举                     | 值                               |
|--------------------------|----------------------------------|
| VerificationPlatformType | MOBILE, PC, CONSOLE |

| 枚举                   | 值                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| VerificationActionType | SIGN_UP, WITHDRAW, LOG_IN, LOG_OUT,<br/>MISSION_ACCEPT, MISSION_CLEAR, MISSION_FAIL,<br/>POST_CREATE, POST_UPDATE, POST_DELETE,<br/>COMMENT_CREATE, COMMENT_UPDATE, COMMENT_DELETE,<br/>LIKE_CREATE, LIKE_DELETE,<br/>ASSET_INFLOW, ASSET_OUTFLOW,<br/>SESSION_START, SESSION_END, STAGE_CLEAR, STAGE_FAIL |

### property 对象

`property` 对象为可选项，包含与行为类型匹配的详细信息。请仅包含如下所示的必要部分。

```json
{
  "property": {
    "mission": {
      ...
    },
    "social": {
      ...
    },
    "game": {
      ...
    },
    "asset": {
      ...
    },
    "device": {
      ...
    }
  }
}
```

#### mission

| 字段             | 类型            | 说明                |
|------------------|-----------------|---------------------|
| missionId        | string          | 任务 ID             |
| rewards          | string          | 奖励信息（字符串格式） |
| expiredTimestamp | ISO-8601 string | 过期时间            |
| detail           | string          | 详细描述            |
| failedReasons    | string          | 失败原因            |

```json
{
  "mission": {
    "missionId": "m_1001",
    "rewards": "point:100",
    "expiredTimestamp": "2025-12-31T23:59:59Z",
    "detail": "邀请3位好友",
    "failedReasons": null
  }
}
```

#### social

| 字段            | 类型   | 说明                          |
|-----------------|--------|-------------------------------|
| contentId       | string | 内容 ID                       |
| contentType     | enum   | 内容类型                      |
| parentContentId | string | 父内容 ID（用于回复/嵌套评论）|
| textLength      | number | 文本长度                      |
| mediaCount      | number | 附加媒体文件数量              |

| 枚举        | 值                                |
|-------------|-----------------------------------|
| contentType | POST, COMMENT, LIKE |

```json
{
  "social": {
    "contentId": "post_abc",
    "contentType": "TEXT",
    "parentContentId": null,
    "textLength": 120,
    "mediaCount": 1
  }
}
```

#### game

| 字段            | 类型    | 说明           |
|-----------------|---------|----------------|
| playDurationSec | number  | 游玩时长（秒） |
| success         | boolean | 是否成功       |
| score           | number  | 达成分数       |

```json
{
  "game": {
    "playDurationSec": 340,
    "success": true,
    "score": 9850
  }
}
```

#### asset

| 字段       | 类型          | 说明                     |
|------------|---------------|--------------------------|
| type       | enum          | 资产类型                 |
| name       | string        | 资产名称                 |
| amount     | string/number | 数量（建议考虑精度问题） |
| actionType | enum          | 变更类型                 |
| isCash     | boolean       | 是否为付费货币           |

| 枚举       | 值                                                                    |
|------------|-----------------------------------------------------------------------|
| actionType | SPEND, EARN, TRANSFER, GRANT, DISCARD |

```json
{
  "asset": {
    "type": "COIN",
    "name": "GOLD",
    "amount": "10.5",
    "actionType": "EARN",
    "isCash": false
  }
}
```

#### device

| 字段        | 类型   | 说明         |
|-------------|--------|--------------|
| os          | string | 操作系统     |
| osVersion   | string | 系统版本     |
| deviceModel | string | 设备型号     |
| networkType | enum   | 网络类型     |

| 枚举        | 值                                                                       |
|-------------|--------------------------------------------------------------------------|
| networkType | WIFI, CELL_3G, CELL_4G, CELL_5G, OTHER |

```json
{
  "device": {
    "os": "Android",
    "osVersion": "14",
    "deviceModel": "SM-S918N",
    "networkType": "WIFI"
  }
}
```

### 响应

当前控制器不返回响应体。

- 200 OK: 成功
- 403 Forbidden: 认证失败
