# Error Responses

This comprehensive guide covers all error responses in the Blog API, including status codes, error formats, troubleshooting steps, and best practices for error handling.

## Error Response Format

The API uses consistent JSON error responses with the following structure:

### Standard Error Response

```json
{
  "code": "ErrorCode",
  "message": "Human-readable error description"
}
```

### Validation Error Response

```json
{
  "code": "ValidationError",
  "errors": {
    "fieldName": {
      "type": "field",
      "value": "invalid-value",
      "msg": "Specific field error message",
      "path": "fieldName",
      "location": "body"
    }
  }
}
```

## HTTP Status Codes

### 2xx Success Codes

| Code | Meaning    | Usage                                             |
| ---- | ---------- | ------------------------------------------------- |
| 200  | OK         | Successful GET, POST (non-creation), PUT requests |
| 201  | Created    | Successful resource creation (POST)               |
| 204  | No Content | Successful DELETE requests                        |

### 4xx Client Error Codes

#### 400 Bad Request

**Usage**: Invalid request data, validation failures, business logic violations

**Common Error Codes**:

- `ValidationError`: Input validation failed
- `BadRequest`: General bad request (e.g., already liked post)

**Examples**:

_Validation Error_:

```json
{
  "code": "ValidationError",
  "errors": {
    "email": {
      "type": "field",
      "value": "invalid-email",
      "msg": "Invalid email address",
      "path": "email",
      "location": "body"
    },
    "password": {
      "type": "field",
      "value": "short",
      "msg": "Password must be at least 8 characters long",
      "path": "password",
      "location": "body"
    }
  }
}
```

_Business Logic Error_:

```json
{
  "code": "BadRequest",
  "message": "You already liked this blog"
}
```

**Troubleshooting**:

1. Validate all required fields are provided
2. Check field formats (email, URLs, etc.)
3. Verify field length constraints
4. Review business logic constraints

---

#### 401 Unauthorized

**Usage**: Authentication issues - missing, invalid, or expired tokens

**Error Code**: `AuthenticationError`

**Common Scenarios**:

- Missing Authorization header
- Invalid JWT token format
- Expired access token
- Invalid refresh token

**Examples**:

_Missing Token_:

```json
{
  "code": "AuthenticationError",
  "message": "Access denied, no token provided"
}
```

_Expired Token_:

```json
{
  "code": "AuthenticationError",
  "message": "Token has expired"
}
```

_Invalid Token_:

```json
{
  "code": "AuthenticationError",
  "message": "Invalid token"
}
```

**Troubleshooting**:

1. Ensure Authorization header is included: `Authorization: Bearer <token>`
2. Check if access token has expired - try refreshing
3. Verify token format is valid JWT
4. Re-authenticate if refresh token is also expired

**Code Example**:

```javascript
const handleAuthError = async (response) => {
  if (response.status === 401) {
    try {
      // Try to refresh token
      const newToken = await refreshAccessToken();
      // Retry original request with new token
      return retryRequest(newToken);
    } catch (refreshError) {
      // Refresh failed, redirect to login
      redirectToLogin();
    }
  }
};
```

---

#### 403 Forbidden

**Usage**: Authorization issues - insufficient permissions

**Error Code**: `AuthorizationError`

**Common Scenarios**:

- User role insufficient for action (regular user trying admin operations)
- Attempting to access/modify resources belonging to other users
- Admin registration with non-whitelisted email

**Examples**:

_Insufficient Role_:

```json
{
  "code": "AuthorizationError",
  "message": "Access denied, insufficient permissions"
}
```

_Admin Registration Denied_:

```json
{
  "code": "AuthorizationError",
  "message": "You cannot register as an admin"
}
```

_Resource Access Denied_:

```json
{
  "code": "AuthorizationError",
  "message": "You can only delete your own comments"
}
```

**Troubleshooting**:

1. Check user role requirements for the endpoint
2. Verify user owns the resource they're trying to modify
3. Confirm admin email is whitelisted for admin registration
4. Review API documentation for required permissions

---

#### 404 Not Found

**Usage**: Requested resource doesn't exist

**Error Code**: `NotFound`

**Common Scenarios**:

- Blog post with given ID/slug doesn't exist
- User with given ID doesn't exist
- Comment with given ID doesn't exist
- Invalid endpoint URL

**Examples**:

_Resource Not Found_:

```json
{
  "code": "NotFound",
  "message": "Blog not found"
}
```

_User Not Found_:

```json
{
  "code": "NotFound",
  "message": "User not found"
}
```

**Troubleshooting**:

1. Verify the resource ID/slug is correct
2. Check if resource was deleted
3. Ensure endpoint URL is correct
4. Confirm resource is accessible with current user permissions

---

#### 413 Payload Too Large

**Usage**: Uploaded file exceeds size limits

