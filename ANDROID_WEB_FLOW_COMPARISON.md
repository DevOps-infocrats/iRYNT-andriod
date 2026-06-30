# Android and Web Flow Comparison

## API Endpoint
- **Android:** Uses `POST /api/v1/attendance/check-in` and `POST /api/v1/attendance/check-out`.
- **Web:** Likely uses a different internal service or the same API but with more context.

## Identity Handling
- **Android:** Sends `driver_profile_id`. Falls back to `user_id` if `driver_profile_id` is null.
- **Web:** Likely uses the database primary keys directly or has access to the full `User` and `Profile` objects.

## Data Visibility (Crucial Difference)
- **Web App Dashboard:** Filters by `company_id` and `circle_id`.
- **Android App:** If the login response doesn't populate `company_id`/`circle_id` in the `User` record, and the backend doesn't update the `User` when the `DriverProfile` is updated, the record might exist in the database but be filtered out from the Web App views.

## Reconcilliation
- **Web:** Can see all records in the DB.
- **Android:** Tries to fetch history via `GET /api/v1/attendance/history` which is currently **MISSING** on the backend. This means the app cannot "see" what the server has if it loses local state.
