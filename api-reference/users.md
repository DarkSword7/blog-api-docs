# Users

The Users API provides endpoints for user profile management and user administration. Regular users can manage their own profiles, while administrators have additional capabilities for user management.

## Endpoints

### Get Current User Profile

Retrieves the profile of the currently authenticated user.

**Endpoint**: `GET /users/current`

**Authentication**: Required (Bearer token)

**Request**: No body required

**Success Response** (200 OK):

```json
{
  "user": {
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
}
```

**Error Responses**:

- `401` - Unauthorized (invalid or missing token)
- `500` - Internal server error

---

### Update Current User Profile

Updates the profile of the currently authenticated user.

**Endpoint**: `PUT /users/current`

**Authentication**: Required (Bearer token)

**Request Body** (all fields optional):

```json
{
  "username": "newusername",
  "email": "newemail@example.com",
  "password": "newpassword123",
  "first_name": "John",
  "last_name": "Doe",
  "website": "https://johndoe.com",
  "facebook": "https://facebook.com/johndoe",
  "instagram": "https://instagram.com/johndoe",
  "linkedin": "https://linkedin.com/in/johndoe",
  "x": "https://x.com/johndoe",
  "youtube": "https://youtube.com/@johndoe"
}
```

**Success Response** (200 OK):

```json
{
  "user": {
    "_id": "60b7c8b4f9b3c12345678901",
    "username": "newusername",
    "email": "newemail@example.com",
    "role": "user",
    "firstName": "John",
    "lastName": "Doe",
    "socialLinks": {
      "website": "https://johndoe.com",
      "facebook": "https://facebook.com/johndoe",
      "instagram": "https://instagram.com/johndoe",
      "linkedin": "https://linkedin.com/in/johndoe",
      "x": "https://x.com/johndoe",
      "youtube": "https://youtube.com/@johndoe"
    },
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-21T09:15:00.000Z"
  }
}
```

**Error Responses**:

- `400` - Validation errors (invalid email, username taken, etc.)
- `401` - Unauthorized
- `404` - User not found
- `500` - Internal server error

**Validation Rules**:

- `username`: Max 20 characters, must be unique
- `email`: Valid email format, max 50 characters, must be unique
- `password`: Min 8 characters
- `first_name`, `last_name`: Max 20 characters each
- Social links: Valid URLs, max 100 characters each

---

### Delete Current User Account

Permanently deletes the currently authenticated user's account and all associated data.

**Endpoint**: `DELETE /users/current`

**Authentication**: Required (Bearer token)

**Request**: No body required

**Success Response** (204 No Content): Empty response body

**Error Responses**:

- `401` - Unauthorized
- `500` - Internal server error

**Warning**: This action is irreversible and will delete:

- User profile and all personal data
- All blog posts authored by the user
- All comments made by the user
- All likes given by the user

---

## Admin-Only Endpoints

The following endpoints require administrator privileges (`role: "admin"`).

### Get All Users

Retrieves a paginated list of all users in the system.

**Endpoint**: `GET /users/`

**Authentication**: Required (Bearer token with admin role)

**Query Parameters**:

- `limit` (optional): Number of users per page (1-50, default: 20)
- `offset` (optional): Number of users to skip (default: 0)

**Example Request**:

```
GET /users/?limit=10&offset=20
```

**Success Response** (200 OK):

```json
{
  "limit": 10,
  "offset": 20,
  "total": 150,
  "users": [
    {
      "_id": "60b7c8b4f9b3c12345678901",
      "username": "user1",
      "email": "user1@example.com",
      "role": "user",
      "firstName": "Alice",
      "lastName": "Smith",
      "socialLinks": {
        "website": "https://alicesmith.com"
      },
      "createdAt": "2024-01-15T10:30:00.000Z",
      "updatedAt": "2024-01-20T15:45:00.000Z"
    },
    {
      "_id": "60b7c8b4f9b3c12345678902",
      "username": "user2",
      "email": "user2@example.com",
      "role": "admin",
      "firstName": "Bob",
      "lastName": "Johnson",
      "socialLinks": {},
      "createdAt": "2024-01-16T11:30:00.000Z",
      "updatedAt": "2024-01-18T14:20:00.000Z"
    }
  ]
}
```

**Error Responses**:

- `400` - Invalid query parameters
- `401` - Unauthorized
- `403` - Forbidden (user is not admin)
- `500` - Internal server error

---

### Get User by ID

Retrieves detailed information about a specific user.

**Endpoint**: `GET /users/{userId}`

**Authentication**: Required (Bearer token with admin role)

**Path Parameters**:

- `userId`: MongoDB ObjectId of the user

**Success Response** (200 OK):

```json
{
  "user": {
    "_id": "60b7c8b4f9b3c12345678901",
    "username": "johndoe",
    "email": "john@example.com",
    "role": "user",
    "firstName": "John",
    "lastName": "Doe",
    "socialLinks": {
      "website": "https://johndoe.com",
      "linkedin": "https://linkedin.com/in/johndoe"
    },
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-20T15:45:00.000Z"
  }
}
```

