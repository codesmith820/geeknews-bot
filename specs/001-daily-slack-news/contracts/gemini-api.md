# Google Gemini API Contract

**Date**: 2024-11-23  
**Feature**: 001-daily-slack-news

## Overview

This document defines the contract for using Google Gemini API to curate articles when more than 50 are available.

## Endpoint

**Base URL**: `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent`  
**Method**: POST  
**Authentication**: API Key via `GEMINI_API_KEY` environment variable

## Request Format

### Headers

```
Content-Type: application/json
```

### Request Body

```json
{
  "contents": [
    {
      "parts": [
        {
          "text": "You are a technology news curator. From the following list of articles published in the last 24 hours, select the 10 most relevant and important articles for software developers and technology professionals. Consider factors like: technical significance, industry impact, developer relevance, and news recency.\n\nArticles:\n1. [Title] - [Description]\n2. [Title] - [Description]\n...\n\nRespond with only the numbers of the selected articles (e.g., '1, 3, 5, 7, 9, 11, 13, 15, 17, 19')"
        }
      ]
    }
  ],
  "generationConfig": {
    "temperature": 0.3,
    "maxOutputTokens": 100
  }
}
```

### Prompt Template

```
You are a technology news curator. From the following list of {count} articles published in the last 24 hours, select the 10 most relevant and important articles for software developers and technology professionals. Consider factors like: technical significance, industry impact, developer relevance, and news recency.

Articles:
{article_list}

Respond with only the numbers of the selected articles separated by commas (e.g., '1, 3, 5, 7, 9, 11, 13, 15, 17, 19')
```

## Response Format

### Success Response

**Status Code**: 200 OK

```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "1, 3, 5, 7, 9, 11, 13, 15, 17, 19"
          }
        ]
      }
    }
  ]
}
```

### Error Responses

**Status Code**: 400 Bad Request

```json
{
  "error": {
    "code": 400,
    "message": "Invalid request",
    "status": "INVALID_ARGUMENT"
  }
}
```

**Status Code**: 401 Unauthorized

```json
{
  "error": {
    "code": 401,
    "message": "API key not valid",
    "status": "UNAUTHENTICATED"
  }
}
```

**Status Code**: 429 Too Many Requests

```json
{
  "error": {
    "code": 429,
    "message": "Resource exhausted",
    "status": "RESOURCE_EXHAUSTED"
  }
}
```

## Response Parsing

1. Extract `text` from `candidates[0].content.parts[0].text`
2. Parse comma-separated numbers
3. Map numbers to article indices (1-based to 0-based)
4. Return selected articles

## Error Handling

- **400 Bad Request**: Invalid prompt or request - log error, use fallback (time-based selection)
- **401 Unauthorized**: Invalid API key - notify administrator, use fallback
- **429 Rate Limited**: Retry with exponential backoff (max 2 retries), then use fallback
- **Network Errors**: Retry once, then use fallback
- **Timeout**: Use fallback immediately (FR-017)

## Fallback Strategy

When LLM API fails (FR-017):

- Select first 10 articles by publication time (most recent)
- Log fallback reason
- Continue with delivery

## Rate Limits

- Gemini API: Check current rate limits in documentation
- Implement retry with exponential backoff
- Consider caching if appropriate

## Cost Considerations

- `gemini-2.5-flash`: Cost-effective model
- Estimate: ~1 API call per day (when >50 articles)
- Monitor usage and costs

## Implementation Notes

- Use `google-genai` Python package
- Log all API requests and responses (FR-014)
- Handle all error cases gracefully
- Always have fallback ready
- Set appropriate timeout (e.g., 30 seconds)