**Error Code**: `ValidationError`

**Scenarios**:

- Blog banner image larger than 2MB
- Request body too large

**Example**:

```json
{
  "code": "ValidationError",
  "message": "File size must be less than 2MB"
}
```

**Troubleshooting**:

1. Compress images before upload
2. Check file size limits in documentation
3. Use appropriate image formats (PNG, JPG, WebP)
4. Consider image optimization tools

---

#### 429 Too Many Requests

**Usage**: Rate limit exceeded

**Headers**: Includes rate limit information

- `RateLimit-Limit`: Total requests allowed
- `RateLimit-Remaining`: Requests remaining in current window
- `RateLimit-Reset`: Seconds until limit resets
- `Retry-After`: Seconds to wait before retrying

**Example Response**:

```http
HTTP/1.1 429 Too Many Requests
RateLimit-Limit: 60
RateLimit-Remaining: 0
RateLimit-Reset: 45
Retry-After: 45
Content-Type: application/json

{
  "error": "You can only make 60 requests every minute."
}
```

**Troubleshooting**:

1. Implement request rate limiting in client
2. Use exponential backoff for retries
3. Check rate limit headers to understand limits
4. Consider caching responses to reduce API calls

---

### 5xx Server Error Codes

#### 500 Internal Server Error

**Usage**: Unexpected server errors

**Error Code**: `ServerError`

**Example**:

```json
{
  "code": "ServerError",
  "message": "Internal server error"
}
```

**Troubleshooting**:

1. Retry the request after a brief delay
2. Check API status page for known issues
3. Contact support if error persists
4. Implement proper error logging in your application

## Error Handling Best Practices

### Client-Side Error Handling

```javascript
class APIClient {
  async makeRequest(url, options = {}) {
    try {
      const response = await fetch(url, {
        ...options,
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${this.accessToken}`,
          ...options.headers,
        },
      });

      if (!response.ok) {
        await this.handleError(response);
      }

      return await response.json();
    } catch (error) {
      console.error("API Request failed:", error);
      throw error;
    }
  }

  async handleError(response) {
    const status = response.status;
    let errorData;

    try {
      errorData = await response.json();
    } catch {
      errorData = { message: "Unknown error occurred" };
    }

    switch (status) {
      case 400:
        if (errorData.code === "ValidationError") {
          throw new ValidationError(errorData.errors);
        }
        throw new BadRequestError(errorData.message);

      case 401:
        // Try to refresh token
        if (await this.tryRefreshToken()) {
          // Token refreshed, caller should retry
          throw new TokenRefreshedError();
        }
        throw new AuthenticationError(errorData.message);

      case 403:
        throw new AuthorizationError(errorData.message);

      case 404:
        throw new NotFoundError(errorData.message);

      case 413:
        throw new PayloadTooLargeError(errorData.message);

      case 429:
        const retryAfter = response.headers.get("Retry-After");
        throw new RateLimitError(errorData.message, retryAfter);

      case 500:
        throw new ServerError(errorData.message);

      default:
        throw new APIError(`HTTP ${status}: ${errorData.message}`);
    }
  }
}

// Custom Error Classes
class APIError extends Error {
  constructor(message) {
    super(message);
    this.name = "APIError";
  }
}

class ValidationError extends APIError {
  constructor(errors) {
    super("Validation failed");
    this.name = "ValidationError";
    this.errors = errors;
  }
}

class AuthenticationError extends APIError {
  constructor(message) {
    super(message);
    this.name = "AuthenticationError";
  }
}

class RateLimitError extends APIError {
  constructor(message, retryAfter) {
    super(message);
    this.name = "RateLimitError";
    this.retryAfter = parseInt(retryAfter) || 60;
  }
}
```

### React Error Handling Example

```jsx
import React, { useState } from "react";

