# Telegram File Leecher — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A public Telegram bot that accepts file/URL submissions, processes downloads, and delivers results via configurable transfer options (DM, cloud storage, HTTP links) with progress tracking and job management.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram users seeking remote file downloads
- Public internet users with large file transfer needs

## Success criteria

- Trackable job status updates for all users
- 100% delivery of completed files via user-selected transfer methods
- Rate-limited abuse prevention without user authentication

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with submission options
- **/grab** (command, actor: user, command: /grab <link>) — Submit a direct URL for leeching
- **Cancel Job** (button, actor: user, callback: job:cancel) — Cancel a pending job by ID

## Flows

### Submission Flow
_Trigger:_ message with file/link

1. Detect submission type (command/upload/forward)
2. Generate job ID and estimate
3. Store job in persistent storage

_Data touched:_ Job

### Progress Flow
_Trigger:_ Job status update

1. Send progress checkpoint
2. Notify on failure/completion
3. Offer transfer options

_Data touched:_ Job, Output

### Cancellation Flow
_Trigger:_ /cancel or button

1. Validate user ownership
2. Mark job as cancelled
3. Release resources

_Data touched:_ Job

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user with rate limits and preferences
  - fields: telegram_id, rate_limit_remaining, preferred_transfer_methods
- **Job** _(retention: persistent)_ — Active leech operation with status tracking
  - fields: job_id, source_url, status, progress_percent, output_files
- **Output** _(retention: session)_ — Downloaded file metadata and delivery options
  - fields: file_name, size_bytes, transfer_method, download_link

## Integrations

- **Telegram** (required) — Bot API messaging
- **Google Drive** (optional) — Cloud storage upload
- **Dropbox** (optional) — Cloud storage upload
- **HTTP Link Generator** (optional) — Public download links
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure cloud storage integrations (Google Drive/Dropbox)
- Set rate limit thresholds
- Adjust job retention TTL (default 7 days)

## Notifications

- Progress updates (start/checkpoint/complete/failure)
- Job expiration warnings (24h before TTL)
- Rate limit approaching alerts

## Permissions & privacy

- Anonymous usage tracking for abuse detection
- Job data retention limited to configured TTL
- No persistent storage of user preferences without explicit consent

## Edge cases

- Failed downloads with automatic retry logic
- Expired jobs with auto-deletion
- Transfer method unavailability during delivery

## Required tests

- End-to-end submission-to-delivery flow with all transfer options
- Job cancellation at various processing stages
- Rate limiting enforcement scenarios

## Assumptions

- Cloud storage integrations will be configured via owner controls
- Default HTTP link generator uses temporary signed URLs
- Rate limits are enforced per-telegram_id
