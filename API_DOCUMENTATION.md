# Kuponna API Documentation

## Authentication API Endpoints

- [POST /api/auth/register](#post-apiauthregister)
- [POST /api/auth/verify-email](#post-apiauthverify-email)
- [POST /api/auth/resend-verification](#post-apiauthresend-verification)
- [POST /api/auth/login](#post-apiauthlogin)
- [POST /api/auth/refresh](#post-apiauthrefresh)
- [POST /api/auth/logout](#post-apiauthlogout)
- [POST /api/auth/forgot-password](#post-apiauthforgot-password)
- [POST /api/auth/resend-reset-code](#post-apiauthresend-reset-code)
- [POST /api/auth/verify-reset-token](#post-apiauthverify-reset-token)
- [POST /api/auth/reset-password](#post-apiauthreset-password)
- [POST /api/auth/change-password](#post-apiauthchange-password)
- [POST /api/auth/resend-email-code](#post-apiauthresend-email-code)
- [POST /api/auth/verify-email-code](#post-apiauthverify-email-code)

---

## Authentication & Authorization Flow

### Overview

The authentication system uses JWT (JSON Web Tokens) for secure authentication. The flow consists of the following steps:

1. **Registration** (`POST /api/auth/register`)
   - Create a new user account. This is the first step for any new user.
2. **Email Verification** (`POST /api/auth/verify-email`)
   - Verify the user's email address using a code sent to their email. Required before login is allowed.
3. **Resend Verification** (`POST /api/auth/resend-verification`)
   - If the user did not receive or lost the verification code, this endpoint resends it.
4. **Login** (`POST /api/auth/login`)
   - Authenticate a user and receive a JWT token. Only possible after email verification.
5. **Token Refresh** (`POST /api/auth/refresh`)
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
11. **Change Password** (`POST /api/auth/change-password`)
    - Change the password for a logged-in user (requires current password).
12. **Resend Email Code (Alternative)** (`POST /api/auth/resend-email-code`)
    - Resend the email verification code using the user's email (alternative to the authenticated resend).
13. **Verify Email Code (Alternative)** (`POST /api/auth/verify-email-code`)
    - Alternative way to verify email using both email and code (useful for mobile or custom flows).

### Token Management

- Tokens are stored in HTTP-only cookies for security
- Tokens expire after 24 hours
- Refresh tokens are handled automatically
- All tokens are signed with HS256 algorithm

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
{
  Cookie: string; // Required, contains httpOnly JWT token
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
{
  Cookie: string; // Required, contains httpOnly JWT token
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

### POST /api/auth/refresh

Refresh an expired access token.

#### Request Headers

```typescript
{
  Cookie: string; // Required, contains httpOnly JWT token
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
{
  Cookie: string; // Required, contains httpOnly JWT token
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
{
  Cookie: string; // Required, contains httpOnly JWT token
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

1. Use TypeScript for type safety
2. Implement proper validation
3. Implement email verification
4. Set verification code expiration

## Security Considerations

1. Tokens are stored in HTTP-only cookies
2. Passwords are hashed using bcrypt
3. Input validation is performed
4. Proper error handling is implemented
5. Secure headers are set
6. Proper logging is implemented
7. Email verification is required
8. Verification codes expire after 30 minutes

## Deals Endpoints

#### Create Deal (Merchant/Admin Only)

```http
POST /deals
```

Request Body:

```json
{
  "title": "string (min 5 characters)",
  "description": "string (min 20 characters)",
  "type": "physical" | "digital",
  "originalPrice": "number",
  "discountedPrice": "number",
  "minGroupSize": "number (min 2)",
  "maxGroupSize": "number (min 2) | null",
  "startDate": "ISO datetime string",
  "endDate": "ISO datetime string",
  "images": "string[]",
  "location": "string | null"
}
```

Response (201):

```json
{
  "id": "uuid",
  "title": "string",
  "description": "string",
  "type": "string",
  "originalPrice": "number",
  "discountedPrice": "number",
  "minGroupSize": "number",
  "maxGroupSize": "number",
  "currentGroupSize": "number",
  "status": "pending_approval",
  "startDate": "string",
  "endDate": "string",
  "images": "string[]",
  "location": "string"
}
```

#### List Deals

```http
GET /deals
```

Query Parameters:

- status: DealStatus (optional)
- type: DealType (optional)
- merchantId: string (optional)

Response (200):

```json
[
  {
    "id": "uuid",
    "title": "string",
    "description": "string",
    "type": "string",
    "originalPrice": "number",
    "discountedPrice": "number",
    "minGroupSize": "number",
    "maxGroupSize": "number",
    "currentGroupSize": "number",
    "status": "string",
    "startDate": "string",
    "endDate": "string",
    "images": "string[]",
    "location": "string"
  }
]
```

#### Join Deal (User Only)

```http
POST /deals/{dealId}/join
```

Response (200):

```json
{
  "message": "Successfully joined the deal",
  "participant": {
    "id": "uuid",
    "userId": "uuid",
    "dealId": "uuid",
    "status": "pending",
    "paymentAmount": "number",
    "joinedAt": "string"
  }
}
```

#### Leave Deal (User Only)

```http
DELETE /deals/{dealId}/join
```

Response (200):

```json
{
  "message": "Successfully left the deal"
}
```

### Admin Endpoints

#### Approve/Reject Deal

```http
PATCH /admin/deals/{dealId}
```

Request Body:

```json
{
  "status": "active" | "cancelled",
  "rejectionReason": "string (optional)"
}
```

Response (200):

```json
{
  "message": "Deal approved/rejected",
  "deal": {
    "id": "uuid",
    "status": "string",
    "rejectionReason": "string"
  }
}
```

#### Get Deal Details (Admin Only)

```http
GET /admin/deals/{dealId}
```

Response (200):

```json
{
  "id": "uuid",
  "title": "string",
  "description": "string",
  "type": "string",
  "originalPrice": "number",
  "discountedPrice": "number",
  "minGroupSize": "number",
  "maxGroupSize": "number",
  "currentGroupSize": "number",
  "status": "string",
  "startDate": "string",
  "endDate": "string",
  "images": "string[]",
  "location": "string",
  "merchant": {
    "id": "uuid",
    "email": "string",
    "firstName": "string",
    "lastName": "string"
  }
}
```

## Data Types

### DealStatus

- `draft`
- `pending_approval`
- `active`
- `completed`
- `cancelled`

### DealType

- `physical`
- `digital`

### ParticipantStatus

- `pending`
- `paid`
- `completed`
- `cancelled`
- `refunded`

### UserRole

- `user`
- `merchant`

### User

```typescript
interface User {
  id: string; // Required, UUID v4 format
  email: string; // Required, valid email format, unique
  firstName: string; // Required, min 2 characters
  lastName: string; // Required, min 2 characters
  role: UserRole; // Required, enum: ['user', 'merchant']
  phoneNumber?: string; // Optional, phone number
  isVerified: boolean; // Required, default: false
  lastLogin: Date | null; // Optional, last successful login date
  createdAt: Date; // Required, automatically set
  updatedAt: Date; // Required, automatically updated
}
```

### LoginCredentials

```typescript
interface LoginCredentials {
  email: string; // Required, valid email format
  password: string; // Required, min 8 characters
}
```

### RegisterData

```typescript
interface RegisterData {
  email: string; // Required, valid email format
  password: string; // Required, min 8 characters, must contain at least 1 number and 1 special character
  firstName: string; // Required, min 2 characters, max 50 characters
  lastName: string; // Required, min 2 characters, max 50 characters
}
```

### TokenResponse

```typescript
interface TokenResponse {
  token: string; // Required, JWT token
  expiresIn: number; // Required, token expiration time in seconds
}
```

### ErrorResponse

```typescript
interface ErrorResponse {
  error: string; // Required, error message
  fields?: {
    // Optional, validation errors by field
    [fieldName: string]: string[];
  };
  code?: string; // Optional, error code for client-side handling
}
```

## Email Verification Flow

The email verification process consists of the following steps:

1. User registration
2. Verification email sent automatically
3. User enters verification code
4. Email verification
5. (Optional) Resend verification code

### Registration with Email Verification

When a user registers, they will automatically receive a verification email containing a 6-digit code. The code is valid for 30 minutes.

**Endpoint:** `POST /api/auth/register`

**Request Body:**

```json
{
  "email": "user@example.com",
  "password": "securepassword",
  "firstName": "John",
  "lastName": "Doe",
  "role": "user",
  "phoneNumber": "+1234567890"
}
```

**Success Response:**

```json
{
  "message": "Registration successful. Please check your email for verification code.",
  "user": {
    "id": "user_id",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "user",
    "phoneNumber": "+1234567890",
    "isVerified": false,
    "lastLogin": null,
    "createdAt": "2024-03-21T12:00:00Z",
    "updatedAt": "2024-03-21T12:00:00Z"
  }
}
```

### Verify Email

After receiving the verification code, the user can verify their email address.

**Endpoint:** `POST /api/auth/verify-email`

**Request Body:**

```

```
