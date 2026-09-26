
# FastAPI Learning Notes

## Introduction

In software development, building a project involves more than writing application logic and deploying it to a server. In many cases, an application needs to communicate with other parts of a system, such as a mobile app, a website, a bot, an admin dashboard, or another software service.

Within a project's architecture, an API creates a communication layer between different systems, allowing applications to send and receive data without depending directly on one another's internal implementation details.

For example, imagine a cryptocurrency price prediction service. This service can accept inputs such as the cryptocurrency name and a time range, process the data, and return the predicted price. The same functionality can then be used by a mobile app, a bot, a dashboard, or any other system.

In the Python ecosystem, **FastAPI** is a modern framework for building APIs and backend services. Designed with a focus on **performance, simplicity, and modern web standards**, it helps developers create clean, scalable, and reliable APIs.

These notes provide a concise, structured overview of the core concepts of FastAPI, starting with foundational topics such as:

- **Path Parameters** and Query Parameters
- The **Request Body** and Pydantic models
- Data validation and error handling
- **Response Models**

They then move on to more advanced topics, including:

- Dependency Injection
- OAuth2 and JWT
- Middleware and CORS
- Database connections
- APIRouter for large projects
- Streaming and Server-Sent Events
- Background Tasks
- Testing and Debugging

These notes aim to give you a clear understanding of FastAPI.

## 1. Path & Query Parameters

In FastAPI, API routes can receive information from the URL path itself (Path) or from additional request parameters (Query). FastAPI uses **Type Hints** to check values (**Validation**) and automatically generate API documentation.

- A **Path Parameter** represents data that forms part of the URL path and is usually used to identify a resource. For example, in `/items/5`, `5` is the value of `item_id`.

- A **Query Parameter** is used for filtering, searching, or optional request settings, as in `/items?skip=10&limit=20`.

- Using **Type Hints**, FastAPI checks data types and returns an appropriate error if a value is invalid.

- An **Enum** can restrict the allowed values of a path parameter, and this restriction is also displayed in the **OpenAPI** documentation.

- Parameters without a default value are required, while parameters with a default value, such as `None` or `0`, are considered optional.

- An **Endpoint** can have multiple **Path Parameters** and **Query Parameters** at the same time and can use different HTTP methods depending on the operation:

| Method | Purpose |
| --- | --- |
| GET | Retrieve information |
| POST | Create or send information |
| PUT | Replace an entire resource |
| PATCH | Modify part of a resource |
| DELETE | Delete information |

## 2. Pydantic and Request Body

**Pydantic** models are used to receive data sent in the **Request Body**. FastAPI automatically converts JSON into a model, performs **Validation**, and handles errors related to invalid data.

- To define a **Request Body**, we create a class that inherits from `BaseModel` and use it as a function parameter. FastAPI automatically **parses** and **validates** the incoming JSON.

- **Body + Path + Query Parameters** can be used together. FastAPI determines which part of the request each parameter comes from based on its data type.

- `Annotated` is the modern way to add metadata and validation rules to parameters, such as:
  - `max_length` and `min_length` for length constraints
  - `pattern` for checking text format
  - `alias` for changing a parameter's name
  - `deprecated` for marking a parameter as deprecated
  - `title` and `description` for better documentation

- `include_in_schema=False` can hide a parameter from the **OpenAPI** documentation.

- A list can be used to receive multiple values in a Query, for example:

  ```python
  q: Annotated[list[str], Query()] = ["foo", "bar"]
  ```

- **Custom Validation**, using tools such as Pydantic's `AfterValidator`, allows you to define custom validation rules when FastAPI's built-in constraints are not sufficient.

## 3. Numeric Validation and Query Models

In addition to checking data types, you can define precise validation rules for numeric values and **Query** parameters. You can also manage several related **Query Parameters** in a **Pydantic** model to keep the code better organized and easier to maintain.

- The following rules can constrain numeric values in **Path and Query Parameters**:

  - `gt` for greater than (`>`)
  - `ge` for greater than or equal to (`≥`)
  - `lt` for less than (`<`)
  - `le` for less than or equal to (`≤`)

- Multiple **Query Parameters** can be grouped in a **Pydantic Model** and declared to FastAPI using `Query()`, improving code organization:

```python
params: Annotated[FilterParams, Query()]
```

- `Field()` can be used to validate each field within the model:

```python
limit: int = Field(gt=0, le=100)
```

- With the following setting:

