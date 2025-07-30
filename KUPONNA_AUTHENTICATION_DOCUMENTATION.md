# Kuponna Authentication API Endpoints

- [POST /api/auth/register](#post-apiauthregister)
- [POST /api/auth/merchant/register](#post-apiauthmerchantregister) **[Mobile App Flow]**
- [PUT /api/auth/merchant](#put-apiauthmerchant) **[Mobile/App Only]**
- [POST /api/auth/verify-email](#post-apiauthverify-email)
- [POST /api/auth/resend-verification](#post-apiauthresend-verification)
- [POST /api/auth/login](#post-apiauthlogin)
- [POST /api/auth/google-login](#post-apiauthgoogle-login) **[Mobile/App Only]**
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

### Environment Variables Required

For Google OAuth functionality to work properly, the following environment variables must be configured:

```bash
# Google OAuth Configuration (Required for /api/auth/google-login)
NEXT_PUBLIC_GOOGLE_OAUTH_CLIENT_ID=your-web-client-id.apps.googleusercontent.com

# Optional: If you have separate mobile credentials
GOOGLE_ANDROID_CLIENT_ID=your-android-client-id.apps.googleusercontent.com
GOOGLE_IOS_CLIENT_ID=your-ios-client-id.apps.googleusercontent.com
```

**How to get these credentials:**

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create or select a project
3. Enable Google+ API or Google Identity API
4. Go to "APIs & Services" → "Credentials"
5. Create OAuth 2.0 Client IDs for:
   - **Web application** (required - used by mobile apps too)
   - **Android** (optional - if you want separate credentials)
   - **iOS** (optional - if you want separate credentials)

**Important:** Mobile apps should use the **Web Client ID** in their configuration, not separate mobile credentials (unless specifically needed).

## Authentication Endpoints

### POST /api/auth/register

Register a new user account.

#### Request Body

```typescript
{
  email: string;      // Required, valid email format
  password: string;   // Required, min 8 characters
  name: string;       // Required, min 2 characters, user's full name
  role: string;       // Required, enum: ['user', 'merchant']
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
    name: string;      // Required, min 2 characters, user's full name
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

### POST /api/auth/merchant/register

Register a new merchant account with business profile (Mobile App Flow).

#### Request Body

```typescript
{
  // Basic user information
  name: string;         // Required, min 2 characters, full name
  email: string;        // Required, valid email format
  password: string;     // Required, min 8 characters
  phoneNumber?: string; // Optional, personal phone number
  
  // Business information
  businessName: string;     // Required, min 2 characters
  businessEmail: string;    // Required, valid email format, must be unique
  businessPhone: string;    // Required, min 10 characters
  businessAddress: string;  // Required, min 5 characters
  businessCategory: string; // Required, min 2 characters
  category?: string;        // Optional, main category ID from /api/categories/grouping
  subCategory?: string;     // Optional, sub category ID
  location?: string;        // Optional, business location
  
  // Document uploads (all optional)
  logo?: string;      // Optional, URL to business logo image
  license?: string;   // Optional, URL to business license document
  banner?: string;    // Optional, URL to business banner/cover image
  fileUrl?: string;   // Optional, URL to additional business documents
}
```

#### Response 201 (application/json)

```typescript
{
  message: string; // Required, "Merchant registered successfully..."
  token: string;   // Required, JWT token for immediate login
  user: {
    id: string;             // Required, UUID v4 format
    name: string;           // Required, user's full name
    email: string;          // Required, valid email format
    role: string;           // Required, "merchant"
    phoneNumber?: string;   // Optional, phone number
    isVerified: boolean;    // Required, false (email verification pending)
    merchantVerified: boolean; // Required, false (admin approval pending)
    location?: string;      // Optional, user location
    category?: string;      // Optional, main category
    subCategory?: string;   // Optional, sub category
    createdAt: string;      // Required, ISO 8601 datetime format
    updatedAt: string;      // Required, ISO 8601 datetime format
  };
  merchantProfile: {
    id: string;           // Required, UUID v4 format
    businessName: string; // Required, business name
    status: string;       // Required, "pending" (awaiting admin approval)
  };
}
```

#### Error Responses

```typescript
// 400 Bad Request - Validation errors
{
  error: string;        // Required, error message
  details?: object[];   // Optional, Zod validation error details
}

// 400 Bad Request - Duplicate user
{
  error: "User already exists"; // User email already registered
}

// 400 Bad Request - Duplicate business email
{
  error: "Business email already registered"; // Business email already in use
}

// 500 Internal Server Error
{
  error: "Internal server error";
}
```

#### Mobile App Flow Integration

The merchant registration follows a 7-step mobile app flow:

1. **Step 1**: User fills basic info (name, email, password)
2. **Step 2**: User selects categories using `/api/categories/grouping`
3. **Step 3**: User enters business details and phone
4. **Step 4**: User uploads documents (logo, license, banner)
5. **Step 5**: Account created with pending status
6. **Step 6**: Email verification required (code sent automatically)
7. **Step 7**: Admin approval needed for `merchantVerified: true`

#### Implementation Notes

- Creates both `User` and `MerchantProfile` records in a single transaction
- Merchant profile status starts as "pending" requiring admin approval
- Email verification code sent automatically to user's email
- User receives JWT token and can login immediately
- Features are limited until both `isVerified` and `merchantVerified` are true
- Document uploads should be completed before calling this endpoint
- Business email must be unique across all merchant profiles
- Uses `/api/categories/grouping` for category selection in mobile UI

#### Usage Example

```typescript
// Mobile app registration
const merchantData = {
  // Basic info (Step 1)
  name: "John Doe",
  email: "john@example.com", 
  password: "securepass123",
  phoneNumber: "+2348012345678",
  
  // Business info (Steps 2-3)
  businessName: "Doe Electronics",
  businessEmail: "info@doeelectronics.com",
  businessPhone: "+2348087654321", 
  businessAddress: "123 Business St, Lagos",
  businessCategory: "Electronics",
  category: "digital-category-uuid",
  subCategory: "electronics-subcategory-uuid",
  location: "Lagos, Nigeria",
  
  // Documents (Step 4)
  logo: "https://cdn.example.com/logo.jpg",
  license: "https://cdn.example.com/license.pdf",
  banner: "https://cdn.example.com/banner.jpg"
};

const response = await fetch('/api/auth/merchant/register', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(merchantData)
});
```

[Back to top](#authentication-api-endpoints)

### PUT /api/auth/merchant

**[Mobile/App Only]** Update merchant profile information (comprehensive update for mobile apps).

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
  // Basic user information (all optional)
  name?: string;        // Min 2 characters, user's full name
  phoneNumber?: string; // Phone number
  location?: string;    // User location
  category?: string;    // Main category ID from /api/categories/grouping
  subCategory?: string; // Sub category ID
  
  // Business information (all optional)
  businessName?: string;     // Min 2 characters, business name
  businessEmail?: string;    // Valid email, must be unique across merchants
  businessPhone?: string;    // Min 10 characters, business phone
  businessAddress?: string;  // Min 5 characters, business address
  businessCategory?: string; // Min 2 characters, business category
  
  // Document uploads (all optional)
  logo?: string;    // URL to business logo image
  license?: string; // URL to business license document
  banner?: string;  // URL to business banner/cover image
  fileUrl?: string; // URL to additional business documents
}
```

#### Response 200 (application/json)

```typescript
{
  message: string; // Required, "Merchant profile updated successfully"
  user: {
    id: string;              // Required, UUID v4 format
    email: string;           // Required, valid email format
    name: string;            // Required, user's full name
    role: string;            // Required, always "merchant"
    phoneNumber?: string;    // Optional, phone number
    location?: string;       // Optional, user location
    category?: string;       // Optional, main category
    subCategory?: string;    // Optional, sub category
    isVerified: boolean;     // Required, email verification status
    merchantVerified: boolean; // Required, admin approval status
    lastLogin: string;       // Required, ISO 8601 datetime format
    createdAt: string;       // Required, ISO 8601 datetime format
    updatedAt: string;       // Required, ISO 8601 datetime format
    // password field is excluded from response
  },
  merchantProfile: {
    id: string;                // Required, UUID v4 format
    userId: string;            // Required, UUID v4 format
    firstName: string;         // Required, extracted from name
    lastName: string;          // Required, extracted from name
    email: string;             // Required, same as user email
    phoneNumber?: string;      // Optional, phone number
    businessName: string;      // Required, business name
    businessEmail: string;     // Required, business email
    businessPhone: string;     // Required, business phone
    businessAddress: string;   // Required, business address
    businessCategory: string;  // Required, business category
    category?: string;         // Optional, main category
    subCategory?: string;      // Optional, sub category
    location?: string;         // Optional, business location
    logo?: string;            // Optional, logo URL
    license?: string;         // Optional, license document URL
    banner?: string;          // Optional, banner image URL
    fileUrl?: string;         // Optional, additional documents URL
    status: string;           // Required, "pending" | "approved" | "cancelled" | "rejected"
    createdAt: string;        // Required, ISO 8601 datetime format
    updatedAt: string;        // Required, ISO 8601 datetime format
  }
}
```

#### Error Responses

**401 Unauthorized** - Authentication required or invalid token:
```typescript
{
  error: string; // "Authentication required" | "No token provided" | "Invalid token" | "User not found"
}
```

**403 Forbidden** - User is not a merchant:
```typescript
{
  error: "Access denied. Merchant role required";
}
```

**404 Not Found** - Merchant profile not found:
```typescript
{
  error: "Merchant profile not found";
}
```

**400 Bad Request** - Validation errors or business email conflict:
```typescript
{
  error: string; // "Invalid input data" | "Business email already registered by another merchant"
  details?: Array<{
    field: string;    // Field name with error
    message: string;  // Error message
  }>
}
```

**500 Internal Server Error**:
```typescript
{
  error: "Internal server error";
}
```

#### Usage Example

```typescript
// Mobile app merchant profile update
const updateMerchantProfile = async (updates: any) => {
  try {
    const response = await fetch('/api/auth/merchant', {
      method: 'PUT',
      headers: {
        'Authorization': `Bearer ${authToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        name: "John Doe Business",
        businessName: "Doe Enterprises",
        businessPhone: "+1234567890",
        businessAddress: "123 Business St, City, State",
        logo: "https://cdn.example.com/logo.jpg",
        banner: "https://cdn.example.com/banner.jpg"
      })
    });

    const data = await response.json();
    
    if (response.ok) {
      console.log('Profile updated:', data.user, data.merchantProfile);
      // Update local state with new profile data
    } else {
      console.error('Update failed:', data.error);
      if (data.details) {
        // Handle validation errors
        data.details.forEach(detail => {
          console.error(`${detail.field}: ${detail.message}`);
        });
      }
    }
  } catch (error) {
    console.error('Network error:', error);
  }
};
```

#### Key Points for Mobile Developers

- **Flexible Updates**: All fields are optional - only send fields you want to update
- **Dual Model Updates**: Updates both User and MerchantProfile models automatically
- **Business Email Uniqueness**: Business email must be unique across all merchants
- **Name Splitting**: The `name` field is automatically split into `firstName` and `lastName` in merchant profile
- **Authentication Required**: Must include valid JWT token with merchant role
- **Status Preservation**: Cannot change merchant profile status (requires admin approval)
- **Document Management**: All document fields accept URLs to uploaded files
- **Category Integration**: Use `/api/categories/grouping` endpoint to get valid category options

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
    name: string;      // Required, min 2 characters, user's full name
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
    name: string;      // Required, min 2 characters, user's full name
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

### POST /api/auth/google-login

**[Mobile/App Only]** Authenticate a user using Google OAuth ID token. This endpoint is specifically designed for mobile and app developers.

#### Request Body

```typescript
{
  idToken: string;      // Required, Google ID token from mobile SDK
  platform?: string;   // Optional, enum: ['android', 'ios']
  deviceInfo?: {        // Optional, device information for security
    deviceId?: string;    // Unique device identifier
    deviceName?: string;  // Device name (e.g., "iPhone 14 Pro")
    osVersion?: string;   // OS version (e.g., "iOS 16.1" or "Android 13")
    appVersion?: string;  // App version (e.g., "1.0.0")
  }
}
```

#### Response 200 (application/json)

```typescript
{
  success: boolean;     // Required, always true for successful requests
  message: string;      // Required, success message
  data: {
    token: string;      // Required, JWT token for API authentication (30 days expiration)
    user: {
      id: string;             // Required, UUID v4 format
      email: string;          // Required, valid email format
      name: string;           // Required, user's full name from Google
      fullName: string;       // Required, same as name (for mobile convenience)
      role: string;           // Required, always "user" for Google signups
      phoneNumber?: string;   // Optional, null for new Google users
      picture?: string;       // Optional, Google profile picture URL 
      isVerified: true;    // Required, automatically true for Google users 
      lastLogin: string;      // Required, ISO 8601 datetime format
      createdAt: string;      // Required, ISO 8601 datetime format
      updatedAt: string;      // Required, ISO 8601 datetime format
    },
    auth: {
      method: string;         // Required, always "google"
      action: string;         // Required, "login" or "signup"
      isNewUser: boolean;     // Required, true if account was just created
      tokenExpiresIn: string; // Required, always "30d" (30 days)
      isVerified: boolean;    // Required, verification status
    },
    appData: {
      shouldCompleteProfile: boolean;   // Required, true if profile is incomplete
      requiresVerification: boolean;    // Required, true if email needs verification
      availableFeatures: {
        canCreateDeals: boolean;        // Required, merchant feature availability
        canJoinGroups: boolean;         // Required, group feature availability
        canMakePurchases: boolean;      // Required, purchase feature availability
      }
    }
  }
}
```

#### Error Responses

**400 Bad Request** - Invalid or expired Google ID token:
```typescript
{
  error: string;        // "Invalid or expired Google ID token"
  details: string;      // "Please try signing in with Google again"
}
```

**400 Bad Request** - Invalid request data:
```typescript
{
  error: string;        // "Invalid request data"
  details: Array<{
    field: string;      // Field name with error
    message: string;    // Error message
  }>
}
```

#### Usage Example

```typescript
// Mobile app implementation
const signInWithGoogle = async () => {
  try {
    // 1. Get Google ID token from your platform's Google Sign-In SDK
    const googleIdToken = await getGoogleIdToken(); // Your implementation
    
    // 2. Send to Kuponna API
    const response = await fetch('/api/auth/google-login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        idToken: googleIdToken,
        platform: Platform.OS, // 'android' or 'ios'
        deviceInfo: {
          deviceId: await getDeviceId(),
          deviceName: await getDeviceName(),
          osVersion: Platform.Version,
          appVersion: '1.0.0'
        }
      })
    });

    const data = await response.json();
    
    if (data.success) {
      // 3. Store the token securely
      await secureStorage.setItem('authToken', data.data.token);
      
      // 4. Handle based on user status
      if (data.data.auth.isNewUser) {
        // Navigate to profile completion or welcome screen
      } else {
        // Navigate to main app
      }
      
      // 5. Use token for subsequent API calls
      const apiResponse = await fetch('/api/protected-endpoint', {
        headers: {
          'Authorization': `Bearer ${data.data.token}`,
          'Content-Type': 'application/json'
        }
      });
    }
  } catch (error) {
    console.error('Google login failed:', error);
  }
};
```

#### Key Points for Mobile Developers

- **Use Web Client ID** in your mobile app configuration (not platform-specific client IDs)
- **Send ID Token** (not access token) from Google Sign-In SDK
- **Store JWT securely** using platform keychain/keystore
- **Token expires in 30 days** - implement refresh logic or re-authenticate
- **Use Bearer token** for all subsequent API calls: `Authorization: Bearer ${token}`

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
    name: string;      // Required, min 2 characters, user's full name
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
    name: string;      // Required, min 2 characters, user's full name
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
  name?: string;           // Optional, user's full name
  phoneNumber?: string;    // Optional, phone number
  picture?: string;        // Optional, profile picture URL
  country?: string;        // Optional, user's country
  timezone?: string;       // Optional, user's timezone
  bio?: string;           // Optional, user biography/description
  pushNotification?: boolean;  // Optional, enable/disable push notifications
  emailNotification?: boolean; // Optional, enable/disable email notifications  
  smsNotification?: boolean;   // Optional, enable/disable SMS notifications
  categories?: string[];   // Optional, array of user's interested categories
  // Note: The following fields cannot be updated via this endpoint:
  // id, email, password, role, createdAt, updatedAt, lastLogin, isVerified, merchantVerified
  // Merchant-specific fields (merchantRole, available_balance, book_balance, monthly_target) are also not updatable through this endpoint
}
```

