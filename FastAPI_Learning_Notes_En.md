# FastAPI Learning Notes

## Introduction

In software development, building a project is not only about writing the application logic and deploying it to a server. In many cases, an application needs to communicate with other parts of a system, such as a mobile application, a website, a bot, a management dashboard, or another software service.

In a project architecture, an API creates a communication layer between different systems and allows applications to send and receive data without being directly dependent on each other's internal implementation details.

For example, imagine a cryptocurrency price prediction service. This service can receive inputs such as the cryptocurrency name and a time range, process the data, and return a predicted price. The same capability can then be used by a mobile application, a bot, a dashboard, or any other system.

In the Python ecosystem, **FastAPI** is a modern web framework for building APIs and backend services. It is designed with a focus on **performance, simplicity, and modern web standards**, helping developers create clean, scalable, and reliable APIs.

In these notes, the core concepts of FastAPI are reviewed in a concise and structured way, starting from fundamental topics such as:

- Path Parameters and Query Parameters
- Request Body and Pydantic Models
- Data Validation and Error Handling
- Response Models

And continuing with more advanced topics such as:

- Dependency Injection
- OAuth2 and JWT
- Middleware and CORS
- Database Integration
- APIRouter for large projects
- Streaming and Server-Sent Events
- Background Tasks
- Testing and Debugging

The goal of these notes is to provide a clear understanding of FastAPI.
