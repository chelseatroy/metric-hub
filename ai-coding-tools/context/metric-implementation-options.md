# Feature Lifecycle Dashboard: Implementation Options

Deep-dive analysis of four candidate features for the lifecycle dashboard PoC.

**Lifecycle Stages:**
- **IMPRESSION** — user saw the feature
- **ENGAGE** — started using it
- **CONVERT** — became an active user
- **DISENGAGE** — stopped using it

**Platform:** Firefox Desktop (all features)
**Ping:** `newtab` (sent on `newtab_session_end` or `component_init`)

---

## 1. Weather Widget

### Metrics Available

| Stage | Metric | Type | Description |
|-------|--------|------|-------------|
| IMPRESSION | `newtab.weather_impression` | event | Weather widget viewed |
| ENGAGE | `newtab.weather_open_provider_url` | event | User opens weather provider link |
| ENGAGE | `newtab.weather_location_selected` | event | User selects a location |
| ENGAGE | `newtab.weather_change_display` | event | User changes weather display mode |
| ENGAGE | `newtab.weather_detect_location` | event | User selects "detect my location" |
| CONVERT | `newtab.weather_opt_in_selection` | event | User answers location opt-in prompt |
| STATE | `newtab.weather_enabled` | boolean | Widget enabled (`browser.newtabpage.activity-stream.showWeather` pref) |
| ERROR | `newtab.weather_load_error` | event | Weather widget failed to load |

### Extra Keys (Event Attributes)

| Metric | Extra Keys |
|--------|------------|
| `weather_impression` | `newtab_visit_id` |
| `weather_change_display` | `newtab_visit_id`, `weather_display_mode` |
| `weather_opt_in_selection` | `newtab_visit_id`, `user_selection` |

### Lifecycle Coverage Analysis

| Stage | Coverage | Notes |
|-------|----------|-------|
| IMPRESSION | ✅ Strong | Explicit `weather_impression` event |
| ENGAGE | ✅ Strong | Multiple engagement events (location, display, provider URL) |
| CONVERT | ✅ Good | `weather_opt_in_selection` captures conversion moment |
| DISENGAGE | ⚠️ Indirect | No explicit "disable" event; must infer from `weather_enabled=false` state changes |

### Recommendation

**Good candidate.** Has explicit events for impression, engagement, and conversion. The DISENGAGE gap can be addressed by tracking `weather_enabled` state transitions (true→false).

---

## 2. Wallpapers

### Metrics Available

| Stage | Metric | Type | Description |
|-------|--------|------|-------------|
| IMPRESSION | `newtab.wallpaper_highlight_cta_click` | event | Feature highlight CTA clicked (discovery) |
| ENGAGE | `newtab.wallpaper_click` | event | User clicks a wallpaper option |
| ENGAGE | `newtab.wallpaper_category_click` | event | User clicks wallpaper category |
| ENGAGE | `newtab.wallpaper_upload` | event | User uploads custom wallpaper |
| DISENGAGE | `newtab.wallpaper_highlight_dismissed` | event | User dismisses wallpaper feature highlight |

### Extra Keys (Event Attributes)

| Metric | Extra Keys |
|--------|------------|
| `wallpaper_click` | `newtab_visit_id`, `selected_wallpaper`, `had_previous_wallpaper`, `had_uploaded_previously` |
| `wallpaper_highlight_cta_click` | `newtab_visit_id` |
| `wallpaper_highlight_dismissed` | `newtab_visit_id` |

### Lifecycle Coverage Analysis

| Stage | Coverage | Notes |
|-------|----------|-------|
| IMPRESSION | ⚠️ Indirect | No explicit "wallpaper section visible" event; `wallpaper_highlight_cta_click` is engagement, not impression |
| ENGAGE | ✅ Strong | Multiple engagement events with rich context (`selected_wallpaper`, previous state) |
| CONVERT | ⚠️ Indirect | `had_previous_wallpaper=false` → `true` on `wallpaper_click` indicates conversion |
| DISENGAGE | ⚠️ Partial | Only captures highlight dismissal, not "user resets to default" |

### Recommendation

**Moderate candidate.** Strong engagement tracking with useful context (which wallpaper, first-time vs returning). Weaker on impression (no passive "saw wallpapers" event) and disengage (only tracks highlight dismissal). Could infer CONVERT from `had_previous_wallpaper` transitions. Could potentially use `selected_wallpaper='none'` as disengage signal.

---

## 3. Widgets (Lists & Focus Timer)

### Lists Widget Metrics

| Stage | Metric | Type | Description |
|-------|--------|------|-------------|
| IMPRESSION | `newtab.widgets_lists_impression` | event | Lists widget viewable on screen |
| ENGAGE | `newtab.widgets_lists_user_event` | event | User creates/edits/deletes lists or tasks |
| DISENGAGE | `newtab.widgets_lists_change_display` | event | User toggles lists widget (enabled pref change) |

### Focus Timer Widget Metrics

| Stage | Metric | Type | Description |
|-------|--------|------|-------------|
| IMPRESSION | `newtab.widgets_timer_impression` | event | Timer widget viewable on screen |
| ENGAGE | `newtab.widgets_timer_user_event` | event | Timer set/play/pause/reset actions |
| ENGAGE | `newtab.widgets_timer_toggle_notification` | event | User toggles timer notifications |
| DISENGAGE | `newtab.widgets_timer_change_display` | event | User toggles timer widget (enabled pref change) |

