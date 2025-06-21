# Data Models

This section provides comprehensive documentation of all data models used throughout the Blog API, including their properties, validation rules, and relationships.

## User Models

### User (Complete Model)

The primary user data model returned in most API responses.

```typescript
interface User {
  _id: string; // MongoDB ObjectId, read-only
  username: string; // Unique username, max 20 characters
  email: string; // Unique email address, max 50 characters
  role: "admin" | "user"; // User role, read-only, default: "user"
  firstName?: string; // Optional first name, max 20 characters
  lastName?: string; // Optional last name, max 20 characters
  socialLinks: {
    website?: string; // Optional website URL, max 100 characters
    facebook?: string; // Optional Facebook URL, max 100 characters
    instagram?: string; // Optional Instagram URL, max 100 characters
    linkedin?: string; // Optional LinkedIn URL, max 100 characters
    x?: string; // Optional X (Twitter) URL, max 100 characters
    youtube?: string; // Optional YouTube URL, max 100 characters
  };
  createdAt: string; // ISO 8601 timestamp, read-only
  updatedAt: string; // ISO 8601 timestamp, read-only
}
```

**Example**:

```json
{
  "_id": "60b7c8b4f9b3c12345678901",
  "username": "johndoe",
  "email": "john@example.com",
  "role": "user",
  "firstName": "John",
  "lastName": "Doe",
  "socialLinks": {
    "website": "https://johndoe.com",
    "linkedin": "https://linkedin.com/in/johndoe",
    "x": "https://x.com/johndoe"
  },
  "createdAt": "2024-01-15T10:30:00.000Z",
  "updatedAt": "2024-01-20T15:45:00.000Z"
}
```

### UserInputRequired (Registration/Login)

Model for user registration and login requests.

```typescript
interface UserInputRequired {
  email: string; // Required, valid email format, max 50 characters
  password: string; // Required, min 8 characters, write-only
  role?: "admin" | "user"; // Optional for registration
}
```

**Validation Rules**:

- `email`: Must be valid email format, unique across all users
- `password`: Minimum 8 characters, not returned in responses
- `role`: Optional, defaults to "user", admin requires whitelist

**Example**:

```json
{
  "email": "newuser@example.com",
  "password": "securepassword123",
  "role": "user"
}
```

### UserUpdateInput (Profile Updates)

Model for updating user profiles.

```typescript
interface UserUpdateInput {
  username?: string; // Optional, max 20 characters, must be unique
  email?: string; // Optional, valid email format, max 50 characters
  password?: string; // Optional, min 8 characters, write-only
  first_name?: string; // Optional, max 20 characters (snake_case in API)
  last_name?: string; // Optional, max 20 characters (snake_case in API)
  website?: string; // Optional, valid URL, max 100 characters
  facebook?: string; // Optional, valid URL, max 100 characters
  instagram?: string; // Optional, valid URL, max 100 characters
  linkedin?: string; // Optional, valid URL, max 100 characters
  x?: string; // Optional, valid URL, max 100 characters
  youtube?: string; // Optional, valid URL, max 100 characters
}
```

**Note**: Field names use snake_case in API requests but are stored as camelCase internally.

---

## Blog Models

### Blog (Complete Model)

The primary blog post data model.

```typescript
interface Blog {
  _id: string; // MongoDB ObjectId, read-only
  title: string; // Blog title, max 180 characters
  slug: string; // URL-friendly identifier, auto-generated, read-only
  content: string; // HTML content of the blog post
  banner: {
    url: string; // Cloudinary URL, read-only
    width: number; // Image width in pixels, read-only
    height: number; // Image height in pixels, read-only
  };
  author: User; // Complete User object, read-only
  viewsCount: number; // Number of views, default: 0, read-only
  likesCount: number; // Number of likes, default: 0, read-only
  commentsCount: number; // Number of comments, default: 0, read-only
  status: "draft" | "published"; // Publication status, default: "draft"
  publishedAt?: string; // ISO 8601 timestamp, read-only, null for drafts
  updatedAt: string; // ISO 8601 timestamp, read-only
}
```

**Example**:

