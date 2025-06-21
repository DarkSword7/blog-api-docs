# Blogs

The Blogs API provides comprehensive functionality for creating, managing, and retrieving blog posts. It includes support for rich content, image uploads, and status management with role-based access controls.

## Endpoints

### Create Blog Post

Creates a new blog post with banner image upload. Admin role required.

**Endpoint**: `POST /blogs/`

**Authentication**: Required (Bearer token with admin role)

**Content-Type**: `multipart/form-data`

**Form Data**:

- `title` (required): Blog post title (max 180 characters)
- `content` (required): HTML content of the blog post
- `status` (optional): "draft" or "published" (default: "draft")
- `banner_image` (required): Image file (PNG/JPG/WebP, max 2MB)

**Success Response** (201 Created):

```json
{
  "blog": {
    "_id": "60b7c8b4f9b3c12345678903",
    "title": "Getting Started with Node.js",
    "slug": "getting-started-with-nodejs",
    "content": "<h1>Introduction</h1><p>Node.js is a powerful runtime...</p>",
    "banner": {
      "url": "https://cloudinary.com/image/upload/v1234567890/banners/abc123.jpg",
      "width": 1200,
      "height": 630
    },
    "author": {
      "_id": "60b7c8b4f9b3c12345678901",
      "username": "admin",
      "email": "admin@example.com",
      "role": "admin"
    },
    "viewsCount": 0,
    "likesCount": 0,
    "commentsCount": 0,
    "status": "draft",
    "publishedAt": null,
    "updatedAt": "2024-01-21T10:30:00.000Z"
  }
}
```

**Error Responses**:

- `400` - Validation errors (missing fields, invalid data)
- `401` - Unauthorized
- `403` - Forbidden (user is not admin)
- `413` - File too large (over 2MB)
- `500` - Internal server error

**Usage Example**:

