# Quick Start Guide: Daily GeekNews Delivery to Slack

**Date**: 2024-11-23  
**Feature**: 001-daily-slack-news

## Prerequisites

- Python 3.9 or higher
- GitHub account with repository access
- Slack workspace with admin permissions
- Google Cloud account with Gemini API access

## Setup Steps

### 1. Clone and Setup Repository

```bash
git clone https://github.com/codesmith820/geeknews-bot.git
cd geeknews-bot
git checkout 001-daily-slack-news
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

Required packages:

- `feedparser>=6.0.10`
- `requests>=2.31.0`
- `pytz>=2023.3`
- `google-genai>=0.2.0`
- `slack-sdk>=3.23.0`

### 3. Configure Slack Webhook

1. Go to https://api.slack.com/apps
2. Create a new app or select existing app
3. Navigate to "Incoming Webhooks"
4. Activate Incoming Webhooks
5. Add New Webhook to Workspace
6. Select target channel
7. Copy webhook URL

### 4. Get Gemini API Key

1. Go to https://makersuite.google.com/app/apikey
2. Create new API key
3. Copy API key

### 5. Configure GitHub Secrets

1. Go to repository Settings → Secrets and variables → Actions
2. Add the following secrets:
   - `SLACK_WEBHOOK_URL`: Your Slack webhook URL
   - `GEMINI_API_KEY`: Your Gemini API key

### 6. Configure Delivery Schedule

Edit `.github/workflows/daily-news.yml`:

```yaml
on:
  schedule:
    - cron: "5 23 * * *" # 08:05 KST (23:05 UTC previous day)
```

Adjust cron expression for your desired delivery time (KST to UTC conversion: subtract 9 hours).

### 7. Test Locally (Optional)

Create `.env` file (don't commit):

```bash
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...
GEMINI_API_KEY=your-api-key
```

Run bot manually:

```bash
python src/bot.py
```

### 8. Enable GitHub Actions Workflow

1. Push code to repository
2. Go to Actions tab
3. Enable workflows if prompted
4. Workflow will run on schedule

## Verification

### Check Workflow Execution

1. Go to repository Actions tab
2. Check "Daily GeekNews Bot" workflow runs
3. View logs for execution details

### Check Slack Channel

1. Open configured Slack channel
2. Verify messages are delivered at scheduled time
3. Check message formatting

## Troubleshooting

### Workflow Not Running

- Check workflow file syntax
- Verify cron schedule is correct
- Check Actions are enabled for repository

### No Messages in Slack

- Verify webhook URL is correct
- Check Slack channel permissions
- Review workflow logs for errors

### LLM API Errors

- Verify API key is correct
- Check API quota/limits
- Review fallback behavior (should use time-based selection)

### RSS Feed Errors

- Verify feed URL is accessible
- Check network connectivity
- Review feed format changes

## Configuration Options

### Environment Variables

- `SLACK_WEBHOOK_URL` (required): Slack webhook URL
- `GEMINI_API_KEY` (required): Gemini API key
- `RSS_URL` (optional): RSS feed URL (default: https://news.hada.io/rss)
- `DEFAULT_DELIVERY_TIME` (optional): Default delivery time in KST (default: 08:00)

### Multiple Channels

To configure multiple channels:

1. Create separate Slack webhooks for each channel
2. Duplicate workflow file or modify to support multiple webhooks
3. Configure different delivery times per channel

## Next Steps

- Review [data-model.md](./data-model.md) for data structures
- Review [contracts/](./contracts/) for API details
- Review [research.md](./research.md) for technology decisions
- Run `/speckit.tasks` to generate implementation tasks
