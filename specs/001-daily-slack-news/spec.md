# Feature Specification: Daily GeekNews Delivery to Slack

**Feature Branch**: `001-daily-slack-news`  
**Created**: 2024-11-23  
**Status**: Draft  
**Input**: User description: "geeknews-bot의 news를 매일 최근 24시간 이내에 나온 뉴스를 slack channel로 전송하는 project"

## Clarifications

### Session 2024-11-23

- Q: When there are no new articles in the last 24 hours, should the system send a notification message to Slack or remain silent? → A: Send no message when there are no articles (silent skip)
- Q: When more than 50 articles are found in the last 24 hours, how should the system handle the Slack message block limit? → A: Use LLM (Gemini API) to curate and summarize articles, displaying maximum 10 most relevant articles
- Q: When an article is missing a required field (title, link, or description), how should the system handle it? → A: Skip any article missing title, link, or description
- Q: What specific information should be logged for each delivery attempt? → A: Log comprehensive details including all article titles, LLM API calls, and full error traces
- Q: When an article has no publication time, how should the system handle it for the 24-hour filtering? → A: Include articles with missing publication time only if total articles are less than 10

## User Scenarios & Testing _(mandatory)_

### User Story 1 - Receive Daily News Digest (Priority: P1)

As a team member, I want to receive a daily digest of recent GeekNews articles in my Slack channel, so that I can stay informed about the latest technology trends without manually visiting the website.

**Why this priority**: This is the core value proposition of the feature - automated delivery of curated news content directly to where team members already work, eliminating the need for active information seeking.

**Independent Test**: Can be fully tested by verifying that news articles from the last 24 hours are successfully delivered to the configured Slack channel at the scheduled time, and users can see formatted messages with article titles, links, and summaries.

**Acceptance Scenarios**:

1. **Given** the system is configured with a valid Slack channel, **When** the scheduled daily delivery time arrives, **Then** the system retrieves news articles published within the last 24 hours and sends them to the Slack channel
2. **Given** there are new articles published in the last 24 hours, **When** the daily delivery runs, **Then** users see formatted messages in Slack containing article titles, clickable links, and brief summaries
3. **Given** there are no new articles in the last 24 hours, **When** the daily delivery runs, **Then** no message is sent to Slack (silent skip)
4. **Given** the system has previously sent articles, **When** the daily delivery runs again, **Then** only articles from the last 24 hours are included (no duplicates from previous deliveries)

---

### User Story 2 - View Formatted News Messages (Priority: P1)

As a team member, I want to see news articles in a well-formatted, readable layout in Slack, so that I can quickly scan and identify articles of interest.

**Why this priority**: The formatting directly impacts user experience and determines whether the feature is actually useful or just creates noise. Poor formatting would reduce adoption.

**Independent Test**: Can be fully tested by verifying that messages sent to Slack display with proper formatting including headers, article titles as clickable links, truncated summaries, and publication timestamps.

**Acceptance Scenarios**:

1. **Given** news articles are being sent to Slack, **When** messages are delivered, **Then** each article is displayed with a clear title that links to the original article
2. **Given** an article has a long description, **When** the message is formatted, **Then** the description is truncated to a reasonable length (approximately 100 characters) with an ellipsis indicator
3. **Given** multiple articles are being sent, **When** messages are delivered, **Then** articles are clearly separated and organized in a readable format
4. **Given** messages include publication information, **When** messages are displayed, **Then** publication dates and times are shown in a consistent, readable format
5. **Given** more than 50 articles are available in the last 24 hours, **When** the system processes articles, **Then** LLM curation is used to select and display the 10 most relevant articles in a single message

---

### User Story 3 - Handle Delivery Failures Gracefully (Priority: P2)

As a system administrator, I want the system to handle errors gracefully when news delivery fails, so that I can be notified of issues and users understand when content is unavailable.

**Why this priority**: Error handling ensures system reliability and maintainability. Without proper error handling, failures could go unnoticed and degrade user trust.

**Independent Test**: Can be fully tested by simulating various failure scenarios (network issues, invalid credentials, source unavailability) and verifying appropriate error handling and notification mechanisms.

**Acceptance Scenarios**:

1. **Given** the news source is temporarily unavailable, **When** the system attempts to retrieve news, **Then** the system handles the error gracefully without crashing and logs the issue appropriately
2. **Given** Slack integration credentials are invalid, **When** the system attempts to send messages, **Then** the system detects the authentication failure and notifies administrators
3. **Given** network connectivity issues occur during delivery, **When** the system encounters the error, **Then** the system retries the operation with appropriate backoff or logs the failure for manual intervention
4. **Given** the news source returns malformed or unexpected data, **When** the system processes the data, **Then** the system handles the error gracefully and continues processing valid articles or skips the delivery with appropriate logging

---

### User Story 5 - Configure Delivery Schedule (Priority: P2)

As a team administrator, I want to configure the delivery time for each Slack channel, so that different teams can receive news at times that work best for their schedules.

**Why this priority**: While not essential for MVP, configurability allows teams to optimize when they receive information based on their work patterns and time zones. This enhances adoption and user satisfaction.

**Independent Test**: Can be fully tested by verifying that administrators can set a custom delivery time for a Slack channel, the system uses the configured time for that channel, and channels without configuration use the default time (8:00 AM KST).

**Acceptance Scenarios**:

1. **Given** a Slack channel is configured with a custom delivery time, **When** the system schedules deliveries, **Then** news is sent to that channel at the configured time
2. **Given** a Slack channel has no custom delivery time configured, **When** the system schedules deliveries, **Then** news is sent to that channel at the default time (8:00 AM KST)
3. **Given** multiple Slack channels are configured with different delivery times, **When** the system runs, **Then** each channel receives news at its configured time
4. **Given** an administrator changes a channel's delivery time, **When** the next scheduled delivery occurs, **Then** the new delivery time is used

---

### User Story 4 - Filter News by Time Window (Priority: P1)

As a team member, I want to receive only news from the last 24 hours, so that I'm not overwhelmed with old content and can focus on recent developments.

**Why this priority**: Time-based filtering is essential to prevent duplicate deliveries and ensure users receive only relevant, recent content. This is a core requirement stated in the feature description.

**Independent Test**: Can be fully tested by verifying that only articles published within the last 24 hours (based on Korean Standard Time) are included in daily deliveries, and articles older than 24 hours are excluded.

**Acceptance Scenarios**:

1. **Given** articles were published 25 hours ago and 2 hours ago, **When** the daily delivery runs, **Then** only the article from 2 hours ago is included
2. **Given** the system runs at different times of day, **When** articles are filtered, **Then** the 24-hour window is calculated correctly relative to the current time
3. **Given** articles have publication times in different time zones, **When** articles are filtered, **Then** all times are normalized correctly to ensure accurate 24-hour filtering
4. **Given** an article has no publication time and total articles are less than 10, **When** the system processes articles, **Then** the article is included and placed at the end of the list
5. **Given** an article has no publication time and total articles are 10 or more, **When** the system processes articles, **Then** the article is excluded (cannot verify 24-hour window)
6. **Given** an article is missing title, link, or description, **When** the system processes articles, **Then** the article is skipped and the skip is logged for monitoring

---

### Edge Cases

- What happens when more than 50 news articles are found in the last 24 hours? → System uses LLM curation to select and display maximum 10 most relevant articles
- How does the system handle articles with missing titles, links, or descriptions? → System skips any article missing title, link, or description and logs the skip for monitoring
- What happens when the news source changes its data format or structure?
- How does the system handle special characters, emojis, or non-standard text encoding in article content?
- What happens if the scheduled delivery time is missed or delayed?
- How does the system handle multiple consecutive delivery failures?
- What happens when LLM API is unavailable or returns an error? → System falls back to sending first 10 articles by publication time (most recent)
- What happens when the news source is permanently unavailable or the feed URL changes?
- How does the system handle articles with extremely long titles or descriptions?
- What happens when Slack channel permissions change or the channel is deleted?
- How does the system handle timezone changes (daylight saving time, etc.)?

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST retrieve news articles from the GeekNews source on a daily schedule
- **FR-002**: System MUST filter articles to include only those published within the last 24 hours (based on Korean Standard Time)
- **FR-003**: System MUST send filtered news articles to a configured Slack channel
- **FR-004**: System MUST format messages with article titles, clickable links, and brief summaries
- **FR-005**: System MUST skip sending messages when no new articles are available in the 24-hour window (silent skip, no notification sent)
- **FR-006**: System MUST prevent duplicate article delivery across multiple daily runs
- **FR-007**: System MUST handle errors gracefully when the news source is unavailable
- **FR-008**: System MUST handle errors gracefully when Slack integration fails
- **FR-017**: System MUST handle errors gracefully when LLM API is unavailable or fails, falling back to sending first 10 articles by publication time (most recent)
- **FR-018**: System MUST skip articles that are missing any required field (title, link, or description) and log skipped articles for monitoring
- **FR-019**: System MUST include articles with missing publication time only when total available articles are less than 10, placing them at the end of the list; otherwise exclude them (cannot verify 24-hour window)
- **FR-009**: System MUST properly handle Korean text encoding and special characters
- **FR-010**: System MUST truncate long article descriptions to maintain readability (approximately 100 characters)
- **FR-011**: System MUST include publication timestamps in messages when available
- **FR-012**: System MUST respect Slack message formatting limits (maximum 50 blocks per message)
- **FR-016**: When more than 50 articles are available, System MUST use LLM-based curation to select and summarize the most relevant articles, displaying a maximum of 10 curated articles
- **FR-013**: System MUST handle timezone conversions correctly (KST to UTC and vice versa)
- **FR-014**: System MUST log comprehensive details for each delivery attempt including: timestamp, article count, all article titles processed, success/failure status, LLM API calls (requests and responses), skipped articles with reasons, and full error traces when failures occur
- **FR-015**: System MUST allow delivery schedule timing to be configured per Slack channel, with a default delivery time of 8:00 AM KST

