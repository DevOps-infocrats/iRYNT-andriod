# Android API Analysis

## 1. Base URL
Current: `http://192.168.1.2:5000` (Defined in `com.vil.mobile.Config.BASE_URL`)

## 2. Retrofit Endpoints

### AuthApi
- `POST api/v1/auth/login`
- `POST api/v1/auth/refresh`

### UserApi
- `GET api/v1/user/profile`
- `POST api/v1/user/profile-pic` (Multipart)
- `POST api/v1/user/documents` (Multipart)

### VehicleApi
- `GET api/v1/vehicles/current`

### AttendanceApi
- `POST api/v1/attendance/check-in`
- `POST api/v1/attendance/check-out`
- `POST api/v1/attendance/gps/sync`
- `GET api/v1/attendance/history`

### DeploymentApi
- `GET api/v1/deployments/current`
- `PUT api/v1/deployments/{id}/status`

## 3. Request DTOs
- `LoginRequest`: `username`, `password`
- `CheckInRequest`: `driver_profile_id`, `latitude`, `longitude`, `accuracy`, `odometer`, `selfie_data` (Base64), `dashboard_data` (Base64)
- `CheckOutRequest`: (Same as CheckInRequest)
- `GpsSyncRequest`: `deployment_id`, `coordinates` (List of `GpsCoordinateDto`)
- `UpdateDeploymentRequest`: `status`

## 4. Response DTOs
- `LoginResponse`: `success`, `message`, `data` (`accessToken`, `refreshToken`, `user`)
- `VehicleResponse`: `success`, `message`, `data` (`VehicleDto`)
- `AttendanceResponse`: `success`, `message`, `data` (`AttendanceRecordDto`)
- `AttendanceHistoryResponse`: `success`, `message`, `data` (List of `AttendanceRecordDto`)
- `DeploymentResponse`: `success`, `message`, `data` (`DeploymentDto`)
- `UserDto`: `id`, `username`, `role`, `phone`, `profile_pic_url`, `driver_profile_id`

## 5. Token Flow
- Handled by `AuthInterceptor`.
- Retrieves token from `TokenManager`.
- Adds `Authorization: Bearer <token>` to requests if token exists.

## 6. GPS Worker Flow
- `GpsSyncWorker` (CoroutineWorker).
- Fetches unsynced logs from `LocationDao`.
- Sends to `api/v1/attendance/gps/sync` in chunks of 50.
- Uses `deployment_id` from `TokenManager` (fallback to "current").

## 7. Attendance Flow
- `AttendanceRepository` saves local record to Room.
- Immediately attempts remote sync via `AttendanceApi.checkIn/checkOut`.
- If remote fails, `AttendanceSyncWorker` handles background retry.
- Images are encoded to Base64 NO_WRAP.

## 8. Deployment Flow
- `DeploymentRepository` calls `api/v1/deployments/current`.
- Stores `deployment_id` in `TokenManager`.
- Update status via `PUT api/v1/deployments/{id}/status`.

## 9. Vehicle Flow
- `VehicleRepository` calls `api/v1/vehicles/current`.

## 10. Notification Flow
- Currently using `MockRepository.getNotifications()`. No API implementation found for notifications yet.
