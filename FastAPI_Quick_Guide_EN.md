# Introduction

In software development, building a project is not only about writing
the application logic and deploying it to a server. In many cases, an
application needs to communicate with other parts of a system, such as a
mobile application, a website, a bot, a management dashboard, or another
software service.

This is where **APIs** become an important part of the application
architecture.

An API creates a communication layer between different systems and
allows applications to send and receive data without being directly
dependent on each other's internal implementation details.

For example, imagine a cryptocurrency price prediction service. This
service can receive inputs such as the cryptocurrency name and a time
range, process the data, and return a predicted price. The same
capability can then be used by a mobile application, a bot, a dashboard,
or any other system.

In the Python ecosystem, **FastAPI** is a modern web framework for
building APIs and backend services. It is designed with a focus on
performance, simplicity, and modern web standards, helping developers
create APIs that are clean, scalable, and reliable.

One of the key features of FastAPI is its use of Python capabilities
such as **Type Hints**. With this feature, FastAPI can understand input
and output data types, automatically validate data, and generate API
documentation without complex configuration.

FastAPI also provides important tools and features such as:

-   **Pydantic** for data management and validation
-   **Dependency Injection** for better code organization
-   **Async Programming** for handling concurrent requests
-   **OpenAPI** for automatic API documentation generation

In this **Quick Guide**, the core concepts of FastAPI are reviewed in a
concise and structured way, starting from fundamental topics such as:

-   Path and Query Parameters
-   Request Body and Pydantic Models
-   Validation and Error Handling
-   Response Models

And continuing with more advanced topics such as:

-   Dependency Injection
-   OAuth2 and JWT
-   Middleware and CORS
-   Database Integration
-   APIRouter for large projects
-   Streaming and Server-Sent Events
-   Background Tasks
-   Testing and Debugging

The goal of this guide is not only to introduce FastAPI features, but
also to provide a clear understanding of where each feature fits in a
modern application architecture and how FastAPI helps build scalable
APIs and backend services.

------------------------------------------------------------------------

# FastAPI Learning Guide

A practical overview of the most important FastAPI concepts, patterns,
and features.

## 1. Path and Query Parameters

### Path Parameters

Path parameters are declared inside the URL using `{name}` and their
type is defined with Python type hints.

``` python
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

FastAPI validates and converts the value automatically.

### Path Parameters with Enum

When a path value must come from a fixed list, use an Enum:

``` python
class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
```

FastAPI automatically validates values and documents them in OpenAPI.

### Query Parameters

Parameters not included in the path are treated as query parameters:

``` python
@app.get("/items/")
async def read_items(skip: int = 0):
    return {"skip": skip}
```

Optional parameters use a default value:

``` python
q: str | None = None
```

Without a default value, they are required.

### HTTP Methods

  Method   Usage
  -------- ----------------------------
  GET      Retrieve information
  POST     Create or send information
  PUT      Replace completely
  PATCH    Partial update
  DELETE   Remove data

------------------------------------------------------------------------

## 2. Pydantic and Request Body

Request bodies are usually defined with Pydantic models.

``` python
class Item(BaseModel):
    name: str
    price: float