### Extra Keys (Event Attributes)

| Metric | Extra Keys |
|--------|------------|
| `widgets_lists_user_event` | `newtab_visit_id`, `user_action` (list_copy, list_create, list_edit, list_delete, task_create, task_edit, task_delete, task_completed) |
| `widgets_timer_user_event` | `newtab_visit_id`, `user_action` (timer_set, timer_play, timer_pause, timer_reset, timer_toggle_focus, timer_toggle_break, timer_end) |

### Lifecycle Coverage Analysis

| Stage | Coverage | Notes |
|-------|----------|-------|
| IMPRESSION | ✅ Strong | Explicit impression events for both widgets |
| ENGAGE | ✅ Strong | Rich `user_action` attribute distinguishes action types |
| CONVERT | ⚠️ Indirect | Could define as first `list_create`/`task_create` or first `timer_set` |
| DISENGAGE | ✅ Strong | Explicit `change_display` events when widgets are disabled |

### Recommendation

**Strong candidate.** Clean lifecycle coverage with explicit impression and disengage events. The `user_action` extra key provides granular engagement data. CONVERT can be defined as first meaningful action (create list/task or set timer). Two widgets means more data points for the PoC.

---

## 4. Trending Searches

### Metrics Available

| Stage | Metric | Type | Description |
|-------|--------|------|-------------|
| IMPRESSION | `newtab.trending_search_impression` | event | Trending search widget visible |
| ENGAGE | `newtab.trending_search_suggestion_open` | event | User opens a suggested search |
| DISENGAGE | `newtab.trending_search_dismiss` | event | User collapses or dismisses widget |

### Additional Related Metrics (urlbar)

| Metric | Type | Description |
|--------|------|-------------|
| `urlbar.picked.trending` | labeled_counter | Trending result picked from URL bar (by index) |
| `urlbar.picked.trending_rich` | labeled_counter | Rich trending result picked from URL bar |
| `urlbar.trending.block` | counter | User blocked trending results |

### Extra Keys (Event Attributes)

| Metric | Extra Keys |
|--------|------------|
| `trending_search_impression` | `newtab_visit_id`, `variant` ("a" or "b") |
| `trending_search_suggestion_open` | `newtab_visit_id`, `variant` |
| `trending_search_dismiss` | `newtab_visit_id`, `variant` |

### Lifecycle Coverage Analysis

| Stage | Coverage | Notes |
|-------|----------|-------|
| IMPRESSION | ✅ Strong | Explicit impression event |
| ENGAGE | ✅ Strong | Explicit click/open event |
| CONVERT | ⚠️ Indirect | Could define as first `suggestion_open` or sustained engagement |
| DISENGAGE | ✅ Strong | Explicit dismiss event (collapse or full dismiss) |

### Recommendation

**Strong candidate.** Cleanest three-event lifecycle (impression → open → dismiss). The `variant` extra key suggests A/B testing capability built in. Missing explicit CONVERT, but simpler feature means conversion could be first use or repeat use. Also has urlbar integration for cross-surface analysis.

---

## Summary Comparison

| Feature | IMPRESSION | ENGAGE | CONVERT | DISENGAGE | Complexity | Recommendation |
|---------|------------|--------|---------|-----------|------------|----------------|
| Weather Widget | ✅ | ✅ | ✅ | ⚠️ | Medium | Good |
| Wallpapers | ⚠️ | ✅ | ⚠️ | ⚠️ | Medium | Moderate |
| Widgets (Lists + Timer) | ✅ | ✅ | ⚠️ | ✅ | Low | **Strong** |
| Trending Searches | ✅ | ✅ | ⚠️ | ✅ | Low | **Strong** |

### Top Recommendations for PoC

1. **Widgets (Lists + Timer)** — Best overall coverage. Explicit events for 3 of 4 stages, rich `user_action` data for engagement analysis, and two separate widgets provide more data. CONVERT can be defined as first meaningful action.

2. **Trending Searches** — Simplest and cleanest. Three explicit events map directly to impression/engage/disengage. Built-in A/B variant tracking. Good for demonstrating the dashboard concept with minimal ambiguity.

3. **Weather Widget** — Strong impression/engage/convert coverage. The opt-in flow provides a clear conversion moment. Would need to derive DISENGAGE from state changes rather than explicit events.

---

## Technical Notes

### Common Infrastructure

All metrics share:
- **Ping:** `newtab`
- **Lifetime:** `ping` (events) or `application` (booleans)
- **Extra key:** `newtab_visit_id` — enables session-level analysis and event sequencing

### Ping Submission Triggers

The `newtab` ping fires on:
- `component_init` — newtab component initialized (captures settings state)
- `newtab_session_end` — newtab visit ended (navigation, tab close, etc.)

### Data Sensitivity

All metrics are classified as `interaction` sensitivity — suitable for general telemetry analysis.

### Pref Dependencies

| Feature | Controlling Pref |
|---------|------------------|
| Weather | `browser.newtabpage.activity-stream.showWeather` |
| Lists | `widgets.lists.enabled` |
| Timer | `widgets.focusTimer.enabled` |
