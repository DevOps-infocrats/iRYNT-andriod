# Attendance Sync Fix Report

## Root Cause
The primary reason attendance marked from the Android application was not appearing in the web application was a mismatch in the request payload and missing identifiers. Specifically:
1. **Missing Fields:** The `CheckInRequest` and `CheckOutRequest` were missing critical fields such as `user_id`, `deployment_id`, and `timestamp` which are used by the backend to link the attendance record to the correct session and display it in the web dashboard.
2. **ID Resolution Issues:** The app was falling back to `user_id` for the `driver_profile_id` field because the backend was not returning `driver_profile_id` in the login response. While the backend tried to handle this, the lack of a dedicated `user_id` field in the request made resolution ambiguous.
3. **Missing Deployment Context:** Attendance requires an active deployment. The app was not sending the `deployment_id` in the attendance request, forcing the backend to look it up, which could fail if the profile ID was ambiguous.
4. **Silent Sync Failures:** The `AttendanceSyncWorker` lacked detailed error logging and didn't capture the reasons for sync failures, making it difficult to diagnose why records remained local.

## Files Inspected
- `com.vil.mobile.ui.state.AttendanceViewModel.kt`
- `com.vil.mobile.model.AttendanceRepository.kt`
- `com.vil.mobile.remote.dto.AttendanceDtos.kt`
- `com.vil.mobile.workers.AttendanceSyncWorker.kt`
- `com.vil.mobile.workers.GpsSyncWorker.kt`
- `com.vil.mobile.security.TokenManager.kt`
- `com.vil.mobile.local.dao.AttendanceDao.kt`

## Files Modified
1. `app/src/main/java/com/vil/mobile/remote/dto/AttendanceDtos.kt`: Added `userId`, `deploymentId`, and `timestamp` to `CheckInRequest` and `CheckOutRequest`.
2. `app/src/main/java/com/vil/mobile/model/AttendanceRepository.kt`: Updated to populate the new request fields and added a robust timestamp.
3. `app/src/main/java/com/vil/mobile/workers/AttendanceSyncWorker.kt`: Updated to populate the new request fields, added `TokenManager` for ID resolution, and improved error logging.

## Exact Lines Changed
- **AttendanceDtos.kt**: Modified `CheckInRequest` and `CheckOutRequest` data classes.
- **AttendanceRepository.kt**: Modified `saveCheckIn` and `updateCheckOut` methods to include `userId`, `deploymentId`, and `timestamp`. Added imports for `SimpleDateFormat` and `Date`.
- **AttendanceSyncWorker.kt**: Modified constructor to include `TokenManager`. Modified `doWork` to resolve IDs and include all required fields in the API requests. Added detailed `Log.e` calls for failed sync attempts.

## Request Payload (Updated)
```json
{
  "driver_profile_id": "uuid-or-fallback",
  "user_id": "uuid",
  "deployment_id": "uuid-or-null",
  "latitude": 28.4595,
  "longitude": 77.0266,
  "accuracy": 8.0,
  "odometer": 1234.5,
  "timestamp": "2023-10-27T10:30:00",
  "selfie_data": "...",
  "dashboard_data": "..."
}
```

## Why changes are safe
- **Minimal Impact:** Only the request DTOs and the code that populates them were changed.
- **Preserved Logic:** Existing local database logic and UI flows remain unchanged.
- **Better Observability:** Added logging allows for easier troubleshooting if issues persist.
- **Backward Compatibility:** Keeping the fallback for `driver_profile_id` ensures that the backend continues to receive what it expects even if it hasn't been updated to return the correct profile ID during login.

## Results
✓ Attendance requests now include full context (`user_id`, `deployment_id`).
✓ Correct `driver_profile_id` or `user_id` fallback is sent explicitly.
✓ Sync failures are now logged with HTTP status codes and error bodies.
✓ Background worker correctly retries with full context on next run.
