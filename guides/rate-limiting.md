# Rate Limiting

To ensure fair usage and protect the API from abuse, rate limiting is applied.

- **Limit:** 60 requests per minute per IP address.
- **Headers:** Standard rate limit headers (`RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset`, `Retry-After`) are included in responses according to the [IETF Draft standard](https://datatracker.ietf.org/doc/html/draft-ietf-httpapi-ratelimit-headers-08).
- **Exceeding Limit:** If you exceed the rate limit, you will receive an HTTP `429 Too Many Requests` response. Check the `Retry-After` header to see when you can make requests again.

**Example 429 Response:**

```http
HTTP/1.1 429 Too Many Requests
RateLimit-Limit: 60
RateLimit-Remaining: 0
RateLimit-Reset: 45  // Seconds until the limit resets
Retry-After: 45      // Seconds to wait before retrying
Content-Type: application/json

{
  "error": "You can only make 60 requests every minute."
}
```
