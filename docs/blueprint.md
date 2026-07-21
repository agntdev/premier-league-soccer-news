# Premier League Soccer News Bot — Bot specification

**Archetype:** content

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that delivers real-time Premier League soccer updates: live scores, match results, and transfer news.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Premier League fans
- soccer enthusiasts
- sports news subscribers

## Success criteria

- Users receive live score alerts within 30 seconds of goals
- Daily transfer digest arrives at 9 AM local time
- Subscribed users get match result notifications shortly after full-time
- No duplicate transfer news is sent to users

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Subscribe to updates and open the main menu
- **/scores** (command, actor: user, command: /scores) — View current live scores
- **/results** (command, actor: user, command: /results) — View yesterday's match results
- **/transfers** (command, actor: user, command: /transfers) — View latest transfer news
- **/subscribe** (command, actor: user, command: /subscribe) — Enable push notifications
- **/unsubscribe** (command, actor: user, command: /unsubscribe) — Stop notifications
- **Subscribe to team alerts** (button, actor: user, callback: subscribe:team_select) — Select a team for targeted alerts
  - inputs: team_selection
  - outputs: confirmation_message
- **View live scores** (button, actor: user, callback: scores:live) — View current live scores
  - outputs: live_scores_list
- **View match results** (button, actor: user, callback: results:yesterday) — View yesterday's match results
  - outputs: match_results_list
- **View transfer news** (button, actor: user, callback: transfers:latest) — View latest transfer news
  - outputs: transfer_news_list

## Flows

### Subscribe
_Trigger:_ /start or /subscribe

1. Open main menu
2. Show subscription options
3. User selects subscription type
4. Confirm subscription
5. Store user preferences

_Data touched:_ user_subscription

### Live Score Alert
_Trigger:_ Live match event (goal, red card)

1. Detect event from data source
2. Identify subscribed users
3. Send push notification with event details

_Data touched:_ match_state, user_subscription

### Match Result Notification
_Trigger:_ Match ends (full-time)

1. Detect match completion
2. Identify subscribed users
3. Send result notification once

_Data touched:_ match_state, user_subscription

### Transfer News Digest
_Trigger:_ Daily at 9 AM

1. Fetch latest transfer news
2. Check for duplicates
3. Send digest to subscribed users

_Data touched:_ transfer_news_history, user_subscription

### On-demand Query
_Trigger:_ /scores, /results, /transfers

1. User sends command
2. Bot retrieves requested data
3. Send response to user

_Data touched:_ match_state, transfer_news_history

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_subscription** _(retention: persistent)_ — Tracks which users are subscribed and their preferences
  - fields: user_id, subscription_type, team_selection, last_notification_time
- **match_state** _(retention: session)_ — Current status of Premier League matches
  - fields: match_id, home_team, away_team, home_score, away_score, status, last_event_time
- **transfer_news** _(retention: persistent)_ — Transfer news items from sports outlets
  - fields: news_id, player_name, from_team, to_team, transfer_fee, source, timestamp
- **transfer_news_history** _(retention: session)_ — Record of sent transfer news to avoid duplicates
  - fields: news_id, user_id, sent_at

## Integrations

- **Telegram** (required) — Bot API messaging
- **Premier League API** (required) — Live scores and match data
- **Sports news API** (required) — Transfer news sourcing
- **Database** (required) — User subscriptions and match state persistence
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View subscription statistics
- Manually trigger transfer news fetch
- Pause live score alerts temporarily
- Reset user subscriptions
- View notification delivery logs

## Notifications

- Live score alerts for subscribed users
- Match result notifications after full-time
- Daily transfer digest at 9 AM local time
- Team-specific alerts for subscribed users

## Permissions & privacy

- Store user_id for subscription management
- Store team preferences for targeted alerts
- No personal data beyond subscription preferences
- Compliance with Telegram Bot API privacy requirements

## Edge cases

- User unsubscribes during live match
- Match ends while user is offline
- Transfer news API rate limits exceeded
- Duplicate transfer news from multiple sources
- User subscribes to non-existent team
- Live score event occurs during maintenance window

## Required tests

- Verify /start opens main menu and shows subscription options
- Test live score alert delivery within 30 seconds of goal
- Confirm match result notification is sent once after full-time
- Validate daily transfer digest arrives at 9 AM local time
- Test team-specific subscription filtering
- Verify no duplicate transfer news is sent
- Confirm unsubscribe stops all notifications
- Test error handling for API failures

## Assumptions

- Updates are pushed via Telegram push notifications (not just on-demand queries)
- Daily transfer digest at 9 AM local time for subscribed users
- Live score alerts only for Premier League matches
- Match results are sent once, shortly after full-time
- Transfer news is sourced from major sports outlets (BBC Sport, Sky Sports, The Athletic)
- Users can subscribe to all updates or choose specific teams for targeted alerts
