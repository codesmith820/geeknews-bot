# Tasks: Daily GeekNews Delivery to Slack

**Input**: Design documents from `/specs/001-daily-slack-news/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Tests are OPTIONAL for this feature. Unit and integration tests can be added in the Polish phase if needed.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US4, US3, US5)
- Include exact file paths in descriptions

## Path Conventions

- **Single project**: `src/`, `tests/` at repository root
- Paths shown below follow the single project structure from plan.md

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project directory structure (src/, tests/unit/, tests/integration/, .github/workflows/)
- [ ] T002 Create requirements.txt with dependencies: feedparser>=6.0.10, requests>=2.31.0, pytz>=2023.3, google-genai>=0.2.0, slack-sdk>=3.23.0, pytest>=7.4.0
- [ ] T003 [P] Create .github/workflows/daily-news.yml workflow file structure
- [ ] T004 [P] Create README.md with project overview and setup instructions
- [ ] T005 [P] Create .gitignore for Python project

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T006 Create src/config.py for environment variable management and configuration loading
- [ ] T007 Create src/logger.py for comprehensive logging utilities (supports FR-014 requirements)
- [ ] T008 Implement configuration validation in src/config.py (validate SLACK_WEBHOOK_URL, GEMINI_API_KEY)
- [ ] T009 [P] Create base error handling module in src/errors.py for custom exceptions

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Receive Daily News Digest (Priority: P1) 🎯 MVP

**Goal**: Core functionality to retrieve GeekNews articles from RSS feed and send them to Slack channel

**Independent Test**: Run bot.py manually with test Slack webhook - verify articles from last 24 hours are delivered to Slack channel

### Implementation for User Story 1

- [ ] T010 [P] [US1] Create src/rss_parser.py with RSS feed fetching using feedparser
- [ ] T011 [US1] Implement fetch_rss_feed() function in src/rss_parser.py to retrieve and parse RSS feed from https://news.hada.io/rss
- [ ] T012 [P] [US1] Create src/slack_client.py with Slack webhook integration using slack-sdk
- [ ] T013 [US1] Implement send_to_slack() function in src/slack_client.py to send messages via webhook URL
- [ ] T014 [US1] Create src/bot.py main entry point that orchestrates RSS fetch and Slack delivery
- [ ] T015 [US1] Implement main() function in src/bot.py that calls RSS parser and Slack client
- [ ] T016 [US1] Add GitHub Actions workflow in .github/workflows/daily-news.yml with cron schedule (default: 23:05 UTC = 08:05 KST)
- [ ] T017 [US1] Configure workflow to inject SLACK_WEBHOOK_URL and GEMINI_API_KEY from GitHub Secrets
- [ ] T018 [US1] Add workflow step to install dependencies and run bot.py

**Checkpoint**: At this point, User Story 1 should be fully functional - RSS feed is fetched and basic message is sent to Slack

---

## Phase 4: User Story 2 - View Formatted News Messages (Priority: P1)

**Goal**: Format news articles using Slack Block Kit for better readability and user experience

**Independent Test**: Send test articles to Slack - verify Block Kit formatting with headers, sections, context blocks, and dividers

### Implementation for User Story 2

- [ ] T019 [US2] Create create_slack_blocks() function in src/slack_client.py to build Block Kit structure
- [ ] T020 [US2] Implement header block in create_slack_blocks() with date format "📅 YYYY-MM-DD GeekNews 브리핑"
- [ ] T021 [US2] Implement section blocks in create_slack_blocks() for each article with title (markdown link), truncated description (~100 chars)
- [ ] T022 [US2] Implement context blocks in create_slack_blocks() for publication timestamps in KST format
- [ ] T023 [US2] Implement divider blocks in create_slack_blocks() to separate articles
- [ ] T024 [US2] Add description truncation logic in src/slack_client.py (FR-010: ~100 characters with ellipsis)
- [ ] T025 [US2] Update send_to_slack() in src/slack_client.py to use Block Kit format instead of plain text
- [ ] T026 [US2] Add validation in create_slack_blocks() to respect 50 block limit per message (FR-012)

**Checkpoint**: At this point, User Stories 1 AND 2 should work together - formatted messages are sent to Slack

---

## Phase 5: User Story 4 - Filter News by Time Window (Priority: P1)

**Goal**: Filter articles to include only those published within the last 24 hours (KST) to prevent duplicates and ensure relevance

**Independent Test**: Test with mock articles having different publication times - verify only articles from last 24 hours are included

### Implementation for User Story 4

- [ ] T027 [US4] Create filter_recent_news() function in src/rss_parser.py for 24-hour time window filtering
- [ ] T028 [US4] Implement timezone conversion logic in filter_recent_news() using pytz (KST to UTC)
- [ ] T029 [US4] Add 24-hour cutoff calculation in filter_recent_news() (current time - 24 hours in UTC)
- [ ] T030 [US4] Implement article validation in filter_recent_news() to skip articles missing title, link, or description (FR-018)
- [ ] T031 [US4] Add logic in filter_recent_news() to handle articles with missing pub_date per FR-019 (include if total < 10, exclude if >= 10)
- [ ] T032 [US4] Implement article sorting in filter_recent_news() (most recent first, articles without pub_date at end if included)
- [ ] T033 [US4] Add logging in filter_recent_news() for skipped articles with reasons
- [ ] T034 [US4] Update src/bot.py to call filter_recent_news() after RSS fetch and before Slack delivery
- [ ] T035 [US4] Add handling in src/bot.py for empty article list (silent skip per FR-005 - no message sent)

**Checkpoint**: At this point, User Stories 1, 2, AND 4 should work together - only recent articles are delivered

---

## Phase 6: User Story 3 - Handle Delivery Failures Gracefully (Priority: P2)

**Goal**: Implement comprehensive error handling and logging for all failure scenarios to ensure system reliability

**Independent Test**: Simulate various failures (network errors, invalid credentials, malformed data) - verify graceful handling and logging

### Implementation for User Story 3

- [ ] T036 [US3] Add error handling in src/rss_parser.py for network timeouts and connection errors (FR-007)
- [ ] T037 [US3] Add error handling in src/rss_parser.py for malformed RSS feed data
- [ ] T038 [US3] Add retry logic in src/rss_parser.py with exponential backoff for transient network failures
- [ ] T039 [US3] Add error handling in src/slack_client.py for invalid webhook URL and authentication failures (FR-008)
- [ ] T040 [US3] Add retry logic in src/slack_client.py with exponential backoff for rate limits and transient failures
- [ ] T041 [US3] Implement comprehensive logging in src/logger.py for all operations (FR-014: timestamp, article count, titles, status)
- [ ] T042 [US3] Add error logging in src/bot.py with full error traces for debugging
- [ ] T043 [US3] Implement DeliveryLog structure in src/logger.py to track delivery attempts (articles_processed, articles_filtered, articles_sent, articles_skipped, status, error_trace)
- [ ] T044 [US3] Add logging for skipped articles with reasons in src/rss_parser.py
- [ ] T045 [US3] Update src/bot.py to catch and log all exceptions without crashing

**Checkpoint**: At this point, all error scenarios are handled gracefully with comprehensive logging

---

## Phase 7: User Story 5 - Configure Delivery Schedule (Priority: P2)

**Goal**: Allow per-channel delivery time configuration with default of 8:00 AM KST

**Independent Test**: Configure different delivery times for test channels - verify deliveries occur at configured times

### Implementation for User Story 5

- [ ] T046 [US5] Create SlackChannelConfig data structure in src/config.py (channel_id, webhook_url, delivery_time, enabled)
- [ ] T047 [US5] Add support for multiple channel configurations in src/config.py (read from environment variables or config file)
- [ ] T048 [US5] Implement default delivery time handling in src/config.py (8:00 AM KST if not specified)
- [ ] T049 [US5] Add timezone conversion logic in src/config.py for delivery_time (KST to UTC for cron)
- [ ] T050 [US5] Update .github/workflows/daily-news.yml to support multiple workflow schedules or dynamic scheduling
- [ ] T051 [US5] Update src/bot.py to iterate over configured channels and send to each at appropriate time
- [ ] T052 [US5] Add channel-specific logging in src/logger.py to track deliveries per channel

**Checkpoint**: At this point, multiple channels can be configured with different delivery times

---

## Phase 8: LLM Curation (Part of User Story 2)

**Goal**: Use Gemini API to curate articles when more than 50 are available, selecting maximum 10 most relevant

**Independent Test**: Test with >50 articles - verify LLM curation selects 10 articles, fallback works if API fails

### Implementation for LLM Curation

- [ ] T053 [US2] Create src/llm_curator.py for Gemini API integration
- [ ] T054 [US2] Implement curate_articles() function in src/llm_curator.py using google-genai client
- [ ] T055 [US2] Add prompt engineering in curate_articles() to select 10 most relevant articles for developers
- [ ] T056 [US2] Implement response parsing in curate_articles() to extract selected article indices
- [ ] T057 [US2] Add fallback logic in curate_articles() to select most recent 10 articles if LLM API fails (FR-017)
- [ ] T058 [US2] Add error handling in curate_articles() for API failures, timeouts, rate limits
- [ ] T059 [US2] Add logging in curate_articles() for LLM API requests and responses (FR-014)
- [ ] T060 [US2] Update src/bot.py to call curate_articles() when filtered articles > 50 (FR-016)
- [ ] T061 [US2] Integrate LLM curation output into Slack message formatting in src/slack_client.py

**Checkpoint**: At this point, LLM curation is integrated and handles large article volumes

---

## Phase 9: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T062 [P] Add Korean text encoding handling in src/rss_parser.py for proper UTF-8 processing (FR-009)
- [ ] T063 [P] Add Unicode normalization in src/rss_parser.py to handle Korean character encoding issues
- [ ] T064 [P] Add validation for extremely long titles/descriptions in src/slack_client.py
- [ ] T065 [P] Add handling for special characters and emojis in article content
- [ ] T066 [P] Update README.md with complete setup instructions from quickstart.md
- [ ] T067 [P] Add code comments and docstrings throughout all modules
- [ ] T068 [P] Create tests/unit/test_rss_parser.py with unit tests for RSS parsing
- [ ] T069 [P] Create tests/unit/test_slack_client.py with unit tests for Slack formatting
- [ ] T070 [P] Create tests/unit/test_llm_curator.py with unit tests for LLM curation
- [ ] T071 [P] Create tests/integration/test_delivery_flow.py with end-to-end integration test
- [ ] T072 [P] Add performance monitoring and execution time logging
- [ ] T073 [P] Validate quickstart.md instructions by following them step-by-step
- [ ] T074 [P] Add error notification mechanism for administrators (e.g., email or separate Slack channel)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (US1 → US2 → US4 → US3 → US5)
- **LLM Curation (Phase 8)**: Depends on US2 (formatting) but can be integrated later
- **Polish (Phase 9)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories - **MVP CORE**
- **User Story 2 (P1)**: Depends on US1 (needs articles to format) - Can start after US1
- **User Story 4 (P1)**: Can start after Foundational (Phase 2) - Integrates with US1/US2 - **CRITICAL for functionality**
- **User Story 3 (P2)**: Can start after US1 - Enhances all stories with error handling
- **User Story 5 (P2)**: Can start after US1 - Adds configuration capability

### Within Each User Story

- Core components before integration
- Error handling after core functionality
- Logging integrated throughout
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel (T003, T004, T005)
- All Foundational tasks marked [P] can run in parallel (T009)
- Once Foundational phase completes:
  - US1 and US4 can start in parallel (different concerns)
  - US2 depends on US1 but can start after US1 core is done
  - US3 can enhance existing stories in parallel
  - US5 can be developed in parallel with other stories
- LLM Curation (Phase 8) can be developed in parallel with US3/US5
- All Polish tasks marked [P] can run in parallel

---

## Parallel Example: User Story 1

```bash
# Launch foundational components in parallel:
Task: "Create src/config.py for environment variable management"
Task: "Create src/logger.py for comprehensive logging utilities"
Task: "Create base error handling module in src/errors.py"

