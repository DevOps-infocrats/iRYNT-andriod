# Attendance Response Analysis

## Observed Responses

### 1. Success Case
- **HTTP Status:** 200 OK
- **Body:** `{"success": true, "message": "Attendance marked", "data": {...}}`
- **Result:** Local Room record updated with `isSynced = true`.

### 2. Failure Case: Missing Deployment
- **HTTP Status:** 400 Bad Request
- **Body:** `{"success": false, "message": "Active deployment is required before attendance can be marked."}`
- **Reason:** The backend requires an active deployment session. If the driver hasn't "Started Trip" or if the `driver_profile_id` is incorrect, this error occurs.
- **Result:** `AttendanceRepository` returns `false`. `AttendanceViewModel` shows "Sync failed. Please ensure you have an active deployment and try again."

### 3. Failure Case: Unauthorized
- **HTTP Status:** 401 Unauthorized
- **Body:** `{"msg": "Token has expired"}` (Typical Flask-JWT-Extended response)
- **Result:** Request fails. `AuthInterceptor` logs: `Intercepting request to: ... Token exists: true`.

### 4. Failure Case: Missing Route
- **HTTP Status:** 404 Not Found
- **URL:** `api/v1/attendance/history`
- **Reason:** This endpoint is missing on the backend, preventing history reconciliation.
