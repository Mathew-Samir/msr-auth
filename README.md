# msr-auth

A lightweight, **backend-agnostic** Angular authentication library that provides a ready-to-use service for common auth flows — login, registration, password recovery, and email verification.

Built with **Angular 21**, published to npm via `ng-packagr`, and designed to drop into any Angular 19–21 project with zero UI opinions.

[![npm version](https://img.shields.io/npm/v/msr-auth.svg)](https://www.npmjs.com/package/msr-auth)

---

## Table of Contents

- [Features](#features)
- [Compatibility](#compatibility)
- [Installation](#installation)
- [Setup (Step by Step)](#setup-step-by-step)
  - [Step 1 — Install the package](#step-1--install-the-package)
  - [Step 2 — Provide HttpClient](#step-2--provide-httpclient)
  - [Step 3 — Configure the library](#step-3--configure-the-library)
  - [Step 4 — Inject & use the service](#step-4--inject--use-the-service)
- [Configuration Reference](#configuration-reference)
  - [AuthConfig](#authconfig)
  - [AuthEndpoints](#authendpoints)
  - [Default Endpoints](#default-endpoints)
- [API Reference](#api-reference)
  - [MsrAuth Service](#msrauth-service)
  - [Method Signatures](#method-signatures)
- [Interfaces](#interfaces)
  - [AuthResponse\<TResponse\>](#authresponsetresponse)
  - [ErrorResponse](#errorresponse)
  - [AuthData](#authdata)
  - [User](#user)
  - [Request Interfaces](#request-interfaces)
- [Advanced Usage](#advanced-usage)
  - [Custom Request & Response Types](#custom-request--response-types)
  - [Error Handling](#error-handling)
- [Project Structure](#project-structure)
- [CI / CD](#ci--cd)
- [Contributing](#contributing)
- [License](#license)

---

## Features

- 🔐 **Full auth flow** — Login, Register, Forgot Password, Reset Password, Email Verification (send + confirm)
- 🧩 **Zero UI** — Pure service layer; bring your own components
- ⚙️ **Configurable endpoints** — Override any or all default API paths
- 🏷️ **Generic types** — Every method accepts custom `TRequest` / `TResponse` generics for full type safety
- 📦 **Tree-shakable** — `providedIn: 'root'` + `sideEffects: false`
- 🔌 **Backend-agnostic** — Works with any REST API that follows a standard JSON response format

---

## Compatibility

| Angular Version | Supported |
| --------------- | --------- |
| 19.x            | ✅        |
| 20.x            | ✅        |
| 21.x            | ✅        |

**Peer dependencies:** `@angular/common` and `@angular/core` (>= 19.0.0 < 22.0.0)

---

## Installation

```bash
npm install msr-auth
```

---

## Setup (Step by Step)

### Step 1 — Install the package

```bash
npm install msr-auth
```

### Step 2 — Provide HttpClient

The library uses Angular's `HttpClient` under the hood, so you must provide it in your application config.

```typescript
// app.config.ts
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideHttpClient(),
    // ... other providers
  ],
};
```

### Step 3 — Configure the library

Use the `provideMsrAuth()` function to register the library with your API base URL and (optionally) override default endpoint paths.

```typescript
// app.config.ts
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';
import { provideMsrAuth } from 'msr-auth';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideHttpClient(),
    provideMsrAuth({
      baseUrl: 'https://api.example.com',
      // Optional: override specific endpoints
      // endpoints: {
      //   login: 'v2/auth/sign-in',
      //   register: 'v2/auth/sign-up',
      // },
    }),
  ],
};
```

### Step 4 — Inject & use the service

```typescript
import { inject } from '@angular/core';
import { MsrAuth, LoginRequest, AuthData, AuthResponse } from 'msr-auth';

export class LoginComponent {
  private auth = inject(MsrAuth);

  onLogin(): void {
    const credentials: LoginRequest = {
      username: 'user@example.com',
      password: 's3cureP@ss',
    };

    this.auth.login(credentials).subscribe({
      next: (response: AuthResponse<AuthData>) => {
        console.log('Token:', response.data?.token);
        console.log('User:', response.data?.user);
      },
      error: (err) => {
        console.error('Login failed:', err);
      },
    });
  }
}
```

---

## Configuration Reference

### AuthConfig

| Property    | Type                      | Required | Description                                 |
| ----------- | ------------------------- | -------- | ------------------------------------------- |
| `baseUrl`   | `string`                  | ✅       | Base URL of your backend API                |
| `endpoints` | `Partial<AuthEndpoints>`  | ❌       | Override one or more default endpoint paths |

### AuthEndpoints

| Property                     | Type     | Description                              |
| ---------------------------- | -------- | ---------------------------------------- |
| `login`                      | `string` | Path for user login                      |
| `register`                   | `string` | Path for user registration               |
| `forgotPassword`             | `string` | Path for forgot-password requests        |
| `resetPassword`              | `string` | Path for resetting passwords             |
| `sendEmailVerification`      | `string` | Path for sending verification emails     |
| `confirmEmailVerification`   | `string` | Path for confirming email verification   |

### Default Endpoints

If you don't override any endpoints, the library uses these defaults:

```
POST  {baseUrl}/auth/login
POST  {baseUrl}/auth/register
POST  {baseUrl}/auth/forgot-password
POST  {baseUrl}/auth/reset-password
POST  {baseUrl}/auth/send-email-verification
POST  {baseUrl}/auth/confirm-email-verification
```

---

## API Reference

### MsrAuth Service

The `MsrAuth` service is `providedIn: 'root'` — inject it anywhere with `inject(MsrAuth)`.

All methods return an `Observable<AuthResponse<TResponse>>` and send a `POST` request to the configured endpoint.

### Method Signatures

| Method                       | Default TRequest                     | Default TResponse | Description                                  |
| ---------------------------- | ------------------------------------ | ----------------- | -------------------------------------------- |
| `login(payload)`             | `LoginRequest`                       | `AuthData`        | Authenticate with credentials                |
| `register(payload)`          | `RegisterRequest`                    | `unknown`         | Create a new user account                    |
| `forgotPassword(payload)`    | `ForgotPasswordRequest`              | `unknown`         | Initiate password recovery                   |
| `resetPassword(payload)`     | `ResetPasswordRequest`               | `unknown`         | Reset password with a token                  |
| `sendEmailVerification(payload)`    | `SendEmailVerificationRequest` | `unknown`         | Send a verification email                    |
| `confirmEmailVerification(payload)` | `ConfirmEmailVerificationRequest` | `unknown`      | Confirm email with a verification code       |

---

## Interfaces

### AuthResponse\<TResponse\>

Standard API response wrapper returned by all methods.

```typescript
interface AuthResponse<TResponse = unknown> {
  status: boolean;   // Success or failure
  code: number;      // HTTP or application-specific status code
  message: string;   // Descriptive message from the server
  data?: TResponse;  // Optional response payload
}
```

### ErrorResponse

Structure for failed API requests.

```typescript
interface ErrorResponse {
  status: boolean;     // Always false for errors
  code: number;        // Error code
  message: string;     // Error message
  errors: ApiError[];  // Field-specific or general errors
}

interface ApiError {
  path: string;     // The field that caused the error (e.g. "email")
  message: string;  // Validation or error message
}
```

### AuthData

Default response type for the `login()` method.

```typescript
interface AuthData {
  token: string;   // JWT or session token
  code?: string;   // Optional verification code
  user: User;      // Authenticated user details
}
```

### User

```typescript
interface User {
  email: string;
  username: string;
  firstName: string;
  lastName: string;
  phone: string;
}
```

### Request Interfaces

All request interfaces include an index signature (`[key: string]: unknown`) so you can pass extra fields safely.

| Interface                          | Required Fields                                             |
| ---------------------------------- | ----------------------------------------------------------- |
| `LoginRequest`                     | `username?`, `password?`                                    |
| `RegisterRequest`                  | `username`, `email`, `firstName`, `lastName`, `phone`, `password?`, `confirmPassword?` |
| `ForgotPasswordRequest`           | `email`                                                     |
| `ResetPasswordRequest`            | `token`, `newPassword?`, `confirmPassword?`                 |
| `SendEmailVerificationRequest`    | `email`                                                     |
| `ConfirmEmailVerificationRequest` | `email`, `code`                                             |

---

## Advanced Usage

### Custom Request & Response Types

Every service method supports **generic type overrides** — you can replace both the request and response types to match your backend's exact contract:

```typescript
// Define your custom types
interface MyLoginPayload {
  email: string;
  password: string;
  rememberMe: boolean;
}

interface MyLoginResponse {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
}

// Use them
this.auth.login<MyLoginResponse, MyLoginPayload>({
  email: 'user@example.com',
  password: 's3cureP@ss',
  rememberMe: true,
}).subscribe((response) => {
  // response.data is typed as MyLoginResponse
  console.log(response.data?.accessToken);
  console.log(response.data?.refreshToken);
});
```

This pattern works for **all six methods** (`login`, `register`, `forgotPassword`, `resetPassword`, `sendEmailVerification`, `confirmEmailVerification`).

### Error Handling

Pair the library with an Angular `HttpInterceptor` to handle errors globally:

```typescript
// auth-error.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { catchError, throwError } from 'rxjs';
import { ErrorResponse } from 'msr-auth';

export const authErrorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((error) => {
      const errorBody = error.error as ErrorResponse;

      if (errorBody?.errors?.length) {
        errorBody.errors.forEach((e) =>
          console.error(`[${e.path}]: ${e.message}`)
        );
      }

      return throwError(() => error);
    })
  );
};
```

Register it in your app config:

```typescript
// app.config.ts
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authErrorInterceptor } from './interceptors/auth-error.interceptor';

provideHttpClient(withInterceptors([authErrorInterceptor]));
```

---

## Project Structure

```
libs/auth-shared-library/
├── .github/workflows/
│   └── publish.yml                # CI/CD — auto-publish to npm on push to main
├── projects/msr-auth/
│   ├── src/
│   │   ├── lib/
│   │   │   ├── config/
│   │   │   │   └── auth.config.ts          # InjectionToken, defaults, provideMsrAuth()
│   │   │   ├── interfaces/
│   │   │   │   └── auth.interface.ts       # All TypeScript interfaces
│   │   │   └── services/
│   │   │       ├── msr-auth.ts             # MsrAuth service (core logic)
│   │   │       └── msr-auth.spec.ts        # Unit tests
│   │   └── public-api.ts                   # Public API surface exports
│   ├── ng-package.json
│   └── package.json                        # npm package metadata (name, version, peers)
├── angular.json
├── package.json
├── tsconfig.json
└── README.md
```

---

## CI / CD

The repository includes a **GitHub Actions** workflow (`.github/workflows/publish.yml`) that automatically builds and publishes the library to npm on every push to `main`.

**Publish workflow:**

1. Checks out the code
2. Sets up Node.js 20
3. Installs dependencies (`npm ci`)
4. Builds the library (`ng build msr-auth --configuration production`)
5. Publishes to npm (`npm publish --access public`)

> **Important:** Bump the `version` in `projects/msr-auth/package.json` before pushing to `main`, otherwise npm will reject the publish.

---

## Contributing

1. Clone the repository
2. Run `npm install`
3. Make your changes in `projects/msr-auth/src/`
4. Build: `ng build msr-auth`
5. Test: `ng test msr-auth`
6. Bump the version in `projects/msr-auth/package.json`
7. Push to `main` to trigger the auto-publish workflow

---

## License

This project is open source.