```python
model_config = {"extra": "forbid"}
```

any extra **Query Parameter** that is not defined in the model is rejected and causes an error.

## 4. Advanced Body Handling and Nested Models

**Request Body** models are not limited to simple data. You can receive multiple models at once, build complex nested models, and define detailed examples for API documentation.

- An **Endpoint** can receive multiple **Body Parameters** at the same time. In this case, FastAPI treats each model as a separate key in the request JSON.

- `Body(embed=True)` allows even a single model to be placed under a specific key in the JSON.

- **Nested** models allow you to structure complex data. One model can contain another model:

```python
image: Image
```

or include lists and sets:

```python
images: list[Image]
tags: set[str]
```

- Models can have multiple levels of nesting, and FastAPI automatically **parses** and **validates** them.

- There are three main levels for displaying examples in **OpenAPI** documentation:

  - `Field(examples=...)` for examples of a specific field
  - `model_config = {"json_schema_extra": ...}` for examples at the model level
  - `Body(openapi_examples=...)` for defining several scenarios at the **Endpoint** level, such as valid or invalid input
 
  ## 5. Cookies and Headers

In addition to **Path**, **Query**, and **Body**, request information can also come from **Cookies** and **Headers**. This data can be handled directly or through **Pydantic** models.

- Receiving **Cookies** and **Headers** is similar to receiving **Query Parameters**, except that the data comes from the **Cookie** or **Header** part of the HTTP request:

  - `Cookie()` reads values stored in cookies
  - `Header()` receives information sent in headers

- FastAPI automatically extracts and validates header and cookie values.

- By default, `Header()` maps a Python variable name to a header name by converting `_` to `-`. Using:

```python
Header(convert_underscores=False)
```

disables this behavior.

- Several related headers or cookies can be grouped in a **Pydantic Model** to keep the code organized, similar to the approach used for a **Query Model**.

## 6. Response Models and Status Codes

A **Response Model** lets you control the structure and content of an API response. In addition to validating the output, it helps prevent sensitive information from being sent unintentionally and standardizes responses.

- Using `response_model` or declaring a function's return type, such as:

```python
-> UserOut
```

lets you specify which data should appear in the API response. FastAPI filters and **validates** the output.

- Sensitive fields such as `password` are excluded from the **Response** if they are not included in the output model, even when they are present in the returned data.

- To return special responses outside the usual model, you can use **Response** objects directly:

  - `JSONResponse` for sending custom JSON
  - `RedirectResponse` for redirecting the user to another route

- The HTTP status code can be specified with:

```python
status_code=201
```

However, using the `status` constants is more readable:

```python
status_code=status.HTTP_201_CREATED
```

- A correctly defined **Response Model** makes the API contract clearer and produces more accurate **OpenAPI** documentation.

## 7. Forms and File Uploads

In addition to receiving **JSON** data, you can receive information from HTML forms and uploaded files. This is useful when building APIs that interact with browsers or files.

- `Form()` can receive data from requests with the following content type:

```text
application/x-www-form-urlencoded
```

such as traditional web **Login** forms.

- There are two main approaches to receiving files:

  - `File()` receives the file contents directly as `bytes` and is suitable for small files.

  - `UploadFile` is suitable for larger files because it uses **streaming** and consumes less memory.

- `File()` or `UploadFile` can be used alongside `Form()` to receive form data and a file together in a single request.

- File uploads require the request to support `multipart/form-data`.

## 8. Error Handling

Errors are part of API design. You can generate standard errors or create custom **Exception Handlers** to control how error responses are presented and formatted.

- Using:

```python
HTTPException(status_code=..., detail=...)
```

makes it easy to return an HTTP error with a specific status code and message.

- `HTTPException` is useful for common errors within **Endpoints**, such as a resource not being found or unauthorized access.

- For greater control, you can define **custom Exception Handlers**, for example:

  - `StarletteHTTPException` for handling HTTP errors
  - `RequestValidationError` for handling input data validation errors

- Custom **Handlers** can change the format of an error response. For example, instead of the default JSON, the response can be returned as **plain text** or in a custom structure.

## 9. Metadata, Documentation, and Partial Updates

In addition to building **Endpoints**, FastAPI lets you improve API documentation for developers and handle partial data changes, such as **PATCH** operations, by updating only the parts the user has submitted.

- Using:

```python
tags=["items"]
```

lets you group endpoints in **Swagger UI**. A function's **docstring** is also displayed as that endpoint's **Description**.