```javascript
const createBlog = async (accessToken, blogData) => {
  const formData = new FormData();
  formData.append("title", blogData.title);
  formData.append("content", blogData.content);
  formData.append("status", blogData.status || "draft");
  formData.append("banner_image", blogData.bannerFile);

  const response = await fetch("/api/v1/blogs/", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${accessToken}`,
      // Note: Don't set Content-Type for FormData
    },
    body: formData,
  });

  return await response.json();
};
```

---

### Get All Blogs

Retrieves a paginated list of blog posts. Admins see all posts; regular users see only published posts.

**Endpoint**: `GET /blogs/`

**Authentication**: Required (Bearer token)

**Query Parameters**:

- `limit` (optional): Number of blogs per page (1-50, default: 20)
- `offset` (optional): Number of blogs to skip (default: 0)

**Success Response** (200 OK):

```json
{
  "limit": 20,
  "offset": 0,
  "total": 45,
  "blogs": [
    {
      "_id": "60b7c8b4f9b3c12345678903",
      "title": "Advanced React Patterns",
      "slug": "advanced-react-patterns",
      "content": "<h1>Introduction</h1><p>React patterns help...</p>",
      "banner": {
        "url": "https://cloudinary.com/image/upload/v1234567890/banners/def456.jpg",
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
  ]
}
```

**Error Responses**:

- `400` - Invalid query parameters
- `401` - Unauthorized
- `500` - Internal server error

---

### Get Blogs by User

Retrieves blog posts created by a specific user.

**Endpoint**: `GET /blogs/user/{userId}`

**Authentication**: Required (Bearer token)

**Path Parameters**:

- `userId`: MongoDB ObjectId of the user

**Query Parameters**:

- `limit` (optional): Number of blogs per page (1-50, default: 20)
- `offset` (optional): Number of blogs to skip (default: 0)

**Success Response** (200 OK):

```json
{
  "limit": 20,
  "offset": 0,
  "total": 12,
  "blogs": [
    {
      "_id": "60b7c8b4f9b3c12345678904",
      "title": "My Journey with TypeScript",
      "slug": "my-journey-with-typescript",
      "content": "<h1>Starting Out</h1><p>When I first heard about TypeScript...</p>",
      "banner": {
        "url": "https://cloudinary.com/image/upload/v1234567890/banners/ghi789.jpg",
        "width": 1200,
        "height": 630
      },
      "author": {
        "_id": "60b7c8b4f9b3c12345678902",
        "username": "developer123",
        "email": "dev@example.com",
        "role": "user"
      },
      "viewsCount": 856,
      "likesCount": 42,
      "commentsCount": 15,
      "status": "published",
      "publishedAt": "2024-01-18T09:15:00.000Z",
      "updatedAt": "2024-01-18T09:15:00.000Z"
    }
  ]
}
```

**Error Responses**:

- `400` - Invalid parameters
- `401` - Unauthorized
- `500` - Internal server error

---

### Get Blog by Slug

Retrieves a single blog post by its unique slug. Increments view count.

**Endpoint**: `GET /blogs/{slug}`

**Authentication**: Required (Bearer token)

**Path Parameters**:

- `slug`: URL-friendly slug of the blog post

**Success Response** (200 OK):

```json
{
  "blog": {
    "_id": "60b7c8b4f9b3c12345678903",
    "title": "Complete Guide to REST APIs",
    "slug": "complete-guide-to-rest-apis",
    "content": "<h1>What are REST APIs?</h1><p>REST (Representational State Transfer) is an architectural style...</p><h2>HTTP Methods</h2><p>REST APIs use standard HTTP methods:</p><ul><li>GET - Retrieve data</li><li>POST - Create new resources</li><li>PUT - Update existing resources</li><li>DELETE - Remove resources</li></ul>",
    "banner": {
      "url": "https://cloudinary.com/image/upload/v1234567890/banners/jkl012.jpg",
      "width": 1200,
      "height": 630
    },
    "author": {
      "_id": "60b7c8b4f9b3c12345678901",
      "username": "admin",
      "email": "admin@example.com",
      "role": "admin",
      "firstName": "Admin",
      "lastName": "User"
    },
    "viewsCount": 2341,
    "likesCount": 156,
    "commentsCount": 48,
    "status": "published",
    "publishedAt": "2024-01-15T11:00:00.000Z",
    "updatedAt": "2024-01-15T11:00:00.000Z"
  }
}
```

**Error Responses**:

- `400` - Invalid slug format
- `401` - Unauthorized
- `403` - Forbidden (trying to access draft as regular user)
- `404` - Blog not found
- `500` - Internal server error

---

### Update Blog Post

Updates an existing blog post. Banner image update is optional. Admin role required.

**Endpoint**: `PUT /blogs/{blogId}`

**Authentication**: Required (Bearer token with admin role)

**Content-Type**: `multipart/form-data`

**Path Parameters**:

- `blogId`: MongoDB ObjectId of the blog post

**Form Data** (all fields optional):

- `title`: Updated blog title (max 180 characters)
- `content`: Updated HTML content
- `status`: "draft" or "published"
- `banner_image`: New banner image file (PNG/JPG/WebP, max 2MB)

**Success Response** (200 OK):

```json
{
  "blog": {
    "_id": "60b7c8b4f9b3c12345678903",
    "title": "Complete Guide to REST APIs - Updated",
    "slug": "complete-guide-to-rest-apis",
    "content": "<h1>What are REST APIs?</h1><p>Updated content with more examples...</p>",
    "banner": {
      "url": "https://cloudinary.com/image/upload/v1234567891/banners/new-banner.jpg",
      "width": 1200,
      "height": 630
    },
    "author": {
      "_id": "60b7c8b4f9b3c12345678901",
      "username": "admin",
      "email": "admin@example.com",
      "role": "admin"
    },
    "viewsCount": 2341,
    "likesCount": 156,
    "commentsCount": 48,
    "status": "published",
    "publishedAt": "2024-01-15T11:00:00.000Z",
    "updatedAt": "2024-01-21T16:45:00.000Z"
  }
}
```

**Error Responses**:

- `400` - Validation errors
- `401` - Unauthorized
- `403` - Forbidden (user is not admin)
- `404` - Blog not found
- `413` - File too large
- `500` - Internal server error

---

### Delete Blog Post

Permanently deletes a blog post and its banner image. Admin role required.

**Endpoint**: `DELETE /blogs/{blogId}`

**Authentication**: Required (Bearer token with admin role)

**Path Parameters**:

- `blogId`: MongoDB ObjectId of the blog post

**Success Response** (204 No Content): Empty response body

**Error Responses**:

- `401` - Unauthorized
- `403` - Forbidden (user is not admin)
- `404` - Blog not found
- `500` - Internal server error

**Note**: This action also deletes all associated comments and likes.

## Blog Status Management

### Draft Status

- Only visible to admins
- Can be edited and updated
- Not included in public blog listings for regular users
- Does not have a `publishedAt` timestamp

### Published Status

- Visible to all authenticated users
- Publicly accessible via slug
- Included in all blog listings
- Has a `publishedAt` timestamp set when first published

## Content Guidelines

### Title Requirements

- Maximum 180 characters
- Should be descriptive and SEO-friendly
- Automatically generates URL slug

### Content Format

- Supports full HTML content
- Recommended to use semantic HTML elements
- Images should be optimized before embedding
- External links should open in new tabs

### Banner Image Requirements

- **Formats**: PNG, JPG, WebP
- **Size Limit**: 2MB maximum
- **Recommended Dimensions**: 1200x630px (OG image ratio)
- **Automatically processed**: Cloudinary optimizes uploaded images

## Code Examples

### Create Blog Post with File Upload

```javascript
const createBlog = async (accessToken, blogData) => {
  const formData = new FormData();
  formData.append("title", blogData.title);
  formData.append("content", blogData.content);
  formData.append("status", blogData.status || "draft");
  formData.append("banner_image", blogData.bannerFile);

  const response = await fetch("/api/v1/blogs/", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${accessToken}`,
    },
    body: formData,
  });

  if (response.ok) {
    return await response.json();
  }
  throw new Error("Failed to create blog post");
};
```

### Get Blog Posts with Pagination

```javascript
const getBlogs = async (accessToken, page = 1, limit = 20) => {
  const offset = (page - 1) * limit;
  const response = await fetch(
    `/api/v1/blogs/?limit=${limit}&offset=${offset}`,
    {
      method: "GET",
      headers: {
        Authorization: `Bearer ${accessToken}`,
        "Content-Type": "application/json",
      },
    }
  );

  if (response.ok) {
    return await response.json();
  }
  throw new Error("Failed to fetch blogs");
};
```

### Complete Blog Management Class

```javascript
class BlogAPI {
  constructor(baseURL, accessToken) {
    this.baseURL = baseURL;
    this.accessToken = accessToken;
  }

  async createBlog(title, content, bannerFile, status = "draft") {
    const formData = new FormData();
    formData.append("title", title);
    formData.append("content", content);
    formData.append("status", status);
    formData.append("banner_image", bannerFile);

    const response = await fetch(`${this.baseURL}/blogs/`, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${this.accessToken}`,
      },
      body: formData,
    });

    return this._handleResponse(response);
  }

  async getAllBlogs(limit = 20, offset = 0) {
    const response = await fetch(
      `${this.baseURL}/blogs/?limit=${limit}&offset=${offset}`,
      {
        method: "GET",
        headers: this._getHeaders(),
      }
    );

    return this._handleResponse(response);
  }

  async getBlogsByUser(userId, limit = 20, offset = 0) {
    const response = await fetch(
      `${this.baseURL}/blogs/user/${userId}?limit=${limit}&offset=${offset}`,
      {
        method: "GET",
        headers: this._getHeaders(),
      }
    );

    return this._handleResponse(response);
  }

  async getBlogBySlug(slug) {
    const response = await fetch(`${this.baseURL}/blogs/${slug}`, {
      method: "GET",
      headers: this._getHeaders(),
    });

    return this._handleResponse(response);
  }

  async updateBlog(blogId, updates, bannerFile = null) {
    const formData = new FormData();

    if (updates.title) formData.append("title", updates.title);
    if (updates.content) formData.append("content", updates.content);
    if (updates.status) formData.append("status", updates.status);
    if (bannerFile) formData.append("banner_image", bannerFile);

    const response = await fetch(`${this.baseURL}/blogs/${blogId}`, {
      method: "PUT",
      headers: {
        Authorization: `Bearer ${this.accessToken}`,
      },
      body: formData,
    });

    return this._handleResponse(response);
  }

  async deleteBlog(blogId) {
    const response = await fetch(`${this.baseURL}/blogs/${blogId}`, {
      method: "DELETE",
      headers: this._getHeaders(),
    });

    if (response.status === 204) {
      return true;
    }
    throw new Error("Failed to delete blog");
  }

  _getHeaders() {
    return {
      Authorization: `Bearer ${this.accessToken}`,
      "Content-Type": "application/json",
    };
  }

  async _handleResponse(response) {
    if (response.ok) {
      return await response.json();
    }

    const error = await response
      .json()
      .catch(() => ({ message: "Unknown error" }));
    throw new Error(error.message || `HTTP ${response.status}`);
  }
}
```

## SEO and Performance

### Automatic Slug Generation

- Slugs are automatically generated from blog titles
- URL-friendly format (lowercase, hyphens instead of spaces)
- Unique across all blog posts

### Image Optimization

- Cloudinary integration for automatic image processing
- Multiple format support (WebP for modern browsers)
- Responsive image delivery

### Content Indexing

- Full-text search capabilities (implementation depends on database)
- Tag-based categorization (can be extended)
- Related post suggestions (can be implemented)

## Role-based Access Summary

### Regular Users (`role: "user"`)

- ✅ View published blog posts
- ✅ Search and filter blogs
- ❌ Create new blog posts
- ❌ Edit existing blog posts
- ❌ View draft posts
- ❌ Delete blog posts

### Administrators (`role: "admin"`)

- ✅ All user permissions
- ✅ Create new blog posts
- ✅ Edit any blog post
- ✅ Delete any blog post
- ✅ View draft posts
- ✅ Manage blog status (draft/published)
- ✅ Upload and manage banner images