**Error Responses**:

- `400` - Invalid userId format
- `401` - Unauthorized
- `403` - Forbidden (user is not admin)
- `404` - User not found
- `500` - Internal server error

---

### Delete User by ID

Permanently deletes a specific user account and all associated data.

**Endpoint**: `DELETE /users/{userId}`

**Authentication**: Required (Bearer token with admin role)

**Path Parameters**:

- `userId`: MongoDB ObjectId of the user to delete

**Success Response** (204 No Content): Empty response body

**Error Responses**:

- `400` - Invalid userId format
- `401` - Unauthorized
- `403` - Forbidden (user is not admin)
- `404` - User not found
- `500` - Internal server error

**Warning**: This action is irreversible and will delete all user data and associated content.

## Code Examples

### Get Current User Profile

```javascript
const getCurrentUser = async (accessToken) => {
  const response = await fetch("/api/v1/users/current", {
    method: "GET",
    headers: {
      Authorization: `Bearer ${accessToken}`,
      "Content-Type": "application/json",
    },
  });

  if (response.ok) {
    const data = await response.json();
    return data.user;
  }
  throw new Error("Failed to fetch user profile");
};
```

### Update User Profile

```javascript
const updateProfile = async (accessToken, updates) => {
  const response = await fetch("/api/v1/users/current", {
    method: "PUT",
    headers: {
      Authorization: `Bearer ${accessToken}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify(updates),
  });

  if (response.ok) {
    const data = await response.json();
    return data.user;
  }

  const error = await response.json();
  throw new Error(error.message || "Failed to update profile");
};
```

### Admin: Get All Users with Pagination

```javascript
const getAllUsers = async (accessToken, limit = 20, offset = 0) => {
  const response = await fetch(
    `/api/v1/users/?limit=${limit}&offset=${offset}`,
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
  throw new Error("Failed to fetch users");
};
```

### Complete User Management Class

```javascript
class UserAPI {
  constructor(baseURL, accessToken) {
    this.baseURL = baseURL;
    this.accessToken = accessToken;
  }

  setAccessToken(token) {
    this.accessToken = token;
  }

  async getCurrentUser() {
    const response = await fetch(`${this.baseURL}/users/current`, {
      method: "GET",
      headers: this._getHeaders(),
    });
    return this._handleResponse(response);
  }

  async updateCurrentUser(updates) {
    const response = await fetch(`${this.baseURL}/users/current`, {
      method: "PUT",
      headers: this._getHeaders(),
      body: JSON.stringify(updates),
    });
    return this._handleResponse(response);
  }

  async deleteCurrentUser() {
    const response = await fetch(`${this.baseURL}/users/current`, {
      method: "DELETE",
      headers: this._getHeaders(),
    });

    if (response.status === 204) {
      return true;
    }
    throw new Error("Failed to delete user");
  }

  // Admin methods
  async getAllUsers(limit = 20, offset = 0) {
    const response = await fetch(
      `${this.baseURL}/users/?limit=${limit}&offset=${offset}`,
      {
        method: "GET",
        headers: this._getHeaders(),
      }
    );
    return this._handleResponse(response);
  }

  async getUserById(userId) {
    const response = await fetch(`${this.baseURL}/users/${userId}`, {
      method: "GET",
      headers: this._getHeaders(),
    });
    return this._handleResponse(response);
  }

  async deleteUserById(userId) {
    const response = await fetch(`${this.baseURL}/users/${userId}`, {
      method: "DELETE",
      headers: this._getHeaders(),
    });

    if (response.status === 204) {
      return true;
    }
    throw new Error("Failed to delete user");
  }

  _getHeaders() {
    return {
      Authorization: `Bearer ${this.accessToken}`,
      "Content-Type": "application/json",
    };
  }

  async _handleResponse(response) {
    if (response.ok) {
      const data = await response.json();
      return data;
    }

    const error = await response
      .json()
      .catch(() => ({ message: "Unknown error" }));
    throw new Error(error.message || `HTTP ${response.status}`);
  }
}
```

## User Roles and Permissions

### Regular User (`role: "user"`)

- ✅ View own profile (`GET /users/current`)
- ✅ Update own profile (`PUT /users/current`)
- ✅ Delete own account (`DELETE /users/current`)
- ❌ View other users' profiles
- ❌ Manage other user accounts

### Administrator (`role: "admin"`)

- ✅ All regular user permissions
- ✅ View all users (`GET /users/`)
- ✅ View any user profile (`GET /users/{userId}`)
- ✅ Delete any user account (`DELETE /users/{userId}`)
- ✅ Access user management features

## Best Practices

1. **Profile Updates**: Only send fields that need to be updated
2. **Password Changes**: Always require current password verification in production
3. **Data Validation**: Validate all input on the client side before sending
4. **Error Handling**: Implement proper error handling for all user operations
5. **Pagination**: Use appropriate page sizes when fetching user lists
6. **Security**: Never expose sensitive user data unnecessarily
