# Changelog

All notable changes to **Info Kierowca Notifier** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [2.4.3] - 2026-10-09

### Fixed
- Restored the 2.4.1 search-date range after the 2.4.2 frontend regression: earliest selectable date is today + 2 days and latest is today + 6 calendar months.
- The same search-date range again applies to both single-center and multi-center modes and is independent of the current booked exam date.
- Preserved the 2.4.1 localization and search-mode UI behavior while retaining the new per-mode WORD-center memory.


## [2.4.2] - 2026-10-09

### Fixed
- Search-mode switching now remembers the WORD-center selection separately for single-center and multi-center modes.
- Switching from multi-center to single-center no longer destroys the previously selected multi-center list.
- Switching back to multi-center restores the user's previous selection of up to 5 WORD centers.
- The remembered selections are persisted in `config.json` and survive application restarts.
- Existing 2.4.1 configurations are migrated automatically from the active `organization_ids` selection; selections already discarded by an older version cannot be reconstructed.


## [2.4.1] - 2026-10-08

### Fixed

#### Poll scheduling
- Removed polling jitter so configured polling intervals are now predictable.
- Saving Settings no longer triggers an immediate availability search.
- Changing the polling interval now reschedules the next search from the moment the settings are saved.
- Increasing or decreasing the polling interval correctly resets the countdown to the newly configured interval.

#### Relogin and scheduled searches
- Fixed an issue where automatic relogin could replace a scheduled availability search.
- A search that becomes due while the session requires renewal is now kept as pending.
- After successful relogin, a pending search is performed immediately.
- The next polling interval starts after the delayed search is actually performed.
- Proactive session renewal no longer causes a scheduled search to be lost.

#### Rate-limit handling
- Server-provided rate-limit timing is now authoritative.
- `x-ratelimit-reset` and `Retry-After` are used to determine when searching may resume after HTTP 429.
- The notifier does not perform an availability search before the server-defined resume time.
- Current slot results are preserved while temporarily rate-limited.
- Improved handling of server-side search quota windows.

#### Search date range
- Removed the previous local 31-day search restriction.
- The earliest selectable search date is now today + 2 days.
- The latest selectable search date is now today + 6 calendar months.
- The same date-range rules are used for single-center and multi-center searches.
- The search range is no longer restricted by the date of an existing booking.

#### Dashboard
- Improved rate-limit status and resume countdown presentation.
- Current slots now represent only the latest successful search instead of being merged with historical results.
- Previously retrieved slots remain visible during rate limiting with an indication that they may no longer be current.
- Improved Polish and English localization.
- Corrected singular/plural forms for available spots.
- Date formatting now follows the language selected in the dashboard.

### Security

- Removed temporary authentication network diagnostics from the production authentication flow.
- Removed HTTP response-body logging from request and parsing error diagnostics.
- Authentication and rate-limit diagnostics now record only non-sensitive metadata.
- Passwords, SMS/OTP codes, session cookies, authorization data, and PKK data are not intentionally written to application logs.
- Authentication-related headers continue to be stripped on cross-origin redirects.
- The dashboard remains bound to the local loopback interface.
- Existing secure native credential-storage requirements remain enforced.

### Diagnostics

- Improved scheduler logging to distinguish timer rescheduling from actual availability searches.
- Added explicit logging when a scheduled search is waiting for relogin.
- Added logging when a pending search resumes immediately after successful relogin.
- Added safe rate-limit diagnostics for session-refresh and availability-search requests.
- Rate-limit diagnostics include HTTP status and available server-provided limit metadata without recording response bodies or authentication data.

### Changed

- Polling frequency remains user-configurable and is not artificially increased by the application.
- Local rate-limit calculations are used only as a fallback when authoritative server timing is unavailable.
- Session refresh is performed before an availability search and failed authentication prevents the scheduled search from being sent until authentication is restored.

### Notes

- Availability-search quotas and limits are controlled by `info-kierowca.pl` and may change independently of the notifier.
- Users may configure frequent polling. If the server-side request limit is reached, the notifier waits until searching is permitted again.
- Session renewal may temporarily delay a scheduled search; after successful renewal, the pending search is executed immediately.
- Antivirus, SmartScreen, or other endpoint-security software may inspect newly built or downloaded executables. This is external to the notifier and is not treated as an application error.


