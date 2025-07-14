# Kuponna Authentication API Endpoints

- [POST /api/auth/register](#post-apiauthregister)
- [POST /api/auth/verify-email](#post-apiauthverify-email)
- [POST /api/auth/resend-verification](#post-apiauthresend-verification)
- [POST /api/auth/login](#post-apiauthlogin)
- [GET /api/auth/refresh](#post-apiauthrefresh)
- [POST /api/auth/logout](#post-apiauthlogout)
- [POST /api/auth/forgot-password](#post-apiauthforgot-password)
- [POST /api/auth/resend-reset-code](#post-apiauthresend-reset-code)
- [POST /api/auth/verify-reset-token](#post-apiauthverify-reset-token)
- [POST /api/auth/reset-password](#post-apiauthreset-password)
- [PATCH /api/auth/change-password](#post-apiauthchange-password)
- [POST /api/auth/resend-email-code](#post-apiauthresend-email-code)
- [POST /api/auth/verify-email-code](#post-apiauthverify-email-code)
- [PATCH /api/user/profile](#patch-apiuserprofile)

---

## Authentication & Authorization Flow

### Overview

The authentication system uses JWT (JSON Web Tokens) for secure authentication. The flow consists of the following steps:

> **Note for App/Mobile Developers:**
> - While web clients use HTTP-only cookies for authentication, mobile and app clients should use the `Authorization: Bearer <token>` header for all authenticated requests. All endpoints that require authentication accept either cookies (for browsers) or bearer tokens (for apps).
> - For mobile apps, implement the "Remember Me" functionality in your login UI and pass the `rememberMe` parameter in the login request to extend session duration to 30 days.
> - When implementing auto-login features, respect the user's choice for session duration and store tokens securely using platform-specific secure storage (Keychain on iOS, Keystore on Android).

1. **Registration** (`POST /api/auth/register`)
   - Create a new user account. This is the first step for any new user.
2. **Email Verification** (`POST /api/auth/verify-email`)
   - Verify the user's email address using a code sent to their email. Required before login is allowed.
3. **Resend Verification** (`POST /api/auth/resend-verification`)
   - If the user did not receive or lost the verification code, this endpoint resends it.
4. **Login** (`POST /api/auth/login`)
   - Authenticate a user and receive a JWT token. Only possible after email verification.
5. **Token Refresh** (`GET /api/auth/refresh`)
   - Automatically refresh an expired JWT token to keep the user logged in without re-entering credentials.
6. **Logout** (`POST /api/auth/logout`)
   - Clear the authentication state and invalidate the user's session.
7. **Forgot Password** (`POST /api/auth/forgot-password`)
   - Initiate a password reset by sending a reset code to the user's email.
8. **Resend Reset Code** (`POST /api/auth/resend-reset-code`)
   - Resend the password reset code if the user did not receive it or it expired.
9. **Verify Reset Token** (`POST /api/auth/verify-reset-token`)
   - Verify the reset code sent to the user's email before allowing password reset.
10. **Reset Password** (`POST /api/auth/reset-password`)
    - Set a new password after verifying the reset code.
11. **Change Password** (`PATCH /api/auth/change-password`)
    - Change the password for a logged-in user (requires current password).
12. **Resend Email Code (Alternative)** (`POST /api/auth/resend-email-code`)
    - Resend the email verification code using the user's email (alternative to the authenticated resend).
13. **Verify Email Code (Alternative)** (`POST /api/auth/verify-email-code`)
    - Alternative way to verify email using both email and code (useful for mobile or custom flows).

### Token Management

- Tokens are stored in HTTP-only cookies for security
- Default token expiration: 24 hours
- Extended token expiration: 30 days (when `rememberMe: true` is used during login)
- Refresh tokens maintain the same expiration duration as the original token
- All tokens are signed with HS256 algorithm

### Remember Me Functionality

The authentication system supports a "Remember Me" option that extends the session duration:

- **Default behavior**: Tokens expire after 24 hours
- **Remember Me enabled**: Tokens expire after 30 days
- **Session persistence**: When refreshing tokens, the system maintains the original expiration duration
- **Implementation**: Pass `rememberMe: true` in the login request body

#### Usage Example

```typescript
// Standard login (24-hour session)
const loginResponse = await fetch('/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'user@example.com',
    password: 'password123',
    rememberMe: false // or omit this field
  })
});

// Extended login (30-day session)
const extendedLoginResponse = await fetch('/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'user@example.com',
    password: 'password123',
    rememberMe: true
  })
});
```

### Security Features

- HTTP-only cookies prevent XSS attacks
- Secure flag in production
- Strict same-site policy
- Password hashing with bcrypt
- CSRF protection
- Rate limiting on auth endpoints

## Authentication Endpoints

### POST /api/auth/register

Register a new user account.

#### Request Body

```typescript
{
  email: string;      // Required, valid email format
  password: string;   // Required, min 8 characters
  firstName: string;  // Required, min 2 characters
  lastName: string;   // Required, min 2 characters
  role: string;      // Required, enum: ['user', 'merchant']
  phoneNumber?: string; // Optional, phone number
}
```

#### Response 201 (application/json)

```typescript
{
  message: string;    // Required, "User registered successfully"
  user: {
    id: string;        // Required, UUID v4 format
    email: string;     // Required, valid email format
    firstName: string; // Required, min 2 characters
    lastName: string;  // Required, min 2 characters
    role: string;      // Required, enum: ['user', 'merchant']
    phoneNumber?: string; // Optional, phone number
    isVerified: boolean;  // Required, default: false
    lastLogin: string | null; // Optional, ISO 8601 datetime format
    createdAt: string; // Required, ISO 8601 datetime format
    updatedAt: string; // Required, ISO 8601 datetime format
  }
}
```

[Back to top](#authentication-api-endpoints)

### POST /api/auth/verify-email

Verify a user's email address using the verification code.

#### Request Headers

```typescript
// For web (browser):
{
  Cookie: string; // Required, contains httpOnly JWT token
}
// For app/mobile:
{
  Authorization: string; // Required, Bearer <JWT token>
}
```

#### Request Body

```typescript
{
  code: string; // Required, 6-character verification code
}
```

#### Response 200 (application/json)

```typescript
{
  message: string;   // Required, "Email verified successfully"
  user: {
    id: string;        // Required, UUID v4 format
    email: string;     // Required, valid email format
    firstName: string; // Required, min 2 characters
    lastName: string;  // Required, min 2 characters
    role: string;      // Required, enum: ['user', 'merchant']
    phoneNumber?: string; // Optional, phone number
    isVerified: boolean;  // Required, true
    lastLogin: string | null; // Optional, ISO 8601 datetime format
    createdAt: string; // Required, ISO 8601 datetime format
    updatedAt: string; // Required, ISO 8601 datetime format
  }
}
```

[Back to top](#authentication-api-endpoints)

### POST /api/auth/resend-verification

Resend the verification email to the user.

#### Request Headers

```typescript
// For web (browser):
{
  Cookie: string; // Required, contains httpOnly JWT token
}
// For app/mobile:
{
  Authorization: string; // Required, Bearer <JWT token>
}
```

#### Response 200 (application/json)

```typescript
{
  message: string; // Required, "Verification email sent successfully"
}
```

### POST /api/auth/login

Authenticate a user and receive an access token.

#### Request Body

```typescript
{
  email: string; // Required, valid email format
  password: string; // Required, min 8 characters
  rememberMe?: boolean; // Optional, extends session to 30 days if true
}
```

#### Response 200 (application/json)

```typescript
{
  message: string;   // Required, "Login successful"
  user: {
    id: string;        // Required, UUID v4 format
    email: string;     // Required, valid email format
    firstName: string; // Required, min 2 characters
    lastName: string;  // Required, min 2 characters
    role: string;      // Required, enum: ['user', 'merchant']
    phoneNumber?: string; // Optional, phone number
    isVerified: boolean;  // Required
    lastLogin: string; // Required, ISO 8601 datetime format
    createdAt: string; // Required, ISO 8601 datetime format
    updatedAt: string; // Required, ISO 8601 datetime format
  }
}
```

[Back to top](#authentication-api-endpoints)

### GET /api/auth/refresh

Refresh an expired access token while maintaining the original session duration.

**Important**: The refresh endpoint automatically detects whether the original token was created with "Remember Me" functionality and maintains the same expiration duration (24 hours or 30 days).

#### Request Headers

```typescript
// For web (browser):
{
  Cookie: string; // Required, contains httpOnly JWT token
}
// For app/mobile:
{
  Authorization: string; // Required, Bearer <JWT token>
}
```

#### Response 200 (application/json)

```typescript
{
  message: string;   // Required, "Token refreshed successfully"
  user: {
    id: string;        // Required, UUID v4 format
    email: string;     // Required, valid email format
    firstName: string; // Required, min 2 characters
    lastName: string;  // Required, min 2 characters
    role: string;      // Required, enum: ['user', 'merchant']
    phoneNumber?: string; // Optional, phone number
    isVerified: boolean;  // Required
    lastLogin: string; // Required, ISO 8601 datetime format
    createdAt: string; // Required, ISO 8601 datetime format
    updatedAt: string; // Required, ISO 8601 datetime format
  }
}
```

[Back to top](#authentication-api-endpoints)

### POST /api/auth/logout

Logout the current user and invalidate their token.

#### Request Headers

```typescript
// For web (browser):
{
  Cookie: string; // Required, contains httpOnly JWT token
}
// For app/mobile:
{
  Authorization: string; // Required, Bearer <JWT token>
}
```

#### Response 200 (application/json)

```typescript
{
  message: string; // Required, "Logged out successfully"
}
```

[Back to top](#authentication-api-endpoints)

### POST /api/auth/forgot-password

Request a password reset by sending an OTP to the user's email.

#### Request Body

```typescript
{
  email: string; // Required, valid email format
}
```

#### Response 200 (application/json)

```typescript
{
  message: string; // Required, "Password reset instructions sent to your email"
}
```

### POST /api/auth/resend-reset-code

Resend the password reset OTP code.

#### Request Body

```typescript
{
  email: string; // Required, valid email format
}
```

#### Response 200 (application/json)

```typescript
{
  message: string; // Required, "Reset code sent successfully"
}
```

### POST /api/auth/verify-reset-token

Verify the OTP token sent to the user's email.

#### Request Body

```typescript
{
  email: string; // Required, valid email format
  token: string; // Required, 6-character OTP code
}
```

#### Response 200 (application/json)

```typescript
{
  message: string; // Required, "Token verified successfully"
  resetToken: string; // Required, JWT token for password reset
}
```

### POST /api/auth/reset-password

Reset the user's password using the verified OTP token.

#### Request Headers

```typescript
// For web (browser):
{
  Authorization: string; // Required, Bearer token from verify-reset-token
}
// For app/mobile:
{
  Authorization: string; // Required, Bearer token from verify-reset-token
}
```

#### Request Body

```typescript
{
  email: string; // Required, valid email format
  token: string; // Required, 6-character OTP code
  newPassword: string; // Required, min 8 characters
}
```

#### Response 200 (application/json)

```typescript
{
  message: string; // Required, "Password reset successfully"
}
```

### POST /api/auth/change-password

Change the user's password while logged in.

#### Request Headers

```typescript
// For web (browser):
{
  Cookie: string; // Required, contains httpOnly JWT token
}
// For app/mobile:
{
  Authorization: string; // Required, Bearer <JWT token>
}
```

#### Request Body

```typescript
{
  currentPassword: string; // Required, current password
  newPassword: string; // Required, new password
}
```

#### Response 200 (application/json)

```typescript
{
  message: string; // Required, "Password changed successfully"
}
```

### POST /api/auth/resend-email-code

Resend the email verification code to the user.

#### Request Body

```typescript
{
  email: string; // Required, valid email format
}
```

#### Response 200 (application/json)

```typescript
{
  message: string; // Required, "Verification code sent successfully"
}
```

### POST /api/auth/verify-email-code

Verify a user's email address using the verification code.

#### Request Body

```typescript
{
  email: string; // Required, valid email format
  code: string; // Required, 6-character verification code
}
```

#### Response 200 (application/json)

```typescript
{
  message: string;   // Required, "Email verified successfully"
  user: {
    id: string;        // Required, UUID v4 format
    email: string;     // Required, valid email format
    firstName: string; // Required, min 2 characters
    lastName: string;  // Required, min 2 characters
    role: string;      // Required, enum: ['user', 'merchant']
    phoneNumber?: string; // Optional, phone number
    isVerified: boolean;  // Required, true
    lastLogin: string | null; // Optional, ISO 8601 datetime format
    createdAt: string; // Required, ISO 8601 datetime format
    updatedAt: string; // Required, ISO 8601 datetime format
  }
}
```

### PATCH /api/user/profile

Update the user's profile information.

#### Request Headers

```typescript
// For web (browser):
{
  Cookie: string; // Required, contains httpOnly JWT token
}
// For app/mobile:
{
  Authorization: string; // Required, Bearer <JWT token>
}
```

#### Request Body

```typescript
{
  firstName?: string;  // Optional, min 2 characters
  lastName?: string;   // Optional, min 2 characters
  phoneNumber?: string; // Optional, phone number
}
```

#### Response 200 (application/json)

```typescript
{
  message: string; // Required, "Profile updated successfully"
  user: {
    id: string;        // Required, UUID v4 format
    email: string;     // Required, valid email format
    firstName: string; // Required, min 2 characters
    lastName: string;  // Required, min 2 characters
    role: string;      // Required, enum: ['user', 'merchant']
    phoneNumber?: string; // Optional, phone number
    isVerified: boolean;  // Required
    lastLogin: string | null; // Optional, ISO 8601 datetime format
    createdAt: string; // Required, ISO 8601 datetime format
    updatedAt: string; // Required, ISO 8601 datetime format
  }
}
```

### PATCH /api/user/profile

Update the current user's profile information.

#### Request Headers

```typescript
// For web (browser):
{
  Cookie: string; // Required, contains httpOnly JWT token
}
// For app/mobile:
{
  Authorization: string; // Required, Bearer <JWT token>
}
```

#### Request Body

```typescript
{
  firstName?: string;    // Optional, min 2 characters
  lastName?: string;     // Optional, min 2 characters
  phoneNumber?: string;  // Optional, phone number
  // Note: The following fields cannot be updated via this endpoint:
  // id, password, createdAt, updatedAt, lastLogin, role, isVerified, merchantVerified
}
```

#### Response 200 (application/json)

```typescript
{
  message: string;   // Required, "Profile updated successfully"
  user: {
    id: string;        // Required, UUID v4 format
    email: string;     // Required, valid email format
    firstName: string; // Required, min 2 characters
    lastName: string;  // Required, min 2 characters
    role: string;      // Required, enum: ['user', 'merchant']
    phoneNumber?: string; // Optional, phone number
    isVerified: boolean;  // Required
    lastLogin: string; // Required, ISO 8601 datetime format
    createdAt: string; // Required, ISO 8601 datetime format
    updatedAt: string; // Required, ISO 8601 datetime format
  }
}
```

[Back to top](#authentication-api-endpoints)

## Error Responses

All endpoints may return the following error responses:

### 400 Bad Request

```typescript
{
  error: string;     // Required, error message (e.g., "Invalid input", "User already exists")
  details?: {        // Optional, present for validation errors
    field: string;   // Field name
    message: string; // Validation error message
  }[];
}
```

### 401 Unauthorized

```typescript
{
  error: string; // Required, error message (e.g., "Invalid credentials")
}
```

### 500 Internal Server Error

```typescript
{
  error: string; // Required, "Internal server error"
}
```

### 403 Forbidden

```typescript
{
  error: string; // Required, error message (e.g., "Email already verified", "Invalid verification code")
}
```

### 429 Too Many Requests

```typescript
{
  error: string; // Required, error message (e.g., "Too many verification attempts. Please try again later")
}
```

## Best Practices

### For Web Developers
1. Use TypeScript for type safety
2. Implement proper validation
3. Implement email verification
4. Set verification code expiration
5. Provide clear UI for "Remember Me" option

### For Mobile/App Developers
1. **Session Management**: 
   - Implement a "Remember for 30 days" toggle in your login UI
   - Store the user's preference locally for better UX
   - Use secure storage for tokens (Keychain/Keystore)

2. **Token Handling**:
   - Always include the `Authorization: Bearer <token>` header for authenticated requests
   - Implement automatic token refresh before expiration
   - Handle different session durations based on user preference

3. **User Experience**:
   - Show session duration information to users
   - Provide easy logout functionality
   - Handle session expiration gracefully

#### Example Mobile Implementation

```typescript
// Example for React Native or similar frameworks
interface LoginRequest {
  email: string;
  password: string;
  rememberMe: boolean;
}

// Login function with Remember Me support
async function login(credentials: LoginRequest) {
  try {
    const response = await fetch('https://your-api.com/api/auth/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(credentials),
    });

    if (response.ok) {
      const data = await response.json();
      
      // Store token securely
      await SecureStorage.setItem('authToken', data.token);
      
      // Store session preference
      await AsyncStorage.setItem('rememberMe', credentials.rememberMe.toString());
      
      return data;
    }
  } catch (error) {
    console.error('Login failed:', error);
    throw error;
  }
}

// Auto-refresh implementation
async function refreshToken() {
  try {
    const token = await SecureStorage.getItem('authToken');
    
    const response = await fetch('https://your-api.com/api/auth/refresh', {
      method: 'GET',
      headers: {
        'Authorization': `Bearer ${token}`,
      },
    });

    if (response.ok) {
      const data = await response.json();
      await SecureStorage.setItem('authToken', data.token);
      return data.token;
    }
  } catch (error) {
    console.error('Token refresh failed:', error);
    // Handle logout or re-authentication
  }
}
```

## Security Considerations

1. Tokens are stored in HTTP-only cookies
2. Passwords are hashed using bcrypt
3. Input validation is performed
4. Proper error handling is implemented
5. Secure headers are set
6. Proper logging is implemented
7. Email verification is required
8. Verification codes expire after 30 minutes