# Launch US1 components that can be parallel:
Task: "Create src/rss_parser.py with RSS feed fetching"
Task: "Create src/slack_client.py with Slack webhook integration"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1 (basic RSS → Slack delivery)
4. **STOP and VALIDATE**: Test User Story 1 independently with test Slack webhook
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo (MVP!)
3. Add User Story 4 → Test independently → Deploy/Demo (Time filtering)
4. Add User Story 2 → Test independently → Deploy/Demo (Formatting)
5. Add LLM Curation → Test independently → Deploy/Demo (Large volume handling)
6. Add User Story 3 → Test independently → Deploy/Demo (Error handling)
7. Add User Story 5 → Test independently → Deploy/Demo (Multi-channel)
8. Each story adds value without breaking previous stories

### Recommended MVP Scope

**Minimum Viable Product**: Phases 1, 2, 3, 4, 5

- Setup and Foundational infrastructure
- User Story 1: Basic RSS → Slack delivery
- User Story 4: 24-hour time filtering
- User Story 2: Block Kit formatting

This delivers a fully functional daily news bot that:

- Retrieves articles from RSS feed
- Filters to last 24 hours
- Formats and sends to Slack
- Handles basic errors

**Extended MVP**: Add Phase 8 (LLM Curation) for handling large article volumes

**Full Feature**: All phases including error handling (US3) and multi-channel configuration (US5)

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1 (RSS + Slack basic)
   - Developer B: User Story 4 (Time filtering) - can work in parallel
3. After US1 core is done:
   - Developer A: User Story 2 (Formatting)
   - Developer B: User Story 4 (Integration)
   - Developer C: LLM Curation (Phase 8)
4. After core stories:
   - Developer A: User Story 3 (Error handling)
   - Developer B: User Story 5 (Multi-channel)
5. All developers: Polish phase

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
- Tests are optional but recommended in Polish phase for production readiness
- LLM Curation is part of US2 but separated into Phase 8 for clarity