- `jsonable_encoder` converts complex data, such as **Pydantic** models, `datetime` values, and other objects, into a structure that can be converted to **JSON**.

- When implementing partial updates with **PATCH**, it is best to change only the fields the user has submitted:

```python
item.model_dump(exclude_unset=True)
```

extracts only the submitted values.

- Then, using:

```python
item.model_copy(update=...)
```

creates a new copy of the model with the changes applied.

- This pattern prevents default values or fields the user has not submitted from unintentionally replacing existing data.

## 10. Dependency Injection

**Dependency Injection** lets you define shared logic, such as authentication, database connections, or parameter processing, once and reuse it across multiple **Endpoints** without duplicating code.

- A regular function that accepts input and returns a value can be injected using:

```python
Depends(func)
```

into an **Endpoint**.

- Shared tasks such as:

  - Pagination
  - Authentication
  - Permission checks
  - Database session management

  can be implemented using dependencies.

- A dependency is not limited to a function. It can:

  - Be a **class**
  - Have its own dependency, known as a **Sub-dependency**
  - Be defined at the **Router** level or for the entire application as a **Global Dependency**

- Using `yield`, you can run code before and after an **Endpoint** executes, for example:

  - Before the request: create a database connection
  - After the response: close the session and release resources

- This structure improves separation of concerns and makes the project easier to maintain.

## 11. Security — OAuth2 + JWT

Authentication systems such as **OAuth2 and JWT** can be used to protect APIs. The user logs in once, receives a token, and sends it with subsequent requests so their identity can be verified.

- With:

```python
OAuth2PasswordBearer(tokenUrl="token")
```

FastAPI specifies that the token is received from the following Header:

```text
Authorization: Bearer <token>
```

- User passwords must not be stored as **Plain Text**. They should be hashed with tools such as `pwdlib` and `PasswordHash`, then checked during login.

- An Endpoint such as `/token` uses:

```python
OAuth2PasswordRequestForm
```

to receive login information and create a **JWT Token** when authentication succeeds.

- A JWT usually contains information such as an expiration time (`exp`) so that the token remains valid only for a limited period.

- Dependencies such as:

  - `get_current_user` to **decode** the token and find the user
  - `get_current_active_user` to check additional user conditions, such as whether the account is active

  are used.

- A simpler approach to access control is to check a custom Header such as:

```text
X-Token
```

However, real authentication systems generally use **OAuth2 + JWT**.

## 12. Middleware and CORS

**Middleware** can apply logic to all requests and responses. **CORS** specifies which clients from other domains are allowed to communicate with the API.

- With:

```python
@app.middleware("http")
```

you can create a general **Middleware** that runs before and after each **Request** is processed, for example:

  - Measuring request processing time
  - Adding a Header to the response
  - Recording request Logs

- With:

```python
app.add_middleware(...)
```

you can add different built-in or custom Middleware components to the application.

- Middleware execution order matters because requests and responses pass through the Middleware chain.

- `CORSMiddleware` controls requests sent from a different **Origin**, such as when the **Frontend** runs on one domain and the **Backend** runs on another.

- Important CORS settings include:

  - `allow_origins` to specify permitted domains
  - `allow_methods` to specify permitted HTTP methods
  - `allow_headers` to specify acceptable Headers

- Correctly configuring **CORS** ensures that only trusted clients can communicate with the API.

## 13. Relational Databases (SQLModel)

**SQLModel** can be used to work with relational databases. It combines **Pydantic** models and an **ORM**. Separating the database, API input, and API output models makes the data structure safer and easier to manage.

- A base model such as:

```python
HeroBase
```

can hold shared fields, and different models can inherit from it.

- The model:

```python
Hero(HeroBase, table=True)
```

represents the actual database table, and **SQLModel** recognizes it as a **Table**.

- Dedicated API models are used to separate responsibilities:

  - `HeroCreate` for data received when creating a record.
  - `HeroUpdate` for changing existing information.
  - `HeroPublic` for data displayed in API responses.

- This separation prevents internal database fields such as:

```python
secret_name
```

from being sent accidentally in the API response.

- This pattern follows the same output-control concept as `response_model`, while working with an **ORM** to keep the database and API structures separate and clean.

## 14. Organizing Large Projects (APIRouter)

To keep the main file from becoming cluttered, **Endpoints** can be divided into smaller sections using **APIRouter**. Each section can have its own routes, dependencies, and settings.

