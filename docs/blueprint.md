# Group Note Manager — Bot specification

**Archetype:** workflow

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Collaborative note-taking bot for Telegram groups with version history, tag-based organization, and optional approval workflow for edits. Tracks revisions, supports Markdown formatting, and notifies group members of changes.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- small work teams
- Telegram group administrators

## Success criteria

- teams can track document revisions with approval workflow
- group members receive configurable notifications about changes

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Show private bot introduction and linked groups
- **/notes** (command, actor: group_member, command: /notes) — List all notes in current group chat
  - inputs: group context
  - outputs: note list with filters
- **/note create** (command, actor: group_member, command: /note create) — Start note creation flow with title/body/tags
  - inputs: title, body, tags, approval flag
  - outputs: new draft note
- **/enable** (command, actor: admin, command: /enable) — Enable notes in this group
  - inputs: group admin confirmation
  - outputs: group settings update
- **View Note** (button, actor: group_member, callback: note:view:<id>) — Display note content with history and actions
  - inputs: note ID
  - outputs: note view with revision history
- **Propose Edit** (button, actor: group_member, callback: note:edit:<id>) — Start edit flow for existing note
  - inputs: note content, change summary
  - outputs: pending revision or immediate update

## Flows

### note_approval
_Trigger:_ edit with approvals_required=true

1. submit edit
2. wait for approval
3. publish revision

_Data touched:_ Note, Revision

### notification_delivery
_Trigger:_ note event (create/publish/approve)

1. check notification settings
2. post to group
3. send author DM

_Data touched:_ GroupSettings

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Note** _(retention: persistent)_ — Collaborative document with version history
  - fields: title, body, tags, status, approvals_required, created_at, updated_at
- **Revision** _(retention: persistent)_ — Snapshot of note content and metadata
  - fields: note_id, author, timestamp, content, change_summary
- **GroupSettings** _(retention: persistent)_ — Per-group configuration
  - fields: enabled, default_approvals_required
- **UserRole** _(retention: session)_ — Implicit group membership status
  - fields: chat_id, user_id, is_admin

## Integrations

- **Telegram** (required) — Group chat messaging and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- enable/disable notes in specific groups
- set default approval requirements per group

## Notifications

- group chat posts for note changes
- direct messages to authors about approvals

## Permissions & privacy

- only group members can view/edit notes
- revision history stored securely
- no personal data beyond Telegram IDs

## Edge cases

- handling approval requests when no approvers are active
- conflict resolution for simultaneous edits
- graceful degradation when storage is unavailable

## Required tests

- approval workflow from proposal to publication
- notification delivery in group and DMs
- revision history restoration

## Assumptions

- Markdown formatting is acceptable for all users
- single approver model meets team needs
- group administrators will properly configure the bot
