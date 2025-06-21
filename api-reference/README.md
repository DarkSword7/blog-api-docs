# API Reference

Welcome to the comprehensive Blog API reference documentation. This section provides detailed information about all available endpoints, request/response formats, authentication, and data models.

## Quick Navigation

### Core Resources

- [Authentication](authentication.md) - User registration, login, and token management
- [Users](users.md) - User profile management and administration
- [Blogs](blogs.md) - Blog post creation, retrieval, and management
- [Comments](comments.md) - Comment creation and management
- [Likes](likes.md) - Blog post liking and unliking

### Additional Information

- [Data Models](data-models.md) - Complete schema definitions
- [Error Responses](error-responses.md) - Error handling and status codes
- [Rate Limiting](../guides/rate-limiting.md) - API usage limits
- [Quick Start Guide](quick-start.md) - Get started with the API

## Base URL

All API endpoints are relative to the following base URL:

```
https://blog-api-dc2g.onrender.com/api/v1
```

## Authentication

Most endpoints require authentication using JWT Bearer tokens. Include your access token in the Authorization header:

```http
Authorization: Bearer <your_access_token>
```

## Response Format

All responses are in JSON format. Successful responses include the requested data, while error responses follow a consistent error format with `code` and `message` fields.

## Getting Started

1. **Register an account** using `POST /auth/register`
2. **Obtain an access token** from the registration response
3. **Make authenticated requests** by including the token in the Authorization header
4. **Refresh tokens** when they expire using `POST /auth/refresh-token`

For detailed examples and usage patterns, see our [Quick Start Guide](quick-start.md).
