# Likes

The Likes API allows users to like and unlike blog posts, providing engagement metrics for content. Each user can like a blog post only once, and the system tracks the total number of likes for each post.

## Endpoints

### Like Blog Post

Adds a like to a specific blog post and increments the blog's like count.

**Endpoint**: `POST /likes/blog/{blogId}`

**Authentication**: Required (Bearer token)

**Path Parameters**:

- `blogId`: MongoDB ObjectId of the blog post to like

**Success Response** (200 OK):

```json
{
  "likesCount": 156
}
```

**Error Responses**:

- `400` - User has already liked this blog post
- `401` - Unauthorized
- `404` - Blog post not found
- `500` - Internal server error

**Example Error Response** (400 Bad Request):

```json
{
  "code": "BadRequest",
  "message": "You already liked this blog"
}
```

---

### Unlike Blog Post

Removes a like from a specific blog post and decrements the blog's like count.

**Endpoint**: `DELETE /likes/blog/{blogId}`

**Authentication**: Required (Bearer token)

**Path Parameters**:

- `blogId`: MongoDB ObjectId of the blog post to unlike

**Success Response** (204 No Content): Empty response body

**Error Responses**:

- `400` - User has not liked this blog post previously
- `401` - Unauthorized
- `404` - Blog post not found
- `500` - Internal server error

**Example Error Response** (400 Bad Request):

```json
{
  "code": "BadRequest",
  "message": "Like not found"
}
```

## Like System Behavior

### One Like Per User

- Each user can like a blog post only once
- Attempting to like an already-liked post returns a 400 error
- Attempting to unlike a post that wasn't liked returns a 400 error

### Like Count Tracking

- Blog posts maintain a `likesCount` field
- Count is automatically incremented/decremented with likes/unlikes
- The like count is returned when liking a post
- Count is visible in blog post responses

### Like Persistence

- Likes are permanently stored in the database
- The system tracks which user liked which post
- Like history is maintained for analytics and user experience

## Code Examples

### Like a Blog Post

```javascript
const likeBlogPost = async (accessToken, blogId) => {
  const response = await fetch(`/api/v1/likes/blog/${blogId}`, {
    method: "POST",
    headers: {
      Authorization: `Bearer ${accessToken}`,
      "Content-Type": "application/json",
    },
  });

  if (response.ok) {
    const data = await response.json();
    return data.likesCount;
  }

  if (response.status === 400) {
    const error = await response.json();
    throw new Error(error.message || "Already liked");
  }

  throw new Error("Failed to like blog post");
};
```

### Unlike a Blog Post

```javascript
const unlikeBlogPost = async (accessToken, blogId) => {
  const response = await fetch(`/api/v1/likes/blog/${blogId}`, {
    method: "DELETE",
    headers: {
      Authorization: `Bearer ${accessToken}`,
      "Content-Type": "application/json",
    },
  });

  if (response.status === 204) {
    return true;
  }

  if (response.status === 400) {
    const error = await response.json();
    throw new Error(error.message || "Not previously liked");
  }

  throw new Error("Failed to unlike blog post");
};
```

### Complete Like Management Class

```javascript
class LikeAPI {
  constructor(baseURL, accessToken) {
    this.baseURL = baseURL;
    this.accessToken = accessToken;
  }

  async likeBlog(blogId) {
    const response = await fetch(`${this.baseURL}/likes/blog/${blogId}`, {
      method: "POST",
      headers: this._getHeaders(),
    });

    if (response.ok) {
      const data = await response.json();
      return data.likesCount;
    }

    return this._handleError(response);
  }

  async unlikeBlog(blogId) {
    const response = await fetch(`${this.baseURL}/likes/blog/${blogId}`, {
      method: "DELETE",
      headers: this._getHeaders(),
    });

    if (response.status === 204) {
      return true;
    }

    return this._handleError(response);
  }

  async toggleLike(blogId, currentlyLiked) {
    try {
      if (currentlyLiked) {
        await this.unlikeBlog(blogId);
        return { liked: false, likesCount: null };
      } else {
        const likesCount = await this.likeBlog(blogId);
        return { liked: true, likesCount };
      }
    } catch (error) {
      throw new Error(`Failed to toggle like: ${error.message}`);
    }
  }

  _getHeaders() {
    return {
      Authorization: `Bearer ${this.accessToken}`,
      "Content-Type": "application/json",
    };
  }

  async _handleError(response) {
    const error = await response
      .json()
      .catch(() => ({ message: "Unknown error" }));
    throw new Error(error.message || `HTTP ${response.status}`);
  }
}
```

