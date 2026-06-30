# Room Sync Analysis

## Attendance Table State
- **`isSynced` flag:**
  - `0` (false) upon initial save.
  - `1` (true) only when API response `isSuccessful` AND `body.success == true`.
- **Pending Records:**
  - Records where `isSynced = 0` remain in the database indefinitely until successfully synced.
  - `AttendanceSyncWorker` attempts to sync these every time it runs.
- **Sync Status:**
  - If the backend returns `400 Bad Request` (e.g., "Missing Deployment"), `isSynced` remains `0`.
  - The worker will retry on every run, but if the underlying issue (e.g., incorrect ID or missing deployment) isn't fixed, it will stay in Room forever.
- **Data Persistence:**
  - `AttendanceDao.deleteAll()` exists but isn't called during normal flow (except maybe on logout).
  - This confirms that unsynced records remain in Room, but they aren't appearing in the Web App because they never reached the server successfully.
