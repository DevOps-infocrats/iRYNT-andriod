# Token Analysis

## Stored User Data
- `TokenManager` stores:
  - `access_token`
  - `refresh_token`
  - `user_id`
  - `user_name`
  - `user_role`
  - `user_phone`
  - `profile_pic_url`
  - `driver_profile_id`
  - `deployment_id`

## Driver Profile ID vs User ID
- In `LoginViewModel`, the `UserDto` from the login response is used to populate these fields.
- `VIL_UUID_DEBUG` logs confirm both `User ID` and `Driver Profile ID` are logged during login.
- If `driver_profile_id` is null in the JSON response, `TokenManager` stores `null`.

## Token Lifespan
- The app uses `EncryptedSharedPreferences` for security.
- `AuthInterceptor` adds the `Authorization: Bearer <token>` header to all outgoing requests.
- If the token is stale/expired, the backend returns `401 Unauthorized`.
- **Note:** `ATTENDANCE_SYNC_ROOT_CAUSE_ANALYSIS.md` mentions that `GpsSyncWorker` ignores `401` errors and returns success, which might be a related issue for location tracking.

## Potential Stale Data
- `TokenManager.clear()` is called during logout, ensuring that old session data is wiped.
- However, if the login response itself lacks the `driver_profile_id`, the app's fallback to `user_id` might be using an ID that the backend doesn't recognize as a valid attendance-eligible profile.