```json
{
  "_id": "60b7c8b4f9b3c12345678903",
  "title": "Complete Guide to REST APIs",
  "slug": "complete-guide-to-rest-apis",
  "content": "<h1>Introduction</h1><p>REST APIs are...</p>",
  "banner": {
    "url": "https://res.cloudinary.com/example/image/upload/v1234567890/banners/abc123.jpg",
    "width": 1200,
    "height": 630
  },
  "author": {
    "_id": "60b7c8b4f9b3c12345678901",
    "username": "admin",
    "email": "admin@example.com",
    "role": "admin"
  },
  "viewsCount": 1250,
  "likesCount": 87,
  "commentsCount": 23,
  "status": "published",
  "publishedAt": "2024-01-20T14:30:00.000Z",
  "updatedAt": "2024-01-20T14:30:00.000Z"
}
```

### BlogInput (Creation)

Model for creating new blog posts.

```typescript
interface BlogInput {
  title: string; // Required, max 180 characters
  content: string; // Required, HTML content
  status?: "draft" | "published"; // Optional, default: "draft"
  banner_image: File; // Required, binary file (PNG/JPG/WebP, max 2MB)
}
```

**Form Data Example**:

```javascript
const formData = new FormData();
formData.append("title", "My New Blog Post");
formData.append("content", "<h1>Hello World</h1><p>This is my first post.</p>");
formData.append("status", "published");
formData.append("banner_image", fileInput.files[0]);
```

### BlogUpdateInput (Updates)

Model for updating existing blog posts.

```typescript
interface BlogUpdateInput {
  title?: string; // Optional, max 180 characters
  content?: string; // Optional, HTML content
  status?: "draft" | "published"; // Optional
  banner_image?: File; // Optional, binary file (PNG/JPG/WebP, max 2MB)
}
```

---

## Comment Models

### Comment (Complete Model)

The primary comment data model.

```typescript
interface Comment {
  _id: string; // MongoDB ObjectId, read-only
  blogId: string; // ObjectId of the associated blog post
  userId: string; // ObjectId of the comment author, read-only
  content: string; // Comment text, max 1000 characters
  likesCount: number; // Number of likes, default: 0, read-only
  createdAt: string; // ISO 8601 timestamp, read-only
  updatedAt: string; // ISO 8601 timestamp, read-only
}
```

**Example**:

```json
{
  "_id": "60b7c8b4f9b3c12345678905",
  "blogId": "60b7c8b4f9b3c12345678903",
  "userId": "60b7c8b4f9b3c12345678901",
  "content": "Great article! The examples really helped me understand the concepts better.",
  "likesCount": 5,
  "createdAt": "2024-01-21T14:30:00.000Z",
  "updatedAt": "2024-01-21T14:30:00.000Z"
}
```

### CommentInput (Creation)

Model for creating new comments.

```typescript
interface CommentInput {
  blogId: string; // Required, ObjectId of the blog post
  content: string; // Required, max 1000 characters
}
```

**Example**:

```json
{
  "blogId": "60b7c8b4f9b3c12345678903",
  "content": "Thanks for sharing this! Very helpful tutorial."
}
```

---

## Response Models

### Paginated Responses

#### PaginatedUsers

```typescript
interface PaginatedUsers {
  limit: number; // Number of users per page
  offset: number; // Number of users skipped
  total: number; // Total number of users
  users: User[]; // Array of User objects
}
```

#### PaginatedBlogs

```typescript
interface PaginatedBlogs {
  limit: number; // Number of blogs per page
  offset: number; // Number of blogs skipped
  total: number; // Total number of blogs
  blogs: Blog[]; // Array of Blog objects
}
```

### Authentication Responses

#### AccessTokenResponse

```typescript
interface AccessTokenResponse {
  accessToken: string; // JWT access token
}
```

#### LoginResponse

Combines access token with user information.

```typescript
interface LoginResponse {
  accessToken: string; // JWT access token
  user: User; // Complete user object
}
```

**Example**:

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "60b7c8b4f9b3c12345678901",
    "username": "johndoe",
    "email": "john@example.com",
    "role": "user"
  }
}
```

---

## Error Models

### ErrorResponse (General Errors)

Standard error response format for most API errors.

```typescript
interface ErrorResponse {
  code: string; // Application-specific error code
  message: string; // Human-readable error message
}
```

**Common Error Codes**:

- `AuthenticationError`: Token missing/invalid/expired
- `AuthorizationError`: Insufficient permissions
- `NotFound`: Resource not found
- `BadRequest`: General bad request
- `ServerError`: Internal server error

**Example**:

```json
{
  "code": "NotFound",
  "message": "Blog post not found"
}
```

### ValidationErrorResponse (Input Validation)

Detailed error response for input validation failures.

```typescript
interface ValidationErrorResponse {
  code: "ValidationError";
  errors: {
    [fieldName: string]: ValidationErrorDetail;
  };
}

