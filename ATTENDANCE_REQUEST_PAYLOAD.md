# Attendance Request Payload Verification

## Current Request Body (Check-In)
```json
{
  "driver_profile_id": "RESOLVED_ID",
  "latitude": 28.1234,
  "longitude": 77.1234,
  "accuracy": 10.5,
  "odometer": 1500.0,
  "selfie_data": "base64...",
  "dashboard_data": "base64..."
}
```

## Missing Fields (As per Phase 2 Checklist)
- `user_id`: Missing
- `deployment_id`: Missing
- `vehicle_id`: Missing
- `helper_profile_id`: Missing
- `timestamp`: Missing

## ID Resolution Issue
- `resolvedDriverId` is currently calculated as: `tokenManager.getDriverProfileId() ?: tokenManager.getUserId() ?: ""`
- If `driver_profile_id` is null (not returned by login API), `user_id` is sent as `driver_profile_id`.
- This mismatch likely causes the backend to fail to link the attendance record to the correct driver profile, or it might be filtered out in the web app dashboard which expects a valid `driver_profile_id`.

## Backend Rejection Reason
- `ATTENDANCE_SYNC_ROOT_CAUSE_ANALYSIS.md` mentions that the backend returns `400 Bad Request` if there is no **Active** deployment.
- The request does not send `deployment_id`, so the backend must be looking up the active deployment for the `driver_profile_id`.
- If the `driver_profile_id` sent is actually the `user_id`, the lookup will fail.
