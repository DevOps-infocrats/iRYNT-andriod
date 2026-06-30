# Android Endpoint Matrix

| Feature | Method | URL | Request DTO | Response DTO | Status |
|---------|--------|-----|-------------|--------------|--------|
| Auth | POST | `api/v1/auth/login` | `LoginRequest` | `LoginResponse` | ✓ Existing |
| Auth | POST | `api/v1/auth/refresh` | `String` (Body) | `LoginResponse` | ✓ Existing |
| Profile | GET | `api/v1/user/profile` | - | `UserDto` | ✓ Existing |
| Profile | POST | `api/v1/user/profile-pic` | `MultipartBody.Part` | `UserDto` | ✓ Existing |
| Profile | POST | `api/v1/user/documents` | `MultipartBody.Part` | `Unit` | ✓ Existing |
| Vehicle | GET | `api/v1/vehicles/current` | - | `VehicleResponse` | ✓ Existing |
| Attendance | POST | `api/v1/attendance/check-in` | `CheckInRequest` | `AttendanceResponse` | ✓ Existing |
| Attendance | POST | `api/v1/attendance/check-out` | `CheckOutRequest` | `AttendanceResponse` | ✓ Existing |
| Attendance | POST | `api/v1/attendance/gps/sync` | `GpsSyncRequest` | `Unit` | ✓ Existing |
| Attendance | GET | `api/v1/attendance/history` | - | `AttendanceHistoryResponse` | ✗ Mismatch/Missing |
| Deployment | GET | `api/v1/deployments/current` | - | `DeploymentResponse` | ✓ Existing |
| Deployment | PUT | `api/v1/deployments/{id}/status` | `UpdateDeploymentRequest` | `DeploymentResponse` | ✓ Existing |

*Note: "Status" indicates if the endpoint was found to be working in previous logs or defined in code. 404 was observed for `/api/v1/attendance/history` on local backend.*
