# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SeriesFlow is a Python Flask web application for intelligent TV episode management. It automates Sonarr-based TV libraries with episode selection, viewing-based automation, and storage cleanup with seeding protection.

**Fork of Episeerr** with enhanced features including minimum retention for torrent seeding protection.

## Running the Application

```bash
# Development (Flask debug mode)
FLASK_DEBUG=true python episeerr.py

# Production (Gunicorn)
gunicorn --workers 2 --bind 0.0.0.0:5002 --access-logfile - --error-logfile - episeerr:app

# Docker
docker-compose up -d
```

Health check endpoint: `GET /api/series-stats`

## Git Commits

Use `diffsense` to generate commit messages (runs locally on Apple Silicon):

```bash
git add <files>
diffsense --nopopup  # generates commit message and commits (no popup for agents)
```

Do NOT write commit messages manually. Always use diffsense.

## Release Process

```bash
./release.sh <version>  # e.g., ./release.sh 2.7.0 or ./release.sh beta-2.7.0
```

This updates VERSION file and triggers GitHub Actions release workflow.

## Architecture

### Core Modules

| File | Purpose |
|------|---------|
| `episeerr.py` | Main Flask app - 50+ routes for web UI, APIs, and webhooks |
| `media_processor.py` | Rule enforcement, cleanup logic, activity tracking |
| `episeerr_utils.py` | Episode selection, Sonarr API interactions, tag management |
| `sonarr_utils.py` | Sonarr API wrapper with caching |

### Request Flow

```
Webhooks (Sonarr/Tautulli/Jellyfin/Jellyseerr)
    ↓
episeerr.py (route handlers)
    ↓
episeerr_utils.py (episode selection) ←→ media_processor.py (cleanup/rules)
    ↓
External APIs (Sonarr, TMDB, Tautulli, Jellyfin)
```

### Key Webhook Handlers

- `/sonarr-webhook` - Series add/update events
- `/webhook` - Tautulli watched episode events
- `/jellyfin-webhook` - Jellyfin playback progress
- `/seerr-webhook` - Jellyseerr/Overseerr request events

### Configuration

- `config/config.json` - Rules and series assignments (JSON, hot-reloadable)
- `.env` - Environment variables (see `.envexample`)

Required env vars: `SONARR_URL`, `SONARR_API_KEY`, `TMDB_API_KEY`

### Data Storage

- `config/` - Rule configuration
- `logs/` - app.log (rotating 10MB), cleanup.log, missing.log
- `data/requests/` - Episode request persistence
- `temp/` - Temporary files

## Key Concepts

**Rules** - Define episode get/keep behavior with grace periods and dormant timers

**Minimum Retention** - Protects files from cleanup for X days (torrent seeding friendly)

**Global Storage Gate** - Single threshold that enables/disables all cleanup operations

**Sonarr Tags** - `episeerr_default` for auto-assignment, `episeerr_select` for episode selection mode

**Activity Tracking** - Series-level activity dates with last season/episode hierarchy for cleanup decisions