## [2.4.0]

### Added

- Added **single-center search mode** using the `OneCenterExam` endpoint.
- Added **multi-center search mode** using the `MultipleCentersExams` endpoint.
- Added support for selecting up to 5 WORD centers in multi-center mode.
- Added automatic padding of multi-center API requests to exactly 5 organization IDs, as required by the API, while filtering filler-center results from returned matches.
- Added support for searching exam slots from **2 days ahead up to 6 calendar months ahead**.
- Added display of **all currently available matching exam slots** on the dashboard, grouped by date.
- Added dedicated handling of HTTP `429 Too Many Requests`.
- Added support for the observed API search limit of **10 requests per rate-limit window**.
- Added support for `x-ratelimit-limit`, `x-ratelimit-remaining`, `x-ratelimit-reset`, and `Retry-After` response headers.
- Added server-directed search resume timing after rate limiting.
- Added local rate-limit timing as a fallback when usable server timing information is unavailable.
- Added `rate_limited` notifier status and dashboard countdown until searching resumes.
- Added tracking of the last successful search independently from the last search attempt.
- Added persistence of the last successfully confirmed slot results during temporary rate limiting.
- Added localized rate-limit and stale-result messages.
- Added locale-aware date formatting for Polish and English.
- Added correct English and Polish available-place pluralization.
- Added `__Host-Http-PUDO-DeviceId` to persisted Profil Zaufany authentication cookies.

### Changed

- Removed the previous local **31-day exam search limit**.
- Search start date is no longer limited by the date of the currently booked exam.
- Search-date validation is now consistent between the frontend and backend.
- Six-month date calculations use calendar-month semantics with safe end-of-month handling.
- Dashboard now presents the complete current set of matching slots instead of only the earliest result.
- Dashboard dates now follow the application's selected language instead of the browser/operating-system locale.
- Dashboard slot-list spacing and scrolling were improved.
- Successful searches returning no matching slots now clear previously displayed current results.
- Rate-limited searches preserve the most recently confirmed results and mark them as potentially stale.
- Polling frequency remains user-configurable between **15 and 1800 seconds**; API quota exhaustion is handled independently by the rate-limit mechanism.
- After HTTP 429, server-provided `x-ratelimit-reset` and `Retry-After` values now take precedence over locally estimated reset timing.
- When both usable server timing values are available, the later time is used with an additional safety margin.
- Locally recorded successful-search timestamps are retained only as a fallback when the server provides no usable reset information.
- Search requests are suppressed while the application is waiting for the server-side rate limit to reset.
- Session refresh requests may continue while search requests are rate-limited, keeping the authenticated session available for the next search.
- English slot-count formatting in logs and push notifications now correctly uses `1 spot` and `N spots`.
- New interface text and validation messages are handled through the centralized `web/localization.py` localization layer.
- Profil Zaufany proactive re-login now ends the current notifier cycle after triggering authentication refresh, preventing continued use of stale in-memory session data.
- Application version changed from **2.3.1 to 2.4.0**.

### Fixed

- Fixed premature search-resume calculation after HTTP 429 when locally estimated timing was earlier than the reset information returned by the server.
- Fixed repeated unnecessary search requests while waiting for the API rate limit to reset.
- Fixed current slot results disappearing when the API temporarily returns HTTP 429.
- Fixed `1 spots` wording in notifier logs and push notifications.
- Fixed Profil Zaufany session persistence by retaining the device-identification cookie required by the authentication flow.
- Fixed restoration of `__Host-Http-*` cookies with appropriate cookie attributes.
- Fixed stale session data being used after proactive Profil Zaufany re-authentication was triggered.
- Fixed obsolete UI messages referring to the old 31-day search window.
- Fixed inconsistent Polish/English date presentation on the dashboard.
- Fixed Polish plural forms for available exam places.
- Fixed end-of-month rollover when calculating the maximum selectable search date six months ahead.
