# Authentication

This API uses JSON Web Tokens (JWT) for securing endpoints. It employs a two-token strategy: an Access Token and a Refresh Token.

## Access Token

- **Usage:** Required for accessing protected API endpoints.
- **Format:** Standard JWT.
- **Transmission:** Must be included in the `Authorization` header of your request using the `Bearer` scheme.

```http
  Authorization: Bearer <your_access_token>
```

- **Lifetime:** Access tokens have a predefined expiration time (configured on the server, e.g., 15 minutes). Once expired, you'll receive a `401 Unauthorized` error.
- **Obtaining:** Received upon successful registration (`/auth/register`) or login (`/auth/login`). Can be renewed using a valid Refresh Token (`/auth/refresh-token`).

## Refresh Token

- **Usage:** Used to obtain a new Access Token when the current one expires, without requiring the user to log in again.
- **Format:** Standard JWT.
- **Transmission:** Automatically handled via an `HttpOnly`, `Secure` (in production), `SameSite=Strict` cookie named `refreshToken`. You generally don't need to manage this token directly in your client-side code.
- **Lifetime:** Refresh tokens have a longer expiration time (configured on the server, e.g., 7 days).
- **Obtaining:** Set automatically as a cookie upon successful registration or login.
- **Invalidation:** Deleted from the server and the cookie is cleared upon logout (`/auth/logout`).

## Authentication Flow

1. **Register or Login:** Call `/auth/register` or `/auth/login` with valid credentials.
   - Receive an `accessToken` in the response body.
   - Receive a `refreshToken` set as an HttpOnly cookie.
2. **Access Protected Resources:** Make requests to other API endpoints, including the `accessToken` in the `Authorization: Bearer <token>` header.
3. **Handle Expired Access Token:** If a request returns a `401 Unauthorized` error indicating an expired access token:
   - Call `POST /auth/refresh-token`. This endpoint uses the `refreshToken` cookie automatically sent by the browser.
   - Receive a new `accessToken` in the response.
   - Retry the original request with the new `accessToken`.
4. **Handle Expired Refresh Token:** If the `/auth/refresh-token` endpoint returns a `401 Unauthorized` error, the refresh token has expired or is invalid. The user must log in again via `/auth/login`.
5. **Logout:** Call `POST /auth/logout`. This requires both the `accessToken` (in the header) and the `refreshToken` (in the cookie) to be valid. It invalidates the refresh token on the server and clears the cookie.
