# Attendance Sync Root Cause Analysis

## 1. Root Causes

### A. Missing Backend History API (Primary Reason for App-Web Mismatch)
The Android app calls `GET /api/v1/attendance/history` to refresh its local records with server-validated data. This endpoint is **missing** in the Flask backend (`api/v1/attendance/routes.py`). 
- **Impact:** Android app cannot confirm if records were successfully saved on the server. If a local record is deleted or the database is cleared, the app shows "Absent" even if the server has the record.
- **Web App Impact:** If the immediate sync fails (see below), the app thinks it will retry, but without a history API, it can never reconcile states.

### B. Silent Failures in UI Flow
`AttendanceViewModel.submitAttendance()` transitions to the "Success" screen immediately after calling `attendanceRepository.saveCheckIn()`, without checking if the remote sync actually succeeded.
- **Impact:** The user sees a success checkmark even if the backend returned `400 Bad Request` (e.g., "Active deployment required") or `401 Unauthorized`.
- **Result:** The record remains local only (`isSynced = false`), but the user is unaware.

### C. Strict Driver Deployment Requirement
The backend `AttendanceGeoService.py` requires Drivers to have an **Active** deployment before marking attendance.
- **Impact:** If the Android app hasn't successfully synced the "Start Trip" action (which sets deployment status to Active), the attendance mark request is rejected with `400 Bad Request`.
- **Result:** Attendance doesn't appear in the web app because the server rejected the creation of the record.

### D. GPS Sync False Success (Silent Data Loss)
`GpsSyncWorker.kt` returns `Result.success()` even if the server returns `401 UNAUTHORIZED`. 
- **Impact:** Location tracks are marked as "Synced" locally but are never actually saved on the server. This prevents admins from seeing the "Live" location status of drivers.

### E. Missing DTO Fields in Login
`User.to_dict()` in the backend does not include `driver_profile_id`.
- **Impact:** The Android app receives `null` for `driverProfileId` and falls back to sending `user_id`. While the backend handles this fallback, it adds complexity and potential for resolution errors.

### F. Dashboard Filtering (Visibility Issue)
The Web Attendance Dashboard (`AttendanceRepository.py`) filters users by `User.company_id` and `User.circle_id`.
- **Impact:** `ensure_helper_profile` updates the **DriverProfile** but NOT the **User** record. If a User has a null company/circle and no active deployment (which provides the project company), they are hidden from filtered admin views even if they marked attendance.

---

## 2. Files Involved

### Android
- `com.vil.mobile.ui.state.AttendanceViewModel.kt`
- `com.vil.mobile.model.AttendanceRepository.kt`
- `com.vil.mobile.workers.GpsSyncWorker.kt`
- `com.vil.mobile.workers.AttendanceSyncWorker.kt`

### Backend
- `app/api/v1/attendance/routes.py`
- `app/api/v1/attendance/controllers.py`
- `app/modules/attendance/services.py`
- `app/modules/attendance/repository.py`
- `app/modules/auth/models.py` (User.to_dict)

---

## 3. API Details

### Request Payload (Example Check-In)
```json
{
  "driver_profile_id": "user-uuid-fallback",
  "latitude": 28.4309,
  "longitude": 77.0371,
  "accuracy": 15.6,
  "odometer": 12450.5,
  "selfie_data": "base64...",
  "dashboard_data": "base64..."
}
```

### Observed Responses
- `401 UNAUTHORIZED`: Missing Authorization Header (Token missing or expired).
- `400 BAD REQUEST`: "Active deployment is required before attendance can be marked." (Driver hasn't started trip).
- `404 NOT FOUND`: `/api/v1/attendance/history` (Missing route).

---

## 4. Minimum Possible Fixes (Proposed)

### Fix 1: Implement Missing API Route (Backend)
Add `/history` route to `app/api/v1/attendance/routes.py` and link it to `AttendanceService.list_attendance_history`.

### Fix 2: Update User DTO (Backend)
Include `driver_profile_id` in `User.to_dict()` to ensure Android app has the correct UUID.

### Fix 3: Fix GPS Worker Result (Android)
Update `GpsSyncWorker.kt` to return `Result.retry()` or `Result.failure()` when `!response.isSuccessful`.

### Fix 4: Validate Sync Result in UI (Android)
Update `AttendanceViewModel.kt` to show an error message if `saveCheckIn` returns false or fails immediately.

### Fix 5: Synchronization Fix (Backend)
In `ensure_helper_profile`, ensure `user.company_id` and `user.circle_id` are updated to match the profile to prevent dashboard filtering issues.