```

FastAPI automatically: - Parses JSON - Validates data - Generates
OpenAPI documentation

You can combine: - Request body - Path parameters - Query parameters

FastAPI identifies each source from the parameter type.

Modern parameter metadata uses `Annotated`:

``` python
q: Annotated[str, Query(max_length=50)]
```

Useful options: - `max_length` - `min_length` - `pattern` - `alias` -
`deprecated` - `description`

------------------------------------------------------------------------

## 3. Numeric Validation and Query Models

Numeric constraints:

-   `ge`: greater than or equal
-   `gt`: greater than
-   `le`: less than or equal
-   `lt`: less than

Example:

``` python
age: int = Field(gt=0, le=120)
```

Multiple query parameters can be grouped:

``` python
filter: Annotated[FilterParams, Query()]
```

To reject unknown parameters:

``` python
model_config = {"extra": "forbid"}
```

------------------------------------------------------------------------

## 4. Advanced Body Features and Nested Models

Multiple body models create separate JSON keys.

`Body(embed=True)` can wrap a single model:

``` python
item: Item = Body(embed=True)
```

Nested models are supported:

``` python
image: Image
images: list[Image]
tags: set[str]
```

Examples can be added at different levels:

-   `Field(examples=...)`
-   `model_config.json_schema_extra`
-   `Body(openapi_examples=...)`

------------------------------------------------------------------------

## 5. Cookies and Headers

Cookies and headers work similarly to query parameters:

``` python
token: str = Cookie()
user_agent: str = Header()
```

`Header(convert_underscores=False)` allows custom header naming.

Cookies and headers can also be grouped using Pydantic models.

------------------------------------------------------------------------

## 6. Response Models and Status Codes

Response models control returned data:

``` python
@app.get("/users/{id}", response_model=UserOut)
```

Sensitive fields can be removed automatically.

Raw responses:

``` python
JSONResponse()
RedirectResponse()
```

Status codes:

``` python
status_code=status.HTTP_201_CREATED
```

------------------------------------------------------------------------

## 7. Forms and File Uploads

Forms:

``` python
username: str = Form()
```

Files:

``` python
file: bytes = File()
```

For large files:

``` python
file: UploadFile
```

Files and forms can be combined.

------------------------------------------------------------------------

## 8. Error Handling

Simple errors:

``` python
raise HTTPException(
    status_code=404,
    detail="Not found"
)
```

Custom handlers can control validation and exception formats.

------------------------------------------------------------------------

## 9. Documentation and Partial Updates

Endpoint grouping:

``` python
tags=["items"]
```

Function docstrings become API descriptions.

Convert complex objects:

``` python
jsonable_encoder(data)
```

PATCH pattern:

``` python
item.model_dump(exclude_unset=True)
```

Only fields sent by the user are updated.

------------------------------------------------------------------------

## 10. Dependency Injection

Dependencies provide reusable logic:

``` python
Depends(function)
```

Common uses: - Authentication - Pagination - Database sessions

Dependencies can: - Be functions or classes - Have sub-dependencies - Be
global or router-level - Use `yield` for cleanup

------------------------------------------------------------------------

## 11. OAuth2 and JWT Security

Common FastAPI security flow:

1.  `OAuth2PasswordBearer` reads Bearer tokens.
2.  Passwords are hashed using tools such as `pwdlib`.
3.  `/token` receives username and password.
4.  JWT tokens are created with expiration (`exp`).
5.  Dependencies decode and validate users.

A simpler alternative is checking custom headers like `X-Token`.

------------------------------------------------------------------------

## 12. Middleware and CORS

Middleware runs before and after requests.

Example:

``` python
@app.middleware("http")
async def middleware(request, call_next):
    response = await call_next(request)
    return response
```

CORS allows frontend applications on different origins:

``` python
CORSMiddleware(
    allow_origins=[...]
)
```

------------------------------------------------------------------------

## 13. Relational Databases with SQLModel

Common model separation:

-   `HeroBase`: shared fields
-   `Hero`: database table
-   `HeroPublic`: API response
-   `HeroCreate`: creation input
-   `HeroUpdate`: update input

This prevents exposing internal fields.

------------------------------------------------------------------------

## 14. Large Project Organization with APIRouter

`APIRouter` organizes applications:

``` python
router = APIRouter()
```

Include routers:

``` python
app.include_router(
    router,
    prefix="/admin",
    tags=["admin"]
)
```

Dependencies can be applied to complete routers.

------------------------------------------------------------------------

## 15. Streaming and Server-Sent Events

Streaming responses use async generators:

``` python
async def stream():
    yield data
```

SSE allows continuous server-to-client events.

Useful for: - Chat streaming - Notifications - Live updates

`ServerSentEvent` supports: - event - id - retry

`Last-Event-ID` can continue interrupted streams.

------------------------------------------------------------------------

## 16. Background Tasks

Background tasks run after the response is sent.

Example:

``` python
background_tasks.add_task(send_email)
```

Useful for: - Logging - Email sending - Small background jobs

Dependencies can also use them.

------------------------------------------------------------------------

## 17. Testing and Debugging

Testing:

``` python
from fastapi.testclient import TestClient
```

`TestClient` allows endpoint testing without running a real server.

Debugging:

``` python
if __name__ == "__main__":
    uvicorn.run(
        app,
        host="0.0.0.0",
        port=8000
    )
```

This allows running the application directly and using breakpoints.

------------------------------------------------------------------------

# Summary

FastAPI combines: - Type hints - Pydantic validation - Automatic
documentation - Dependency injection - Security tools - Database
patterns - Async capabilities

These features make it suitable for building modern, scalable APIs
quickly.
