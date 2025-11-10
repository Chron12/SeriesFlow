# SeriesFlow

**Intelligent episode management for Sonarr** - Get episodes as you watch, clean up automatically with seeding protection, and manage storage efficiently.

> **Note:** SeriesFlow is a fork of [Episeerr](https://github.com/Vansmak/episeerr) with enhanced features including **minimum retention for seeding protection** and continued development.

---

- [SeriesFlow](#seriesflow)
  - [What's New in SeriesFlow](#whats-new-in-seriesflow)
  - [What It Does](#what-it-does)
  - [Quick Start](#quick-start)
    - [Full Setup (All Features)](#full-setup-all-features)
    - [Basic Setup (Works Immediately)](#basic-setup-works-immediately)
    - [Optional Additions (Add Only What You Want)](#optional-additions-add-only-what-you-want)
  - [How It Works](#how-it-works)
    - [Smart Rules](#smart-rules)
    - [Grace Periods](#grace-periods)
    - [Minimum Retention (NEW!)](#minimum-retention-new)
    - [Example: Popular Show Rule](#example-popular-show-rule)
    - [Storage Gate](#storage-gate)
  - [Three Ways to Use SeriesFlow (Pick What You Need)](#three-ways-to-use-seriesflow-pick-what-you-need)
    - [🎯 **Just Episode Selection**](#-just-episode-selection)
    - [⚡ **Add Viewing Automation**](#-add-viewing-automation)
    - [💾 **Add Storage Management**](#-add-storage-management)
  - [Key Benefits](#key-benefits)
  - [Documentation](#documentation)
  - [Support](#support)

## What's New in SeriesFlow

✨ **v2.6.0 - Seeding Protection**
- **Minimum Retention Days**: Protect files for a minimum time before ANY cleanup can occur
- Perfect for seeders: Files stay on server for 14 days (configurable) even if watched
- Works across all cleanup functions (grace_watched, grace_unwatched, dormant)
- Clear logging shows which files are protected for seeding

## What It Does

SeriesFlow automates your TV library with three simple features:

🎯 **Episode Selection** - Choose exactly which episodes you want
⚡ **Smart Rules** - Next episode ready when you watch, old episodes cleaned up automatically
💾 **Smart Cleanup** - Automatic cleanup with seeding protection and storage awareness

## Quick Start

### Full Setup (All Features)

```yaml
services:
  seriesflow:
    image: vansmak/episeerr:latest  # Uses upstream Episeerr image (compatible)
    environment:
      # Required for all features
      - SONARR_URL=http://your-sonarr:8989
      - SONARR_API_KEY=your_sonarr_api_key
      - TMDB_API_KEY=your_tmdb_api_key

      # Add your seer info if you want to manage by episode
      - JELLYSEERR_URL=http://your-overseer-or-jellyseerr-url
      - JELLYSEERR_API_KEY=your_api_key

      # Add these ONLY if you want viewing automation
      - TAUTULLI_URL=http://your-tautulli:8181
      - TAUTULLI_API_KEY=your_tautulli_key
      # Or use Jellyfin instead
      - JELLYFIN_URL=http://your-jellyfin-url
      - JELLYFIN_API_KEY=your_jellyfin_key
      - JELLYFIN_USER_ID=your_user_id

      # Optional quicklinks
      - CUSTOMAPP_URL=http://192.168.1.100:8080
      - CUSTOMAPP_NAME=My Custom App
      - CUSTOMAPP_ICON=fas fa-cog

    volumes:
      - ./config:/app/config
      - ./logs:/app/logs
      - ./data:/app/data
      - ./temp:/app/temp
    ports:
      - "5002:5002"
    restart: unless-stopped
```

### Basic Setup (Works Immediately)

1. **Start container** and go to `http://your-server:5002`
2. **That's it!** You can now use episode selection

### Optional Additions (Add Only What You Want)

- **Storage cleanup**: Set threshold in Scheduler page
- **Smart rules**: Create rules for automatic management
- **Viewing automation**: Add webhooks for next episode ready
- **Add `watched` tag in Sonarr**: Removes these series from Series Management

---

## How It Works

### Smart Rules

Create rules with the dropdown system:

**Get Episodes:**
- Type: Episodes/Seasons/All + Count
- Example: "3 episodes" = next 3 episodes ready

**Keep Episodes:**
- Type: Episodes/Seasons/All + Count
- Example: "1 season" = keep current season after watching

### Grace Periods

Create rules with independent grace timers:

**Grace Watched (Rotating Collection):**
- Your kept episodes expire after X days of inactivity
- Example: 14 days = watched episodes rotate out after 2 weeks

**Grace Unwatched (Watch Deadlines):**
- New episodes get X days to be watched if no activity
- Example: 10 days = pressure to watch new content

**Dormant Timer:**
- Removes content from abandoned shows
- Example: 30 days = if no activity for a month, clean up the show

### Minimum Retention (NEW!)

**Seeding Protection:**
- Files must stay on server for X days before ANY cleanup
- Protects files for seeding even if they've been watched
- Example: 14 days = files seed for 2 weeks minimum
- Overrides all other cleanup rules until retention period passes

### Example: Popular Show Rule

```log
Get: 5 episodes (next 5 episodes ready)
Keep: 2 episodes (last 2 watched episodes)
Grace: 7 days (keep last 2 watched episodes, delete after a week)
Dormant: 60 days (cleanup if abandoned for 2 months)
Min Retention: 14 days (protect for seeding)
```

**What happens:**
1. Watch E10 → Get E11-E15, Keep E9-E10
2. After 7 days → E9-10 eligible for deletion BUT protected for seeding
3. After 14 days → E9-10 deleted (grace + retention both passed)
4. After 60 days no activity → Delete show (series abandoned)

### Storage Gate

- Set one global threshold: "Keep 20GB free"
- Cleanup only runs when below threshold
- Stops immediately when back above threshold
- Only affects shows with grace/dormant timers
- Respects minimum retention even during storage pressure

---

## Three Ways to Use SeriesFlow (Pick What You Need)

### 🎯 **Just Episode Selection**

Good for picking specific episodes, even across seasons.

- **Setup**: Just the 3 required environment variables
- **Create sonarr and optional seer webhooks**
- **No rules needed, no webhooks required**
- **Use**: Manual episode selection interface only

### ⚡ **Add Viewing Automation**

Next episode ready as you watch (optional upgrade).

- **Setup**: Add Tautulli/Jellyfin webhook + create rules
- **No storage management required**
- **Use**: Episodes managed automatically as you watch

### 💾 **Add Storage Management**

Automatic cleanup with seeding protection when storage gets low (optional upgrade).

- **Setup**: Set storage threshold + add grace/dormant/min retention timers to rules
- **No viewing automation required**
- **Use**: Hands-off storage management with torrent-friendly retention

---

## Key Benefits

✅ **Intuitive**: Dropdown system makes rules easy to understand
✅ **Smart**: Grace periods that actually make sense
✅ **Seeding-Friendly**: Minimum retention protects files for seeders
✅ **Safe**: Storage gate prevents unnecessary cleanup
✅ **Flexible**: Use only the features you need
✅ **Storage-Aware**: Cleanup respects your storage limits

---

Screenshot <img width="1856" height="1301" alt="SeriesFlow" src="https://github.com/user-attachments/assets/ddad6213-ea53-4af9-9997-2a1f605b827c" />

---

## Documentation

**[📚 Full Documentation](./docs/)** - Complete guides and setup

**Quick Links:**
- [Installation Guide](./docs/installation.md) - Docker setup and configuration
- [Rules Guide](./docs/rules-guide.md) - Creating and managing rules
- [Episode Selection](./docs/episode-selection.md) - Manual episode management
- [Storage Gate](./docs/global_storage_gate_guide.md) - Automatic cleanup system

---

## Support

- **Issues**: [GitHub Issues](https://github.com/Chron12/SeriesFlow/issues)
- **Discussions**: [GitHub Discussions](https://github.com/Chron12/SeriesFlow/discussions)
- **Original Project**: [Episeerr by Vansmak](https://github.com/Vansmak/episeerr)

☕ **Support Development**: Rarely coffee, likely Red Bull, probably weed money.

*Intelligent episode management with seeding protection that just works.*