### React Like Button Component

```jsx
import React, { useState } from "react";

const LikeButton = ({
  blogId,
  initialLikesCount,
  initiallyLiked,
  accessToken,
}) => {
  const [likesCount, setLikesCount] = useState(initialLikesCount);
  const [isLiked, setIsLiked] = useState(initiallyLiked);
  const [loading, setLoading] = useState(false);

  const likeAPI = new LikeAPI("/api/v1", accessToken);

  const handleToggleLike = async () => {
    if (loading) return;

    setLoading(true);
    try {
      const result = await likeAPI.toggleLike(blogId, isLiked);
      setIsLiked(result.liked);

      if (result.liked) {
        setLikesCount(result.likesCount);
      } else {
        // Decrement count when unliking
        setLikesCount((prev) => Math.max(0, prev - 1));
      }
    } catch (error) {
      console.error("Failed to toggle like:", error);
      alert(error.message);
    }
    setLoading(false);
  };

  return (
    <button
      onClick={handleToggleLike}
      disabled={loading}
      className={`like-button ${isLiked ? "liked" : ""}`}
      aria-label={isLiked ? "Unlike this post" : "Like this post"}
    >
      <span className="like-icon">{isLiked ? "❤️" : "🤍"}</span>
      <span className="like-count">{likesCount}</span>
      {loading && <span className="loading">...</span>}
    </button>
  );
};

// Usage
const BlogPost = ({ blog, userLikes, accessToken }) => {
  const isLiked = userLikes.includes(blog._id);

  return (
    <div className="blog-post">
      <h1>{blog.title}</h1>
      <div className="blog-content">{blog.content}</div>

      <div className="blog-actions">
        <LikeButton
          blogId={blog._id}
          initialLikesCount={blog.likesCount}
          initiallyLiked={isLiked}
          accessToken={accessToken}
        />
      </div>
    </div>
  );
};
```

### Bulk Like Status Check

```javascript
// Helper function to check like status for multiple blogs
const checkLikeStatuses = async (accessToken, blogIds) => {
  // Note: This would require a custom endpoint or multiple API calls
  // Current API doesn't have bulk like status checking

  const likeStatuses = {};

  for (const blogId of blogIds) {
    try {
      // This is a workaround - try to like and immediately unlike
      // In a real implementation, you'd want a dedicated endpoint
      await likeBlog(accessToken, blogId);
      await unlikeBlog(accessToken, blogId);
      likeStatuses[blogId] = false; // Not previously liked
    } catch (error) {
      if (error.message.includes("already liked")) {
        likeStatuses[blogId] = true; // Already liked
      } else {
        likeStatuses[blogId] = false; // Assume not liked on error
      }
    }
  }

  return likeStatuses;
};
```

## User Experience Patterns

### Optimistic Updates

For better user experience, implement optimistic updates:

```javascript
const OptimisticLikeButton = ({
  blogId,
  initialLikesCount,
  initiallyLiked,
  accessToken,
}) => {
  const [likesCount, setLikesCount] = useState(initialLikesCount);
  const [isLiked, setIsLiked] = useState(initiallyLiked);
  const [loading, setLoading] = useState(false);

  const handleToggleLike = async () => {
    // Optimistic update
    const newLikedState = !isLiked;
    const newCount = newLikedState ? likesCount + 1 : likesCount - 1;

    setIsLiked(newLikedState);
    setLikesCount(newCount);
    setLoading(true);

    try {
      const likeAPI = new LikeAPI("/api/v1", accessToken);

      if (newLikedState) {
        const actualCount = await likeAPI.likeBlog(blogId);
        setLikesCount(actualCount); // Use server count for accuracy
      } else {
        await likeAPI.unlikeBlog(blogId);
      }
    } catch (error) {
      // Revert optimistic update on error
      setIsLiked(isLiked);
      setLikesCount(likesCount);
      console.error("Failed to toggle like:", error);
    }

    setLoading(false);
  };

  return (
    <button onClick={handleToggleLike} disabled={loading}>
      {isLiked ? "❤️" : "🤍"} {likesCount}
    </button>
  );
};
```

