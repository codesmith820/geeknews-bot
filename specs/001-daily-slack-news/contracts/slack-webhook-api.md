# Slack Incoming Webhook API Contract

**Date**: 2024-11-23  
**Feature**: 001-daily-slack-news

## Overview

This document defines the contract for sending messages to Slack via Incoming Webhooks.

## Endpoint

**URL**: Provided via `SLACK_WEBHOOK_URL` environment variable  
**Method**: POST  
**Content-Type**: application/json

## Request Format

### Block Kit Message Structure

```json
{
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "📅 2024-11-23 GeekNews 브리핑"
      }
    },
    {
      "type": "divider"
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*<https://example.com/article|Article Title>*\nArticle description truncated to ~100 characters..."
      }
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "plain_text",
          "text": "발행: 2024-11-23 08:30",
          "emoji": true
        }
      ]
    },
    {
      "type": "divider"
    }
  ]
}
```

### Constraints

- Maximum 50 blocks per message (FR-012)
- Each article uses ~5 blocks (section, context, divider)
- Maximum ~10 articles per message

## Response Format

### Success Response

**Status Code**: 200 OK

**Body**: `ok`

### Error Responses

**Status Code**: 400 Bad Request

```json
{
  "ok": false,
  "error": "invalid_blocks"
}
```

**Status Code**: 401 Unauthorized

```json
{
  "ok": false,
  "error": "invalid_token"
}
```

**Status Code**: 429 Too Many Requests

```json
{
  "ok": false,
  "error": "rate_limited"
}
```

## Error Handling

- **400 Bad Request**: Invalid message format - log error, skip delivery
- **401 Unauthorized**: Invalid webhook URL - notify administrator, log error
- **429 Rate Limited**: Retry with exponential backoff (max 3 retries)
- **Network Errors**: Retry with exponential backoff (max 3 retries)
- **Timeout**: Retry once, then log and skip

## Rate Limits

- Slack Webhooks: ~1 message per second per webhook
- For multiple channels: Send sequentially with small delay

## Implementation Notes

- Use `requests.post()` with JSON payload
- Set `Content-Type: application/json` header
- Handle all error status codes
- Implement retry logic for transient failures
- Log all API calls for monitoring (FR-014)
