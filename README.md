<!-- TOC -->

* [WGA API Documentation](#wga-api-documentation)
* [Development Server Information](#development-server-information)
    * [URL](#url)
    * [API Key](#api-key)
* [API List](#api-list)
    * [Register Verification Event API](#register-verification-event-api)
        * [Authentication](#authentication)
        * [Request Body](#request-body)
        * [Fields](#fields)
        * [property Object](#property-object)
            * [mission](#mission)
            * [social](#social)
            * [game](#game)
            * [asset](#asset)
            * [device](#device)
        * [Responses](#responses)

<!-- TOC -->

# WGA API Documentation

# Development Server Information

## URL

### Production Server
`https://api.wga.xyz/`

### Develope Server
`https://dev-api.wga.xyz/`

## API Key

Please include the API Key in the `X-WGA-API-Key` header of your requests.

# API List

## Register Verification Event API

`POST /api/verification-events`

This API is used by clients (partner services) to register specific user actions (Social, Game, Mission, Asset, etc.) as verification events. The WGA server stores these events and utilizes them in subsequent verification, batch processing, and on-chain anchoring pipelines.

### Authentication

This API utilizes `WgaProjectContext.projectId()`. Therefore, requests must include project authentication (e.g., API Key or JWT). The `projectId` is extracted from the authentication credentials and injected internally within the server.

### Request Body

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

### Fields

| Field                    | Type            | Required | Description                               |
|--------------------------|-----------------|----------|-------------------------------------------|
| verificationEventType    | enum            | ✅        | Event domain category                                |
| verificationPlatformType | enum            | ✅        | Platform where the event occurred                                    |
| verificationActionType   | enum            | ✅        | Type of user action                                 |
| clientTimestamp          | ISO-8601 string | ✅        | Event occurrence time (Instant) based on the client                  |
| clientUserId             | string          | ✅        | User identifier from the partner service                           |
| ip                       | string          | ❌        | Client IP address                                  |
| userAgent                | string          | ❌        | User-Agent string                                |
| country                  | string          | ❌        | Country code (ISO 3166-1 alpha-2) <br/>e.g., KR, US |
| property                 | object          | ❌        | Detailed metadata specific to the action                              |

| Enum                  | Value                |
|-----------------------|----------------------|
| VerificationEventType | SOCIAL, GAME |

| Enum                     | Value                            |
|--------------------------|----------------------------------|
| VerificationPlatformType | MOBILE, PC, CONSOLE |

| Enum                   | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| VerificationActionType | SIGN_UP, WITHDRAW, LOG_IN, LOG_OUT,<br/>MISSION_ACCEPT, MISSION_CLEAR, MISSION_FAIL,<br/>POST_CREATE, POST_UPDATE, POST_DELETE,<br/>COMMENT_CREATE, COMMENT_UPDATE, COMMENT_DELETE,<br/>LIKE_CREATE, LIKE_DELETE,<br/>ASSET_INFLOW, ASSET_OUTFLOW,<br/>SESSION_START, SESSION_END, STAGE_CLEAR, STAGE_FAIL |

### property Object

The `property` object is optional and contains detailed information matching the action type. Include only the necessary sections as shown below.

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

| Field            | Type            | Description    |
|------------------|-----------------|----------------|
| missionId        | string          | Mission ID          |
| rewards          | string          | Reward info (String format) |
| expiredTimestamp | ISO-8601 string | Expiration time          |
| detail           | string          | Detailed description          |
| failedReasons    | string          | Reasons for failure          |

```json
{
  "mission": {
    "missionId": "m_1001",
    "rewards": "point:100",
    "expiredTimestamp": "2025-12-31T23:59:59Z",
    "detail": "Invite 3 friends",
    "failedReasons": null
  }
}
```

#### social

| Field           | Type   | Description          |
|-----------------|--------|----------------------|
| contentId       | string | Content ID               |
| contentType     | enum   | Type of content               |
| parentContentId | string | ID of the parent content (for replies/nested comments) |
| textLength      | number | Length of the text               |
| mediaCount      | number | Number of attached media files             |      

| Enum        | Value                             |
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

| Field           | Type    | Description |
|-----------------|---------|-------------|
| playDurationSec | number  | Play time in seconds  |
| success         | boolean | Success status       |
| score           | number  | Achieved score          |

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

| Field      | Type          | Description    |
|------------|---------------|----------------|
| type       | enum          | Asset type          |
| name       | string        | Asset name          |
| amount     | string/number | Quantity (Precision consideration recommended) |
| actionType | enum          | Change type          |
| isCash     | boolean       | Whether it is a paid currency       |

| Enum       | Value                                                                 |
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

| Field       | Type   | Description |
|-------------|--------|-------------|
| os          | string | Operating System        |
| osVersion   | string | OS Version       |
| deviceModel | string | Device Model     |
| networkType | enum   | Network type     |

| Enum        | Value                                                                    |
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

### Responses

The controller currently does not return a response body.

- 200 OK: Success
- 403 Forbidden: Authentication failure
