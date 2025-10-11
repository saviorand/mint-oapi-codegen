# Mint Client Example

This example demonstrates generating a Mint language client from an OpenAPI specification.

## Files

- `api.yaml` - OpenAPI 3.0 specification for a simple blog API
- `cfg.yaml` - Configuration file for oapi-codegen
- `blog-api.gen.mint` - Generated Mint code (created by running the generator)

## Generating the Mint Client

From this directory, run:

```bash
go run ../../cmd/oapi-codegen/oapi-codegen.go -config cfg.yaml api.yaml
```

This will generate `blog-api.gen.mint` containing:
- Mint type definitions for all schemas
- A `BlogApi` Provider with methods for each API operation

## Generated Code Structure

The generated file includes:

### Types
```mint
type User {
  bio : Maybe(String),
  email : String,
  id : Number,
  username : String
}

type Post {
  authorId : Number,
  content : String,
  id : Number,
  published : Maybe(Bool),
  title : String
}
```

### Provider
```mint
// Subscription type (required for providers, unused for HTTP client)
type BlogApiSubscription {
  dummy : String
}

provider BlogApi : BlogApiSubscription {
  state baseUrl : String = "https://api.example.com/v1"
  
  fun getUser(userId : Number) : Promise(Result(Http.ErrorResponse, User)) {
    // ...
  }
  
  fun listPosts(params : ListPostsParams) : Promise(Result(Http.ErrorResponse, Array(Post))) {
    // ...
  }
}
```

## Using the Generated Client

In your Mint application, you can use the generated client like this:

```mint
component Main {
  fun loadUser : Promise(Void) {
    let response = await BlogApi.getUser(1)
    
    case response {
      Result.Ok(user) => {
        Debug.log(user.username)
        void
      }
      Result.Err(error) => {
        Debug.log("Error loading user:")
        Debug.log(error)
        void
      }
    }
  }
  
  fun render : Html {
    <div>
      <button onClick={loadUser}>
        "Load User"
      </button>
    </div>
  }
}
```

## API Endpoints

The example API includes:

- `GET /users/{userId}` - Get a user by ID
- `GET /posts` - List all posts (with optional limit/offset)
- `POST /posts` - Create a new post
- `GET /posts/{postId}` - Get a post by ID

## Configuration

The `cfg.yaml` file specifies:
- `package: BlogApi` - The name of the generated Provider
- `output: blog-api.gen.mint` - Output file name
- `generate.models: true` - Generate type definitions
- `generate.client: true` - Generate client Provider

## Customizing

You can modify `api.yaml` to add more endpoints, change schemas, or adjust parameters. Then regenerate the client to see your changes reflected in the Mint code.

