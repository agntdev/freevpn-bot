# FreeVPN Config Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Provides non-technical users with instant, ready-to-use WireGuard VPN configurations via Telegram buttons. Delivers .conf files and QR codes with one-click, enforcing soft quotas and short expiration to balance usability and abuse prevention.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- ordinary end users with little technical knowledge

## Success criteria

- User receives working VPN config file and instructions within 2 seconds of button tap

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with 'Get VPN' and 'Info / Limits' buttons
- **Get VPN** (button, actor: user, callback: vpn:request) — Generate and deliver new WireGuard config file and QR code
- **Info / Limits** (button, actor: user, callback: info:limits) — Display current quotas, expiry, and usage instructions

## Flows

### vpn_delivery
_Trigger:_ vpn:request

1. Check user quota and server capacity
2. Generate WireGuard config and QR code
3. Send .conf file + QR code + one-line instruction

_Data touched:_ User account, VPN profile, Issuance record

### admin_notification
_Trigger:_ issuance_complete

1. Format compact alert with user ID and profile ID
2. Send to admin Telegram channel

_Data touched:_ Issuance record

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User account** _(retention: persistent)_ — Telegram user ID, display name, optional email
  - fields: telegram_id, display_name, email
- **VPN profile** _(retention: session)_ — Generated WireGuard config with 24h expiry
  - fields: config_data, expiry_time, server_location
- **Issuance record** _(retention: persistent)_ — Log of config delivery events
  - fields: telegram_id, profile_id, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging and file delivery
- **Admin Telegram Channel** (required) — Issuance alerts and health notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure admin notification Telegram chat ID
- Adjust quota limits (GB/day)
- Set server pool retention policy

## Notifications

- Admin channel alerts on new issuance (telegram_id + profile_id + timestamp)

## Permissions & privacy

- Store only telegram_id, display_name, and issuance logs
- Auto-delete user data after 30 days

## Edge cases

- Quota exceeded - show retry window
- No available servers - suggest wait time
- Invalid config request - retry or error message

## Required tests

- End-to-end test: User taps 'Get VPN' → receives config file and QR code within 2s
- Admin channel receives notification on successful issuance

## Assumptions

- Default 5GB/day quota per user
- WireGuard is preferred over OpenVPN for simplicity
- Admin channel is pre-configured by owner
