# User Switching & Data Sync Fix Report

## 1. Root Causes
- **ViewModel Scoping**: ViewModels were scoped to `MainActivity`, which remains alive between logouts. They retained stale data from previous users.
- **Initialization Over-Reliance**: `ProfileViewModel` and others only initialized state once, failing to pick up new tokens/IDs after a user switch.
- **Global Worker Scheduling**: `GpsSyncWorker` was scheduled once globally in `onCreate`, potentially syncing logs across different user sessions.
- **Partial Logout Logic**: Logout only cleared `TokenManager` and `AttendanceDao`, leaving in-memory UI states untouched.

## 2. Files Inspected
- `com.vil.mobile.MainActivity.kt`
- `com.vil.mobile.ui.state.DashboardViewModel.kt`
- `com.vil.mobile.ui.state.AttendanceViewModel.kt`
- `com.vil.mobile.ui.state.ProfileViewModel.kt`
- `com.vil.mobile.ui.login.LoginViewModel.kt`
- `com.vil.mobile.ui.DriverProfileScreen.kt`

## 3. Files Modified
- `app/src/main/java/com/vil/mobile/MainActivity.kt`
- `app/src/main/java/com/vil/mobile/ui/state/DashboardViewModel.kt`
- `app/src/main/java/com/vil/mobile/ui/state/AttendanceViewModel.kt`
- `app/src/main/java/com/vil/mobile/ui/state/ProfileViewModel.kt`
- `app/src/main/java/com/vil/mobile/ui/login/LoginViewModel.kt`
- `app/src/main/java/com/vil/mobile/ui/DriverProfileScreen.kt`

## 4. Exact Changes Made
- **Implemented `resetState()`**: Added to `DashboardViewModel`, `AttendanceViewModel`, and `ProfileViewModel` to clear in-memory state flows.
- **Centralized `handleLogout`**: Unified logout logic in `MainActivity` to:
    - Reset all ViewModels.
    - Cancel all `WorkManager` tasks.
    - Clear `TokenManager` (secure prefs).
    - Stop the `LocationTrackingService`.
- **Session-Aware Worker Scheduling**: Moved `scheduleGpsSync()` from `onCreate` to the successful login callback.
- **Proactive Profile Fetch**: Added `loadProfile()` to `ProfileViewModel` and triggered it via `LaunchedEffect` in `DriverProfileScreen` to ensure fresh server data is always shown.
- **Immediate Session Refresh**: Added `refreshSession()` to `ProfileViewModel` to update in-memory user info immediately after a successful login.

## 5. Why changes are safe
- **Minimal Footprint**: Used existing ViewModel methods and localized changes to the navigation/session boundaries.
- **No UI Alterations**: preserved the original design and user flows.
- **No Schema Changes**: Leveraged existing DAO methods for data purging.

## 6. Regression Results
- ✓ User A login -> Logout -> User B login: No data from User A is visible.
- ✓ GPS logs for User A are cleared/stopped upon logout.
- ✓ Profile screen pulls latest metadata from server every time it opens.
- ✓ Dashboard correctly refreshes for each new user.
