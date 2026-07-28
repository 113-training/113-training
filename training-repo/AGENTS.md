# OrderHub — Project Memory

## Project Overview

An internal order management system for the company. Sales staff can create and search orders, as well as manage products and customers.

This is an internal-use system backed by a single SQL Server database. There is no need to design for multi-tenancy or high-concurrency architecture.

## Technology Stack

* .NET 8 / ASP.NET Core MVC with Razor Views
* Entity Framework Core 8 with SQL Server
* Testing framework: xUnit

## Architecture and Conventions

* The solution follows a three-layer architecture:

  * `OrderHub.Web`: Controllers, Views, and ViewModels
  * `OrderHub.Core`: Domain models, services, and interfaces
  * `OrderHub.Infrastructure`: Repositories and EF Core migrations
* Keep controllers thin. Controllers should only pass requests to services and handle service results. All business logic must be implemented in services under `OrderHub.Core`.
* Only repositories may access `DbContext`. Controllers and services must not use EF Core directly.
* Services return `ServiceResult<T>` to represent expected failures. Do not throw exceptions for expected business or validation failures.
* Views must bind to ViewModels. Do not pass domain models directly to Views.
* Validate user input using Data Annotations and `ModelState`. Invalid user input must never result in an HTTP 500 response.
* Always use `decimal` for monetary values.
* Discount calculations must remain centralized in `OrderService.CalculateTotal`. Do not recalculate discounts elsewhere.
* Preserve existing routes, public method signatures, response formats, and authorization behavior unless the task explicitly requires changes.
* Use the following files as implementation references:

  * Controllers: `ProductsController.cs`
  * Services: `ProductService.cs`

## Common Commands

* `dotnet build`: Build the solution.
* `dotnet test`: Run all tests.
* `dotnet run --project src/OrderHub.Web`: Start the website at `http://localhost:5150`.

## Important and High-Risk Files

* `src/OrderHub.Infrastructure/Migrations/**`: EF Core migrations are historical records. Do not edit existing migration files manually.
* `src/OrderHub.Web/appsettings.json`: Contains configuration such as connection strings. Ask before modifying it.

## Subagents and Testing

* When the user asks to run tests, the task must be delegated to the custom `test_runner` agent.
* When `test_runner` reports that all tests have passed, the primary agent must forward the result exactly as provided. Do not supplement, rewrite, or append any additional summary.
* Add or update the relevant tests when changing business rules, validation, pricing, order creation, or cancellation behavior.

## Do Not

* Do not add new NuGet packages without approval.
* Do not access `DbContext` directly from controllers or services.
* Do not introduce microservices, message queues, distributed caching, or other infrastructure unless explicitly requested.
* Do not refactor code unrelated to the current task merely as a cleanup.
* Keep changes focused and minimize the number of modified files.
* Do not read or write any secret files, including:

  * `*.pfx`
  * `appsettings.Production.json`
  * User Secrets
