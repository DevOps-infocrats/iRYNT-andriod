# JWT Flow Analysis

## 1. Token Storage
- **Location:** `EncryptedSharedPreferences` with filename `secure_prefs`.
- **Keys:** `access_token`, `refresh_token`.
- **Method:** `TokenManager.saveTokens(accessToken, refreshToken)` uses `prefs.edit().apply{...}.commit()`.

## 2. Token Retrieval
- **Method:** `TokenManager.getAccessToken()` uses `prefs.getString("access_token", null)`.
- **Context:** Used by `AuthInterceptor` to attach the token to outgoing requests.

## 3. Token Existence
- **Initial State:** Null before login.
- **After Login:** Correctly saved (Logged: `Saving tokens successful: true`).
- **Observation:** `AuthInterceptor` reports `Token exists: false` even after successful login logs.

## 4. Authorization Header Attachment
- **Mechanism:** `AuthInterceptor` intercepts all requests.
- **Logic:**
  ```kotlin
  val accessToken = tokenManager.getAccessToken()
  if (accessToken != null) {
      val authenticatedRequest = originalRequest.newBuilder()
          .header("Authorization", "Bearer $accessToken")
          .build()
      chain.proceed(authenticatedRequest)
  } else {
      chain.proceed(originalRequest)
  }
  ```
- **Issue:** `accessToken` is null in `AuthInterceptor` for requests following login, leading to "Missing Authorization Header" from backend.

## 5. Potential Root Causes
- **Singleton Desync:** Although Hilt is used with `@Singleton`, multiple instances of `TokenManager` might exist if something is bypassing Hilt or if `EncryptedSharedPreferences` initialization is problematic.
- **Context Issues:** `TokenManager` uses `EncryptedSharedPreferences` which relies on a `MasterKey` and `context`.
- **Timing:** `LoginViewModel` saves the token and immediately navigates/triggers requests. `AuthInterceptor` might be seeing a stale state if the `SharedPreferences` instance hasn't updated its memory cache.
