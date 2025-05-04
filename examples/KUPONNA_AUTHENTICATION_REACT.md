# React Authentication Implementation Guide

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
- [POST /api/auth/verify-email-code](#post-apiauthverify-email-code)
- [POST /api/auth/resend-email-code](#post-apiauthresend-email-code)

## Overview

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
12. **Verify Email Code (Alternative)** (`POST /api/auth/verify-email-code`)
    - Alternative way to verify email using both email and code (useful for mobile or custom flows).
13. **Resend Email Code (Alternative)** (`POST /api/auth/resend-email-code`)
    - Resend the email verification code using the user's email (alternative to the authenticated resend).

---

# Table of Contents

1. [Setup](#setup)
2. [Authentication Context](#authentication-context)
3. [Protected Routes](#protected-routes)
4. [Login Implementation](#login-implementation)
5. [Registration Implementation](#registration-implementation)
6. [Email Verification](#email-verification)
7. [Auto Login](#auto-login)
8. [Logout Implementation](#logout-implementation)
9. [Error Handling](#error-handling)
10. [Password Reset Implementation](#password-reset-implementation)

## Setup

First, install the required dependencies:

```bash
npm install react-router-dom @types/react-router-dom
```

## Authentication Context

Create a context to manage authentication state:

````typescript
// src/contexts/AuthContext.tsx
import { createContext, useContext, useState, useEffect } from "react";

interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  role: "user" | "merchant";
  phoneNumber?: string;
  isVerified: boolean;
  lastLogin: string | null;
  createdAt: string;
  updatedAt: string;
}

interface AuthContextType {
  user: User | null;
  loading: boolean;
  login: (email: string, password: string) => Promise<void>;
  register: (userData: RegisterData) => Promise<void>;
  logout: () => Promise<void>;
  verifyEmail: (code: string) => Promise<void>;
  resendVerificationEmail: () => Promise<void>;
  forgotPassword: (email: string) => Promise<void>;
  verifyResetToken: (email: string, token: string) => Promise<void>;
  resetPassword: (
    email: string,
    token: string,
    newPassword: string
  ) => Promise<void>;
  changePassword: (
    currentPassword: string,
    newPassword: string
  ) => Promise<void>;
  resendResetCode: (email: string) => Promise<void>;
  verifyEmailCode: (email: string, code: string) => Promise<void>;
  resendEmailCode: (email: string) => Promise<void>;
  isVerified: boolean;
}

interface RegisterData {
  email: string;
  password: string;
  firstName: string;
  lastName: string;
  role: "user" | "merchant";
  phoneNumber?: string;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [isVerified, setIsVerified] = useState<boolean>(false);

  useEffect(() => {
    checkAuth();
  }, []);

  const checkAuth = async () => {
    try {
      const response = await fetch("/api/auth/refresh", {
        method: "POST",
        credentials: "include",
      });

      if (response.ok) {
        const data = await response.json();
        setUser(data.user);
        setIsVerified(data.user.isVerified);
      }
    } catch (error) {
      console.error("Auth check failed:", error);
    } finally {
      setLoading(false);
    }
  };

  const login = async (email: string, password: string) => {
    const response = await fetch("/api/auth/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      credentials: "include",
      body: JSON.stringify({ email, password }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Login failed");
    }

    const data = await response.json();
    setUser(data.user);
    setIsVerified(data.user.isVerified);
  };

  const register = async (userData: RegisterData) => {
    const response = await fetch("/api/auth/register", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      credentials: "include",
      body: JSON.stringify(userData),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Registration failed");
    }

    const data = await response.json();
    setUser(data.user);
    setIsVerified(data.user.isVerified);
  };

  const logout = async () => {
    await fetch("/api/auth/logout", {
      method: "POST",
      credentials: "include",
    });
    setUser(null);
    setIsVerified(false);
  };

  const verifyEmail = async (code: string) => {
    const response = await fetch("/api/auth/verify-email", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      credentials: "include",
      body: JSON.stringify({ code }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Verification failed");
    }

    const data = await response.json();
    setUser(data.user);
    setIsVerified(true);
  };

  const resendVerificationEmail = async () => {
    const response = await fetch("/api/auth/resend-verification", {
      method: "POST",
      credentials: "include",
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Failed to resend verification email");
    }
  };

  const forgotPassword = async (email: string) => {
    const response = await fetch("/api/auth/forgot-password", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Failed to send reset instructions");
    }
  };

  const verifyResetToken = async (email: string, token: string) => {
    const response = await fetch("/api/auth/verify-reset-token", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, token }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Invalid or expired token");
    }
  };

  const resetPassword = async (
    email: string,
    token: string,
    newPassword: string
  ) => {
    const response = await fetch("/api/auth/reset-password", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, token, newPassword }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Failed to reset password");
    }
  };

  const changePassword = async (
    currentPassword: string,
    newPassword: string
  ) => {
    const response = await fetch("/api/auth/change-password", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      credentials: "include",
      body: JSON.stringify({ currentPassword, newPassword }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Failed to change password");
    }
  };

  const resendResetCode = async (email: string) => {
    const response = await fetch("/api/auth/resend-reset-code", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Failed to resend reset code");
    }
  };

  const verifyEmailCode = async (email: string, code: string) => {
    const response = await fetch("/api/auth/verify-email-code", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, code }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Verification failed");
    }

    const data = await response.json();
    setUser(data.user);
    setIsVerified(true);
  };

  const resendEmailCode = async (email: string) => {
    const response = await fetch("/api/auth/resend-email-code", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Failed to resend verification code");
    }
  };

  return (
    <AuthContext.Provider
      value={{
        user,
        loading,
        login,
        register,
        logout,
        verifyEmail,
        resendVerificationEmail,
        forgotPassword,
        verifyResetToken,
        resetPassword,
        changePassword,
        resendResetCode,
        verifyEmailCode,
        resendEmailCode,
        isVerified,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error("useAuth must be used within an AuthProvider");
  }
  return context;
};

## Protected Routes

Create a component to protect routes that require authentication:

```typescript
// src/components/ProtectedRoute.tsx
import { Navigate, useLocation } from "react-router-dom";
import { useAuth } from "../contexts/AuthContext";

export function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { user, loading } = useAuth();
  const location = useLocation();

  if (loading) {
    return <div>Loading...</div>;
  }

  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <>{children}</>;
}
````

## Login Implementation

Create a login page component:

```typescript
// src/pages/LoginPage.tsx
import { useState } from "react";
import { useNavigate, useLocation, Link } from "react-router-dom";
import { useAuth } from "../contexts/AuthContext";

export function LoginPage() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();

  const from = location.state?.from?.pathname || "/dashboard";

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      await login(email, password);
      navigate(from, { replace: true });
    } catch (error) {
      setError(error instanceof Error ? error.message : "Login failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {error && <div>{error}</div>}

      <input
        type="email"
        required
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email address"
      />

      <input
        type="password"
        required
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />

      <button type="submit" disabled={loading}>
        {loading ? "Signing in..." : "Sign in"}
      </button>

      <Link to="/register">Don't have an account? Sign up</Link>
    </form>
  );
}
```

## Registration Implementation

Create a registration page component:

```typescript
// src/pages/RegisterPage.tsx
import { useState } from "react";
import { useNavigate, Link } from "react-router-dom";
import { useAuth } from "../contexts/AuthContext";

export function RegisterPage() {
  const [formData, setFormData] = useState({
    email: "",
    password: "",
    firstName: "",
    lastName: "",
    role: "user" as const,
    phoneNumber: "",
  });
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);
  const { register } = useAuth();
  const navigate = useNavigate();

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      await register(formData);
      navigate("/verify-email");
    } catch (error) {
      setError(error instanceof Error ? error.message : "Registration failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {error && <div>{error}</div>}

      <input
        type="text"
        name="firstName"
        required
        value={formData.firstName}
        onChange={handleChange}
        placeholder="First name"
      />

      <input
        type="text"
        name="lastName"
        required
        value={formData.lastName}
        onChange={handleChange}
        placeholder="Last name"
      />

      <input
        type="email"
        name="email"
        required
        value={formData.email}
        onChange={handleChange}
        placeholder="Email address"
      />

      <input
        type="password"
        name="password"
        required
        value={formData.password}
        onChange={handleChange}
        placeholder="Password"
      />

      <input
        type="tel"
        name="phoneNumber"
        value={formData.phoneNumber}
        onChange={handleChange}
        placeholder="Phone number (optional)"
      />

      <select
        name="role"
        value={formData.role}
        onChange={(e) =>
          setFormData((prev) => ({
            ...prev,
            role: e.target.value as "user" | "merchant",
          }))
        }
      >
        <option value="user">User</option>
        <option value="merchant">Merchant</option>
      </select>

      <button type="submit" disabled={loading}>
        {loading ? "Creating account..." : "Create account"}
      </button>

      <Link to="/login">Already have an account? Sign in</Link>
    </form>
  );
}
```

## Email Verification

After registration, users need to verify their email address. Here's how to implement the email verification flow:

### Verification Context

First, add the verification state and methods to your authentication context:

```typescript
interface AuthContextType {
  // ... existing properties ...
  isVerified: boolean;
  verifyEmail: (code: string) => Promise<void>;
  resendVerification: () => Promise<void>;
}

export const AuthContext = createContext<AuthContextType | undefined>(
  undefined
);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  // ... existing state ...
  const [isVerified, setIsVerified] = useState<boolean>(false);

  const verifyEmail = async (code: string) => {
    try {
      const response = await fetch("/api/auth/verify-email", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify({ code }),
      });

      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.message);
      }

      const data = await response.json();
      setIsVerified(true);
      setUser(data.user);
    } catch (error) {
      throw error;
    }
  };

  const resendVerification = async () => {
    try {
      const response = await fetch("/api/auth/resend-verification", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.message);
      }

      return await response.json();
    } catch (error) {
      throw error;
    }
  };

  // ... existing code ...

  return (
    <AuthContext.Provider
      value={{
        // ... existing values ...
        isVerified,
        verifyEmail,
        resendVerification,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}
```

### Verification Component

Create a verification component to handle the email verification process:

```typescript
import { useState } from "react";
import { useAuth } from "../contexts/AuthContext";

export function EmailVerification() {
  const [code, setCode] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);
  const { verifyEmail, resendVerification } = useAuth();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      await verifyEmail(code);
      // Redirect to dashboard or home page after successful verification
    } catch (error) {
      setError(error.message);
    } finally {
      setLoading(false);
    }
  };

  const handleResend = async () => {
    setError("");
    setLoading(true);

    try {
      await resendVerification();
      // Show success message
    } catch (error) {
      setError(error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="verification-container">
      <h2>Verify Your Email</h2>
      <p>Please enter the verification code sent to your email.</p>

      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={code}
          onChange={(e) => setCode(e.target.value)}
          placeholder="Enter verification code"
          maxLength={6}
          required
        />
        <button type="submit" disabled={loading}>
          {loading ? "Verifying..." : "Verify Email"}
        </button>
      </form>

      <button
        onClick={handleResend}
        disabled={loading}
        className="resend-button"
      >
        Resend Verification Code
      </button>

      {error && <p className="error">{error}</p>}
    </div>
  );
}
```

### Protected Route with Verification Check

Update your protected route component to check for email verification:

```typescript
import { Navigate } from "react-router-dom";
import { useAuth } from "../contexts/AuthContext";

interface ProtectedRouteProps {
  children: React.ReactNode;
  requireVerification?: boolean;
}

export function ProtectedRoute({
  children,
  requireVerification = true,
}: ProtectedRouteProps) {
  const { user, isVerified } = useAuth();

  if (!user) {
    return <Navigate to="/login" />;
  }

  if (requireVerification && !isVerified) {
    return <Navigate to="/verify-email" />;
  }

  return <>{children}</>;
}
```

### Usage Example

```typescript
import { BrowserRouter, Routes, Route } from "react-router-dom";
import { AuthProvider } from "./contexts/AuthContext";
import { ProtectedRoute } from "./components/ProtectedRoute";
import { EmailVerification } from "./components/EmailVerification";

function App() {
  return (
    <BrowserRouter>
      <AuthProvider>
        <Routes>
          <Route path="/login" element={<Login />} />
          <Route path="/register" element={<Register />} />
          <Route path="/verify-email" element={<EmailVerification />} />
          <Route
            path="/dashboard"
            element={
              <ProtectedRoute>
                <Dashboard />
              </ProtectedRoute>
            }
          />
        </Routes>
      </AuthProvider>
    </BrowserRouter>
  );
}
```

### Styling Example

```css
.verification-container {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.verification-container h2 {
  margin-bottom: 1rem;
  text-align: center;
}

.verification-container form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.verification-container input {
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

.verification-container button {
  padding: 0.75rem;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
}

.verification-container button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.resend-button {
  margin-top: 1rem;
  background-color: transparent !important;
  color: #007bff !important;
  text-decoration: underline;
}

.error {
  color: #dc3545;
  margin-top: 1rem;
  text-align: center;
}
```

### Best Practices

1. Always check for email verification status before allowing access to protected routes
2. Provide clear feedback to users about the verification process
3. Implement a resend verification code feature with a cooldown period
4. Store the verification status in the authentication context
5. Handle all possible error cases and display appropriate messages
6. Use loading states to prevent multiple submissions
7. Implement proper form validation for the verification code
8. Consider implementing a countdown timer for the verification code expiration

## Auto Login

The auto-login functionality is handled by the `checkAuth` function in the `AuthContext`. It runs when the app starts and when the `AuthProvider` is mounted. Here's how it works:

1. The app starts and the `AuthProvider` is mounted
2. `checkAuth` is called automatically via `useEffect`
3. It attempts to refresh the token using the `/api/auth/refresh` endpoint
4. If successful, the user is automatically logged in
5. If unsuccessful, the user remains logged out

## Logout Implementation

Create a logout button component:

```typescript
// src/components/LogoutButton.tsx
import { useAuth } from "../contexts/AuthContext";

export function LogoutButton() {
  const { logout } = useAuth();

  const handleLogout = async () => {
    try {
      await logout();
    } catch (error) {
      console.error("Logout failed:", error);
    }
  };

  return <button onClick={handleLogout}>Sign out</button>;
}
```

## Error Handling

The authentication context includes error handling for all operations. Here's how to handle errors in your components:

```typescript
try {
  await login(email, password);
  // Success - navigate to dashboard
} catch (error) {
  // Handle error
  setError(error instanceof Error ? error.message : "An error occurred");
}
```

Common error scenarios:

- Invalid credentials
- Network errors
- Server errors
- Token expiration
- Invalid input

## App Setup

Set up your app with the authentication provider and routes:

```typescript
// src/App.tsx
import { BrowserRouter as Router, Routes, Route } from "react-router-dom";
import { AuthProvider } from "./contexts/AuthContext";
import { ProtectedRoute } from "./components/ProtectedRoute";
import { LoginPage } from "./pages/LoginPage";
import { RegisterPage } from "./pages/RegisterPage";
import { Dashboard } from "./pages/Dashboard";

function App() {
  return (
    <Router>
      <AuthProvider>
        <Routes>
          <Route path="/login" element={<LoginPage />} />
          <Route path="/register" element={<RegisterPage />} />
          <Route
            path="/dashboard"
            element={
              <ProtectedRoute>
                <Dashboard />
              </ProtectedRoute>
            }
          />
          <Route path="/" element={<Navigate to="/dashboard" replace />} />
        </Routes>
      </AuthProvider>
    </Router>
  );
}

export default App;
```

This implementation provides a complete authentication flow with:

- User registration
- Login/logout functionality
- Protected routes
- Auto-login with token refresh
- Error handling
- Type safety with TypeScript
- Modern UI with Tailwind CSS
- Responsive design
- Loading states
- Form validation

## API Response Types

### Login Response

```typescript
interface LoginResponse {
  message: string; // Required, e.g., "Login successful"
  user: {
    id: string; // Required, UUID format
    email: string; // Required, valid email format
    firstName: string; // Required, min 2 characters
    lastName: string; // Required, min 2 characters
    role: string; // Required, enum: 'user', 'merchant', 'admin'
  };
}
```

### Register Response

```typescript
interface RegisterResponse {
  message: string; // Required, e.g., "User registered successfully"
  user: {
    id: string; // Required, UUID format
    email: string; // Required, valid email format
    firstName: string; // Required, min 2 characters
    lastName: string; // Required, min 2 characters
    role: string; // Required, enum: 'user', 'merchant', 'admin'
    createdAt: string; // Required, ISO datetime string
    updatedAt: string; // Required, ISO datetime string
  };
}
```

### Error Response

```typescript
interface ErrorResponse {
  error: string; // Required, error message
}
```

This implementation provides a complete authentication flow with:

- User registration
- Login/logout functionality
- Protected routes
- Auto-login with token refresh
- Error handling
- Type safety with TypeScript

## Password Reset Implementation

### Reset Context

Add password reset functionality to your authentication context:

```typescript
interface AuthContextType {
  // ... existing properties ...
  forgotPassword: (email: string) => Promise<void>;
  verifyResetToken: (email: string, token: string) => Promise<string>;
  resetPassword: (
    newPassword: string,
    confirmPassword: string
  ) => Promise<void>;
  resendResetCode: (email: string) => Promise<void>;
  changePassword: (
    currentPassword: string,
    newPassword: string
  ) => Promise<void>;
}

export const AuthContext = createContext<AuthContextType | undefined>(
  undefined
);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  // ... existing state ...

  const forgotPassword = async (email: string) => {
    try {
      const response = await fetch("/api/auth/forgot-password", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ email }),
      });

      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.message);
      }

      return await response.json();
    } catch (error) {
      throw error;
    }
  };

  const verifyResetToken = async (email: string, token: string) => {
    try {
      const response = await fetch("/api/auth/verify-reset-token", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ email, token }),
      });

      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.message);
      }

      const data = await response.json();
      return data.resetToken;
    } catch (error) {
      throw error;
    }
  };

  const resetPassword = async (
    newPassword: string,
    confirmPassword: string
  ) => {
    try {
      const response = await fetch("/api/auth/reset-password", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${resetToken}`,
        },
        body: JSON.stringify({ newPassword, confirmPassword }),
      });

      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.message);
      }

      return await response.json();
    } catch (error) {
      throw error;
    }
  };

  const resendResetCode = async (email: string) => {
    try {
      const response = await fetch("/api/auth/resend-reset-code", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ email }),
      });

      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.message);
      }

      return await response.json();
    } catch (error) {
      throw error;
    }
  };

  const changePassword = async (
    currentPassword: string,
    newPassword: string
  ) => {
    try {
      const response = await fetch("/api/auth/change-password", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify({ currentPassword, newPassword }),
      });

      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.message);
      }

      return await response.json();
    } catch (error) {
      throw error;
    }
  };

  return (
    <AuthContext.Provider
      value={{
        // ... existing values ...
        forgotPassword,
        verifyResetToken,
        resetPassword,
        resendResetCode,
        changePassword,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}
```

### Password Reset Components

Create components for the password reset flow:

```typescript
// ForgotPassword.tsx
export function ForgotPassword() {
  const [email, setEmail] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);
  const [success, setSuccess] = useState(false);
  const { forgotPassword } = useAuth();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      await forgotPassword(email);
      setSuccess(true);
    } catch (error) {
      setError(error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="forgot-password-container">
      <h2>Reset Password</h2>
      {success ? (
        <div className="success-message">
          <p>Password reset instructions have been sent to your email.</p>
          <Link to="/verify-reset-token">Enter Reset Code</Link>
        </div>
      ) : (
        <form onSubmit={handleSubmit}>
          <input
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            placeholder="Enter your email"
            required
          />
          <button type="submit" disabled={loading}>
            {loading ? "Sending..." : "Send Reset Instructions"}
          </button>
        </form>
      )}
      {error && <p className="error">{error}</p>}
    </div>
  );
}

// VerifyResetToken.tsx
export function VerifyResetToken() {
  const [email, setEmail] = useState("");
  const [token, setToken] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);
  const { verifyResetToken } = useAuth();
  const navigate = useNavigate();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      const resetToken = await verifyResetToken(email, token);
      navigate("/reset-password", { state: { resetToken } });
    } catch (error) {
      setError(error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="verify-token-container">
      <h2>Enter Reset Code</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="Enter your email"
          required
        />
        <input
          type="text"
          value={token}
          onChange={(e) => setToken(e.target.value)}
          placeholder="Enter reset code"
          maxLength={6}
          required
        />
        <button type="submit" disabled={loading}>
          {loading ? "Verifying..." : "Verify Code"}
        </button>
      </form>
      {error && <p className="error">{error}</p>}
    </div>
  );
}

// ResetPassword.tsx
export function ResetPassword() {
  const [newPassword, setNewPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);
  const { resetPassword } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");

    if (newPassword !== confirmPassword) {
      setError("Passwords do not match");
      return;
    }

    setLoading(true);

    try {
      await resetPassword(newPassword, confirmPassword);
      navigate("/login", { state: { message: "Password reset successfully" } });
    } catch (error) {
      setError(error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="reset-password-container">
      <h2>Set New Password</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="password"
          value={newPassword}
          onChange={(e) => setNewPassword(e.target.value)}
          placeholder="New password"
          required
          minLength={8}
        />
        <input
          type="password"
          value={confirmPassword}
          onChange={(e) => setConfirmPassword(e.target.value)}
          placeholder="Confirm new password"
          required
          minLength={8}
        />
        <button type="submit" disabled={loading}>
          {loading ? "Resetting..." : "Reset Password"}
        </button>
      </form>
      {error && <p className="error">{error}</p>}
    </div>
  );
}

// ChangePassword.tsx
export function ChangePassword() {
  const [currentPassword, setCurrentPassword] = useState("");
  const [newPassword, setNewPassword] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);
  const { changePassword } = useAuth();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");

    if (newPassword !== confirmPassword) {
      setError("Passwords do not match");
      return;
    }

    setLoading(true);

    try {
      await changePassword(currentPassword, newPassword);
      // Show success message and clear form
      setCurrentPassword("");
      setNewPassword("");
    } catch (error) {
      setError(error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="change-password-container">
      <h2>Change Password</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="password"
          value={currentPassword}
          onChange={(e) => setCurrentPassword(e.target.value)}
          placeholder="Current password"
          required
        />
        <input
          type="password"
          value={newPassword}
          onChange={(e) => setNewPassword(e.target.value)}
          placeholder="New password"
          required
          minLength={8}
        />
        <button type="submit" disabled={loading}>
          {loading ? "Changing..." : "Change Password"}
        </button>
      </form>
      {error && <p className="error">{error}</p>}
    </div>
  );
}
```

### Styling Example

```css
.forgot-password-container,
.verify-token-container,
.reset-password-container,
.change-password-container {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.success-message {
  text-align: center;
  padding: 1rem;
  background-color: #d4edda;
  border: 1px solid #c3e6cb;
  border-radius: 4px;
  margin-bottom: 1rem;
}

.success-message a {
  color: #155724;
  text-decoration: none;
  font-weight: bold;
}

.success-message a:hover {
  text-decoration: underline;
}

form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

input {
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

button {
  padding: 0.75rem;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.error {
  color: #dc3545;
  margin-top: 1rem;
  text-align: center;
}
```

### Best Practices

1. Always validate passwords on both client and server side
2. Implement proper error handling for all password-related operations
3. Use secure password requirements (minimum length, special characters, etc.)
4. Provide clear feedback to users about password requirements
5. Implement rate limiting for password reset attempts
6. Use secure token generation and validation
7. Implement proper session management after password changes
8. Consider implementing password strength indicators
9. Use proper HTTP security headers
10. Implement proper logging for security events

[Back to top](#authentication-api-endpoints)