### Debounced Likes

To prevent rapid clicking issues:

```javascript
import { useCallback, useRef } from "react";

const useDebouncedLike = (callback, delay = 500) => {
  const timeoutRef = useRef();

  return useCallback(
    (...args) => {
      clearTimeout(timeoutRef.current);
      timeoutRef.current = setTimeout(() => callback(...args), delay);
    },
    [callback, delay]
  );
};

const DebouncedLikeButton = ({ blogId, accessToken }) => {
  const [state, setState] = useState({ liked: false, count: 0 });

  const handleLike = async () => {
    const likeAPI = new LikeAPI("/api/v1", accessToken);
    try {
      const result = await likeAPI.toggleLike(blogId, state.liked);
      setState({
        liked: result.liked,
        count: result.likesCount || state.count,
      });
    } catch (error) {
      console.error("Like failed:", error);
    }
  };

  const debouncedLike = useDebouncedLike(handleLike);

  return (
    <button onClick={debouncedLike}>
      {state.liked ? "❤️" : "🤍"} {state.count}
    </button>
  );
};
```

## Analytics and Insights

### Like Metrics

The like system provides valuable engagement metrics:

- **Total Likes**: Overall engagement across all blog posts
- **Like Rate**: Percentage of viewers who like a post
- **Popular Content**: Posts with highest like counts
- **User Engagement**: Most active users based on likes given

### Potential Analytics Endpoints

While not currently implemented, these could be valuable additions:

```javascript
// Example additional endpoints that could be implemented
const getTopLikedPosts = async (accessToken, limit = 10) => {
  // GET /analytics/posts/top-liked?limit=10
};

const getUserLikeHistory = async (accessToken, userId) => {
  // GET /users/{userId}/likes
};

const getBlogLikeHistory = async (accessToken, blogId) => {
  // GET /blogs/{blogId}/likes
};
```

## Best Practices

### For Users

1. **Meaningful Likes**: Like content that genuinely provides value
2. **Authentic Engagement**: Avoid like/unlike spamming
3. **Discover Content**: Use likes to bookmark interesting posts

### For Developers

1. **Error Handling**: Always handle like/unlike errors gracefully
2. **User Feedback**: Provide clear visual feedback for like actions
3. **Performance**: Implement optimistic updates for better UX
4. **Rate Limiting**: Consider implementing like rate limiting
5. **Analytics**: Track like patterns for content insights

### Security Considerations

1. **Authentication**: Always verify user authentication
2. **Duplicate Prevention**: Enforce one-like-per-user rule
3. **Rate Limiting**: Prevent like spam attacks
4. **Input Validation**: Validate blogId parameters

## Integration with Other Features

### Comments Integration

```javascript
const BlogEngagement = ({ blog, accessToken }) => {
  return (
    <div className="engagement-section">
      <LikeButton
        blogId={blog._id}
        initialLikesCount={blog.likesCount}
        accessToken={accessToken}
      />
      <CommentsSection
        blogId={blog._id}
        commentsCount={blog.commentsCount}
        accessToken={accessToken}
      />
    </div>
  );
};
```

### Social Sharing

```javascript
const SocialActions = ({ blog, isLiked, accessToken }) => {
  const shareText = `Check out this article: ${blog.title}`;
  const shareUrl = `https://blog.example.com/posts/${blog.slug}`;

  return (
    <div className="social-actions">
      <LikeButton blogId={blog._id} accessToken={accessToken} />
      <button
        onClick={() => navigator.share({ text: shareText, url: shareUrl })}
      >
        Share
      </button>
      <button onClick={() => copyToClipboard(shareUrl)}>Copy Link</button>
    </div>
  );
};
```

The Likes API provides a foundation for user engagement and content popularity metrics. While simple in design, it can be extended with additional features like comment likes, user like histories, and advanced analytics as the platform grows.
