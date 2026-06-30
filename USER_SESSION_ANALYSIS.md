# User Session & Data Sync Analysis

## 1. Data Storage & Persistence
- **Tokens & Basic Profile**: Stored in `EncryptedSharedPreferences` via `TokenManager`. Includes JWT tokens, user ID, name, role, and driver profile ID.
- **Attendance History**: Stored in a local Room database (`VILDatabase`). User isolation is implemented via `driverProfileId` in DAO queries.
- **GPS Logs**: Stored in Room (`gps_logs` table).

## 2. In-Memory State (Caches)
- **ViewModels**: `DashboardViewModel`, `AttendanceViewModel`, and `ProfileViewModel` are scoped to `MainActivity`. They hold `MutableStateFlow` objects that persist as long as the Activity is alive.
- **State Persistence Issue**: When a user logs out, the app navigates back to the Login screen, but the `MainActivity` is not finished. The ViewModels retain their last known `UiState`.
- **User Switching**: If a second user logs in, the ViewModels may briefly expose the first user's data before the `loadDashboard()` or equivalent refresh calls complete.

## 3. Session Management Flow
- **Login**: `LoginViewModel` calls `auth/login`, saves data to `TokenManager`, and updates `loginSuccess` state.
- **Logout**: `LoginViewModel.logout()` currently:
    1. Calls `attendanceRepository.clearLocalData()` (clears Room attendance table).
    2. Calls `tokenManager.clear()` (clears SharedPrefs).
    3. Resets its own `LoginUiState`.
- **Missing Logout Actions**:
    - Does NOT reset `DashboardViewModel` state.
    - Does NOT reset `AttendanceViewModel` state.
    - Does NOT reset `ProfileViewModel` state.
    - Does NOT cancel active `WorkManager` tasks (though some may fail gracefully due to missing tokens).
    - Does NOT stop the `LocationTrackingService` foreground service (handled partially in `MainActivity`).

## 4. Proposed Minimal Fixes
- Add `resetState()` methods to all session-aware ViewModels.
- Ensure `LoginViewModel.logout()` triggers these resets.
- Ensure all repositories invalidate any in-memory caches upon logout.
- Force a fresh fetch from the network immediately upon dashboard/profile loading for the new user.
