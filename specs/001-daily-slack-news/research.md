# Research: Daily GeekNews Delivery to Slack

**Date**: 2024-11-23  
**Feature**: 001-daily-slack-news

## Technology Decisions

### RSS Feed Parsing

**Decision**: Use `feedparser` library for RSS/Atom feed parsing

**Rationale**:

- Industry standard for RSS/Atom parsing in Python
- Handles malformed XML gracefully with fault tolerance
- Automatically handles encoding detection and conversion
- Supports both RSS 2.0 and Atom formats
- Well-maintained and widely used

**Alternatives considered**:

- Manual XML parsing with `xml.etree.ElementTree`: More control but requires handling RSS/Atom variations manually
- `beautifulsoup4`: Overkill for structured RSS feeds, better for HTML scraping
- Custom parser: Unnecessary complexity for standard RSS format

**Implementation notes**:

- Use `feedparser.parse()` with RSS URL
- Handle `feedparser` exceptions for malformed feeds
- Access entries via `feed.entries` list
- Use `published_parsed` for timezone-aware date handling

---

### HTTP Client Library

**Decision**: Use `requests` library for HTTP operations

**Rationale**:

- De facto standard for HTTP in Python
- Simple API with good error handling
- Supports timeout configuration
- User-Agent header customization
- Better control than feedparser's built-in HTTP

**Alternatives considered**:

- `httpx`: Modern async alternative, but overkill for scheduled sync operations
- `urllib`: Built-in but more verbose and less user-friendly
- `feedparser` built-in HTTP: Less control over headers and error handling

**Implementation notes**:

- Set appropriate User-Agent header
- Configure timeout (e.g., 30 seconds)
- Handle `requests.exceptions` for network errors
- Use session for connection pooling if needed

---

### Timezone Handling

**Decision**: Use `pytz` library for timezone conversions

**Rationale**:

- Comprehensive timezone database (IANA timezone database)
- Handles KST (Asia/Seoul) and UTC conversions accurately
- Supports daylight saving time transitions
- Well-tested and reliable

**Alternatives considered**:

- `zoneinfo` (Python 3.9+): Modern alternative but requires system timezone data
- Manual offset calculation: Error-prone, doesn't handle DST
- `dateutil.tz`: Good alternative but pytz is more widely used

**Implementation notes**:

- Use `pytz.timezone('Asia/Seoul')` for KST
- Use `pytz.utc` for UTC
- Convert RSS pubDate to UTC for comparison
- Format times in KST for display

---

### LLM API Integration

**Decision**: Use Google Gemini API (via `google-genai` package) for article curation

**Rationale**:

- User specified Gemini API in clarifications
- Good performance for text summarization and selection tasks
- Cost-effective for curation use case
- Reliable API with good documentation

**Alternatives considered**:

- OpenAI GPT API: More expensive, similar capabilities
- Local LLM models: Requires infrastructure, slower inference
- Rule-based filtering: Less intelligent, doesn't understand article relevance

**Implementation notes**:

- Use `gemini-2.5-flash` model for cost efficiency
- Implement fallback to time-based selection if API fails
- Cache API responses if appropriate
- Handle rate limits and API errors gracefully
- Prompt engineering: "Select the 10 most relevant and important technology news articles from this list..."

---

### Slack Integration

**Decision**: Use `slack-sdk` (Python Slack SDK) for Slack API integration

**Rationale**:

- Official Slack SDK for Python
- Supports both Webhooks and Web API
- Built-in Block Kit support
- Good error handling and retry logic
- Well-maintained by Slack

**Alternatives considered**:

- Direct HTTP requests to Slack Webhook: More control but more code
- `slackclient` (legacy): Deprecated, replaced by slack-sdk
- Custom implementation: Unnecessary when official SDK exists

**Implementation notes**:

- Use Incoming Webhooks for simple message posting (initial implementation)
- Use Block Kit for rich message formatting
- Handle Slack API rate limits
- Implement retry logic for transient failures
- Support both webhook URL and bot token methods

---

### Scheduling Platform

**Decision**: Use GitHub Actions for scheduled execution

**Rationale**:

- Free for public repositories
- Integrated with code repository
- No server infrastructure needed
- Built-in cron scheduling
- Easy secret management
- Stateless execution aligns with feature requirements

**Alternatives considered**:

- AWS Lambda + EventBridge: More complex setup, costs money
- Cron on VPS: Requires server maintenance and costs
- Heroku Scheduler: Platform dependency, costs money
- Local cron: Requires always-on machine

**Implementation notes**:

- Use `schedule` trigger with cron syntax
- Set time to avoid UTC 00:00 peak (e.g., 23:05 UTC = 08:05 KST)
- Store Slack webhook URL and Gemini API key as GitHub Secrets
- Handle workflow_dispatch for manual testing
- Use ubuntu-latest runner

---

### Message Formatting

**Decision**: Use Slack Block Kit for message formatting

**Rationale**:

- Official Slack UI framework
- Rich, structured message layouts
- Better UX than plain text
- Supports headers, sections, dividers, context blocks
- 50 block limit per message (handled by curation)

**Alternatives considered**:

- Plain text messages: Less readable, poor UX
- Attachments (legacy): Deprecated, Block Kit is modern approach
- Custom formatting: Unnecessary when Block Kit provides all needed components

**Implementation notes**:

- Header block for date/title
- Section blocks for each article (title + link + truncated description)
- Context block for publication time
- Divider blocks for separation
- Respect 50 block limit (max ~10 articles per message with formatting)

---

### Error Handling Strategy

**Decision**: Comprehensive logging with graceful degradation

**Rationale**:

- Stateless system (GitHub Actions) requires detailed logs for debugging
- Multiple external dependencies (RSS, Slack, LLM) need robust error handling
- User experience should not break on partial failures

**Implementation notes**:

- Log all operations with timestamps
- Log LLM API requests/responses for debugging
- Log skipped articles with reasons
- Implement fallbacks: LLM failure → time-based selection, RSS failure → skip delivery, Slack failure → log and retry
- Never crash silently - always log errors

---

### Configuration Management

**Decision**: Environment variables + GitHub Secrets

**Rationale**:

- GitHub Actions provides secure secret storage
- Environment variables are standard for cloud/serverless
- No need for config files in stateless execution
- Easy to update without code changes

**Alternatives considered**:

- Config files: Requires storage, more complex in serverless
- Database: Overkill for simple key-value config
- Hardcoded values: Security risk, not flexible

**Implementation notes**:

- Use `os.environ.get()` for configuration
- Required: `SLACK_WEBHOOK_URL`, `GEMINI_API_KEY`
- Optional: `RSS_URL` (default to GeekNews), `DEFAULT_DELIVERY_TIME` (default to 08:00 KST)
- Validate required config on startup

---

## Integration Patterns

### RSS Feed → Article Processing Flow

1. Fetch RSS feed with `feedparser.parse()`
2. Parse entries and extract: title, link, description, pubDate, guid
3. Filter by publication time (last 24 hours, KST)
4. Validate required fields (title, link, description)
5. Handle missing publication times per FR-019
6. Pass to curation if >50 articles

### LLM Curation Flow

1. Prepare article list with titles and descriptions
2. Call Gemini API with prompt: "Select 10 most relevant articles..."
3. Parse API response to get selected article indices/IDs
4. Fallback to time-based selection (most recent 10) if API fails
5. Log API call details for monitoring

### Slack Message Formatting Flow

1. Build Block Kit structure:
   - Header: "📅 YYYY-MM-DD GeekNews 브리핑"
   - Divider
   - For each article: Section block with title (link), truncated description, context block with time
   - Divider between articles
2. Validate block count (max 50)
3. Send via Slack Webhook API
4. Handle rate limits and retries

### Error Recovery Patterns

- **RSS Feed Unavailable**: Log error, skip delivery, don't crash
- **LLM API Failure**: Log error, fallback to time-based selection, continue
- **Slack API Failure**: Log error, retry with exponential backoff (max 3 retries)
- **Malformed Data**: Skip invalid articles, log reason, continue with valid ones
- **Network Timeout**: Retry once, then log and skip

---

## Performance Considerations

- **RSS Parsing**: Should complete in <5 seconds for typical feed
- **LLM API Call**: May take 10-30 seconds, acceptable for daily delivery
- **Slack Message Send**: <2 seconds per message
- **Total Execution Time**: Target <2 minutes for 100 articles, well within GitHub Actions limits

## Security Considerations

- Store API keys in GitHub Secrets (never in code)
- Use HTTPS for all external API calls
- Validate and sanitize RSS feed data before processing
- Don't log sensitive information (API keys, full webhook URLs)
- Use environment variables for all configuration

## Testing Strategy

- **Unit Tests**: Mock external APIs (RSS, Slack, LLM), test filtering logic, timezone conversions
- **Integration Tests**: Test full flow with test Slack channel and mock RSS feed
- **Error Scenario Tests**: Test all error paths and fallbacks
- **Performance Tests**: Verify execution time with large article sets
