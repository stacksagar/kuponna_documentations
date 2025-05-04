# React Native Authentication Implementation Guide

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
3. [Navigation Setup](#navigation-setup)
4. [Login Implementation](#login-implementation)
5. [Registration Implementation](#registration-implementation)
6. [Email Verification](#email-verification)
7. [Auto Login](#auto-login)
8. [Logout Implementation](#logout-implementation)
9. [Error Handling](#error-handling)
10. [API Response Types](#api-response-types)
11. [Password Reset Implementation](#password-reset-implementation)

## Setup

First, install the required dependencies:

```bash
npm install @react-navigation/native @react-navigation/native-stack @react-native-async-storage/async-storage react-native-safe-area-context react-native-screens
```

## Authentication Context

Create a context to manage authentication state:

````typescript
// src/contexts/AuthContext.tsx
import { createContext, useContext, useState, useEffect } from "react";
import AsyncStorage from "@react-native-async-storage/async-storage";

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

  useEffect(() => {
    checkAuth();
  }, []);

  const checkAuth = async () => {
    try {
      const token = await AsyncStorage.getItem("token");
      if (token) {
        const response = await fetch("YOUR_API_URL/api/auth/refresh", {
          method: "POST",
          headers: {
            Authorization: `Bearer ${token}`,
          },
        });

        if (response.ok) {
          const data = await response.json();
          setUser(data.user);
          await AsyncStorage.setItem("token", data.token);
        } else {
          await AsyncStorage.removeItem("token");
        }
      }
    } catch (error) {
      console.error("Auth check failed:", error);
    } finally {
      setLoading(false);
    }
  };

  const login = async (email: string, password: string) => {
    const response = await fetch("YOUR_API_URL/api/auth/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, password }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Login failed");
    }

    const data = await response.json();
    await AsyncStorage.setItem("token", data.token);
    setUser(data.user);
  };

  const register = async (userData: RegisterData) => {
    const response = await fetch("YOUR_API_URL/api/auth/register", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(userData),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Registration failed");
    }

    const data = await response.json();
    await AsyncStorage.setItem("token", data.token);
    setUser(data.user);
  };

  const logout = async () => {
    await AsyncStorage.removeItem("token");
    setUser(null);
  };

  const verifyEmail = async (code: string) => {
    const token = await AsyncStorage.getItem("token");
    const response = await fetch("YOUR_API_URL/api/auth/verify-email", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${token}`,
      },
      body: JSON.stringify({ code }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Verification failed");
    }

    const data = await response.json();
    setUser(data.user);
  };

  const resendVerificationEmail = async () => {
    const token = await AsyncStorage.getItem("token");
    const response = await fetch("YOUR_API_URL/api/auth/resend-verification", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Failed to resend verification email");
    }
  };

  const forgotPassword = async (email: string) => {
    const response = await fetch("YOUR_API_URL/api/auth/forgot-password", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Failed to initiate password reset");
    }
  };

  const verifyResetToken = async (email: string, token: string) => {
    const response = await fetch("YOUR_API_URL/api/auth/verify-reset-token", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, token }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Invalid or expired reset token");
    }
  };

  const resetPassword = async (
    email: string,
    token: string,
    newPassword: string
  ) => {
    const response = await fetch("YOUR_API_URL/api/auth/reset-password", {
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
    const token = await AsyncStorage.getItem("token");
    const response = await fetch("YOUR_API_URL/api/auth/change-password", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${token}`,
      },
      body: JSON.stringify({ currentPassword, newPassword }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Failed to change password");
    }
  };

  const resendResetCode = async (email: string) => {
    const response = await fetch("YOUR_API_URL/api/auth/resend-reset-code", {
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
    const response = await fetch("YOUR_API_URL/api/auth/verify-email-code", {
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
  };

  const resendEmailCode = async (email: string) => {
    const response = await fetch("YOUR_API_URL/api/auth/resend-email-code", {
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

## Navigation Setup

Set up the navigation structure with authentication flow:

```typescript
// src/navigation/index.tsx
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import { useAuth } from "../contexts/AuthContext";
import { LoginScreen } from "../screens/LoginScreen";
import { RegisterScreen } from "../screens/RegisterScreen";
import { HomeScreen } from "../screens/HomeScreen";
import { ProfileScreen } from "../screens/ProfileScreen";
import { LoadingScreen } from "../screens/LoadingScreen";
import { VerifyEmailScreen } from "../screens/VerifyEmailScreen";

const Stack = createNativeStackNavigator();

export function Navigation() {
  const { user, loading } = useAuth();

  if (loading) {
    return <LoadingScreen />;
  }

  return (
    <NavigationContainer>
      <Stack.Navigator>
        {user ? (
          // Authenticated stack
          <>
            {!user.isVerified ? (
              <Stack.Screen
                name="VerifyEmail"
                component={VerifyEmailScreen}
                options={{ headerShown: false }}
              />
            ) : (
              <>
                <Stack.Screen
                  name="Home"
                  component={HomeScreen}
                  options={{ headerShown: false }}
                />
                <Stack.Screen
                  name="Profile"
                  component={ProfileScreen}
                  options={{ title: "Profile" }}
                />
              </>
            )}
          </>
        ) : (
          // Auth stack
          <>
            <Stack.Screen
              name="Login"
              component={LoginScreen}
              options={{ headerShown: false }}
            />
            <Stack.Screen
              name="Register"
              component={RegisterScreen}
              options={{ headerShown: false }}
            />
          </>
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
}
````

## Login Implementation

Create a login screen:

```typescript
// src/screens/LoginScreen.tsx
import { useState } from "react";
import {
  View,
  TextInput,
  TouchableOpacity,
  Text,
  Alert,
  ActivityIndicator,
} from "react-native";
import { useAuth } from "../contexts/AuthContext";
import type { NativeStackNavigationProp } from "@react-navigation/native-stack";

type LoginScreenProps = {
  navigation: NativeStackNavigationProp<any>;
};

export function LoginScreen({ navigation }: LoginScreenProps) {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [loading, setLoading] = useState(false);
  const { login } = useAuth();

  const handleLogin = async () => {
    if (!email || !password) {
      Alert.alert("Error", "Please fill in all fields");
      return;
    }

    setLoading(true);
    try {
      await login(email, password);
    } catch (error) {
      Alert.alert(
        "Error",
        error instanceof Error ? error.message : "Login failed"
      );
    } finally {
      setLoading(false);
    }
  };

  return (
    <View>
      <Text>Welcome Back</Text>

      <TextInput
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        autoCapitalize="none"
        keyboardType="email-address"
        editable={!loading}
      />

      <TextInput
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
        editable={!loading}
      />

      <TouchableOpacity onPress={handleLogin} disabled={loading}>
        {loading ? <ActivityIndicator /> : <Text>Sign In</Text>}
      </TouchableOpacity>

      <TouchableOpacity onPress={() => navigation.navigate("Register")}>
        <Text>Don't have an account? Sign up</Text>
      </TouchableOpacity>
    </View>
  );
}
```

## Registration Implementation

Create a registration screen:

```typescript
// src/screens/RegisterScreen.tsx
import { useState } from "react";
import {
  View,
  TextInput,
  TouchableOpacity,
  Text,
  Alert,
  ActivityIndicator,
} from "react-native";
import { useAuth } from "../contexts/AuthContext";
import type { NativeStackNavigationProp } from "@react-navigation/native-stack";

type RegisterScreenProps = {
  navigation: NativeStackNavigationProp<any>;
};

export function RegisterScreen({ navigation }: RegisterScreenProps) {
  const [formData, setFormData] = useState({
    email: "",
    password: "",
    firstName: "",
    lastName: "",
    role: "user" as const,
    phoneNumber: "",
  });
  const [loading, setLoading] = useState(false);
  const { register } = useAuth();

  const handleChange = (field: string, value: string) => {
    setFormData((prev) => ({ ...prev, [field]: value }));
  };

  const handleRegister = async () => {
    if (
      !formData.email ||
      !formData.password ||
      !formData.firstName ||
      !formData.lastName
    ) {
      Alert.alert("Error", "Please fill in all required fields");
      return;
    }

    setLoading(true);
    try {
      await register(formData);
      navigation.replace("VerifyEmail");
    } catch (error) {
      Alert.alert(
        "Registration Failed",
        error instanceof Error ? error.message : "Please try again"
      );
    } finally {
      setLoading(false);
    }
  };

  return (
    <View>
      <Text>Create Account</Text>

      <TextInput
        placeholder="First Name"
        value={formData.firstName}
        onChangeText={(value) => handleChange("firstName", value)}
        editable={!loading}
      />

      <TextInput
        placeholder="Last Name"
        value={formData.lastName}
        onChangeText={(value) => handleChange("lastName", value)}
        editable={!loading}
      />

      <TextInput
        placeholder="Email"
        value={formData.email}
        onChangeText={(value) => handleChange("email", value)}
        autoCapitalize="none"
        keyboardType="email-address"
        editable={!loading}
      />

      <TextInput
        placeholder="Password"
        value={formData.password}
        onChangeText={(value) => handleChange("password", value)}
        secureTextEntry
        editable={!loading}
      />

      <TextInput
        placeholder="Phone Number (optional)"
        value={formData.phoneNumber}
        onChangeText={(value) => handleChange("phoneNumber", value)}
        keyboardType="phone-pad"
        editable={!loading}
      />

      <TouchableOpacity onPress={handleRegister} disabled={loading}>
        {loading ? <ActivityIndicator /> : <Text>Create Account</Text>}
      </TouchableOpacity>

      <TouchableOpacity onPress={() => navigation.navigate("Login")}>
        <Text>Already have an account? Sign in</Text>
      </TouchableOpacity>
    </View>
  );
}
```

## Email Verification

After registration, users need to verify their email address. Here's how to implement the email verification flow in React Native:

### 1. Update AuthContext

Add verification-related methods to the AuthContext:

```typescript
interface AuthContextType {
  // ... existing properties ...
  verifyEmail: (code: string) => Promise<void>;
  resendVerificationEmail: () => Promise<void>;
}

export function AuthProvider({ children }: { children: React.ReactNode }) {
  // ... existing code ...

  const verifyEmail = async (code: string) => {
    const token = await AsyncStorage.getItem("token");
    const response = await fetch("YOUR_API_URL/api/auth/verify-email", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${token}`,
      },
      body: JSON.stringify({ code }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Verification failed");
    }

    const data = await response.json();
    setUser(data.user);
  };

  const resendVerificationEmail = async () => {
    const token = await AsyncStorage.getItem("token");
    const response = await fetch("YOUR_API_URL/api/auth/resend-verification", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || "Failed to resend verification email");
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
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}
```

### 2. Create Verification Screen

Create a screen for email verification:

```typescript
// src/screens/VerifyEmailScreen.tsx
import { useState } from "react";
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  Alert,
  ActivityIndicator,
  StyleSheet,
} from "react-native";
import { useAuth } from "../contexts/AuthContext";
import type { NativeStackNavigationProp } from "@react-navigation/native-stack";

type VerifyEmailScreenProps = {
  navigation: NativeStackNavigationProp<any>;
};

export function VerifyEmailScreen({ navigation }: VerifyEmailScreenProps) {
  const [code, setCode] = useState("");
  const [loading, setLoading] = useState(false);
  const [resending, setResending] = useState(false);
  const { verifyEmail, resendVerificationEmail, user } = useAuth();

  const handleVerify = async () => {
    if (!code) {
      Alert.alert("Error", "Please enter the verification code");
      return;
    }

    setLoading(true);
    try {
      await verifyEmail(code);
      navigation.replace("Home");
    } catch (error) {
      Alert.alert(
        "Verification Failed",
        error instanceof Error ? error.message : "Please try again"
      );
    } finally {
      setLoading(false);
    }
  };

  const handleResend = async () => {
    setResending(true);
    try {
      await resendVerificationEmail();
      Alert.alert(
        "Success",
        "Verification email sent. Please check your inbox."
      );
    } catch (error) {
      Alert.alert(
        "Error",
        error instanceof Error ? error.message : "Failed to resend email"
      );
    } finally {
      setResending(false);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Verify Your Email</Text>
      <Text style={styles.subtitle}>
        Please enter the verification code sent to {user?.email}
      </Text>

      <TextInput
        style={styles.input}
        placeholder="Enter verification code"
        value={code}
        onChangeText={setCode}
        keyboardType="number-pad"
        maxLength={6}
      />

      <TouchableOpacity
        style={styles.button}
        onPress={handleVerify}
        disabled={loading}
      >
        {loading ? (
          <ActivityIndicator color="#fff" />
        ) : (
          <Text style={styles.buttonText}>Verify Email</Text>
        )}
      </TouchableOpacity>

      <TouchableOpacity
        style={styles.resendButton}
        onPress={handleResend}
        disabled={resending}
      >
        {resending ? (
          <ActivityIndicator color="#007AFF" />
        ) : (
          <Text style={styles.resendText}>Resend Verification Email</Text>
        )}
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: "center",
  },
  title: {
    fontSize: 24,
    fontWeight: "bold",
    marginBottom: 10,
    textAlign: "center",
  },
  subtitle: {
    fontSize: 16,
    color: "#666",
    marginBottom: 20,
    textAlign: "center",
  },
  input: {
    borderWidth: 1,
    borderColor: "#ddd",
    padding: 15,
    borderRadius: 8,
    marginBottom: 20,
    fontSize: 16,
  },
  button: {
    backgroundColor: "#007AFF",
    padding: 15,
    borderRadius: 8,
    alignItems: "center",
  },
  buttonText: {
    color: "#fff",
    fontSize: 16,
    fontWeight: "bold",
  },
  resendButton: {
    marginTop: 15,
    padding: 15,
    alignItems: "center",
  },
  resendText: {
    color: "#007AFF",
    fontSize: 16,
  },
});
```

### 3. Update Navigation

Modify the navigation setup to include the verification screen:

```typescript
// src/navigation/index.tsx
export function Navigation() {
  const { user, loading } = useAuth();

  if (loading) {
    return <LoadingScreen />;
  }

  return (
    <NavigationContainer>
      <Stack.Navigator>
        {user ? (
          // Authenticated stack
          <>
            {!user.isVerified ? (
              <Stack.Screen
                name="VerifyEmail"
                component={VerifyEmailScreen}
                options={{ headerShown: false }}
              />
            ) : (
              <>
                <Stack.Screen
                  name="Home"
                  component={HomeScreen}
                  options={{ headerShown: false }}
                />
                <Stack.Screen
                  name="Profile"
                  component={ProfileScreen}
                  options={{ title: "Profile" }}
                />
              </>
            )}
          </>
        ) : (
          // Auth stack
          <>
            <Stack.Screen
              name="Login"
              component={LoginScreen}
              options={{ headerShown: false }}
            />
            <Stack.Screen
              name="Register"
              component={RegisterScreen}
              options={{ headerShown: false }}
            />
          </>
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

### 4. Update Registration Flow

Modify the registration screen to navigate to verification:

```typescript
// src/screens/RegisterScreen.tsx
export function RegisterScreen({ navigation }: RegisterScreenProps) {
  // ... existing code ...

  const handleRegister = async () => {
    if (!validateForm()) {
      return;
    }

    setLoading(true);
    try {
      await register(formData);
      navigation.replace("VerifyEmail");
    } catch (error) {
      Alert.alert(
        "Registration Failed",
        error instanceof Error ? error.message : "Please try again"
      );
    } finally {
      setLoading(false);
    }
  };

  // ... rest of the component
}
```

### 5. API Endpoints

The following API endpoints are available for email verification:

1. `POST /api/auth/verify-email`

   - Request body: `{ code: string }`
   - Headers: `Authorization: Bearer <token>`
   - Verifies the email using the provided code
   - Returns updated user object

2. `POST /api/auth/resend-verification`
   - Headers: `Authorization: Bearer <token>`
   - Resends the verification email
   - Returns success message

### 6. Usage Example

```typescript
// In your app navigation
import { VerifyEmailScreen } from "./screens/VerifyEmailScreen";

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen
          name="VerifyEmail"
          component={VerifyEmailScreen}
          options={{ headerShown: false }}
        />
        {/* ... other screens ... */}
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

## Auto Login

The auto-login functionality is handled by the `checkAuth` function in the `AuthContext`. It runs when the app starts and when the `AuthProvider` is mounted. Here's how it works:

1. The app starts and the `AuthProvider` is mounted
2. `checkAuth` is called automatically via `useEffect`
3. It checks for a stored token in AsyncStorage
4. If a token exists, it attempts to refresh it using the `/api/auth/refresh` endpoint
5. If successful, the user is automatically logged in
6. If unsuccessful, the token is removed and the user remains logged out

## Logout Implementation

Add a logout button to your profile screen:

```typescript
// src/screens/ProfileScreen.tsx
import { View, Text, TouchableOpacity, Alert } from "react-native";
import { useAuth } from "../contexts/AuthContext";

export function ProfileScreen() {
  const { user, logout } = useAuth();

  const handleLogout = async () => {
    try {
      await logout();
    } catch (error) {
      Alert.alert("Error", "Failed to logout");
    }
  };

  return (
    <View>
      <Text>Profile</Text>
      <Text>Welcome, {user?.firstName}!</Text>

      <TouchableOpacity onPress={handleLogout}>
        <Text>Sign Out</Text>
      </TouchableOpacity>
    </View>
  );
}
```

## Error Handling

The authentication context includes error handling for all operations. Here's how to handle errors in your components:

```typescript
try {
  await login(email, password);
  // Success - navigation is handled by the auth context
} catch (error) {
  // Handle error
  Alert.alert(
    "Error",
    error instanceof Error ? error.message : "An error occurred"
  );
}
```

Common error scenarios:

- Invalid credentials
- Network errors
- Server errors
- Token expiration
- Invalid input

## App Setup

Finally, set up your app with the authentication provider:

```typescript
// App.tsx
import { AuthProvider } from "./src/contexts/AuthContext";
import { Navigation } from "./src/navigation";

export default function App() {
  return (
    <AuthProvider>
      <Navigation />
    </AuthProvider>
  );
}
```

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
- Protected navigation
- Auto-login with token refresh
- Error handling
- Type safety with TypeScript
- Modern UI with React Native components
- Loading states
- Form validation
- Secure token storage with AsyncStorage

## Password Reset Implementation

Create screens for the password reset flow:

```typescript
// src/screens/ForgotPasswordScreen.tsx
import React, { useState } from "react";
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  ActivityIndicator,
} from "react-native";
import { useAuth } from "../contexts/AuthContext";

export function ForgotPasswordScreen({ navigation }) {
  const [email, setEmail] = useState("");
  const [message, setMessage] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);
  const { forgotPassword } = useAuth();

  const handleSubmit = async () => {
    setError("");
    setMessage("");
    setLoading(true);

    try {
      await forgotPassword(email);
      setMessage("Password reset instructions sent to your email");
      navigation.navigate("ResetPassword", { email });
    } catch (err) {
      setError(
        err instanceof Error ? err.message : "Failed to send reset instructions"
      );
    } finally {
      setLoading(false);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Forgot Password</Text>
      {error ? <Text style={styles.error}>{error}</Text> : null}
      {message ? <Text style={styles.success}>{message}</Text> : null}
      <TextInput
        style={styles.input}
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
      />
      <TouchableOpacity
        style={styles.button}
        onPress={handleSubmit}
        disabled={loading}
      >
        {loading ? (
          <ActivityIndicator color="#fff" />
        ) : (
          <Text style={styles.buttonText}>Send Reset Instructions</Text>
        )}
      </TouchableOpacity>
    </View>
  );
}

// src/screens/ResetPasswordScreen.tsx
import React, { useState } from "react";
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  ActivityIndicator,
} from "react-native";
import { useAuth } from "../contexts/AuthContext";

export function ResetPasswordScreen({ route, navigation }) {
  const { email } = route.params;
  const [token, setToken] = useState("");
  const [newPassword, setNewPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);
  const [step, setStep] = useState<"verify" | "reset">("verify");
  const { verifyResetToken, resetPassword } = useAuth();

  const handleVerifyToken = async () => {
    setError("");
    setLoading(true);

    try {
      await verifyResetToken(email, token);
      setStep("reset");
    } catch (err) {
      setError(err instanceof Error ? err.message : "Invalid or expired token");
    } finally {
      setLoading(false);
    }
  };

  const handleResetPassword = async () => {
    setError("");

    if (newPassword !== confirmPassword) {
      setError("Passwords do not match");
      return;
    }

    if (newPassword.length < 8) {
      setError("Password must be at least 8 characters long");
      return;
    }

    setLoading(true);

    try {
      await resetPassword(email, token, newPassword);
      navigation.replace("Login", { message: "Password reset successfully" });
    } catch (err) {
      setError(err instanceof Error ? err.message : "Failed to reset password");
    } finally {
      setLoading(false);
    }
  };

  if (step === "verify") {
    return (
      <View style={styles.container}>
        <Text style={styles.title}>Enter Reset Code</Text>
        {error ? <Text style={styles.error}>{error}</Text> : null}
        <TextInput
          style={styles.input}
          placeholder="Reset Code"
          value={token}
          onChangeText={setToken}
          autoCapitalize="none"
        />
        <TouchableOpacity
          style={styles.button}
          onPress={handleVerifyToken}
          disabled={loading}
        >
          {loading ? (
            <ActivityIndicator color="#fff" />
          ) : (
            <Text style={styles.buttonText}>Verify Code</Text>
          )}
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Reset Password</Text>
      {error ? <Text style={styles.error}>{error}</Text> : null}
      <TextInput
        style={styles.input}
        placeholder="New Password"
        value={newPassword}
        onChangeText={setNewPassword}
        secureTextEntry
      />
      <TextInput
        style={styles.input}
        placeholder="Confirm Password"
        value={confirmPassword}
        onChangeText={setConfirmPassword}
        secureTextEntry
      />
      <TouchableOpacity
        style={styles.button}
        onPress={handleResetPassword}
        disabled={loading}
      >
        {loading ? (
          <ActivityIndicator color="#fff" />
        ) : (
          <Text style={styles.buttonText}>Reset Password</Text>
        )}
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: "#fff",
  },
  title: {
    fontSize: 24,
    fontWeight: "bold",
    marginBottom: 20,
    textAlign: "center",
  },
  input: {
    borderWidth: 1,
    borderColor: "#ddd",
    padding: 15,
    borderRadius: 5,
    marginBottom: 15,
  },
  button: {
    backgroundColor: "#007AFF",
    padding: 15,
    borderRadius: 5,
    alignItems: "center",
  },
  buttonText: {
    color: "#fff",
    fontSize: 16,
    fontWeight: "bold",
  },
  error: {
    color: "#ff3b30",
    marginBottom: 10,
    textAlign: "center",
  },
  success: {
    color: "#34c759",
    marginBottom: 10,
    textAlign: "center",
  },
});
```

Update your navigation configuration to include the new screens:

```typitten
// src/navigation/AuthStack.tsx
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import { ForgotPasswordScreen } from "../screens/ForgotPasswordScreen";
import { ResetPasswordScreen } from "../screens/ResetPasswordScreen";

const Stack = createNativeStackNavigator();

export function AuthStack() {
  return (
    <Stack.Navigator>
      {/* ... existing screens ... */}
      <Stack.Screen
        name="ForgotPassword"
        component={ForgotPasswordScreen}
        options={{ title: "Forgot Password" }}
      />
      <Stack.Screen
        name="ResetPassword"
        component={ResetPasswordScreen}
        options={{ title: "Reset Password" }}
      />
    </Stack.Navigator>
  );
}
```

Add a link to the forgot password screen in your login screen:

```typescript
// src/screens/LoginScreen.tsx
// ... existing imports ...

export function LoginScreen({ navigation }) {
  // ... existing code ...

  return (
    <View style={styles.container}>
      {/* ... existing form fields ... */}
      <TouchableOpacity
        onPress={() => navigation.navigate("ForgotPassword")}
        style={styles.forgotPassword}
      >
        <Text style={styles.forgotPasswordText}>Forgot Password?</Text>
      </TouchableOpacity>
      {/* ... existing buttons ... */}
    </View>
  );
}

const styles = StyleSheet.create({
  // ... existing styles ...
  forgotPassword: {
    alignSelf: "center",
    marginVertical: 10,
  },
  forgotPasswordText: {
    color: "#007AFF",
    fontSize: 16,
  },
});
```

[Back to top](#authentication-api-endpoints)
