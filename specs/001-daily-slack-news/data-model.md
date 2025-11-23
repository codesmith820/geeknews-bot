# Data Model: Daily GeekNews Delivery to Slack

**Date**: 2024-11-23  
**Feature**: 001-daily-slack-news

## Entities

### NewsArticle

Represents a single news item retrieved from GeekNews RSS feed.

**Fields**:

- `title` (string, required): Article title
- `link` (string, required): URL to original article or GeekNews summary page
- `description` (string, required): Article summary or excerpt
- `pub_date` (datetime, optional): Publication date/time in UTC
- `guid` (string, optional): Unique identifier for the article
- `raw_entry` (dict, optional): Original feedparser entry for debugging

**Validation Rules**:

- Must have title, link, and description (FR-018)
- If pub_date is missing, article is only included if total articles < 10 (FR-019)
- Title and link are required for inclusion in delivery

**State Transitions**:

- `fetched` → Retrieved from RSS feed
- `validated` → Passed required field checks
- `filtered` → Passed 24-hour time window check
- `curated` → Selected by LLM (if >50 articles)
- `formatted` → Converted to Slack Block Kit format
- `sent` → Successfully delivered to Slack
- `skipped` → Excluded due to validation failure or filtering

---

### SlackChannelConfig

Represents configuration for a Slack channel delivery target.

**Fields**:

- `channel_id` (string, required): Slack channel identifier
- `webhook_url` (string, required): Incoming Webhook URL for the channel
- `delivery_time` (time, optional): Scheduled delivery time in KST (default: 08:00)
- `enabled` (boolean, default: true): Whether deliveries are active for this channel

**Validation Rules**:

- webhook_url must be valid HTTPS URL
- delivery_time must be in KST timezone
- channel_id must be non-empty

**State Transitions**:

- `configured` → Initial setup complete
- `active` → Enabled and ready for deliveries
- `disabled` → Temporarily disabled
- `error` → Configuration error detected

---

### DeliveryLog

Represents a comprehensive record of each delivery attempt.

**Fields**:

- `timestamp` (datetime, required): Delivery attempt timestamp (UTC)
- `channel_id` (string, required): Target Slack channel
- `articles_processed` (int, required): Total articles retrieved from RSS
- `articles_filtered` (int, required): Articles passing 24-hour filter
- `articles_sent` (int, required): Articles successfully sent to Slack
- `articles_skipped` (int, required): Articles skipped (with reasons)
- `article_titles` (list[string], required): All article titles processed
- `llm_api_calls` (list[dict], optional): LLM API request/response details
- `status` (string, required): "success", "partial", or "failure"
- `error_trace` (string, optional): Full error trace if failure occurred
- `execution_time_seconds` (float, required): Total execution time

**Validation Rules**:

- articles_sent + articles_skipped <= articles_filtered
- status must be one of: "success", "partial", "failure"
- If status is "failure", error_trace must be present

**State Transitions**:

- `started` → Delivery attempt initiated
- `rss_fetched` → RSS feed retrieved
- `articles_filtered` → Time-based filtering complete
- `llm_curated` → LLM curation applied (if needed)
- `slack_sent` → Messages sent to Slack
- `completed` → Delivery attempt finished (success or failure)

---

### DeliverySchedule

Represents the timing configuration for daily news deliveries.

**Fields**:

- `channel_id` (string, required): Associated Slack channel
- `delivery_time_kst` (time, required): Delivery time in KST (default: 08:00)
- `cron_expression` (string, computed): GitHub Actions cron expression (UTC equivalent)
- `timezone` (string, default: "Asia/Seoul"): Timezone for delivery time

**Validation Rules**:

- delivery_time_kst must be valid time
- cron_expression computed as: KST time - 9 hours = UTC time
- Example: 08:00 KST = 23:00 UTC (previous day)

**State Transitions**:

- `configured` → Schedule set up
- `active` → Scheduled and running
- `paused` → Temporarily paused

---

## Relationships

- **NewsArticle** → **DeliveryLog**: Many-to-one (many articles processed in one delivery)
- **SlackChannelConfig** → **DeliverySchedule**: One-to-one (each channel has one schedule)
- **SlackChannelConfig** → **DeliveryLog**: One-to-many (each channel has many delivery logs)

## Data Flow

1. **RSS Feed** → Parse → **NewsArticle[]** (fetched state)
2. **NewsArticle[]** → Validate → **NewsArticle[]** (validated state, some skipped)
3. **NewsArticle[]** → Filter by time → **NewsArticle[]** (filtered state)
4. **NewsArticle[]** → LLM Curation (if >50) → **NewsArticle[]** (curated, max 10)
5. **NewsArticle[]** → Format → **Slack Block Kit JSON**
6. **Slack Block Kit JSON** → Send → **Slack Channel**
7. **All steps** → Log → **DeliveryLog** (completed state)

## Validation Rules Summary

### Article Validation (FR-018)

- Skip if missing title
- Skip if missing link
- Skip if missing description
- Log skip reason

### Time Filtering (FR-002, FR-019)

- Include if pub_date within last 24 hours (KST)
- Include if pub_date missing AND total articles < 10 (place at end)
- Exclude if pub_date missing AND total articles >= 10

### LLM Curation (FR-016)

- Apply if articles_filtered > 50
- Select maximum 10 most relevant articles
- Fallback to most recent 10 if LLM fails (FR-017)

### Message Formatting (FR-012)

- Maximum 50 blocks per Slack message
- With formatting: ~5 blocks per article (header, section, context, divider)
- Maximum ~10 articles per message

## Data Persistence

**Note**: This is a stateless system running on GitHub Actions. No persistent storage is used.

- Configuration: Stored in GitHub Secrets (environment variables)
- Delivery state: Not persisted (time-based filtering prevents duplicates)
- Logs: Output to GitHub Actions logs (no database storage)

If future persistence is needed:

- Consider storing last delivery timestamp in GitHub repository file
- Or use external storage (S3, database) for delivery history
