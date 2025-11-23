# Implementation Plan: Daily GeekNews Delivery to Slack

**Branch**: `001-daily-slack-news` | **Date**: 2024-11-23 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-daily-slack-news/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

This feature implements an automated daily news delivery system that retrieves GeekNews articles from the last 24 hours and sends them to configured Slack channels. The system uses RSS feed parsing, LLM-based curation (Google Gemini API) for handling large article volumes, and GitHub Actions for scheduled execution. Key capabilities include time-based filtering, Slack Block Kit formatting, comprehensive error handling, and per-channel delivery schedule configuration.

## Technical Context

**Language/Version**: Python 3.9+  
**Primary Dependencies**: feedparser, requests, pytz, google-genai (Gemini API), slack-sdk  
**Storage**: N/A (stateless execution via GitHub Actions)  
**Testing**: pytest  
**Target Platform**: Linux (GitHub Actions runners - ubuntu-latest)  
**Project Type**: single (Python script/service)  
**Performance Goals**: Process and deliver up to 100 articles per day within 5 minutes of scheduled time  
**Constraints**: Must complete within GitHub Actions timeout limits, handle RSS feed parsing errors gracefully, respect Slack API rate limits  
**Scale/Scope**: Single RSS feed source, multiple Slack channels, daily execution schedule, up to 100 articles per delivery

## Constitution Check

_GATE: Must pass before Phase 0 research. Re-check after Phase 1 design._

**Note**: Constitution file (`.specify/memory/constitution.md`) appears to be a template and does not contain specific project principles. Proceeding with standard best practices:

- ✅ **Testability**: Feature spec includes clear acceptance criteria and testable requirements
- ✅ **Observability**: Comprehensive logging requirements defined (FR-014)
- ✅ **Error Handling**: Multiple requirements for graceful error handling (FR-007, FR-008, FR-017)
- ✅ **Documentation**: Specification includes detailed user stories and requirements

**No violations detected** - proceeding to Phase 0.

## Project Structure

### Documentation (this feature)

```text
specs/001-daily-slack-news/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
geeknews-bot/
├── .github/
│   └── workflows/
│       └── daily-news.yml      # GitHub Actions workflow
├── src/
│   ├── bot.py                  # Main bot logic
│   ├── rss_parser.py           # RSS feed parsing
│   ├── slack_client.py         # Slack API integration
│   ├── llm_curator.py          # LLM-based article curation
│   ├── config.py               # Configuration management
│   └── logger.py               # Logging utilities
├── tests/
│   ├── unit/
│   │   ├── test_rss_parser.py
│   │   ├── test_slack_client.py
│   │   └── test_llm_curator.py
│   └── integration/
│       └── test_delivery_flow.py
├── requirements.txt            # Python dependencies
└── README.md
```

**Structure Decision**: Single Python project structure with modular components (RSS parser, Slack client, LLM curator) separated into distinct modules. This follows separation of concerns while maintaining simplicity for a single-purpose bot service.

## Phase 0: Research Complete

**Status**: ✅ Complete  
**Output**: `research.md`

All technology decisions documented:

- RSS feed parsing (feedparser)
- HTTP client (requests)
- Timezone handling (pytz)
- LLM API integration (Google Gemini)
- Slack integration (slack-sdk)
- Scheduling platform (GitHub Actions)
- Message formatting (Slack Block Kit)
- Error handling strategy
- Configuration management

## Phase 1: Design Complete

**Status**: ✅ Complete  
**Outputs**:

- `data-model.md` - Entity definitions and relationships
- `contracts/slack-webhook-api.md` - Slack API contract
- `contracts/gemini-api.md` - Gemini API contract
- `contracts/rss-feed.md` - RSS feed contract
- `quickstart.md` - Setup and configuration guide
- `.cursor/rules/specify-rules.mdc` - Agent context updated

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No violations to justify.
