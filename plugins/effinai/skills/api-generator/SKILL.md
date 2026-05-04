---
name: api-generator
description: "Generate REST or GraphQL APIs from specifications or requirements"
version: "1.0.0"
tags: [engineering, api, rest, graphql, generation]
createdBy: "built-in"
status: "active"
---

# API Generator

## Activation
When the user asks to create an API, generate endpoints, or scaffold API routes.

## Workflow

### Step 1: Gather Specification
From the user, determine:
1. **API style**: REST or GraphQL
2. **Resources/entities**: What data does the API manage?
3. **Operations**: CRUD? Custom actions?
4. **Authentication**: JWT, API key, OAuth, none?
5. **Framework**: Express, Fastify, NestJS, Django, FastAPI, etc.

If a spec file exists (OpenAPI YAML, GraphQL schema), read it.

### Step 2: Design the API

**For REST:**
Follow RESTful conventions:
- `GET /resources` — List (with pagination, filtering, sorting)
- `GET /resources/:id` — Get single resource
- `POST /resources` — Create
- `PUT /resources/:id` — Full update
- `PATCH /resources/:id` — Partial update
- `DELETE /resources/:id` — Delete

Design principles:
- Use plural nouns for resource names
- Use HTTP status codes correctly (201 Created, 204 No Content, 404, 422)
- Version the API (`/api/v1/`)
- Use consistent error response format
- Include pagination metadata in list responses

**For GraphQL:**
- Define types with proper nullability
- Use input types for mutations
- Implement connection-based pagination (Relay spec)
- Add proper error handling with extensions

### Step 3: Generate Code Structure

```
src/
  routes/           # or controllers/
    [resource].ts   # Route handlers
  middleware/
    auth.ts         # Authentication middleware
    validate.ts     # Request validation
    error.ts        # Error handling
  schemas/
    [resource].ts   # Validation schemas (Zod/Joi)
  services/
    [resource].ts   # Business logic
  models/
    [resource].ts   # Data models/types
```

### Step 4: Generate for Each Resource
1. **Route/Controller** — HTTP handlers with proper status codes
2. **Validation schema** — Input validation with Zod or Joi
3. **Service layer** — Business logic separated from HTTP
4. **Type definitions** — Request/response TypeScript types
5. **Error handling** — Consistent error responses

### Step 5: Add Cross-Cutting Concerns
- Request validation middleware
- Authentication middleware
- Rate limiting configuration
- CORS configuration
- Request logging
- Error handling middleware with proper HTTP codes

### Step 6: Generate OpenAPI Spec
If REST, generate an OpenAPI 3.0 spec that documents all endpoints.

## Output
- Generated route files with full CRUD operations
- Validation schemas for all inputs
- Middleware for auth and error handling
- OpenAPI spec or GraphQL schema file
- Example requests for testing (curl or httpie format)

## Rules
- Always include input validation — never trust client data
- Separate route handlers from business logic
- Use proper HTTP status codes (not 200 for everything)
- Include pagination for all list endpoints
- Add rate limiting to authentication endpoints
- Generate TypeScript types for request/response bodies
