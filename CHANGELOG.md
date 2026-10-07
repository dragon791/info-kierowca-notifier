# Changelog

All notable changes to this project will be documented in this file.

## [2.4.0] - 2026-10-08

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
