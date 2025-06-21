# Introduction

Welcome to the Blog API documentation! This API provides functionalities for managing blog posts, users, comments, and likes.

## Base URL

All API endpoints are relative to the following base URL: /api/v1

For example, the user registration endpoint is `/api/v1/auth/register`.

## Overview

- **API Version:** 1.0.0
- **Authentication:** Uses JWT Bearer tokens for access and JWT refresh tokens (via HTTP-only cookies) for session renewal.
- **Authorization:** Role-based access control ('admin', 'user'). Specific roles are required for certain endpoints.
- **Rate Limiting:** Applied globally (60 requests per minute per IP). Exceeding the limit returns a `429 Too Many Requests` error.
- **Input Validation:** Uses `express-validator`. Invalid requests return detailed `400 Bad Request` errors.
- **Content Format:** Primarily JSON (`application/json`). File uploads use `multipart/form-data`.