#### Response 200 (application/json)

```typescript
{
  message: string; // Required, "Profile updated successfully"
  user: {
    id: string;                   // Required, UUID v4 format
    email: string;                // Required, valid email format
    name: string;                 // Required, user's full name
    role: string;                 // Required, enum: ['user', 'merchant', 'admin']
    phoneNumber?: string;         // Optional, phone number
    picture?: string;             // Optional, profile picture URL
    country?: string;             // Optional, user's country
    timezone?: string;            // Optional, user's timezone
    bio?: string;                 // Optional, user biography/description
    isVerified: boolean;          // Required, email verification status
    merchantVerified?: boolean;   // Optional, merchant verification status (for merchant users)
    pushNotification: boolean;    // Required, push notification preference
    emailNotification: boolean;   // Required, email notification preference
    smsNotification: boolean;     // Required, SMS notification preference
    categories: string[];         // Required, array of user's interested categories
    lastLogin: string | null;     // Optional, ISO 8601 datetime format
    createdAt: string;            // Required, ISO 8601 datetime format
    updatedAt: string;            // Required, ISO 8601 datetime format
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

## Data Models

### MerchantProfile Interface

```typescript
interface MerchantProfile {
  id: string;                    // Required, UUID v4 format
  userId: string;                // Required, UUID v4 format, foreign key to User
  firstName: string;             // Required, merchant's first name
  lastName: string;              // Required, merchant's last name
  email: string;                 // Required, merchant's personal email
  phoneNumber?: string;          // Optional, merchant's personal phone
  businessName: string;          // Required, business name
  businessEmail: string;         // Required, business email (unique)
  businessPhone: string;         // Required, business phone number
  businessAddress: string;       // Required, business address
  businessCategory: string;      // Required, business category
  category?: string;             // Optional, main category UUID
  subCategory?: string;          // Optional, sub category UUID
  location?: string;             // Optional, business location
  logo?: string;                 // Optional, URL to business logo
  license?: string;              // Optional, URL to business license document
  banner?: string;               // Optional, URL to business banner image
  fileUrl?: string;              // Optional, URL to additional documents
  status: "pending" | "approved" | "cancelled" | "rejected"; // Required
  createdAt: string;             // Required, ISO 8601 datetime format
  updatedAt: string;             // Required, ISO 8601 datetime format
  // Relationships
  user?: User;                   // Optional, associated user object
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