- `APIRouter()` works like a small version of **FastAPI** and allows you to organize endpoints in separate files.

- After defining a Router, use:

```python
app.include_router(
    router,
    prefix="...",
    tags=["..."],
    dependencies=[...]
)
```

to add it to the main application.

- With `prefix`, you can define a shared path for a group of endpoints, such as:

```text
/users
/admin
```

- With `tags`, you can group endpoints in **Swagger UI**.

- With `dependencies`, you can apply shared dependencies to the entire Router. For example, all `/admin` routes can require authentication.

- In large projects, sections such as users, products, or authentication usually have separate Routers, making the project easier to maintain and extend.

- In `pyproject.toml`, the following setting:

```toml
[tool.fastapi]
entrypoint = "app.main:app"
```

specifies the location of the main **FastAPI** object so FastAPI tools know where to run the application.

## 15. Streaming and Real-Time Communication (Streaming / SSE)

When sending long or continuous data, you do not need to wait for the entire response to be ready. **Streaming** sends data in stages, while **SSE** **Pushes** real-time events to the client.

- When an `async` function produces data with `yield`, FastAPI can send the response as a **Stream**, meaning the data is sent gradually rather than all at once.

- This approach is useful for:

  - Displaying the output of long-running processes
  - Sending real-time logs
  - Streaming responses generated by AI models

  These are typical use cases.

- **SSE (Server-Sent Events)** sends one-way events from the server to the client, such as gradually sending the tokens of a chat response.

- `EventSourceResponse` can create an **SSE** connection and continuously send events.

- `ServerSentEvent` lets you configure information for each event:

  - `event` for the event type
  - `id` for identifying the event
  - `retry` for specifying the reconnection retry time

- With the following Header:

```text
Last-Event-ID
```

If the connection is interrupted, the client can resume the stream from the last event.

- **SSE** is not limited to `GET` and can also be used for scenarios such as streaming the response to a `POST` request.

  ## 16. Background Tasks

Some tasks do not need to finish before responding to the user. **Background Tasks** can move these operations until after the response is sent, reducing the user's waiting time.

- Using:

```python
BackgroundTasks.add_task(func, ...)
```

you can register a task to run after the **Response** is sent to the user.

- This capability is suitable for tasks such as:

  - Sending email
  - Recording Logs
  - Lightweight background processing

  This is useful because the user does not have to wait for these tasks to finish.

- A **Background Task** can be used directly inside an **Endpoint** or received and managed through a **Dependency**.

- This approach is suitable for short, nonessential tasks. For heavy, long-running processing, it is generally better to use a queue system such as **Celery** or a similar tool.

  ## 17. Testing and Debugging

**Endpoints** can be tested without actually running a server. During development, the application can also be run in a way that allows you to use a **Debugger** and set **breakpoints** to find errors.

- Using:

```python
TestClient
```

the `fastapi.testclient` package can be used to write automated tests for **Endpoints**. This approach simulates HTTP requests without requiring a real server to run.

- Tests can check things such as:

  - Whether the **Response** is correct
  - HTTP status codes
  - Input validation
  - Endpoint behavior

  These tests are used to verify them.

- For local debugging, **Uvicorn** can be run directly inside the file:

```python
if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

- This approach lets you run the file directly, set a **breakpoint**, and inspect the code line by line in the **Debugger**.

  ## Swagger UI

**Swagger UI** is an interactive user interface for viewing and testing APIs based on the **OpenAPI** standard. It displays API documentation as a web page and lets you send requests to **Endpoints** without requiring separate tools.

**Result:**

- FastAPI automatically generates **OpenAPI** documentation and provides **Swagger UI** as the default interface at:

```text
/docs
```

FastAPI makes this interface available there.

- In **Swagger UI**, you can:

  - View the list of endpoints.
  - Inspect the API's input and output parameters.
  - Run `GET`, `POST`, `PUT`, `DELETE`, and other requests directly.
  - View responses and HTTP status codes.

- This tool is highly useful for **Frontend** and **Backend** developers and for API testing because it lets you inspect API behavior without writing a separate client.

**Example:**

When a FastAPI project is running, **Swagger UI** documentation is usually available at:

```text
http://127.0.0.1:8000/docs
```

In addition to **Swagger UI**, FastAPI automatically provides alternative **ReDoc** documentation at:

```text
http://127.0.0.1:8000/redoc
```

FastAPI provides this documentation there.
