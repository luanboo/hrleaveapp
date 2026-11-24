# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

HR leave tracker front end. The whole app is one self-contained file, `index.html` (inline CSS and JS, no dependencies). There is no build step, package manager, linter or test suite, and no backend.

Run it by opening `index.html` in a browser (or serving the folder with any static server).

## Architecture

All state lives in a single object `S` (`users`, `holidays`, `requests`, `currentUser`), persisted to `localStorage` under the key `leave-tracker-v1` by `save()`. If the seed data shape in `seed()` changes, bump the key or old browser data will be loaded in the old shape. Transient UI state (active tab, calendar month, highlighted request) is in a separate `ui` object and is not persisted.

Rendering is full re-render: `render()` rebuilds the nav and the active tab's HTML via one `view*` function per tab (`viewMine`, `viewApprovals`, `viewCalendar`, `viewHolidays`). Event handling is delegated on `document` using `data-tab`, `data-act` and `data-id` attributes. When adding an action, add a `data-act` and a case in the click switch.

Because `render()` replaces the DOM, the apply form is only rebuilt on actions. Typing in it calls `updatePreview()` (live working-day count and validation) and never `render()`, so don't call `render()` from the form's `input` handler.

Business rules are in the "business logic" section and are shared across tabs:
- `workingDays(start, end)` counts Mon–Fri days not in `S.holidays`. A request's `days` is calculated once at submit and stored, so editing holidays later does not change existing requests.
- `balance(userId, type)` derives taken from `approved` requests and pending from `pending` ones. Nothing is stored, so balances can't drift.
- `evaluateForm()` is the single validator for new requests: date order, zero working days, overlap with the user's own requests, and available balance net of pending days.
- `overlapsFor(req)` finds other people's approved or pending leave overlapping a request. It feeds the manager's approval cards. The calendar's "People off" row uses the `OVERLAP_WARN` constant.
- `decide()` is the approve/reject path: rejection requires a comment, and approval re-checks the balance.

Dates are `YYYY-MM-DD` strings, compared lexically. Always go through `parse()` and `iso()` (local-time) rather than `new Date('YYYY-MM-DD')`, which parses as UTC and shifts days. All user-entered text must pass through `esc()` before going into template strings.

Roles are a demo switcher (`#userSel`), not authentication. Manager-only UI is gated by `isManager()`. Managers cannot approve their own requests, which are excluded from the queue.