interface ValidationErrorDetail {
  type: string; // Error type (e.g., "field")
  value: any; // The invalid value provided
  msg: string; // Error message for this field
  path: string; // Field path
  location: string; // Where the field was found (e.g., "body")
}
```

**Example**:

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

---

## Data Types and Formats

### MongoDB ObjectId

All `_id` fields use MongoDB ObjectId format:

- **Format**: 24-character hexadecimal string
- **Example**: `"60b7c8b4f9b3c12345678901"`
- **Validation**: Must be a valid ObjectId when provided

### ISO 8601 Timestamps

All date/time fields use ISO 8601 format:

- **Format**: `YYYY-MM-DDTHH:mm:ss.sssZ`
- **Example**: `"2024-01-21T14:30:00.000Z"`
- **Timezone**: Always UTC (Z suffix)

### URLs

All URL fields must be valid URLs:

- **Format**: Must include protocol (http:// or https://)
- **Example**: `"https://example.com"`
- **Validation**: Must pass URL validation

### File Uploads

Banner image files have specific requirements:

- **Formats**: PNG, JPEG, WebP
- **Size Limit**: 2MB maximum
- **Processing**: Automatically optimized via Cloudinary
- **Response**: Returns processed image URL with dimensions

## Model Relationships

### User → Blogs (One-to-Many)

- Users can author multiple blog posts
- Blog posts reference the author via `author` field
- Deleting a user cascades to their blog posts

### Blog → Comments (One-to-Many)

- Blog posts can have multiple comments
- Comments reference the blog via `blogId` field
- Deleting a blog cascades to its comments

### User → Comments (One-to-Many)

- Users can create multiple comments
- Comments reference the user via `userId` field
- Deleting a user cascades to their comments

### User → Likes (Many-to-Many)

- Users can like multiple blog posts
- Blog posts can be liked by multiple users
- Relationship tracked in separate likes collection
- Aggregate counts stored in blog posts

## Field Validation Summary

| Field               | Type   | Required | Max Length | Format       | Unique |
| ------------------- | ------ | -------- | ---------- | ------------ | ------ |
| User.username       | string | ✓        | 20         | alphanumeric | ✓      |
| User.email          | string | ✓        | 50         | email        | ✓      |
| User.password       | string | ✓        | -          | min 8 chars  | -      |
| User.firstName      | string | -        | 20         | text         | -      |
| User.lastName       | string | -        | 20         | text         | -      |
| User.socialLinks.\* | string | -        | 100        | URL          | -      |
| Blog.title          | string | ✓        | 180        | text         | -      |
| Blog.content        | string | ✓        | -          | HTML         | -      |
| Comment.content     | string | ✓        | 1000       | text         | -      |

## TypeScript Definitions

For TypeScript applications, you can use these complete type definitions:

```typescript
// Copy and paste into your TypeScript project
export interface User {
  _id: string;
  username: string;
  email: string;
  role: "admin" | "user";
  firstName?: string;
  lastName?: string;
  socialLinks: {
    website?: string;
    facebook?: string;
    instagram?: string;
    linkedin?: string;
    x?: string;
    youtube?: string;
  };
  createdAt: string;
  updatedAt: string;
}

export interface Blog {
  _id: string;
  title: string;
  slug: string;
  content: string;
  banner: {
    url: string;
    width: number;
    height: number;
  };
  author: User;
  viewsCount: number;
  likesCount: number;
  commentsCount: number;
  status: "draft" | "published";
  publishedAt?: string;
  updatedAt: string;
}

export interface Comment {
  _id: string;
  blogId: string;
  userId: string;
  content: string;
  likesCount: number;
  createdAt: string;
  updatedAt: string;
}

export interface ErrorResponse {
  code: string;
  message: string;
}

export interface ValidationErrorResponse {
  code: "ValidationError";
  errors: {
    [fieldName: string]: {
      type: string;
      value: any;
      msg: string;
      path: string;
      location: string;
    };
  };
}
```