const BlogForm = () => {
  const [errors, setErrors] = useState({});
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (formData) => {
    setLoading(true);
    setErrors({});

    try {
      const apiClient = new APIClient();
      await apiClient.createBlog(formData);
      // Success handling
    } catch (error) {
      if (error instanceof ValidationError) {
        // Handle validation errors
        setErrors(error.errors);
      } else if (error instanceof AuthenticationError) {
        // Redirect to login
        redirectToLogin();
      } else if (error instanceof RateLimitError) {
        // Show rate limit message
        setErrors({
          general: `Rate limit exceeded. Try again in ${error.retryAfter} seconds.`,
        });
      } else {
        // General error handling
        setErrors({
          general: error.message || "An unexpected error occurred",
        });
      }
    }
    setLoading(false);
  };

  return (
    <form onSubmit={handleSubmit}>
      {errors.general && <div className="error-banner">{errors.general}</div>}

      <input type="text" name="title" className={errors.title ? "error" : ""} />
      {errors.title && <span className="field-error">{errors.title.msg}</span>}

      {/* Other form fields... */}
    </form>
  );
};
```

### Retry Logic with Exponential Backoff

```javascript
class RetryableAPIClient extends APIClient {
  async makeRequestWithRetry(url, options = {}, maxRetries = 3) {
    let lastError;

    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        return await this.makeRequest(url, options);
      } catch (error) {
        lastError = error;

        // Don't retry client errors (4xx) except 429
        if (error.status >= 400 && error.status < 500 && error.status !== 429) {
          throw error;
        }

        // Don't retry on last attempt
        if (attempt === maxRetries) {
          throw error;
        }

        // Calculate delay with exponential backoff
        const delay = Math.min(1000 * Math.pow(2, attempt), 10000);

        // Add jitter to prevent thundering herd
        const jitter = Math.random() * 1000;

        await new Promise((resolve) => setTimeout(resolve, delay + jitter));
      }
    }

    throw lastError;
  }
}
```

## Error Code Reference

### Authentication Errors (`AuthenticationError`)

| Message                            | Cause                         | Solution                          |
| ---------------------------------- | ----------------------------- | --------------------------------- |
| "Access denied, no token provided" | Missing Authorization header  | Add Bearer token to request       |
| "Invalid token"                    | Malformed JWT                 | Check token format and generation |
| "Token has expired"                | Expired access token          | Refresh token or re-authenticate  |
| "Invalid refresh token"            | Expired/invalid refresh token | Re-authenticate user              |

### Authorization Errors (`AuthorizationError`)

| Message                                   | Cause                              | Solution                                  |
| ----------------------------------------- | ---------------------------------- | ----------------------------------------- |
| "Access denied, insufficient permissions" | User role insufficient             | Check role requirements                   |
| "You cannot register as an admin"         | Non-whitelisted admin registration | Use whitelisted email or register as user |
| "You can only delete your own comments"   | Trying to delete others' content   | Verify resource ownership                 |

### Validation Errors (`ValidationError`)

| Field        | Common Messages                               | Solution                 |
| ------------ | --------------------------------------------- | ------------------------ |
| email        | "Invalid email address"                       | Use valid email format   |
| password     | "Password must be at least 8 characters long" | Increase password length |
| title        | "Title is required"                           | Provide blog title       |
| content      | "Content is required"                         | Provide blog content     |
| banner_image | "File size must be less than 2MB"             | Compress image file      |

### Business Logic Errors (`BadRequest`)

| Message                       | Cause                           | Solution                        |
| ----------------------------- | ------------------------------- | ------------------------------- |
| "You already liked this blog" | Duplicate like attempt          | Check like status before liking |
| "Like not found"              | Trying to unlike non-liked post | Verify post is liked            |
| "Blog not found"              | Invalid blog ID/slug            | Check blog exists               |

## Testing Error Scenarios

```javascript
// Test suite for error handling
describe("API Error Handling", () => {
  test("handles validation errors", async () => {
    const invalidData = { email: "invalid", password: "123" };

    try {
      await apiClient.register(invalidData);
    } catch (error) {
      expect(error).toBeInstanceOf(ValidationError);
      expect(error.errors.email.msg).toContain("Invalid email");
      expect(error.errors.password.msg).toContain("8 characters");
    }
  });

  test("handles authentication errors", async () => {
    const expiredToken = "expired.jwt.token";

    try {
      await apiClient.getCurrentUser(expiredToken);
    } catch (error) {
      expect(error).toBeInstanceOf(AuthenticationError);
      expect(error.message).toContain("expired");
    }
  });

  test("handles rate limiting", async () => {
    // Make requests until rate limited
    const promises = Array(100)
      .fill()
      .map(() => apiClient.getBlogs());

    try {
      await Promise.all(promises);
    } catch (error) {
      expect(error).toBeInstanceOf(RateLimitError);
      expect(error.retryAfter).toBeGreaterThan(0);
    }
  });
});
```

## Monitoring and Logging

```javascript
class APIClientWithLogging extends APIClient {
  async handleError(response) {
    const errorData = await response.json().catch(() => ({}));

    // Log error for monitoring
    this.logError({
      status: response.status,
      url: response.url,
      method: response.method || "GET",
      error: errorData,
      timestamp: new Date().toISOString(),
      userId: this.getCurrentUserId(),
    });

    // Continue with normal error handling
    await super.handleError(response);
  }

  logError(errorInfo) {
    // Send to logging service
    console.error("API Error:", errorInfo);

    // Optional: Send to analytics service
    if (typeof analytics !== "undefined") {
      analytics.track("API Error", errorInfo);
    }
  }
}
```

This comprehensive error handling guide ensures robust client-side error management and provides clear troubleshooting paths for common issues encountered when using the Blog API.
