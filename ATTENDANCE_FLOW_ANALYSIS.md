# Attendance Flow Analysis

## 1. Local Room Save Flow
- `AttendanceViewModel.submitAttendance()` creates an `AttendanceRecord`.
- Calls `AttendanceRepository.saveCheckIn()` or `updateCheckOut()`.
- `AttendanceRepository` adds metadata (selfie paths, driverProfileId).
- `AttendanceRepository` converts `AttendanceRecord` to `AttendanceEntity`.
- `AttendanceDao.insertAttendance()` or `updateAttendance()` saves it to the `attendance` table.
- Default `isSynced` is `false`.

## 2. Sync Flow
- Immediate Sync: `AttendanceRepository` calls `remoteRepository.checkIn()` or `checkOut()` right after Room save.
- If API call is successful (`response.isSuccessful && body?.success == true`), it updates the entity in Room with `isSynced = true`.
- Background Sync: `AttendanceViewModel` calls `triggerSync()` which enqueues `AttendanceSyncWorker`.
- `AttendanceSyncWorker` queries `attendanceDao.getUnsyncedAttendance()`, iterates through them, and attempts API calls.

## 3. API Request Payload
- **Check-In Request:**
  - `driver_profile_id`: String
  - `latitude`: Double
  - `longitude`: Double
  - `accuracy`: Double
  - `odometer`: Double
  - `selfie_data`: Base64 String
  - `dashboard_data`: Base64 String
- **Check-Out Request:** Same fields as Check-In.

## 4. Response DTO
- `AttendanceResponse`:
  - `success`: Boolean
  - `message`: String
  - `data`: `AttendanceRecordDto` (contains calculated fields like `total_hours`, `status`, etc.)

## 5. Sync Worker Behavior
- Fetches all records where `isSynced = 0`.
- For each record, checks `isCompleted` to decide between `check-in` and `check-out` API.
- If API call succeeds, updates `isSynced = 1`.
- If any record fails to sync, it returns `Result.retry()`.

## 6. Driver ID Source
- `tokenManager.getDriverProfileId() ?: tokenManager.getUserId() ?: ""`

## 7. User ID Source
- `tokenManager.getUserId()` (Stored in `user_id` preference).

## 8. Token Source
- `TokenManager` manages `access_token` and `refresh_token` in `EncryptedSharedPreferences`.

## 9. Attendance Status Handling
- Local: "Present" or "Half Day" based on 10:30 AM cutoff.
- Remote: Server likely calculates its own status and returns it in `AttendanceRecordDto`.

## 10. Failure Paths
- Network failure: Sync fails, `isSynced` remains `false`.
- API error (e.g., 400 Bad Request): Sync fails, `isSynced` remains `false`.
- `AttendanceSyncWorker` will retry on next run.
- `AttendanceViewModel` shows error if immediate sync fails.
