---
name: api-docs-generator
description: "Generate OpenAPI/Swagger specs from existing API code"
version: "1.0.0"
tags: [documentation, api, openapi, swagger]
createdBy: "built-in"
status: "active"
---

# API Documentation Generator

## Activation
When the user asks to generate API docs, create an OpenAPI spec, or document endpoints.

## Workflow

### Step 1: Discover Endpoints
1. Find route files (routes/, controllers/, app.ts, server.ts)
2. Extract all HTTP method + path combinations
3. Identify request parameters (path, query, body, headers)
4. Identify response shapes from return statements or types
5. Find authentication requirements from middleware

### Step 2: Analyze Each Endpoint
For each endpoint, determine:
- HTTP method and path
- Request body schema (from validation or types)
- Query parameters and their types
- Path parameters
- Response schema (from return types or actual responses)
- Authentication requirement
- Rate limiting
- Error responses

### Step 3: Generate OpenAPI 3.0 Spec

```yaml
openapi: 3.0.3
info:
  title: [Project Name] API
  version: 1.0.0
  description: [Generated from source code analysis]

servers:
  - url: http://localhost:3000/api/v1
    description: Local development

paths:
  /resource:
    get:
      summary: List resources
      operationId: listResources
      tags: [Resources]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
      responses:
        "200":
          description: Success
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ResourceList"

components:
  schemas:
    Resource:
      type: object
      properties:
        id:
          type: string
          format: uuid
      required: [id]

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

### Step 4: Validate the Spec
- Ensure all $ref references resolve
- Verify required fields are listed
- Check response codes are appropriate
- Validate example values match schemas

### Step 5: Output Options
1. **openapi.yaml** — Full OpenAPI spec file
2. **Inline JSDoc** — Add @swagger comments to route files
3. **Markdown** — Human-readable API docs table

## Rules
- Generate from actual code, not assumptions
- Include error response schemas (400, 401, 404, 500)
- Document authentication requirements per endpoint
- Include request/response examples with realistic data
- Follow OpenAPI 3.0 spec strictly for tool compatibility
- Save to docs/ directory unless user specifies otherwise