### Key Entities _(include if feature involves data)_

- **News Article**: Represents a single news item from GeekNews, containing title, link, description/summary, publication date/time, and unique identifier
- **Delivery Schedule**: Represents the timing configuration for when daily news deliveries should occur
- **Slack Channel Configuration**: Represents the target Slack channel, integration credentials for message delivery, and configured delivery time (defaults to 8:00 AM KST if not specified)
- **Delivery Log**: Represents a comprehensive record of each delivery attempt, including timestamp, number of articles processed and sent, all article titles, LLM API interaction details, skipped articles with reasons, success/failure status, and complete error traces

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: System successfully delivers news articles to Slack channel on schedule 95% of the time over a 30-day period
- **SC-002**: Users receive news articles within 5 minutes of the scheduled delivery time
- **SC-003**: All articles included in deliveries are from within the last 24 hours (100% accuracy)
- **SC-004**: Zero duplicate articles are delivered across consecutive daily runs
- **SC-005**: System handles and recovers from temporary source unavailability (network errors, timeouts) without manual intervention in 90% of cases
- **SC-006**: Messages are formatted correctly and readable in Slack (no formatting errors, broken links, or encoding issues) for 99% of delivered articles
- **SC-007**: When more than 50 articles are available, system successfully uses LLM curation to deliver the 10 most relevant articles in a single message
- **SC-008**: System processes and delivers up to 100 articles per day without performance degradation

## Assumptions

- GeekNews provides a reliable RSS feed at a consistent URL
- Slack workspace and channel are already set up and accessible
- Delivery schedule will be once per day per channel at a configurable time
- Default delivery time of 8:00 AM KST aligns with users preferring to receive news in the morning (before work hours) to review during the day
- Channel administrators can configure delivery times to match their team's preferences
- Korean Standard Time (KST, UTC+9) is the appropriate timezone for filtering and display
- News source maintains consistent data structure and encoding
- Slack integration credentials can be securely stored and accessed
- System has network access to both GeekNews source and Slack API
- Users have appropriate permissions to view the target Slack channel
- Article descriptions can be safely truncated at approximately 100 characters without losing critical information
- LLM API (Google Gemini) is available and accessible for article curation when needed
- LLM curation will select the most relevant and important articles from a larger set

## Dependencies

- Access to GeekNews RSS feed (https://news.hada.io/rss)
- Slack workspace with appropriate permissions
- Slack integration credentials (webhook URL or bot token)
- LLM API access (Google Gemini API) for article curation and summarization
- System capable of running scheduled tasks
- Network connectivity to GeekNews, Slack services, and LLM API

## Out of Scope

- User interaction with the bot (e.g., requesting specific news, filtering by topic)
- Storing news articles in a database for historical access
- Supporting multiple news sources beyond GeekNews
- Customizable delivery schedules per individual user (channel-level configuration is in scope)
- News article search or filtering capabilities
- Analytics or reporting on article engagement
- Integration with other messaging platforms (Discord, Teams, etc.)
- Advanced AI-powered article analysis beyond basic curation (LLM-based curation for article selection when >50 articles is in scope)
- User preferences for article categories or topics
