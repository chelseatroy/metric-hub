# Plan: Widgets Feature Lifecycle Dashboard Config

## What
Create a new TOML config file at `looker/feature-analytics/widgets.toml` using the new dashboard config format for the Widgets feature (Lists & Focus Timer).

## File to create

**`looker/feature-analytics/widgets.toml`** — single file covering both widgets, since they belong to the same product area, share the same newtab ping, and are grouped together in the context doc.

## TOML structure

Uses the format from Rob's draft:
- `[newtab_widgets]` — dashboard header with `title` and `product`
- `[metrics.<name>]` — each metric definition

## Metrics (19 total)

### Lists Widget (10 metrics)

| Stage | Metric Name | Type | Event / Source |
|-------|-------------|------|---------------|
| IMPRESSION | `lists_impressions` | `event_count` | `newtab.widgets_lists_impression` |
| ENGAGE | `lists_user_events` | `event_count` | `newtab.widgets_lists_user_event` |
| ENGAGE | `lists_user_events_by_action` | `event_extra` (count) | extra: `user_action` |
| ENGAGE | `lists_engagement_rate` | `ratio` | `lists_user_events / lists_impressions` |
| CONVERT | `lists_list_creates` | `event_count` | where: `user_action = 'list_create'` |
| CONVERT | `lists_task_creates` | `event_count` | where: `user_action = 'task_create'` |
| CONVERT | `lists_conversion_events` | `event_count` | where: `user_action IN ('list_create', 'task_create')` |
| CONVERT | `lists_conversion_rate` | `ratio` | `lists_conversion_events / lists_impressions` |
| DISENGAGE | `lists_change_display` | `event_count` | `newtab.widgets_lists_change_display` |
| DISENGAGE | `lists_disengage_rate` | `ratio` | `lists_change_display / lists_impressions` |

### Focus Timer Widget (9 metrics)

| Stage | Metric Name | Type | Event / Source |
|-------|-------------|------|---------------|
| IMPRESSION | `timer_impressions` | `event_count` | `newtab.widgets_timer_impression` |
| ENGAGE | `timer_user_events` | `event_count` | `newtab.widgets_timer_user_event` |
| ENGAGE | `timer_user_events_by_action` | `event_extra` (count) | extra: `user_action` |
| ENGAGE | `timer_toggle_notification` | `event_count` | `newtab.widgets_timer_toggle_notification` |
| ENGAGE | `timer_engagement_rate` | `ratio` | `timer_user_events / timer_impressions` |
| CONVERT | `timer_set_events` | `event_count` | where: `user_action = 'timer_set'` |
| CONVERT | `timer_conversion_rate` | `ratio` | `timer_set_events / timer_impressions` |
| DISENGAGE | `timer_change_display` | `event_count` | `newtab.widgets_timer_change_display` |
| DISENGAGE | `timer_disengage_rate` | `ratio` | `timer_change_display / timer_impressions` |

## Key design decisions

1. **Conversion definition**: Lists = `list_create` or `task_create`; Timer = `timer_set`. These represent the moment a user goes from viewing to actively using.
2. **Disengage counts all toggles**: The `change_display` events fire for both enable and disable. Counting both is simpler for the PoC and still informative (high toggle counts indicate usability issues).
3. **`graph` omitted**: Defaults to `true` per the format spec. All metrics should be graphed.
4. **Ratios all use impressions as denominator**: Standard funnel calculation.

## Steps

1. Create directory `looker/feature-analytics/`
2. Create `looker/feature-analytics/widgets.toml` with the config
