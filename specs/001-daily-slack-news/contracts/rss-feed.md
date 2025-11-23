# GeekNews RSS Feed Contract

**Date**: 2024-11-23  
**Feature**: 001-daily-slack-news

## Overview

This document defines the expected structure of the GeekNews RSS feed.

## Endpoint

**URL**: `https://news.hada.io/rss`  
**Method**: GET  
**Content-Type**: application/rss+xml or application/xml

## Expected Feed Structure

### RSS 2.0 Format

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
  <channel>
    <title>GeekNews</title>
    <link>https://news.hada.io</link>
    <description>...</description>
    <item>
      <title>Article Title</title>
      <link>https://news.hada.io/...</link>
      <description>Article description or summary</description>
      <pubDate>Mon, 23 Nov 2024 08:30:00 +0900</pubDate>
      <guid>unique-article-id</guid>
    </item>
    ...
  </channel>
</rss>
```

## Required Fields per Article

- `title` (string, required): Article title
- `link` (string, required): URL to article
- `description` (string, required): Article summary
- `pubDate` (string, optional): Publication date in RFC 822 format with timezone

## Data Extraction

Using `feedparser` library:

```python
feed = feedparser.parse(rss_url)
for entry in feed.entries:
    title = entry.title
    link = entry.link
    description = entry.description
    pub_date = entry.published_parsed  # struct_time in UTC
    guid = entry.get('id', entry.link)
```

## Timezone Handling

- RSS `pubDate` may include timezone (e.g., `+0900` for KST)
- `feedparser` converts to UTC automatically
- Use `published_parsed` for timezone-aware datetime conversion
- Convert to KST for filtering (last 24 hours)

## Error Handling

- **Network Errors**: Retry with exponential backoff (max 3 retries)
- **Malformed XML**: Log error, skip delivery
- **Missing Required Fields**: Skip article, log reason (FR-018)
- **Timeout**: Retry once, then log and skip delivery
- **404 Not Found**: Log error, notify administrator

## Validation Rules

1. **Title**: Must be non-empty string
2. **Link**: Must be valid URL
3. **Description**: Must be non-empty string
4. **pubDate**: Optional, but if present must be parseable

## Edge Cases

- **Missing pubDate**: Handle per FR-019 (include if total < 10)
- **Invalid URLs**: Skip article
- **Encoding Issues**: feedparser handles automatically, but log if detected
- **Very Long Descriptions**: Truncate to ~100 characters (FR-010)

## Rate Limiting

- RSS feeds typically don't have strict rate limits
- Respect `robots.txt` if present
- Use reasonable User-Agent header
- Cache feed if appropriate (but not for this daily delivery use case)

## Implementation Notes

- Use `feedparser.parse()` for parsing
- Handle `feedparser` exceptions
- Validate all extracted fields
- Log parsing errors
- Support both RSS 2.0 and Atom formats (feedparser handles both)
