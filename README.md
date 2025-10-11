# mint-oapi-codegen

A fork of [oapi-codegen](https://github.com/oapi-codegen/oapi-codegen) that generates [Mint language](https://mint-lang.com) client code from OpenAPI 3.0 specifications.

## Overview

`mint-oapi-codegen` converts OpenAPI specifications into idiomatic Mint code, including:
- **Type definitions** for all schemas (records, enums, etc.)
- **HTTP client code** as Mint Providers with async/await Promise-based methods
- **Type-safe API calls** with proper error handling

This tool helps you reduce boilerplate when integrating Mint applications with REST APIs, allowing you to focus on business logic instead of HTTP plumbing.

## Features

✅ **Generates Mint Types** - Proper Mint record types with Maybe for optional fields  
✅ **HTTP Client Provider** - Ready-to-use Provider with all API operations  
✅ **Query Parameters** - Automatic URL encoding and optional parameter handling  
✅ **Request Bodies** - JSON encoding with proper type checking  
✅ **Path Parameters** - Type-safe path interpolation  
✅ **Header Parameters** - Both required and optional headers supported  
✅ **Error Handling** - Result types for safe error handling  
✅ **Enum Support** - Generates Mint modules for enum values

## Installation

Requires Go 1.21 or later.

```bash
# Clone the repository
git clone https://github.com/yourusername/mint-oapi-codegen.git
cd mint-oapi-codegen

# Build the binary
go build -o mint-oapi-codegen ./cmd/oapi-codegen

# Or run directly
go run ./cmd/oapi-codegen/oapi-codegen.go -config cfg.yaml api.yaml
```

## Quick Start

### 1. Create an OpenAPI Specification

Create an `api.yaml` file:

```yaml
openapi: "3.0.0"
info:
  version: 1.0.0
  title: Blog API
servers:
  - url: https://api.example.com/v1
paths:
  /users/{userId}:
    get:
      operationId: getUser
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/User"
components:
  schemas:
    User:
      type: object
      required:
        - id
        - username
        - email
      properties:
        id:
          type: integer
        username:
          type: string
        email:
          type: string
        bio:
          type: string
```

### 2. Create a Configuration File

Create a `cfg.yaml` file:

```yaml
package: BlogApi
output: blog-api.gen.mint
generate:
  models: true
  client: true
```

### 3. Generate Mint Code

```bash
go run ./cmd/oapi-codegen/oapi-codegen.go -config cfg.yaml api.yaml
```

### 4. Use in Your Mint Application

```mint
component Main {
  fun loadUser : Promise(Void) {
    let response = await BlogApi.getUser(1)
    
    case response {
      Result.Ok(user) => {
        Debug.log("User: " + user.username)
        void
      }
      
      Result.Err(error) => {
        Debug.log("Error loading user")
        void
      }
    }
  }
  
  fun render : Html {
    <button onClick={loadUser}>
      "Load User"
    </button>
  }
}
```

## Generated Code Structure

### Types

The generator creates proper Mint record types:

```mint
type User {
  bio : Maybe(String),
  email : String,
  id : Number,
  username : String
}
```

### Provider

The generator creates a Provider with HTTP client methods:

```mint
provider BlogApi : BlogApiSubscription {
  state baseUrl : String = "https://api.example.com/v1"
  
  fun update {
    void
  }

  fun getUser(userId : Number) : Promise(Result(Http.ErrorResponse, User)) {
    let url = "#{baseUrl}/users/#{userId}"
    
    let request = Http.get(url)
    
    let Ok(httpResponse) =
      await Http.send(request) or return Result.Err({...})
    
    let JSON(object) =
      httpResponse.body or return Result.Err({...})
    
    decode object as User
    |> Result.mapError((error : Object.Error) : Http.ErrorResponse {...})
  }
}
```

### Parameters

Query and header parameters are handled automatically:

```mint
// Parameter type
type ListPostsParams {
  limit : Maybe(Number),
  offset : Maybe(Number)
}

// Usage in function
fun listPosts(params : ListPostsParams) : Promise(Result(Http.ErrorResponse, Array(Post))) {
  let queryParams =
    [
      Maybe.map(params.limit, (value : Number) : Tuple(String, String) { 
        {"limit", Number.toString(value)} 
      }),
      Maybe.map(params.offset, (value : Number) : Tuple(String, String) { 
        {"offset", Number.toString(value)} 
      })
    ]
    |> Array.compact()
  
  // ... builds query string and makes request
}
```

## Configuration Options

The configuration file (`cfg.yaml`) supports:

| Option | Description | Example |
|--------|-------------|---------|
| `package` | Name of the generated Provider | `BlogApi` |
| `output` | Output file path | `blog-api.gen.mint` |
| `generate.models` | Generate type definitions | `true` |
| `generate.client` | Generate HTTP client Provider | `true` |

## Type Mappings

OpenAPI types are mapped to Mint as follows:

| OpenAPI Type | Format | Mint Type |
|--------------|--------|-----------|
| `string` | - | `String` |
| `string` | `email` | `String` |
| `string` | `date` | `String` (ISO 8601) |
| `string` | `date-time` | `String` (ISO 8601) |
| `string` | `uuid` | `String` |
| `string` | `byte` | `String` (base64) |
| `string` | `binary` | `String` |
| `integer` | - | `Number` |
| `integer` | `int32` | `Number` |
| `integer` | `int64` | `Number` |
| `number` | - | `Number` |
| `number` | `float` | `Number` |
| `number` | `double` | `Number` |
| `boolean` | - | `Bool` |
| `array` | - | `Array(T)` |
| `object` | - | Named type |

Optional fields use `Maybe(T)`.

## Examples

See the [examples/mint-client](./examples/mint-client) directory for a complete working example with:
- Sample OpenAPI specification
- Configuration file
- Generated Mint code
- Usage examples

## Supported Features

### ✅ Currently Supported

- [x] Schema types (objects, arrays, primitives)
- [x] Path parameters
- [x] Query parameters (required and optional)
- [x] Header parameters (required and optional)
- [x] Request bodies (JSON)
- [x] Response bodies (JSON)
- [x] Enums
- [x] References (`$ref`)
- [x] Required vs optional fields
- [x] Inline object extraction
- [x] All HTTP methods (GET, POST, PUT, DELETE, PATCH)

### 🚧 Planned Features

- [ ] Multiple response status codes
- [ ] Multiple content types (XML, form data, etc.)
- [ ] Response headers in return types
- [ ] Cookie parameters
- [ ] File uploads/downloads
- [ ] Authentication schemes (OAuth, API keys, etc.)

### ❌ Not Supported

- Server-side code generation
- oneOf/anyOf/allOf unions
- Webhooks
- Callbacks

## Differences from Original oapi-codegen

This fork:
- Generates **Mint** code instead of Go
- Focuses on **client-side** code only (no server generation)
- Uses Mint's **Promise** and **Result** types for async operations
- Uses Mint's **Provider** pattern for state management
- Generates idiomatic Mint with proper casing and naming conventions

## Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## Known Issues

See [TESTING-SUMMARY.md](./examples/mint-client/TESTING-SUMMARY.md) for known limitations and testing results.

## License

Apache 2.0 - See [LICENSE](./LICENSE) file for details.

This is a fork of [oapi-codegen](https://github.com/oapi-codegen/oapi-codegen) by DeepMap, Inc.

## Credits

This project is based on [oapi-codegen](https://github.com/oapi-codegen/oapi-codegen) and uses its core parsing and code generation infrastructure, adapted for the Mint language.

Special thanks to the oapi-codegen maintainers and contributors for creating such a well-structured and extensible codebase.
