# 📚 Index of Interview Questions - ASP.NET Core

1. [Explain how routing works in ASP.NET Core](#-1-explain-how-routing-works-in-aspnet-core)
2. [What is Middleware and in What Order Do They Execute?](#-2-what-is-middleware-and-in-what-order-do-they-execute)
3. [How Can You Stop Other Middlewares from Executing?](#-3-how-can-you-stop-other-middlewares-from-executing)
4. [What Is the Difference Between MVC and Razor Pages?](#-4-what-is-the-difference-between-mvc-and-razor-pages)
5. [Name 3 Ways to Create Middleware in ASP.NET Core](#-5-name-3-ways-to-create-middleware-in-aspnet-core)
6. [Explain How `appsettings.json` Configuration Layering Works](#-6-explain-how-appsettingsjson-configuration-layering-works)
7. [What Is the Difference Between Singleton, Scoped, and Transient Services?](#-7-what-is-the-difference-between-singleton-scoped-and-transient-services)
8. [How to Use a Scoped Service Inside a Singleton Service in ASP.NET Core?](#-8-how-to-use-a-scoped-service-inside-a-singleton-service-in-aspnet-core)
9. [How to Execute Code When the Application Is Starting and Stopping?](#-9-how-to-execute-code-when-the-application-is-starting-and-stopping)
10. [What Is a Background Service in ASP.NET Core?](#-10-what-is-a-background-service-in-aspnet-core)
11. [Name a Few Ways to Read Data from `appsettings.json` Configuration](#-11-name-a-few-ways-to-read-data-from-appsettingsjson-configuration)
12. [What Is the Options Pattern in ASP.NET Core?](#-12-what-is-the-options-pattern-in-aspnet-core)
13. [Name the Use Cases for `IOptionsSnapshot` and `IOptionsMonitor`](#-13-name-the-use-cases-for-ioptionssnapshot-and-ioptionsmonitor)
14. [How to Validate Configuration in ASP.NET Core?](#-14-how-to-validate-configuration-in-aspnet-core)
15. [What Is the Difference Between DataAnnotations and FluentValidation?](#-15-what-is-the-difference-between-dataannotations-and-fluentvalidation)
16. [What Are the Controller Filter Attributes in ASP.NET Core?](#-16-what-are-the-controller-filter-attributes-in-aspnet-core)
17. [Why Are Minimal APIs Faster Than Controllers in ASP.NET Core?](#-17-why-are-minimal-apis-faster-than-controllers-in-aspnet-core)
18. [How to Add Authorization to an ASP.NET Core Project?](#-18-how-to-add-authorization-to-an-aspnet-core-project)
19. [How to Add Authorization to All Controller's Methods Except One?](#-19-how-to-add-authorization-to-all-controllers-methods-except-one)
20. [How Would You Implement Log-In Functionality in ASP.NET Core?](#-20-how-would-you-implement-log-in-functionality-in-aspnet-core)
21. [Explain How JWT Tokens Work](#-21-explain-how-jwt-tokens-work)
22. [Explain Refresh Tokens and How They Work](#-22-explain-refresh-tokens-and-how-they-work)

---

# ✅ 1. Explain how routing works in ASP.NET Core

## 🧠 What is Routing?

Routing in **ASP.NET Core** is the mechanism that **maps incoming HTTP requests to corresponding controller actions or endpoints** in your application.

> In simpler terms: Routing decides **"which code to execute"** based on the **URL** of the request.

---

## 🏗️ Types of Routing

ASP.NET Core supports two main types of routing:

### 1. **Conventional Routing** (mostly used in MVC)

Defined in `Program.cs` or `Startup.cs`, this uses **patterns** like `{controller=Home}/{action=Index}/{id?}`.

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

🔸 A request to `/products/details/5` would be routed to:

```plaintext
Controller: ProductsController
Action: Details
Parameter: id = 5
```

---

### 2. **Attribute Routing** (defined directly on controllers/actions)

This gives you **more control** and is preferred for APIs.

```csharp
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetProductById(int id)
    {
        // Logic here
        return Ok($"Product {id}");
    }
}
```

🔸 A request to `GET /api/products/10` will invoke `GetProductById(10)`.

---

## 🧩 Routing Components

* **Endpoints**: Final target of the route (usually a controller action or Razor page).
* **Middleware**: ASP.NET Core uses middleware pipeline; routing must be added before endpoint execution.
* **Route templates**: Patterns like `{id:int}` or `{slug:alpha}` that can include constraints and defaults.

---

## 🔄 Routing in Minimal APIs (.NET 6+)

```csharp
var app = WebApplication.Create();

app.MapGet("/hello/{name}", (string name) => $"Hello {name}!");

app.Run();
```

🔸 Request to `/hello/Albert` returns: `Hello Albert`.

---

## ✅ Practical Example (MVC)

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

```csharp
// Controllers/HomeController.cs
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }

    public IActionResult About(int id)
    {
        return Content($"About Page with ID: {id}");
    }
}
```

🌐 Request to `/home/about/3` → matches `About(int id)` with `id = 3`.

---

## 📌 Summary

| Concept               | Description                              |
| --------------------- | ---------------------------------------- |
| **Routing**           | Matches URL path to a controller/action  |
| **Conventional**      | Based on URL patterns                    |
| **Attribute Routing** | Uses `[Route]`, `[HttpGet]`, etc.        |
| **Minimal API**       | Uses `MapGet`, `MapPost`, etc. directly  |
| **Constraints**       | Restrict route values by type or pattern |

---

# ✅ 2. What is Middleware and in What Order Do They Execute?

## 🧠 What is Middleware?

**Middleware** in ASP.NET Core is a software component that is assembled into the application pipeline to **handle requests and responses**.

> Middleware can:
>
> * Inspect or modify the incoming request.
> * Pass the request to the next component in the pipeline.
> * Inspect or modify the outgoing response.

---

## 🧱 Middleware Execution Flow

```
[Client Request]
        ↓
Middleware A
        ↓
Middleware B
        ↓
Endpoint (e.g., Controller, Razor Page)
        ↑
Middleware B (response)
        ↑
Middleware A (response)
        ↑
[Client Response]
```

---

## ⚙️ Where is Middleware Configured?

Middleware is added in `Program.cs` using methods like `.UseXyz()`:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseMiddleware<LoggingMiddleware>();
app.UseRouting();          // Important: before endpoints
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();      // Defines endpoints

app.Run();
```

---

## 🚦 Execution Order Matters

Middleware is executed **in the order it is added**. If the order is incorrect, it can break functionality.

✅ For example, you must call `UseAuthentication()` **before** `UseAuthorization()` — otherwise, the user won't be authenticated when authorization runs.

---

## 🛠️ Custom Middleware Example

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine($"[IN] Request: {context.Request.Path}");
        await _next(context); // Pass to the next middleware
        Console.WriteLine($"[OUT] Response: {context.Response.StatusCode}");
    }
}
```

Add it to the pipeline:

```csharp
app.UseMiddleware<LoggingMiddleware>();
```

---

## 🔁 Full Example with Output

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("➡️ Middleware 1 - Before");
    await next();
    Console.WriteLine("⬅️ Middleware 1 - After");
});

app.Use(async (context, next) =>
{
    Console.WriteLine("➡️ Middleware 2 - Before");
    await next();
    Console.WriteLine("⬅️ Middleware 2 - After");
});

app.Run(async context =>
{
    Console.WriteLine("🎯 Endpoint reached");
    await context.Response.WriteAsync("Hello Middleware!");
});
```

🖨️ Console output:

```
➡️ Middleware 1 - Before
➡️ Middleware 2 - Before
🎯 Endpoint reached
⬅️ Middleware 2 - After
⬅️ Middleware 1 - After
```

---

## 📌 Summary Table

| Concept      | Explanation                                                |
| ------------ | ---------------------------------------------------------- |
| Middleware   | A component that processes HTTP requests/responses         |
| Order        | Determined by the order added in `Program.cs`              |
| Control flow | Uses `await next()` to pass control to the next middleware |
| Custom       | Built by implementing a class with `InvokeAsync()` method  |

---

# ✅ 3. How Can You Stop Other Middlewares from Executing?

## 🧠 Overview

In ASP.NET Core, each middleware decides **whether to call the next middleware** in the pipeline using the `await next()` call.

If you **do not call `await next()`**, the request **will not continue** to the next middleware or endpoint — effectively **stopping** the execution of the pipeline.

---

## 🔧 Basic Structure of Middleware

```csharp
app.Use(async (context, next) =>
{
    // Pre-processing logic
    await next(); // Passes control to the next middleware
    // Post-processing logic
});
```

To stop the pipeline, simply **omit `await next()`**:

```csharp
app.Use(async (context, next) =>
{
    context.Response.StatusCode = 403;
    await context.Response.WriteAsync("Access Denied");

    // No call to next() → pipeline ends here
});
```

---

## 🧪 Practical Example: Short-circuiting the pipeline

```csharp
app.Use(async (context, next) =>
{
    if (!context.Request.Headers.ContainsKey("X-Auth"))
    {
        context.Response.StatusCode = 401;
        await context.Response.WriteAsync("Unauthorized");
        return; // Don't call next() → stop here
    }

    await next(); // Continue if X-Auth header exists
});

app.Use(async (context, next) =>
{
    Console.WriteLine("✅ This middleware only runs if authorized.");
    await next();
});

app.Run(async context =>
{
    await context.Response.WriteAsync("Final endpoint reached.");
});
```

🧪 Behavior:

* If `X-Auth` header is missing → returns `401 Unauthorized` and stops.
* If `X-Auth` is present → the pipeline continues as normal.

---

## 🛑 Other Ways to Short-Circuit

1. **Return a response early:**

```csharp
context.Response.StatusCode = 400;
await context.Response.WriteAsync("Bad Request");
```

2. **Use `Run()` instead of `Use()`**
   `app.Run()` does **not pass** control to any further middleware.

```csharp
app.Run(async context =>
{
    await context.Response.WriteAsync("This ends the pipeline.");
});
```

If `app.Run()` is hit, **no more middleware after it is executed**.

---

## 📌 Summary

| Action                              | Result                                   |
| ----------------------------------- | ---------------------------------------- |
| Omit `await next()`                 | Stops the pipeline                       |
| Use `return` after writing response | Prevents further processing              |
| Use `app.Run()`                     | Defines terminal middleware (no `next`)  |
| Common use cases                    | Auth checks, global error handling, etc. |

---

# ✅ 4. What Is the Difference Between MVC and Razor Pages?

## 🧠 Overview

Both **MVC** and **Razor Pages** are part of the ASP.NET Core web framework.
They are used to build **dynamic web applications** — but they differ in **structure, routing, and separation of concerns**.

---

## 📦 ASP.NET Core MVC

**MVC (Model-View-Controller)** separates the application into 3 components:

* **Model** – Business/data logic
* **View** – UI layer (.cshtml)
* **Controller** – Handles requests and returns Views or data

### 🧱 Structure (Typical MVC)

```
/Controllers/HomeController.cs
/Views/Home/Index.cshtml
/Models/Product.cs
```

### 📄 Example – MVC Controller

```csharp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View(); // Looks for Views/Home/Index.cshtml
    }
}
```

### ➕ Pros

* Clear separation of concerns
* Better suited for large apps
* Great for REST APIs (with `ApiController`)

---

## 📘 Razor Pages

**Razor Pages** is a **page-based** architecture introduced in ASP.NET Core 2.0.
Each page combines the **view** and its **logic** into a single `.cshtml` + `.cshtml.cs` pair.

### 🧱 Structure (Typical Razor Pages)

```
/Pages/Index.cshtml
/Pages/Index.cshtml.cs
```

### 📄 Example – Razor Page

```csharp
// Pages/Index.cshtml.cs
public class IndexModel : PageModel
{
    public string Message { get; set; }

    public void OnGet()
    {
        Message = "Hello from Razor Page!";
    }
}
```

```html
<!-- Pages/Index.cshtml -->
@page
@model IndexModel
<h2>@Model.Message</h2>
```

### ➕ Pros

* Simpler for small or form-based apps
* Less boilerplate than MVC
* Better organization for page-centric logic

---

## 🔁 Key Differences

| Feature        | MVC                             | Razor Pages                     |
| -------------- | ------------------------------- | ------------------------------- |
| Pattern        | Controller-based                | Page-based                      |
| File structure | Separated (Controllers + Views) | Combined (.cshtml + .cshtml.cs) |
| Routing        | Conventional or attribute-based | Based on file path              |
| REST APIs      | Very suitable                   | Not ideal                       |
| Simplicity     | More complex                    | Easier to get started           |
| Use case       | Large web apps, APIs            | Simple web apps, forms          |

---

## 📝 When to Use What?

* ✅ Use **MVC** if:

  * You're building a **REST API**
  * Your app has **complex routing**
  * You prefer **separation of concerns**

* ✅ Use **Razor Pages** if:

  * You're building a **simple web UI**
  * You want a **quick and clean page-based model**
  * You're creating **form-based pages** (CRUD)

---

# ✅ 5. Name 3 Ways to Create Middleware in ASP.NET Core

ASP.NET Core offers several ways to create and register middleware in the HTTP request pipeline.

Here are **three main ways** to do it:

---

## 1️⃣ **Use Inline Middleware with `app.Use`**

This is the **simplest way** to create middleware using a lambda function.

### ✅ Example:

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("➡️ Inline Middleware");
    await next(); // Passes control to the next middleware
});
```

### 🔎 When to use:

* For quick logic
* Logging, header checks, etc.
* No need for custom class

---

## 2️⃣ **Create a Reusable Class and Register with `app.UseMiddleware<T>()`**

You can define a custom middleware class with a constructor and `InvokeAsync()` method.

### ✅ Example:

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine($"Request: {context.Request.Path}");
        await _next(context); // Call the next middleware
    }
}
```

Register it in `Program.cs`:

```csharp
app.UseMiddleware<LoggingMiddleware>();
```

### 🔎 When to use:

* Reusable across projects
* Complex logic
* Injecting services via constructor

---

## 3️⃣ **Use Extension Method to Register Middleware**

You can wrap the middleware logic in an **extension method** for cleaner code and reuse.

### ✅ Example:

#### Middleware class:

```csharp
public class SecurityHeadersMiddleware
{
    private readonly RequestDelegate _next;

    public SecurityHeadersMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
        await _next(context);
    }
}
```

#### Extension method:

```csharp
public static class SecurityHeadersMiddlewareExtensions
{
    public static IApplicationBuilder UseSecurityHeaders(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<SecurityHeadersMiddleware>();
    }
}
```

#### Register in `Program.cs`:

```csharp
app.UseSecurityHeaders();
```

### 🔎 When to use:

* Cleaner and descriptive pipeline
* Middleware libraries or shared projects

---

## 📌 Summary

| Method                 | Description                                 | Use Case                          |
| ---------------------- | ------------------------------------------- | --------------------------------- |
| `app.Use(...)`         | Inline middleware with lambda               | Simple or quick logic             |
| `UseMiddleware<T>()`   | Class-based middleware                      | Reusable, testable, DI support    |
| `UseCustomExtension()` | Extension method wrapping class-based logic | Clean syntax, library-style reuse |

---

# ✅ 6. Explain How `appsettings.json` Configuration Layering Works

## 🧠 What is Configuration Layering?

In **ASP.NET Core**, configuration settings are loaded from **multiple sources** in a specific **priority order**.
Each source can **override** values from the previous one — this is known as **configuration layering**.

One of the most common configuration files is `appsettings.json`.

---

## 🧩 Configuration Sources (Default Order)

ASP.NET Core loads configuration **in this order** (later wins):

1. `appsettings.json`
2. `appsettings.{Environment}.json` (e.g., `appsettings.Development.json`)
3. **Environment Variables**
4. **Command-line arguments**
5. **Secrets manager** (for development)
6. Any **custom configuration providers**

> 🔁 Later sources override earlier ones.

---

## 📁 Example File Structure

```plaintext
appsettings.json
appsettings.Development.json
appsettings.Production.json
```

---

## 📄 Example: `appsettings.json`

```json
{
  "AppSettings": {
    "Title": "My App",
    "ShowBeta": false
  }
}
```

## 📄 Example: `appsettings.Development.json`

```json
{
  "AppSettings": {
    "ShowBeta": true
  }
}
```

### ✅ Final configuration in **Development** environment:

```json
{
  "Title": "My App",
  "ShowBeta": true   // Overridden by appsettings.Development.json
}
```

---

## 🔧 How It's Loaded in `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

// This is what happens under the hood:
builder.Configuration
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", optional: true)
    .AddEnvironmentVariables()
    .AddCommandLine(args);
```

---

## 🛠️ How to Use It in Code

Inject `IConfiguration`:

```csharp
public class HomeController : Controller
{
    private readonly IConfiguration _config;

    public HomeController(IConfiguration config)
    {
        _config = config;
    }

    public IActionResult Index()
    {
        var title = _config["AppSettings:Title"];
        return Content($"App Title: {title}");
    }
}
```

---

## 🧪 Environment Selection

You can set the environment using:

```bash
# Linux/macOS
export ASPNETCORE_ENVIRONMENT=Production

# Windows CMD
set ASPNETCORE_ENVIRONMENT=Production
```

Or via `launchSettings.json` in development:

```json
"profiles": {
  "MyApp": {
    "environmentVariables": {
      "ASPNETCORE_ENVIRONMENT": "Development"
    }
  }
}
```

---

## 📌 Summary Table

| Source                           | Priority | Overwrites Previous? |
| -------------------------------- | -------- | -------------------- |
| `appsettings.json`               | Low      | No                   |
| `appsettings.{Environment}.json` | Medium   | Yes                  |
| Environment variables            | High     | Yes                  |
| Command-line args                | Highest  | Yes                  |

---

# ✅ 7. What Is the Difference Between Singleton, Scoped, and Transient Services?

In **ASP.NET Core**, you register services in the **Dependency Injection (DI) container**.
When registering services, you choose their **lifetime**, which controls **how and when** instances are created.

---

## 🧠 1. Singleton

* **One instance** is created for the **entire application lifetime**.
* The same instance is **shared across all requests and users**.

```csharp
builder.Services.AddSingleton<IMyService, MyService>();
```

### ✅ Use when:

* The service is **stateless** or **expensive to create**.
* You want to **cache** or **share resources**.

### ⚠️ Be careful:

* Avoid using **HttpContext** or per-request data.
* Not thread-safe by default.

---

## 🧠 2. Scoped

* A **new instance is created per HTTP request**.
* All components within that request share the same instance.

```csharp
builder.Services.AddScoped<IMyService, MyService>();
```

### ✅ Use when:

* You need to maintain **state within a single request** (e.g., database context).
* Safe to use services that depend on the current request (e.g., `HttpContextAccessor`).

---

## 🧠 3. Transient

* A **new instance is created every time** the service is requested.

```csharp
builder.Services.AddTransient<IMyService, MyService>();
```

### ✅ Use when:

* The service is **lightweight** and **stateless**.
* You don't need to share state at all.

---

## 🧪 Practical Example

```csharp
public interface IOperationService
{
    Guid Id { get; }
}

public class OperationService : IOperationService
{
    public Guid Id { get; } = Guid.NewGuid();
}
```

Register services:

```csharp
builder.Services.AddSingleton<IOperationService, OperationService>();   // Singleton
builder.Services.AddScoped<IOperationService, OperationService>();      // Scoped
builder.Services.AddTransient<IOperationService, OperationService>();   // Transient
```

Inject into controller:

```csharp
public class DemoController : Controller
{
    private readonly IOperationService _service1;
    private readonly IOperationService _service2;

    public DemoController(IOperationService service1, IOperationService service2)
    {
        _service1 = service1;
        _service2 = service2;
    }

    public IActionResult Index()
    {
        return Content($"Service1 ID: {_service1.Id} | Service2 ID: {_service2.Id}");
    }
}
```

* **Singleton**: Both IDs are **the same** across all requests.
* **Scoped**: Same ID within one request, different across requests.
* **Transient**: Different ID every time it is injected.

---

## 📌 Summary Table

| Lifetime  | Instance Per | Shared Between | Typical Use Case                              |
| --------- | ------------ | -------------- | --------------------------------------------- |
| Singleton | Application  | All requests   | Logging, caching, config, DI container itself |
| Scoped    | HTTP Request | Same request   | DbContext, business logic                     |
| Transient | Every call   | No sharing     | Lightweight services, mapping                 |

---

# ✅ 8. How to Use a Scoped Service Inside a Singleton Service in ASP.NET Core?

## 🧠 Problem Summary

In **ASP.NET Core**, directly injecting a **scoped service into a singleton** is **not allowed** and will cause a runtime exception or incorrect behavior.

> ❌ Why?
> Because scoped services are **tied to the HTTP request**, and singleton services **live for the entire app lifetime**. Mixing them can cause memory leaks or unexpected results.

---

## 🛠️ ✅ Recommended Solution: Use `IServiceProvider` or `IServiceScopeFactory`

Instead of direct injection, use **manual service resolution** within a **scope**.

---

## 🔧 Method 1: Using `IServiceScopeFactory`

```csharp
public interface IScopedService
{
    string GetRequestId();
}

public class ScopedService : IScopedService
{
    private readonly string _requestId = Guid.NewGuid().ToString();

    public string GetRequestId() => _requestId;
}
```

### ✅ Singleton that depends on scoped service:

```csharp
public class MySingletonService
{
    private readonly IServiceScopeFactory _scopeFactory;

    public MySingletonService(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    public void DoWork()
    {
        using var scope = _scopeFactory.CreateScope();
        var scopedService = scope.ServiceProvider.GetRequiredService<IScopedService>();
        Console.WriteLine($"Scoped Request ID: {scopedService.GetRequestId()}");
    }
}
```

### Register in `Program.cs`:

```csharp
builder.Services.AddScoped<IScopedService, ScopedService>();
builder.Services.AddSingleton<MySingletonService>();
```

---

## 🔧 Method 2: Using `IServiceProvider` directly (less preferred)

```csharp
public class MySingletonService
{
    private readonly IServiceProvider _serviceProvider;

    public MySingletonService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public void DoWork()
    {
        using var scope = _serviceProvider.CreateScope();
        var scoped = scope.ServiceProvider.GetRequiredService<IScopedService>();
        Console.WriteLine(scoped.GetRequestId());
    }
}
```

> ⚠️ Warning: Avoid storing `IServiceProvider` long-term inside the singleton. Only use it temporarily within the method that needs the scoped dependency.

---

## ❌ What NOT to do

This will **compile** but cause **runtime issues** or unexpected behavior:

```csharp
// BAD PRACTICE: Injecting a scoped service directly into singleton
public class BadSingleton
{
    public BadSingleton(IScopedService scoped) { }
}
```

---

## 📌 Summary

| Approach                       | Safe? | Description                                     |
| ------------------------------ | ----- | ----------------------------------------------- |
| Inject `IServiceScopeFactory`  | ✅     | Recommended: short-lived scope inside singleton |
| Inject `IServiceProvider`      | ✅     | Acceptable: create scope manually               |
| Inject scoped service directly | ❌     | Dangerous: breaks DI lifetime rules             |

---

# ✅ 9. How to Execute Code When the Application Is Starting and Stopping?

In **ASP.NET Core**, you can hook into application **lifetime events** to run custom logic **on startup** and **on shutdown**.

---

## 🚀 On Application Start / Stop

ASP.NET Core provides the `IHostApplicationLifetime` (before .NET 6) or `IHostApplicationLifetime` / `IHostApplicationBuilder` in newer versions to access:

* `ApplicationStarted` – app has fully started
* `ApplicationStopping` – app is shutting down (graceful)
* `ApplicationStopped` – app has fully stopped

---

## ✅ Using `IHostApplicationLifetime` (Startup-based approach)

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app, IHostApplicationLifetime lifetime)
    {
        lifetime.ApplicationStarted.Register(() =>
        {
            Console.WriteLine("🚀 App started");
            // e.g., seed database, start background jobs
        });

        lifetime.ApplicationStopping.Register(() =>
        {
            Console.WriteLine("🛑 App stopping");
            // e.g., flush logs, dispose services
        });

        lifetime.ApplicationStopped.Register(() =>
        {
            Console.WriteLine("✅ App stopped");
        });

        app.UseRouting();
    }
}
```

---

## ✅ Using `Program.cs` in .NET 6 / 7 / 8

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

var lifetime = app.Lifetime;

lifetime.ApplicationStarted.Register(() =>
{
    Console.WriteLine("🚀 Application Started");
});

lifetime.ApplicationStopping.Register(() =>
{
    Console.WriteLine("🛑 Application Stopping");
});

lifetime.ApplicationStopped.Register(() =>
{
    Console.WriteLine("✅ Application Stopped");
});

app.Run();
```

---

## ✅ Option: Use `IHostedService` for Long-Running Background Tasks

You can create a class that implements `IHostedService` to run logic on app start and shutdown.

```csharp
public class StartupTask : IHostedService
{
    public Task StartAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("🚀 Startup logic here");
        return Task.CompletedTask;
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("🛑 Shutdown logic here");
        return Task.CompletedTask;
    }
}
```

Register it:

```csharp
builder.Services.AddHostedService<StartupTask>();
```

---

## 📌 Summary Table

| Action               | Best Approach                                                 |
| -------------------- | ------------------------------------------------------------- |
| Run code on startup  | `ApplicationStarted.Register()` or `StartAsync()`             |
| Run code on shutdown | `ApplicationStopping` / `ApplicationStopped` or `StopAsync()` |
| For background logic | Implement `IHostedService`                                    |

---

# ✅ 10. What Is a Background Service in ASP.NET Core?

## 🧠 Definition

A **Background Service** is a long-running task that executes in the background independently of incoming HTTP requests.

In ASP.NET Core, background services are typically implemented using the **`BackgroundService`** base class, which is part of the **`Microsoft.Extensions.Hosting`** namespace.

---

## 🧱 Use Cases

Background services are useful for tasks like:

* Processing queued messages
* Periodic jobs (e.g., cleanup, reporting)
* Polling external APIs
* Sending emails
* Listening to events from a queue (e.g., RabbitMQ, Kafka)

---

## 🛠️ How to Create a Background Service

### ✅ Step 1: Create the service

```csharp
public class MyBackgroundService : BackgroundService
{
    private readonly ILogger<MyBackgroundService> _logger;

    public MyBackgroundService(ILogger<MyBackgroundService> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("Background service is starting.");

        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Background service is working...");
            await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
        }

        _logger.LogInformation("Background service is stopping.");
    }
}
```

### ✅ Step 2: Register the service in `Program.cs`

```csharp
builder.Services.AddHostedService<MyBackgroundService>();
```

---

## 🔁 Lifecycle

A `BackgroundService` starts **when the application starts**, and it stops **gracefully** when the app is shutting down.

* You implement your logic in `ExecuteAsync(CancellationToken stoppingToken)`.
* The framework handles the lifetime automatically.
* You should **listen to the cancellation token** to gracefully stop your service.

---

## 📦 Other Ways to Run Background Tasks

| Option                  | Use Case                                        |
| ----------------------- | ----------------------------------------------- |
| `IHostedService`        | Base interface for background workers           |
| `BackgroundService`     | Abstract class for long-running background task |
| `Task.Run()` in startup | ❌ Not recommended for long-term processing      |
| Timer-based tasks       | Use `System.Threading.Timer` in hosted service  |

---

## ⚠️ Best Practices

* Always monitor the `stoppingToken` to shut down gracefully.
* Avoid infinite loops without `await` → they can freeze the thread.
* Use `try/catch` inside the loop to prevent crashes.
* Offload heavy work using channels or queues.

---

## ✅ Summary

| Concept            | Explanation                                          |
| ------------------ | ---------------------------------------------------- |
| Background Service | Long-running task that runs outside of HTTP pipeline |
| Based on           | `BackgroundService` or `IHostedService`              |
| Starts             | When app starts                                      |
| Stops              | When app is shutting down                            |
| Ideal for          | Message queues, scheduled jobs, processing tasks     |

---

# ✅ 11. Name a Few Ways to Read Data from `appsettings.json` Configuration

In ASP.NET Core, the `appsettings.json` file is commonly used to store configuration data such as connection strings, custom settings, feature toggles, etc.

There are **multiple ways** to read data from it:

---

## 🧩 Example `appsettings.json`

```json
{
  "AppSettings": {
    "SiteName": "MyApp",
    "MaxItems": 10
  },
  "ConnectionStrings": {
    "Default": "Server=.;Database=MyDb;Trusted_Connection=True;"
  }
}
```

---

## 1️⃣ **Using `IConfiguration` Interface (Key-Based Access)**

### ✅ Example:

```csharp
public class HomeController : Controller
{
    private readonly IConfiguration _config;

    public HomeController(IConfiguration config)
    {
        _config = config;
    }

    public IActionResult Index()
    {
        var siteName = _config["AppSettings:SiteName"];
        var connection = _config.GetConnectionString("Default");

        return Content($"Site: {siteName} | DB: {connection}");
    }
}
```

### 🔎 Notes:

* Use colon (`:`) for nested sections.
* Use `GetConnectionString()` for the `ConnectionStrings` section.

---

## 2️⃣ **Binding to a POCO (Plain C# Class)**

Create a class that matches the section structure:

```csharp
public class AppSettings
{
    public string SiteName { get; set; }
    public int MaxItems { get; set; }
}
```

### ✅ Register it in `Program.cs`:

```csharp
builder.Services.Configure<AppSettings>(builder.Configuration.GetSection("AppSettings"));
```

### ✅ Inject using `IOptions<T>`:

```csharp
public class HomeController : Controller
{
    private readonly AppSettings _settings;

    public HomeController(IOptions<AppSettings> options)
    {
        _settings = options.Value;
    }

    public IActionResult Index()
    {
        return Content($"Site: {_settings.SiteName}, Max: {_settings.MaxItems}");
    }
}
```

---

## 3️⃣ **Using `GetSection().Get<T>()` for Manual Binding**

No need for `IOptions`, useful in non-DI contexts:

```csharp
var settings = builder.Configuration.GetSection("AppSettings").Get<AppSettings>();
Console.WriteLine(settings.SiteName);
```

---

## 4️⃣ **Using `IOptionsSnapshot<T>` or `IOptionsMonitor<T>`**

* `IOptionsSnapshot<T>` – for **scoped** lifetime, gets updated per request.
* `IOptionsMonitor<T>` – for **singleton** services, detects **real-time changes** if `reloadOnChange` is enabled.

### Example:

```csharp
public class MyService
{
    private readonly AppSettings _currentSettings;

    public MyService(IOptionsMonitor<AppSettings> monitor)
    {
        _currentSettings = monitor.CurrentValue;
    }
}
```

---

## 📌 Summary Table

| Method                                 | Use Case                           | Lifetime         |
| -------------------------------------- | ---------------------------------- | ---------------- |
| `IConfiguration["Key"]`                | Simple key access                  | Any              |
| `IConfiguration.GetSection().Get<T>()` | Bind to class manually             | Any              |
| `IOptions<T>`                          | Auto-bind class via DI             | Singleton/Scoped |
| `IOptionsSnapshot<T>`                  | Updated on each request            | Scoped           |
| `IOptionsMonitor<T>`                   | Live-updating, background services | Singleton        |

---

# ✅ 12. What Is the Options Pattern in ASP.NET Core?

## 🧠 Definition

The **Options Pattern** in ASP.NET Core is a way to **bind configuration sections** (like from `appsettings.json`) to **strongly-typed classes** and inject them using **dependency injection (DI)**.

> It helps keep configuration code clean, type-safe, and maintainable.

---

## 🔧 Why Use the Options Pattern?

* ✅ Strongly-typed access to config values
* ✅ Supports validation and reload on change
* ✅ Follows dependency injection principles

---

## 📄 Example `appsettings.json`

```json
{
  "AppSettings": {
    "SiteTitle": "MyApp",
    "MaxItems": 10
  }
}
```

---

## 🧱 Step-by-Step Implementation

### 1️⃣ Create a POCO class

```csharp
public class AppSettings
{
    public string SiteTitle { get; set; }
    public int MaxItems { get; set; }
}
```

---

### 2️⃣ Register the configuration in `Program.cs`

```csharp
builder.Services.Configure<AppSettings>(
    builder.Configuration.GetSection("AppSettings"));
```

This binds the `AppSettings` section to the `AppSettings` class and makes it injectable.

---

### 3️⃣ Inject using `IOptions<T>` in a service or controller

```csharp
public class HomeController : Controller
{
    private readonly AppSettings _settings;

    public HomeController(IOptions<AppSettings> options)
    {
        _settings = options.Value;
    }

    public IActionResult Index()
    {
        return Content($"Title: {_settings.SiteTitle}, Max: {_settings.MaxItems}");
    }
}
```

---

## 🔁 Advanced Variants

### 🌀 `IOptionsSnapshot<T>`

* For **scoped** lifetimes (e.g., per HTTP request)
* Captures configuration as it was at the start of the request

```csharp
public HomeController(IOptionsSnapshot<AppSettings> options)
```

---

### 🧲 `IOptionsMonitor<T>`

* For **singleton** services
* Automatically **detects changes** in config if `reloadOnChange: true`

```csharp
public class BackgroundWorker
{
    public BackgroundWorker(IOptionsMonitor<AppSettings> monitor)
    {
        var settings = monitor.CurrentValue;
    }
}
```

---

## ⚠️ Common Mistakes

* Forgetting to register `.Configure<T>()` in `Program.cs`
* Using `IOptionsSnapshot` in singleton services (not allowed)
* Accessing config directly with strings (`_config["SomeKey"]`) — less safe

---

## 📌 Summary Table

| Interface             | Lifetime  | Reload on Change | Use Case                     |
| --------------------- | --------- | ---------------- | ---------------------------- |
| `IOptions<T>`         | Singleton | ❌                | Basic usage in most services |
| `IOptionsSnapshot<T>` | Scoped    | ✅ (per request)  | Web apps (controllers)       |
| `IOptionsMonitor<T>`  | Singleton | ✅ (real-time)    | Background workers, caching  |

---

# ✅ 13. Name the Use Cases for `IOptionsSnapshot` and `IOptionsMonitor`

ASP.NET Core provides **`IOptions<T>`**, **`IOptionsSnapshot<T>`**, and **`IOptionsMonitor<T>`** to access configuration data.

Each serves a different purpose depending on the **lifetime** of the service and the **need for runtime updates**.

---

## 🌀 `IOptionsSnapshot<T>`

### 🧠 What is it?

* Retrieves a **fresh configuration instance per HTTP request**
* Scoped lifetime
* Suitable for **web applications** where config may change between requests

### ✅ Use Cases

| Use Case                         | Description                                                                         |
| -------------------------------- | ----------------------------------------------------------------------------------- |
| 🔁 **Per-request configuration** | Inject in a controller or scoped service to get updated values per request          |
| 🧪 **Testing new features**      | Useful when config values are toggled frequently (e.g., A/B testing, feature flags) |
| 🌐 **Multi-tenant apps**         | Each request might load a different tenant's configuration                          |

### ❌ Not suitable for:

* Singleton services (won't work, throws error)

---

## 🧲 `IOptionsMonitor<T>`

### 🧠 What is it?

* **Singleton-safe** and supports **real-time config changes**
* Monitors the underlying configuration file (e.g., `appsettings.json`)
* Triggers a **callback when config changes**

### ✅ Use Cases

| Use Case                          | Description                                                                      |
| --------------------------------- | -------------------------------------------------------------------------------- |
| 🧵 **Background services**        | Use in long-running or singleton services (e.g., workers, hosted services)       |
| 🔄 **Live configuration reloads** | Auto-reloads values when `appsettings.json` changes and `reloadOnChange` is true |
| 🔔 **On-change notifications**    | Subscribe to `OnChange()` for reacting to config updates                         |
| 💡 **Caching and tuning**         | Useful for changing thresholds or feature toggles without restarting the app     |

---

### 🔧 Example: Reacting to changes with `IOptionsMonitor`

```csharp
public class FeatureToggleService
{
    public FeatureToggleService(IOptionsMonitor<AppSettings> monitor)
    {
        monitor.OnChange(settings =>
        {
            Console.WriteLine($"🔄 Config updated: FeatureX = {settings.EnableFeatureX}");
        });
    }
}
```

---

## 🧪 Summary Table

| Feature                  | `IOptionsSnapshot<T>`         | `IOptionsMonitor<T>`                          |
| ------------------------ | ----------------------------- | --------------------------------------------- |
| Lifetime                 | Scoped                        | Singleton                                     |
| Refresh on change        | Yes (per request)             | Yes (real-time)                               |
| Use in controllers       | ✅                             | ✅                                             |
| Use in singletons        | ❌                             | ✅                                             |
| Detect changes via event | ❌                             | ✅ `.OnChange()`                               |
| Common use case          | Per-request config (web apps) | Live monitoring (background services, caches) |

---

# ✅ 14. How to Validate Configuration in ASP.NET Core?

## 🧠 Why Validate Configuration?

When using the **Options pattern** to bind config sections (like `appsettings.json`), it's important to ensure the values:

* Are **not null or empty**
* Fall within an expected **range**
* Meet **required business rules**

If misconfigured, the app should **fail fast** at startup rather than throw runtime exceptions.

---

## 🔧 3 Main Ways to Validate Configuration

---

## 1️⃣ **Data Annotations + `ValidateDataAnnotations()`**

### ✅ Example Configuration Section

```json
"AppSettings": {
  "SiteTitle": "My Site",
  "MaxItems": 20
}
```

### ✅ POCO with Annotations

```csharp
public class AppSettings
{
    [Required]
    public string SiteTitle { get; set; }

    [Range(1, 100)]
    public int MaxItems { get; set; }
}
```

### ✅ Register and Validate in `Program.cs`

```csharp
builder.Services.AddOptions<AppSettings>()
    .Bind(builder.Configuration.GetSection("AppSettings"))
    .ValidateDataAnnotations(); // enables [Required], [Range], etc.
```

---

## 2️⃣ **Custom Validation with `.Validate(...)`**

Use a lambda or method to write your own rules.

```csharp
builder.Services.AddOptions<AppSettings>()
    .Bind(builder.Configuration.GetSection("AppSettings"))
    .Validate(settings => settings.MaxItems > 0 && settings.MaxItems < 100,
              "MaxItems must be between 1 and 99");
```

### ✅ You can also extract it into a static method:

```csharp
.Validate(AppSettingsValidator.Validate, "Custom validation failed");
```

---

## 3️⃣ **Fail Fast on Startup with `.ValidateOnStart()`**

By default, validation happens **only when the option is first accessed**.
To **fail at app startup**, add:

```csharp
.ValidateOnStart();
```

🔁 Combine everything:

```csharp
builder.Services.AddOptions<AppSettings>()
    .Bind(builder.Configuration.GetSection("AppSettings"))
    .ValidateDataAnnotations()
    .Validate(settings => settings.SiteTitle.StartsWith("My"), "Must start with 'My'")
    .ValidateOnStart();
```

---

## 🔁 Advanced: Use `IValidateOptions<T>`

For reusable, injectable validators:

```csharp
public class AppSettingsValidator : IValidateOptions<AppSettings>
{
    public ValidateOptionsResult Validate(string name, AppSettings settings)
    {
        if (string.IsNullOrWhiteSpace(settings.SiteTitle))
            return ValidateOptionsResult.Fail("SiteTitle is required.");

        if (settings.MaxItems <= 0)
            return ValidateOptionsResult.Fail("MaxItems must be positive.");

        return ValidateOptionsResult.Success;
    }
}
```

Register it:

```csharp
builder.Services.AddSingleton<IValidateOptions<AppSettings>, AppSettingsValidator>();
```

---

## 📌 Summary Table

| Method                      | Description                           | Fails on startup?            |
| --------------------------- | ------------------------------------- | ---------------------------- |
| `ValidateDataAnnotations()` | Uses `[Required]`, `[Range]`, etc.    | ❌ (unless `ValidateOnStart`) |
| `Validate(...)`             | Custom logic with lambda              | ❌ (unless `ValidateOnStart`) |
| `ValidateOnStart()`         | Forces validation on app startup      | ✅                            |
| `IValidateOptions<T>`       | Full custom reusable validation logic | ✅                            |

---

# ✅ 15. What Is the Difference Between DataAnnotations and FluentValidation?

Both **Data Annotations** and **FluentValidation** are used to validate input data in ASP.NET Core — typically for models received from forms or APIs.

They differ in **flexibility, maintainability, and separation of concerns**.

---

## 📘 1. Data Annotations (Built-in)

### 🧠 What Is It?

Validation attributes added **directly to model properties** using `[Required]`, `[StringLength]`, `[Range]`, etc.

### ✅ Example:

```csharp
public class Product
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; }

    [Range(1, 999)]
    public decimal Price { get; set; }
}
```

### ➕ Pros

* ✅ Built-in to .NET
* ✅ Simple for small models
* ✅ Recognized by MVC, Blazor, Swagger, etc.

### ➖ Cons

* ❌ Validation logic is **mixed with your model**
* ❌ Not ideal for **complex rules or conditions**
* ❌ Less flexible and hard to unit test

---

## 📘 2. FluentValidation (External Library)

### 🧠 What Is It?

A **separate class-based validation library** using a fluent API — highly customizable and testable.

### ✅ Install via NuGet:

```
dotnet add package FluentValidation.AspNetCore
```

### ✅ Example:

```csharp
public class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class ProductValidator : AbstractValidator<Product>
{
    public ProductValidator()
    {
        RuleFor(p => p.Name)
            .NotEmpty()
            .MaximumLength(100);

        RuleFor(p => p.Price)
            .InclusiveBetween(1, 999);
    }
}
```

Register it in `Program.cs`:

```csharp
builder.Services.AddValidatorsFromAssemblyContaining<ProductValidator>();
builder.Services.AddFluentValidationAutoValidation(); // Enables automatic model validation
```

---

### ➕ Pros

* ✅ **Separation of concerns** (model != validation logic)
* ✅ Fluent and readable API
* ✅ Easily unit testable
* ✅ Conditional validation (`When(...)`)
* ✅ Complex rules and rule sets

### ➖ Cons

* ❌ Requires additional NuGet package
* ❌ Slightly more setup for small apps

---

## 📊 Comparison Table

| Feature                    | Data Annotations   | FluentValidation               |
| -------------------------- | ------------------ | ------------------------------ |
| 🔌 Built-in support        | ✅ Yes              | ❌ Requires NuGet package       |
| 📦 External dependency     | ❌ None             | ✅ Yes (`FluentValidation`)     |
| 🧼 Separation of concerns  | ❌ Mixed with model | ✅ Separate validator classes   |
| 🔁 Conditional validation  | ❌ Limited          | ✅ Powerful `When(...)` support |
| 🧪 Unit testable           | ❌ Hard             | ✅ Easily testable              |
| 🛠 Complex rules           | ❌ Not ideal        | ✅ Fully supported              |
| 🧩 Suitable for large apps | ⚠️ Limited         | ✅ Highly recommended           |

---

## ✅ Summary

| Use Data Annotations when...                  |
| --------------------------------------------- |
| - You have **simple** validations             |
| - You want to **keep things minimal**         |
| - You work in **Blazor or MVC with metadata** |

| Use FluentValidation when...                 |
| -------------------------------------------- |
| - You need **advanced or conditional** logic |
| - You want **testable, clean code**          |
| - You're working on **scalable projects**    |

---

# ✅ 16. What Are the Controller Filter Attributes in ASP.NET Core?

## 🧠 What Are Filter Attributes?

**Filter attributes** in ASP.NET Core are a powerful way to execute logic **before or after** key stages in the request processing pipeline.

They apply to **controllers** or **actions** and are used for:

* Authorization
* Error handling
* Logging
* Caching
* Modifying requests or responses

---

## 🧩 Types of Filter Attributes

ASP.NET Core supports several **built-in filter types**:

| Filter Type   | Interface              | Purpose                                    |
| ------------- | ---------------------- | ------------------------------------------ |
| Authorization | `IAuthorizationFilter` | Runs before anything else (access control) |
| Resource      | `IResourceFilter`      | Runs before/after model binding            |
| Action        | `IActionFilter`        | Runs before/after action execution         |
| Exception     | `IExceptionFilter`     | Handles unhandled exceptions               |
| Result        | `IResultFilter`        | Runs before/after the result is executed   |

---

## 🎯 1. `[Authorize]`

Used to restrict access to actions/controllers based on user roles or policies.

```csharp
[Authorize(Roles = "Admin")]
public class AdminController : Controller
{
    public IActionResult Index() => View();
}
```

---

## 🎯 2. `[AllowAnonymous]`

Overrides `[Authorize]` to allow anonymous access.

```csharp
[AllowAnonymous]
public IActionResult Public() => View();
```

---

## 🎯 3. `[ValidateAntiForgeryToken]`

Validates the anti-forgery token in POST forms to prevent CSRF.

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult SubmitForm(MyFormModel model)
{
    // Process form
}
```

---

## 🎯 4. `[Produces]`, `[ProducesResponseType]`

Defines the media type or HTTP response status codes for better Swagger docs and client understanding.

```csharp
[Produces("application/json")]
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public IActionResult GetItem() { ... }
```

---

## 🎯 5. `[ServiceFilter]` / `[TypeFilter]`

Used to apply **custom filters** (with DI support).

```csharp
[ServiceFilter(typeof(MyCustomActionFilter))]
public IActionResult SecureArea() => View();
```

> Register `MyCustomActionFilter` in DI.

---

## 🛠️ Custom Action Filter Example

Create a logging filter:

```csharp
public class LogActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        Console.WriteLine("Before executing action");
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        Console.WriteLine("After executing action");
    }
}
```

Register in DI and apply:

```csharp
builder.Services.AddScoped<LogActionFilter>();

[ServiceFilter(typeof(LogActionFilter))]
public IActionResult Index() => View();
```

---

## 📌 Summary Table

| Attribute                          | Description                                             |
| ---------------------------------- | ------------------------------------------------------- |
| `[Authorize]`                      | Restricts access based on user roles or policies        |
| `[AllowAnonymous]`                 | Allows access even if `[Authorize]` is applied globally |
| `[ValidateAntiForgeryToken]`       | Validates anti-CSRF token                               |
| `[Produces]`                       | Sets expected content type of response                  |
| `[ProducesResponseType]`           | Documents possible return codes                         |
| `[ServiceFilter]` / `[TypeFilter]` | Inject and apply custom filters                         |

---

## 🔁 Filter Execution Order

```plaintext
AuthorizationFilter
 ↓
ResourceFilter
 ↓
ActionFilter
 ↓
Controller Action
 ↑
ResultFilter
 ↑
ExceptionFilter (if error)
```

---

# ✅ 17. Why Are Minimal APIs Faster Than Controllers in ASP.NET Core?

## 🧠 Overview

**Minimal APIs** were introduced in **.NET 6** as a lightweight alternative to traditional **controller-based APIs**.

They offer **better performance** due to fewer abstractions and less overhead in the request-processing pipeline.

---

## ⚡ Reasons Why Minimal APIs Are Faster

---

### 1️⃣ ✅ **Fewer Middleware Components**

* Minimal APIs don't require the **MVC middleware**, **routing metadata**, or **model binding system** that controllers use.
* No controller discovery, no `ActionDescriptor`, no attribute routing logic.

```csharp
app.MapGet("/hello", () => "Hello World");
```

✅ This bypasses:

* Controller/action resolution
* Filter pipelines
* View rendering (not needed for APIs)

---

### 2️⃣ ✅ **No Controller/Action Abstraction**

* In MVC, each controller and action is wrapped in a rich metadata object.
* Minimal APIs use **direct function handlers**, which are **resolved inline**.

```csharp
public string SayHello() => "Hello";           // MVC: wrapped in ControllerActionDescriptor
app.MapGet("/hello", SayHello);               // Minimal: direct delegate, no extra wrapping
```

---

### 3️⃣ ✅ **No Model Binding System (unless explicitly used)**

* Minimal APIs support **manual parsing** (e.g., from the query string or route).
* You opt in to `FromBody`, `FromQuery`, etc., only when needed.

```csharp
app.MapPost("/greet", (string name) => $"Hello, {name}!");
```

📉 MVC will look for:

* `[FromBody]`, `[FromQuery]`, `[FromRoute]`
* Validation
* Custom binders

📈 Minimal APIs just inject values via parameters directly.

---

### 4️⃣ ✅ **No Filter Pipeline**

* MVC uses filters (`IActionFilter`, `IResultFilter`, etc.)
* Minimal APIs skip this unless you **explicitly add middleware**

Result: **faster execution** with fewer lifecycle hooks.

---

### 5️⃣ ✅ **Compiled Endpoint Tree (as of .NET 7/8)**

* Endpoints in Minimal APIs can be **analyzed at compile time** using source generators.
* This improves **routing performance** and **removes reflection-based lookups**.

---

## 🧪 Performance Benchmarks

* Microsoft's internal benchmarks show **Minimal APIs are up to 30–50% faster** in high-throughput scenarios.
* Especially noticeable with:

  * Short request/response lifecycles
  * Lightweight JSON operations
  * No business logic in controller base classes

---

## 📌 Summary Table

| Feature       | MVC Controllers              | Minimal APIs                   |
| ------------- | ---------------------------- | ------------------------------ |
| Routing       | Attribute + Convention-based | Inline and explicit            |
| Middleware    | Requires MVC pipeline        | Lightweight routing            |
| Filters       | Enabled by default           | Not used unless added manually |
| Metadata      | Heavy action descriptors     | Lightweight delegate handlers  |
| Model Binding | Fully automatic + validation | Manual unless configured       |
| Performance   | Slower                       | ✅ Faster                       |

---

## ✅ Conclusion

**Minimal APIs are faster** because they:

* Skip controller and action resolution
* Avoid filters and attribute parsing
* Execute less middleware
* Have lower memory and CPU overhead

> Ideal for: **microservices**, **IoT**, **high-performance APIs**, or **simple CRUD services**.

---

# ✅ 18. How to Add Authorization to an ASP.NET Core Project?

## 🧠 What Is Authorization?

**Authorization** determines **what a user is allowed to do** after being authenticated.
In ASP.NET Core, authorization is handled via **policies**, **roles**, and **claims-based logic**.

---

## 🧱 Step-by-Step: Adding Authorization

---

### ✅ 1️⃣ Enable Authentication (required before authorization)

You need to first **add authentication**, using:

* JWT Bearer
* Cookies
* Identity
* External providers (Google, Facebook, etc.)

### Example (JWT-based):

```csharp
builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        options.Authority = "https://your-auth-server/";
        options.Audience = "your-api";
    });
```

> 🔑 This is required before `UseAuthorization()` will work.

---

### ✅ 2️⃣ Register Authorization Middleware

```csharp
builder.Services.AddAuthorization(); // optional if using default policy

var app = builder.Build();

app.UseAuthentication();   // 🔑 MUST come before
app.UseAuthorization();
```

---

### ✅ 3️⃣ Protect Endpoints with `[Authorize]`

You can apply authorization globally, at controller, or at action level:

#### Controller-wide:

```csharp
[Authorize]
public class AdminController : Controller
{
    public IActionResult Index() => View();
}
```

#### Specific action:

```csharp
[Authorize]
public IActionResult Dashboard() => View();
```

#### Allow anonymous access:

```csharp
[AllowAnonymous]
public IActionResult Public() => View();
```

---

### ✅ 4️⃣ Role-Based Authorization

```csharp
[Authorize(Roles = "Admin")]
public IActionResult AdminPanel() => View();
```

> Ensure the authenticated user has a `role` claim.

---

### ✅ 5️⃣ Policy-Based Authorization

#### Define policy:

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("Over18", policy =>
        policy.RequireClaim("Age", "18", "19", "20", "21"));
});
```

#### Apply policy:

```csharp
[Authorize(Policy = "Over18")]
public IActionResult RestrictedArea() => View();
```

---

### ✅ 6️⃣ Minimal APIs Authorization

```csharp
app.MapGet("/secret", [Authorize] () => "Protected data");
```

Or with policy:

```csharp
app.MapGet("/admin", [Authorize(Roles = "Admin")] () => "Admin zone");
```

---

## 🛠️ Custom Authorization Handler (Advanced)

Implement `IAuthorizationHandler` for complex rules:

```csharp
public class AgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    public AgeRequirement(int age) => MinimumAge = age;
}

public class AgeHandler : AuthorizationHandler<AgeRequirement>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, AgeRequirement requirement)
    {
        if (context.User.HasClaim("Age", "18"))
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }
}
```

Register in DI:

```csharp
builder.Services.AddSingleton<IAuthorizationHandler, AgeHandler>();
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("Minimum18", policy =>
        policy.Requirements.Add(new AgeRequirement(18)));
});
```

---

## 📌 Summary Table

| Step                | Code / Description                               |
| ------------------- | ------------------------------------------------ |
| Register auth       | `AddAuthentication()`, then `AddAuthorization()` |
| Middleware order    | `UseAuthentication()` → `UseAuthorization()`     |
| Secure endpoints    | Use `[Authorize]` or `.RequireAuthorization()`   |
| Role-based checks   | `[Authorize(Roles = "...")]`                     |
| Policy-based checks | Define with `AddPolicy()`                        |
| Allow public access | Use `[AllowAnonymous]`                           |

---

# ✅ 19. How to Add Authorization to All Controller's Methods Except One?

## 🧠 Goal

You want to:

* **Protect** all actions in a controller by default.
* **Allow anonymous access** to **only one specific method**.

This is easily done using `[Authorize]` at the controller level, and `[AllowAnonymous]` on the exception.

---

## ✅ Step-by-Step Example

### 🔧 1. Apply `[Authorize]` at the controller level

This makes all actions **require authentication** by default:

```csharp
[Authorize]
public class AccountController : Controller
{
    public IActionResult Dashboard()
    {
        return View("Dashboard"); // 🔒 Requires auth
    }

    public IActionResult Settings()
    {
        return View("Settings"); // 🔒 Requires auth
    }

    // 👇 This one will be excluded
}
```

---

### 🔓 2. Use `[AllowAnonymous]` on the specific method

This allows **unauthenticated access** to that single method:

```csharp
[AllowAnonymous]
public IActionResult Login()
{
    return View("Login"); // 🔓 Public access
}
```

---

## ✅ Full Example

```csharp
[Authorize] // Applies to all actions
public class AccountController : Controller
{
    public IActionResult Profile() => View();        // 🔒 Protected
    public IActionResult Dashboard() => View();      // 🔒 Protected

    [AllowAnonymous]
    public IActionResult Login() => View();          // 🔓 Public
}
```

---

## 🔄 Minimal API Equivalent

```csharp
app.MapGroup("/account")
   .RequireAuthorization()
   .MapGet("/dashboard", () => "Dashboard")        // 🔒 Authorized
   .MapGet("/login", [AllowAnonymous] () => "Login"); // 🔓 Anonymous
```

---

## 📌 Summary

| Attribute          | Applies to       | Description                            |
| ------------------ | ---------------- | -------------------------------------- |
| `[Authorize]`      | Controller/class | Secures all actions in the controller  |
| `[AllowAnonymous]` | Action/method    | Overrides `[Authorize]` for one action |

> ✅ This is the **recommended and cleanest approach** for this scenario.

---

# ✅ 20. How Would You Implement Log-In Functionality in ASP.NET Core?

## 🧠 Overview

Login functionality involves:

1. **Receiving credentials**
2. **Validating the user**
3. **Generating a token or cookie**
4. **Returning authentication response**

We'll focus on a **JWT-based login** commonly used in APIs.

---

## 📦 Technologies

* ASP.NET Core 6/7/8
* Entity Framework Core (optional for user lookup)
* JWT Bearer tokens via `System.IdentityModel.Tokens.Jwt`

---

## 🔐 Step-by-Step: JWT Login Flow

---

### ✅ 1. Configure JWT Authentication in `Program.cs`

```csharp
var key = Encoding.UTF8.GetBytes("YourSuperSecretKey!");

builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = "MyApp",
            ValidAudience = "MyAppUsers",
            IssuerSigningKey = new SymmetricSecurityKey(key)
        };
    });

builder.Services.AddAuthorization();
```

---

### ✅ 2. Create a Login Request Model

```csharp
public class LoginRequest
{
    public string Username { get; set; }
    public string Password { get; set; }
}
```

---

### ✅ 3. Create an Endpoint for Log-In

```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    [HttpPost("login")]
    public IActionResult Login([FromBody] LoginRequest request)
    {
        // ✅ 1. Validate user (in real app, query database)
        if (request.Username != "admin" || request.Password != "1234")
            return Unauthorized("Invalid credentials");

        // ✅ 2. Create claims
        var claims = new[]
        {
            new Claim(ClaimTypes.Name, request.Username),
            new Claim(ClaimTypes.Role, "Admin")
        };

        // ✅ 3. Create JWT token
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("YourSuperSecretKey!"));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: "MyApp",
            audience: "MyAppUsers",
            claims: claims,
            expires: DateTime.UtcNow.AddHours(1),
            signingCredentials: creds);

        var tokenString = new JwtSecurityTokenHandler().WriteToken(token);

        // ✅ 4. Return token
        return Ok(new { token = tokenString });
    }
}
```

---

### ✅ 4. Protect Other Endpoints Using `[Authorize]`

```csharp
[Authorize]
[HttpGet("profile")]
public IActionResult Profile()
{
    var user = User.Identity?.Name;
    return Ok($"Hello, {user}!");
}
```

---

### ✅ 5. Call It From Frontend (e.g., using fetch)

```javascript
const response = await fetch('/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ username: 'admin', password: '1234' })
});

const data = await response.json();
localStorage.setItem('jwt', data.token);
```

Include the token in future requests:

```javascript
fetch('/api/protected', {
  headers: { 'Authorization': 'Bearer ' + localStorage.getItem('jwt') }
});
```

---

## 🔄 Alternative: Cookie-Based Auth (for MVC or Blazor)

* Use `AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)`
* Use `SignInAsync()` in controller
* Use `[Authorize]` as usual
* No frontend token needed (browser manages the cookie)

---

## 📌 Summary Table

| Step                      | Description                                    |
| ------------------------- | ---------------------------------------------- |
| 1. Receive credentials    | From a POST `/login` request                   |
| 2. Validate credentials   | Check username/password (ideally via database) |
| 3. Generate token         | Using `JwtSecurityTokenHandler`                |
| 4. Return token to client | Typically in response body                     |
| 5. Protect APIs           | Use `[Authorize]` and JWT middleware           |

---

# ✅ 21. Explain How JWT Tokens Work

## 🧠 What Is a JWT?

A **JWT (JSON Web Token)** is a compact, URL-safe token format used for **stateless authentication**.

It's digitally signed and **self-contained**, meaning it carries user information (claims) and can be **validated without querying the database**.

---

## 🧱 JWT Structure

A JWT consists of **three parts**, separated by dots (`.`):

```
HEADER.PAYLOAD.SIGNATURE
```

### 1️⃣ Header (Base64-encoded)

Describes the token type and signing algorithm:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

---

### 2️⃣ Payload (Base64-encoded)

Contains **claims** — user info, roles, expiration, etc.:

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "role": "Admin",
  "exp": 1717263600
}
```

---

### 3️⃣ Signature

Ensures the token has not been tampered with.
It's created using:

```
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

---

## 🔐 How JWT Tokens Work (Login Flow)

### 🔁 1. Client Sends Credentials

```http
POST /login
{
  "username": "john",
  "password": "1234"
}
```

---

### 🧠 2. Server Validates User and Generates Token

If credentials are correct:

* The server creates a JWT with user info
* It signs the token with a secret key
* It returns the token to the client

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

### 📲 3. Client Stores Token

* Typically in `localStorage`, `sessionStorage`, or a secure cookie.

---

### 🔐 4. Client Sends Token in Requests

For protected routes:

```http
GET /api/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

---

### ✅ 5. Server Verifies the Token

* Checks signature validity (using secret)
* Validates expiration and claims
* If valid → grants access
* If invalid → returns `401 Unauthorized`

---

## 🛠️ JWT in ASP.NET Core

### ✅ Add JWT Authentication in `Program.cs`

```csharp
builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = "MyApp",
            ValidAudience = "MyUsers",
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes("YourSuperSecretKey!"))
        };
    });
```

### ✅ Protect APIs with `[Authorize]`

```csharp
[Authorize]
[HttpGet("profile")]
public IActionResult GetProfile() => Ok("You're authenticated!");
```

---

## 📌 JWT Pros and Cons

| ✅ Pros                             | ⚠️ Cons                                |
| ---------------------------------- | -------------------------------------- |
| Stateless (no server-side session) | If stolen, token can be reused         |
| Portable (used across services)    | No server-side revocation by default   |
| Fast verification                  | Token size can grow with more claims   |
| Works well with SPA/mobile apps    | Must be securely stored (e.g., no XSS) |

---

## 🧪 Useful Claims in JWT

| Claim  | Description                      |
| ------ | -------------------------------- |
| `sub`  | Subject (user ID)                |
| `name` | Username or full name            |
| `role` | Role of the user (e.g., "Admin") |
| `exp`  | Expiration timestamp (UNIX)      |
| `iat`  | Issued at                        |
| `iss`  | Issuer                           |
| `aud`  | Audience                         |

---

## ✅ Summary

* JWT is a **self-contained**, signed token used for **stateless authentication**
* Contains user info (claims), expiration, and a signature
* Stored on the client and sent in the `Authorization` header
* Server verifies the token using a **secret key** on every request

---

# ✅ 22. Explain Refresh Tokens and How They Work

## 🧠 What Is a Refresh Token?

A **refresh token** is a **long-lived credential** used to **obtain a new JWT access token** without requiring the user to log in again.

It is stored securely and used **only when the short-lived JWT expires**.

> 🔁 **Access token** = short-lived (e.g., 15 minutes)
> 🔁 **Refresh token** = long-lived (e.g., 7 days or 30 days)

---

## 📦 Why Use Refresh Tokens?

* Improves **security** by limiting JWT lifetime
* Avoids forcing the user to log in frequently
* Enables **stateless authentication** with minimal server storage

---

## 🔄 Refresh Token Flow (Step-by-Step)

### 1️⃣ User logs in with credentials

* Server verifies credentials
* Server returns:

  * JWT access token (short-lived)
  * Refresh token (long-lived)

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "d2f3c21a4a8b4e35a..."
}
```

---

### 2️⃣ Client stores both tokens securely

* Access token → used in `Authorization` header
* Refresh token → stored in `HttpOnly` cookie or secure storage

---

### 3️⃣ When access token expires:

* Client sends a **refresh request** with the refresh token

```http
POST /auth/refresh
{
  "refreshToken": "d2f3c21a4a8b4e35a..."
}
```

---

### 4️⃣ Server validates the refresh token:

* Checks expiration
* Confirms it's still valid (e.g., in DB or in-memory)
* If valid:

  * Issues a new access token (and optionally a new refresh token)

---

### 5️⃣ Client receives new tokens and continues

---

## 🛠️ Refresh Tokens in ASP.NET Core (Simplified Example)

### 🔧 Model

```csharp
public class RefreshToken
{
    public string Token { get; set; }
    public string UserId { get; set; }
    public DateTime Expires { get; set; }
    public bool IsExpired => DateTime.UtcNow >= Expires;
}
```

---

### 🔐 When Logging In

```csharp
var refreshToken = new RefreshToken
{
    Token = Guid.NewGuid().ToString(),
    UserId = user.Id,
    Expires = DateTime.UtcNow.AddDays(7)
};

// Save to DB or cache

return Ok(new
{
    accessToken = jwtToken,
    refreshToken = refreshToken.Token
});
```

---

### 🔁 Refresh Endpoint

```csharp
[HttpPost("refresh")]
public IActionResult Refresh([FromBody] string refreshToken)
{
    var storedToken = _db.RefreshTokens.FirstOrDefault(t => t.Token == refreshToken);

    if (storedToken == null || storedToken.IsExpired)
        return Unauthorized();

    var newAccessToken = _jwtService.GenerateAccessToken(storedToken.UserId);
    var newRefreshToken = Guid.NewGuid().ToString();

    // Update DB with new refresh token
    storedToken.Token = newRefreshToken;
    storedToken.Expires = DateTime.UtcNow.AddDays(7);
    _db.SaveChanges();

    return Ok(new
    {
        accessToken = newAccessToken,
        refreshToken = newRefreshToken
    });
}
```

---

## 🧾 Where to Store Refresh Tokens?

| Storage Method      | Pros                                 | Cons                             |
| ------------------- | ------------------------------------ | -------------------------------- |
| **HttpOnly Cookie** | Safer from XSS attacks               | Must deal with CSRF protection   |
| **Local Storage**   | Easier to access with JS             | Vulnerable to XSS if not secured |
| **Database**        | Can revoke/expire tokens server-side | Adds DB overhead                 |
| **In-memory cache** | Fast lookup                          | Lost on server restart           |

---

## ✅ Summary

| Concept        | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| Access token   | Short-lived JWT used in each request                         |
| Refresh token  | Long-lived token used to request new access tokens           |
| Main benefit   | Improves security and user experience                        |
| Storage        | Stored securely (cookie, DB, localStorage)                   |
| Implementation | Requires secure generation, validation, and revocation logic |

---

# ✅ 23. How Would You Implement an Application That Allows Access to Certain Resources If a User Has Specific Permissions?

## 🧠 Goal

Implement **fine-grained authorization** based on **user-specific permissions**, not just roles.

> 🔒 Example:
>
> * `user1` can **ReadProducts**
> * `user2` can **EditUsers**
> * You want to restrict access to endpoints based on **custom permissions**

---

## ✅ Step-by-Step: Implement Permission-Based Authorization

---

## 1️⃣ Define the Permissions

Create a static class to hold permission constants:

```csharp
public static class Permissions
{
    public const string ReadProducts = "Permissions.ReadProducts";
    public const string EditUsers = "Permissions.EditUsers";
}
```

---

## 2️⃣ Store Permissions in User Claims

✅ When generating a JWT, include permissions as claims:

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.UserName),
    new Claim("Permission", Permissions.ReadProducts),
    new Claim("Permission", Permissions.EditUsers)
};

var token = new JwtSecurityToken(
    issuer: "MyApp",
    audience: "MyAppUsers",
    claims: claims,
    expires: DateTime.UtcNow.AddHours(1),
    signingCredentials: creds);
```

> ✅ Each permission is a separate `Permission` claim.

---

## 3️⃣ Create a Custom Authorization Requirement

```csharp
public class PermissionRequirement : IAuthorizationRequirement
{
    public string Permission { get; }

    public PermissionRequirement(string permission)
    {
        Permission = permission;
    }
}
```

---

## 4️⃣ Create a Custom Authorization Handler

```csharp
public class PermissionHandler : AuthorizationHandler<PermissionRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        PermissionRequirement requirement)
    {
        var hasPermission = context.User.Claims
            .Any(c => c.Type == "Permission" && c.Value == requirement.Permission);

        if (hasPermission)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

---

## 5️⃣ Register the Handler in `Program.cs`

```csharp
builder.Services.AddSingleton<IAuthorizationHandler, PermissionHandler>();
```

---

## 6️⃣ Define Policies for Each Permission

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("CanReadProducts", policy =>
        policy.Requirements.Add(new PermissionRequirement(Permissions.ReadProducts)));

    options.AddPolicy("CanEditUsers", policy =>
        policy.Requirements.Add(new PermissionRequirement(Permissions.EditUsers)));
});
```

---

## 7️⃣ Apply the Policy to Controllers or Actions

```csharp
[Authorize(Policy = "CanReadProducts")]
[HttpGet("products")]
public IActionResult GetProducts()
{
    return Ok("You have permission to read products.");
}

[Authorize(Policy = "CanEditUsers")]
[HttpPost("users/edit")]
public IActionResult EditUser()
{
    return Ok("You can edit users.");
}
```

---

## 🧪 Minimal API Example

```csharp
app.MapGet("/products", [Authorize(Policy = "CanReadProducts")] () => "Products data");
```

---

## 📌 Summary Table

| Step                                 | Action                                   |
| ------------------------------------ | ---------------------------------------- |
| 1. Define permissions                | Use constants or enums                   |
| 2. Add claims to user                | Include `"Permission"` claims in the JWT |
| 3. Create `PermissionRequirement`    | Custom requirement for each permission   |
| 4. Implement `PermissionHandler`     | Checks if user has the required claim    |
| 5. Register in DI                    | Register `IAuthorizationHandler`         |
| 6. Define policies                   | One policy per permission                |
| 7. Use `[Authorize(Policy = "...")]` | Protect endpoints accordingly            |

---

# ✅ 24. What Is `HostedService` Used For in ASP.NET Core?

## 🧠 What Is a Hosted Service?

A **Hosted Service** in ASP.NET Core is a background task that runs **alongside the web application** — outside the request/response cycle.

It implements the **`IHostedService`** interface or derives from **`BackgroundService`**, and is managed by the ASP.NET Core **generic host**.

> 🛠️ Hosted services are ideal for tasks that need to run continuously, periodically, or in response to startup/shutdown events.

---

## 🧱 Common Use Cases

| Use Case                         | Description                        |
| -------------------------------- | ---------------------------------- |
| 🧵 Long-running background tasks | e.g., queue processing, workers    |
| 🔄 Periodic jobs                 | e.g., cleanup tasks, status checks |
| 🌐 Polling external services     | e.g., API watchers, RSS fetchers   |
| 🔔 Scheduled notifications       | e.g., email reminders, alerts      |
| 💬 Message queue consumers       | e.g., RabbitMQ, Kafka consumers    |

---

## 🧪 Example: Basic Hosted Service

### ✅ Create a class implementing `BackgroundService`

```csharp
public class MyBackgroundWorker : BackgroundService
{
    private readonly ILogger<MyBackgroundWorker> _logger;

    public MyBackgroundWorker(ILogger<MyBackgroundWorker> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("Background service is starting.");

        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Working at: {time}", DateTimeOffset.Now);
            await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
        }

        _logger.LogInformation("Background service is stopping.");
    }
}
```

---

### ✅ Register the service in `Program.cs`

```csharp
builder.Services.AddHostedService<MyBackgroundWorker>();
```

---

## 🔁 Lifecycle Methods in `IHostedService`

If you implement `IHostedService` manually (not via `BackgroundService`), you define:

```csharp
public class MyWorker : IHostedService
{
    public Task StartAsync(CancellationToken cancellationToken)
    {
        // Initialization logic
        return Task.CompletedTask;
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        // Cleanup logic
        return Task.CompletedTask;
    }
}
```

---

## 🔐 Hosted Service with Dependency Injection

You can inject any service into the constructor:

```csharp
public MyWorker(ILogger<MyWorker> logger, IMyService service) { ... }
```

> The hosted service itself is **registered as singleton**, but injected services can be **scoped** via `IServiceScopeFactory`.

---

## ⚠️ Best Practices

* Always check `stoppingToken.IsCancellationRequested` in loops.
* Use `try/catch` inside `ExecuteAsync` to avoid crashing the host.
* For scoped dependencies (e.g., `DbContext`), use `IServiceScopeFactory`.

---

## 📌 Summary Table

| Feature             | Explanation                                       |
| ------------------- | ------------------------------------------------- |
| `IHostedService`    | Interface to implement background tasks           |
| `BackgroundService` | Base class with `ExecuteAsync()`                  |
| Lifetime            | Runs from app start to shutdown                   |
| Common use cases    | Background jobs, queue consumers, scheduled tasks |
| Registration        | Via `AddHostedService<T>()` in DI                 |

---

# ✅ 25. Explain the Difference Between `PeriodicTimer` and `await Task.Delay()`

## 🧠 Overview

Both `PeriodicTimer` and `Task.Delay()` are used for **delaying execution in asynchronous code**, especially in **background services**, but they differ significantly in **precision**, **control**, and **design purpose**.

---

## 🔁 1️⃣ `await Task.Delay(...)`

### 🧱 What It Is

* A general-purpose method that delays execution for a **specific duration**.
* Commonly used in loops to simulate periodic tasks.

### ✅ Example:

```csharp
while (!cancellationToken.IsCancellationRequested)
{
    DoWork();
    await Task.Delay(TimeSpan.FromSeconds(5), cancellationToken);
}
```

### ⚠️ Problem

* Delay happens **after** `DoWork()`, so total interval = `DoWork duration + delay`.
* Risk of **drift** over time.

---

## ⏱️ 2️⃣ `PeriodicTimer`

### 🧱 What It Is

* Introduced in **.NET 6**.
* A **specialized timer** for running periodic tasks with **accurate intervals**, regardless of task duration.

### ✅ Example:

```csharp
var timer = new PeriodicTimer(TimeSpan.FromSeconds(5));

while (await timer.WaitForNextTickAsync(cancellationToken))
{
    DoWork();
}
```

### 🧠 How It Works

* Automatically maintains a **steady interval** between ticks.
* If `DoWork()` takes too long, ticks may be **skipped**, avoiding backlog.
* Cleaner and safer for scheduled intervals.

---

## ⚖️ Side-by-Side Comparison

| Feature              | `Task.Delay()`                       | `PeriodicTimer`                             |
| -------------------- | ------------------------------------ | ------------------------------------------- |
| Delay behavior       | Manual delay after work              | Waits for next tick                         |
| Drift control        | ❌ Drift increases with task duration | ✅ Maintains accurate intervals              |
| Skips missed ticks   | ❌ No — just delays again             | ✅ Yes — does not queue missed ticks         |
| Cancellation support | ✅ via `CancellationToken`            | ✅ via `WaitForNextTickAsync(token)`         |
| Best use case        | Simple delays or one-off waits       | Accurate recurring tasks in background jobs |
| Introduced in        | .NET 4.5+                            | ✅ .NET 6 and later                          |

---

## 🛠️ When to Use What?

| Scenario                           | Recommended Option |
| ---------------------------------- | ------------------ |
| Run a task every fixed interval    | `PeriodicTimer`    |
| Delay after a task finishes        | `Task.Delay()`     |
| Need precise scheduling            | `PeriodicTimer`    |
| Need compatibility with .NET 5/4.8 | `Task.Delay()`     |

---

## 🧪 Example in a BackgroundService

### ❌ Using `Task.Delay`:

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    while (!stoppingToken.IsCancellationRequested)
    {
        DoWork();
        await Task.Delay(5000, stoppingToken); // includes work time + delay
    }
}
```

### ✅ Using `PeriodicTimer`:

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    var timer = new PeriodicTimer(TimeSpan.FromSeconds(5));

    while (await timer.WaitForNextTickAsync(stoppingToken))
    {
        DoWork(); // task duration doesn't affect next tick
    }
}
```

---

## ✅ Summary

* Use `**PeriodicTimer**` for **precise, recurring tasks** (introduced in .NET 6).
* Use `**Task.Delay**` for **simple delays**, especially in older .NET versions or when drift isn’t critical.

---

# ✅ 26. What Is HSTS?

## 🧠 Definition

**HSTS** stands for **HTTP Strict Transport Security**.
It’s a web security policy mechanism that forces browsers to **always use HTTPS** when communicating with your site — even if the user types `http://`.

> ✅ HSTS protects users from **man-in-the-middle (MITM) attacks** and **protocol downgrade attacks**.

---

## 🔐 How Does HSTS Work?

1. When a browser receives an HTTP response with an **`Strict-Transport-Security`** header, it remembers that:

   * This domain **must only be accessed via HTTPS**
   * For a specified **max duration**
2. Next time the user types `http://your-site.com`, the browser **automatically upgrades** it to `https://your-site.com` — **without asking the server**.

---

## 📄 Header Example

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

### Header breakdown:

* `max-age=31536000`: tells browser to enforce HTTPS for 1 year (in seconds)
* `includeSubDomains`: applies rule to all subdomains
* `preload`: indicates the domain wants to be included in [Chrome's HSTS preload list](https://hstspreload.org)

---

## 🛠️ Enable HSTS in ASP.NET Core

### ✅ In `Program.cs`:

```csharp
var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts(); // Enables HSTS middleware
}

app.UseHttpsRedirection();
```

### ✅ Optional: Configure options

```csharp
builder.Services.AddHsts(options =>
{
    options.Preload = true;
    options.IncludeSubDomains = true;
    options.MaxAge = TimeSpan.FromDays(365);
});
```

> 💡 You should **not use HSTS in development** — only in **production**!

---

## ❗ Important Notes

* **HSTS is enforced by the browser** — the server can’t force HTTPS if the client ignores it.
* **First request** may still be over HTTP unless:

  * The domain is on the HSTS preload list, or
  * You always redirect to HTTPS (`UseHttpsRedirection`)
* Don’t enable HSTS unless your site has **100% HTTPS coverage**

---

## 📌 Summary Table

| Feature                 | Description                                                         |
| ----------------------- | ------------------------------------------------------------------- |
| HSTS                    | HTTP Strict Transport Security                                      |
| Purpose                 | Forces client browsers to use HTTPS                                 |
| Header                  | `Strict-Transport-Security`                                         |
| ASP.NET Core middleware | `app.UseHsts()`                                                     |
| Environment restriction | ✅ Only in production                                                |
| Preloading              | Add your domain to the [HSTS preload list](https://hstspreload.org) |

---

# ✅ 27. How to Return a File from an API Endpoint in ASP.NET Core?

## 🧠 Overview

In ASP.NET Core, you can return a file from a controller or minimal API using the built-in **`File()`** helper method.

> You can return files like:
>
> * PDF, images, Excel, Word
> * Binary streams or physical files
> * Downloadable content

---

## 🧱 Common Return Types

| Return Method                         | Description                                |
| ------------------------------------- | ------------------------------------------ |
| `File(byte[], contentType, fileName)` | File from memory (e.g., generated content) |
| `File(Stream, contentType)`           | File from a stream                         |
| `PhysicalFile(...)`                   | File from physical path                    |
| `VirtualFile(...)`                    | File from wwwroot (virtual path)           |

---

## ✅ 1. Return a File from Disk

```csharp
[HttpGet("download")]
public IActionResult Download()
{
    var filePath = Path.Combine(Directory.GetCurrentDirectory(), "Files/report.pdf");
    var contentType = "application/pdf";
    var fileName = "report.pdf";

    return PhysicalFile(filePath, contentType, fileName);
}
```

---

## ✅ 2. Return a File from a `byte[]` Array

```csharp
[HttpGet("invoice")]
public IActionResult GetInvoice()
{
    byte[] fileBytes = System.IO.File.ReadAllBytes("Files/invoice.pdf");
    return File(fileBytes, "application/pdf", "invoice.pdf");
}
```

---

## ✅ 3. Return a File from a `Stream`

```csharp
[HttpGet("stream")]
public IActionResult DownloadStream()
{
    var filePath = "Files/image.png";
    var stream = new FileStream(filePath, FileMode.Open, FileAccess.Read);
    return File(stream, "image/png", "image.png");
}
```

---

## ✅ 4. Return File from `wwwroot` (Virtual File)

```csharp
[HttpGet("logo")]
public IActionResult GetLogo()
{
    return VirtualFile("/assets/logo.png", "image/png", "company-logo.png");
}
```

> 📁 Make sure the file is in `wwwroot/assets/` and `UseStaticFiles()` is enabled.

---

## ✅ 5. Return File with Minimal API

```csharp
app.MapGet("/download", () =>
{
    var filePath = Path.Combine(Directory.GetCurrentDirectory(), "Files/manual.pdf");
    return Results.File(filePath, "application/pdf", "manual.pdf");
});
```

---

## 📄 Common MIME Types

| File Type | Content Type                                                              |
| --------- | ------------------------------------------------------------------------- |
| PDF       | `application/pdf`                                                         |
| PNG       | `image/png`                                                               |
| JPEG      | `image/jpeg`                                                              |
| ZIP       | `application/zip`                                                         |
| CSV       | `text/csv`                                                                |
| DOCX      | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` |

---

## ⚠️ Tips

* Always validate file paths to avoid directory traversal attacks.
* Use streaming (`FileStream`) for large files to avoid memory issues.
* Return proper content types to allow preview/download in browsers.

---

## 📌 Summary

| Approach            | Use When                                     |
| ------------------- | -------------------------------------------- |
| `PhysicalFile()`    | Returning a file from disk                   |
| `File(byte[], ...)` | Returning generated content or database BLOB |
| `File(Stream, ...)` | Efficient large file transfer                |
| `VirtualFile()`     | Serving files from `wwwroot`                 |
| `Results.File()`    | In Minimal APIs                              |

---

# ✅ 28. How to Accept a File via an API Endpoint in ASP.NET Core?

## 🧠 Overview

In ASP.NET Core, you accept uploaded files using the `IFormFile` type.
The client must send the file using a **`multipart/form-data`** request.

> ✅ `IFormFile` represents a file sent with the HTTP request body.

---

## 🧱 Example: Single File Upload

### ✅ 1. Create the Endpoint

```csharp
[HttpPost("upload")]
public async Task<IActionResult> UploadFile(IFormFile file)
{
    if (file == null || file.Length == 0)
        return BadRequest("No file uploaded.");

    var path = Path.Combine(Directory.GetCurrentDirectory(), "Uploads", file.FileName);

    using (var stream = new FileStream(path, FileMode.Create))
    {
        await file.CopyToAsync(stream);
    }

    return Ok("File uploaded successfully.");
}
```

---

### 🔧 Notes

* `file.Length`: checks file size
* `file.FileName`: original filename from client
* `file.CopyToAsync(...)`: writes to disk

---

## 🧪 Sample Request (via Postman or Curl)

```
POST /upload
Content-Type: multipart/form-data
Body: form-data
Key: file (type: File) → choose a file to send
```

Or with `curl`:

```bash
curl -F "file=@example.pdf" https://localhost:5001/upload
```

---

## 📚 Accepting Multiple Files

```csharp
[HttpPost("upload/multiple")]
public async Task<IActionResult> UploadFiles(List<IFormFile> files)
{
    foreach (var file in files)
    {
        var path = Path.Combine("Uploads", file.FileName);
        using var stream = new FileStream(path, FileMode.Create);
        await file.CopyToAsync(stream);
    }

    return Ok("All files uploaded successfully.");
}
```

> 🔑 Make sure the frontend uses `input type="file" multiple` and the request is multipart.

---

## 📦 File Size and Limitations

By default, ASP.NET Core allows file uploads up to **30 MB**.

You can increase the limit in `Program.cs`:

```csharp
builder.Services.Configure<FormOptions>(options =>
{
    options.MultipartBodyLengthLimit = 100_000_000; // 100 MB
});
```

---

## 🛡️ Security Tips

| Recommendation                        | Reason                       |
| ------------------------------------- | ---------------------------- |
| Validate file extensions and size     | Prevent malicious uploads    |
| Never trust `file.FileName`           | May be spoofed               |
| Sanitize and rename uploaded files    | Avoid path traversal attacks |
| Store files in a non-public directory | Protect sensitive data       |

---

## 🧩 Minimal API Version

```csharp
app.MapPost("/upload", async (IFormFile file) =>
{
    var path = Path.Combine("Uploads", file.FileName);
    using var stream = new FileStream(path, FileMode.Create);
    await file.CopyToAsync(stream);
    return Results.Ok("Uploaded");
});
```

---

## 📌 Summary

| Feature               | API Usage                   |
| --------------------- | --------------------------- |
| Single file upload    | `IFormFile file`            |
| Multiple files        | `List<IFormFile> files`     |
| Content-Type required | `multipart/form-data`       |
| Copy to disk          | `file.CopyToAsync(stream)`  |
| File size limit       | Configure via `FormOptions` |

---

# ✅ 29. How to Access Query String Parameters in an API Endpoint?

## 🧠 What Is a Query String?

A **query string** is part of the URL that comes after the `?` and provides additional parameters:

```
GET /products?category=books&page=2
```

---

## 📘 Controller-Based API: Use Method Parameters

ASP.NET Core automatically binds query string values to **action method parameters**.

### ✅ Example:

```csharp
[HttpGet("search")]
public IActionResult Search(string category, int page = 1)
{
    return Ok($"Category: {category}, Page: {page}");
}
```

🧪 URL to call:

```
GET /api/search?category=books&page=2
```

> No attribute like `[FromQuery]` is needed — it works by default.

---

### 🔎 Optional: Use `[FromQuery]` Explicitly

```csharp
[HttpGet("filter")]
public IActionResult Filter([FromQuery] string sortBy, [FromQuery] int limit = 10)
{
    return Ok($"Sort: {sortBy}, Limit: {limit}");
}
```

---

## 📘 Model Binding with Complex Objects

You can bind query params to a class automatically:

```csharp
public class ProductFilter
{
    public string Category { get; set; }
    public int Page { get; set; }
}

[HttpGet("products")]
public IActionResult GetProducts([FromQuery] ProductFilter filter)
{
    return Ok($"Category: {filter.Category}, Page: {filter.Page}");
}
```

🧪 Request:

```
GET /products?category=books&page=3
```

---

## 📘 Minimal APIs: Just Use Parameters

### ✅ Example:

```csharp
app.MapGet("/search", (string keyword, int page) =>
{
    return Results.Ok($"Keyword: {keyword}, Page: {page}");
});
```

🧪 Call:

```
GET /search?keyword=aspnet&page=1
```

ASP.NET Core binds query parameters automatically.

---

### 🔎 Optional: Use `HttpContext.Request.Query`

Use this only when you want **manual access**:

```csharp
[HttpGet("manual")]
public IActionResult Manual()
{
    var query = HttpContext.Request.Query;
    var value = query["key"];
    return Ok($"Value: {value}");
}
```

---

## 📌 Summary

| Method                 | Description                        |
| ---------------------- | ---------------------------------- |
| `string param`         | Automatically binds query strings  |
| `[FromQuery] param`    | Explicitly binds from query        |
| `ComplexType`          | Binds to a class via `[FromQuery]` |
| `Request.Query["key"]` | Manual access via `HttpContext`    |
| Minimal API parameters | Directly bind from query           |

---

# ✅ 30. How to Get Current Logged-In User Information in ASP.NET Core?

## 🧠 Overview

In ASP.NET Core, once a user is authenticated, their identity is available via the `HttpContext.User` property — a `ClaimsPrincipal` object that contains all the **claims** from the token or cookie.

---

## ✅ 1. Access `User` in a Controller or Razor Page

```csharp
public class AccountController : Controller
{
    [Authorize]
    public IActionResult Profile()
    {
        var username = User.Identity?.Name; // from ClaimTypes.Name
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        var role = User.FindFirst(ClaimTypes.Role)?.Value;

        return Ok(new { Username = username, UserId = userId, Role = role });
    }
}
```

> 🧠 `User` is available by default in any controller or Razor page via `ControllerBase.User`.

---

## ✅ 2. Access Claims in Minimal API

```csharp
app.MapGet("/me", [Authorize] (HttpContext http) =>
{
    var user = http.User;

    var username = user.Identity?.Name;
    var email = user.FindFirst(ClaimTypes.Email)?.Value;

    return Results.Ok(new { username, email });
});
```

---

## ✅ 3. Common Claim Types

| Claim Type                         | Description                        |
| ---------------------------------- | ---------------------------------- |
| `ClaimTypes.Name`                  | Usually the username               |
| `ClaimTypes.Email`                 | User’s email address               |
| `ClaimTypes.Role`                  | User’s role                        |
| `ClaimTypes.NameIdentifier`        | User ID (usually a GUID or DB ID)  |
| Custom claim (e.g. `"Permission"`) | Any additional value your app uses |

---

## ✅ 4. Injecting `IHttpContextAccessor` (Outside Controllers)

Use this when accessing the user in **services or background tasks**:

### Register in `Program.cs`:

```csharp
builder.Services.AddHttpContextAccessor();
```

### Use in a service:

```csharp
public class MyService
{
    private readonly IHttpContextAccessor _accessor;

    public MyService(IHttpContextAccessor accessor)
    {
        _accessor = accessor;
    }

    public string GetUserId()
    {
        var user = _accessor.HttpContext?.User;
        return user?.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    }
}
```

---

## 📌 Summary

| Method                      | Use Case                              |
| --------------------------- | ------------------------------------- |
| `User.Identity.Name`        | Get current username                  |
| `User.FindFirst(...).Value` | Read specific claims                  |
| `HttpContext.User`          | Available in controllers & middleware |
| `IHttpContextAccessor`      | Use in services or outside pipeline   |

---

✅ Requires the user to be **authenticated** via:

* JWT tokens (`Authorization: Bearer`)
* Cookie authentication
* External providers (Google, Azure AD, etc.)

---

# ✅ 31. How to Inject Dependencies in Minimal APIs in ASP.NET Core?

## 🧠 Overview

Minimal APIs in ASP.NET Core allow **dependency injection (DI)** directly into the **route handler parameters** — no need for controllers or constructors.

> ✅ You can inject any registered service (e.g. DbContext, logger, custom service) by adding it as a **parameter** in the endpoint lambda.

---

## ✅ 1. Inject via Lambda Parameters

ASP.NET Core automatically resolves parameters from the **DI container** if their types are registered.

### 🧱 Example:

```csharp
app.MapGet("/greet", (ILogger<Program> logger) =>
{
    logger.LogInformation("Hello from Minimal API");
    return Results.Ok("Hello");
});
```

> The framework knows `ILogger<Program>` is a DI service, so it injects it automatically.

---

## ✅ 2. Inject Custom Services

### 🔧 Register a service in `Program.cs`

```csharp
builder.Services.AddScoped<IGreetingService, GreetingService>();
```

### ✅ Use it in a route

```csharp
app.MapGet("/hello", (IGreetingService greetingService) =>
{
    return Results.Ok(greetingService.SayHello());
});
```

```csharp
public interface IGreetingService
{
    string SayHello();
}

public class GreetingService : IGreetingService
{
    public string SayHello() => "Hello from service!";
}
```

---

## ✅ 3. Inject Multiple Services

```csharp
app.MapGet("/info", (
    ILogger<Program> logger,
    IMyService service,
    IConfiguration config) =>
{
    var message = config["AppSettings:Message"];
    logger.LogInformation("Accessed /info endpoint");
    return Results.Ok(new { data = service.GetData(), message });
});
```

---

## ✅ 4. Inject Services + Route Parameters

```csharp
app.MapGet("/product/{id}", (int id, IProductService productService) =>
{
    var product = productService.GetById(id);
    return product is not null ? Results.Ok(product) : Results.NotFound();
});
```

---

## 🛠 Advanced: Inject Scoped Services Manually (if needed)

```csharp
app.MapGet("/scoped", async (IServiceScopeFactory scopeFactory) =>
{
    using var scope = scopeFactory.CreateScope();
    var dbContext = scope.ServiceProvider.GetRequiredService<MyDbContext>();
    var items = await dbContext.Items.ToListAsync();
    return Results.Ok(items);
});
```

---

## 🔎 When DI Works

| Type             | Can be Injected? | Example                  |
| ---------------- | ---------------- | ------------------------ |
| `ILogger<T>`     | ✅ Yes            | Logging                  |
| `IConfiguration` | ✅ Yes            | Access appsettings.json  |
| `IOptions<T>`    | ✅ Yes            | Typed config settings    |
| `DbContext`      | ✅ Yes            | Data access with EF Core |
| Custom services  | ✅ Yes            | Must be registered in DI |
| HttpContext      | ✅ Yes            | `(HttpContext http)`     |

---

## ✅ Summary

| Method                     | Description                                 |
| -------------------------- | ------------------------------------------- |
| Lambda parameter injection | Most common in Minimal APIs                 |
| Register in `Services`     | Use `AddScoped`, `AddSingleton`, etc.       |
| Works for any DI type      | Including `ILogger`, `DbContext`, `Options` |
| Route params + services    | Combine route inputs with injected services |

---

# ✅ 32. How Would You Structure Your Minimal API Endpoints?

## 🧠 Overview

Minimal APIs are great for small or fast projects, but **structuring them properly** is essential for:

* Maintainability
* Separation of concerns
* Testability
* Scalability

---

## 🔧 Recommended Structure for Minimal APIs

### 📁 Suggested Folder Layout

```
/MyApp
│
├── Program.cs
├── Endpoints
│   ├── ProductsEndpoints.cs
│   ├── UsersEndpoints.cs
│
├── Services
│   ├── IProductService.cs
│   ├── ProductService.cs
│
├── Models
│   ├── Product.cs
│   ├── User.cs
│
├── Data
│   └── AppDbContext.cs
```

---

## 🧩 Step-by-Step Example

---

### ✅ 1. Define Routes in Dedicated Endpoint Files

#### 🗂 `Endpoints/ProductsEndpoints.cs`

```csharp
public static class ProductsEndpoints
{
    public static void MapProducts(this IEndpointRouteBuilder app)
    {
        app.MapGet("/products", async (IProductService service) =>
        {
            var products = await service.GetAllAsync();
            return Results.Ok(products);
        });

        app.MapPost("/products", async (Product product, IProductService service) =>
        {
            await service.CreateAsync(product);
            return Results.Created($"/products/{product.Id}", product);
        });
    }
}
```

---

### ✅ 2. Register Endpoints in `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapProducts(); // 👈 Extension method for clean grouping

app.Run();
```

---

### ✅ 3. Organize Business Logic in Services

#### 🗂 `Services/IProductService.cs`

```csharp
public interface IProductService
{
    Task<IEnumerable<Product>> GetAllAsync();
    Task CreateAsync(Product product);
}
```

#### 🗂 `Services/ProductService.cs`

```csharp
public class ProductService : IProductService
{
    private readonly AppDbContext _context;

    public ProductService(AppDbContext context) => _context = context;

    public async Task<IEnumerable<Product>> GetAllAsync() => await _context.Products.ToListAsync();

    public async Task CreateAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
    }
}
```

Register in `Program.cs`:

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
```

---

## ➕ Optional Enhancements

### ✅ Use `RouteGroupBuilder` (Grouping in .NET 7+)

```csharp
public static void MapProducts(this IEndpointRouteBuilder app)
{
    var group = app.MapGroup("/products").RequireAuthorization();

    group.MapGet("/", async (IProductService s) => Results.Ok(await s.GetAllAsync()));
    group.MapPost("/", async (Product p, IProductService s) => {
        await s.CreateAsync(p);
        return Results.Created($"/products/{p.Id}", p);
    });
}
```

### ✅ Add OpenAPI Documentation with Annotations

Use `Swashbuckle.AspNetCore` or `MinimalApis.Extensions` for better Swagger docs.

---

## 📌 Best Practices

| Practice                    | Why it matters                            |
| --------------------------- | ----------------------------------------- |
| Group endpoints by resource | Improves clarity (e.g., `/products`)      |
| Use extension methods       | Keeps `Program.cs` clean                  |
| Keep logic in services      | Promotes SRP and testability              |
| Use DTOs and validation     | Prevents over-posting, enforces contracts |
| Prefer async I/O            | Improves scalability                      |

---

## ✅ Summary

| Task              | Recommended Approach                      |
| ----------------- | ----------------------------------------- |
| Group routes      | Use `MapGroup()` or endpoint extensions   |
| Split logic       | Into `/Endpoints`, `/Services`, `/Models` |
| Avoid fat lambdas | Delegate business logic to services       |
| Secure endpoints  | Use `[Authorize]`, policies, route groups |
| Add validation    | Use FluentValidation or manual checks     |

---

# ✅ 33. What Is Output Caching in ASP.NET Core?

## 🧠 Definition

**Output Caching** is a performance optimization technique where **the result of a controller action or endpoint is cached** and **served to subsequent requests without re-executing the action**.

> 🔁 It avoids unnecessary computation and database access for frequently requested data.

---

## 🚀 Key Benefits

* ✅ Reduces server processing
* ✅ Improves response time
* ✅ Lowers database load
* ✅ Works well for **read-heavy endpoints**

---

## 🔧 Output Caching vs Response Caching

| Feature              | Output Caching                | Response Caching                    |
| -------------------- | ----------------------------- | ----------------------------------- |
| Stores entire output | ✅ Yes (server-side)           | ❌ No (relies on client/proxy)       |
| Based on parameters  | ✅ Yes (query, headers, route) | ❌ Limited                           |
| Supports policies    | ✅ Yes                         | ❌ Minimal                           |
| Middleware support   | ✅ .NET 7+ built-in            | ✅ Older alternative (cache headers) |

---

## 📦 Available In

* ✅ Built-in since **ASP.NET Core 7**
* ✅ Enhanced in **ASP.NET Core 8** with policies and fine-grained control

---

## ✅ Example: Enabling Output Caching

### 1️⃣ In `Program.cs`:

```csharp
builder.Services.AddOutputCache(); // Register output cache service

var app = builder.Build();

app.UseOutputCache(); // Add middleware to pipeline
```

---

### 2️⃣ Apply Output Caching to an Endpoint

#### ✅ Minimal API:

```csharp
app.MapGet("/time", () =>
{
    return DateTime.UtcNow.ToString("HH:mm:ss");
})
.CacheOutput(p => p.Expire(TimeSpan.FromSeconds(10)));
```

#### ✅ MVC Controller:

```csharp
[HttpGet]
[OutputCache(Duration = 10)]
public IActionResult GetTime()
{
    return Ok(DateTime.UtcNow.ToString("HH:mm:ss"));
}
```

🕒 Within 10 seconds, repeated calls return the **cached response** — after that, it's refreshed.

---

## 🔧 Customize Cache with Policies (ASP.NET Core 8+)

```csharp
builder.Services.AddOutputCache(options =>
{
    options.AddPolicy("ShortCache", policy => policy.Expire(TimeSpan.FromSeconds(5)));
});
```

Apply policy:

```csharp
app.MapGet("/products", GetProducts).CacheOutput("ShortCache");
```

---

## ❗ Caching Based on Input

You can vary caching based on:

* Query parameters
* Route parameters
* Request headers

```csharp
.CacheOutput(p => p
    .Expire(TimeSpan.FromSeconds(10))
    .SetVaryByQuery("category"));
```

---

## 🔥 Cache Invalidation

In .NET 8+, you can programmatically **evict cache entries**:

```csharp
var cache = app.Services.GetRequiredService<IOutputCacheStore>();
await cache.EvictByTagAsync("Products", CancellationToken.None);
```

---

## 📌 Summary Table

| Feature         | Description                                              |
| --------------- | -------------------------------------------------------- |
| Output Caching  | Caches the entire HTTP response                          |
| Location        | Server-side                                              |
| Supported since | .NET 7+                                                  |
| Used via        | `UseOutputCache()` + `[OutputCache]` or `.CacheOutput()` |
| Customization   | Supports policies, expiration, vary-by                   |
| Ideal for       | Read-heavy, infrequently changing endpoints              |

---

# ✅ 34. Difference Between `IMemoryCache` and `IDistributedCache` in ASP.NET Core

## 🧠 Overview

Both `IMemoryCache` and `IDistributedCache` are interfaces for **caching data** in ASP.NET Core, but they serve different purposes and work differently.

---

## 📦 1. `IMemoryCache`

### 🧱 Description

* Stores data **in memory (RAM)** of the **local web server**
* Fastest access time (in-process)
* Suitable for **single-instance apps**

### ✅ Example

```csharp
var cache = app.Services.GetRequiredService<IMemoryCache>();

cache.Set("key", "value", TimeSpan.FromMinutes(10));

if (cache.TryGetValue("key", out string result))
{
    Console.WriteLine(result); // "value"
}
```

---

### ➕ Pros

* ✅ Super fast (in-memory)
* ✅ Easy to use
* ✅ Supports expiration, eviction, and cache priority

### ➖ Cons

* ❌ Not shared between servers
* ❌ Cache is lost if the app restarts
* ❌ Not ideal for load-balanced or distributed environments

---

## 📦 2. `IDistributedCache`

### 🧱 Description

* Stores cache **outside the app** (e.g., Redis, SQL Server)
* Designed for **distributed systems or cloud apps**
* All servers in a web farm can **share** the cache

### ✅ Example

```csharp
var cache = app.Services.GetRequiredService<IDistributedCache>();

await cache.SetStringAsync("key", "value");

var value = await cache.GetStringAsync("key");
Console.WriteLine(value); // "value"
```

---

### Supported Providers

* Redis (`AddStackExchangeRedisCache`)
* SQL Server (`AddDistributedSqlServerCache`)
* Custom implementations

---

### ➕ Pros

* ✅ Shared across multiple instances
* ✅ Survives app restarts
* ✅ Suitable for microservices and cloud apps

### ➖ Cons

* ❌ Slightly slower (network latency)
* ❌ Requires external infrastructure (Redis, SQL, etc.)
* ❌ No eviction or expiration logic by default (must configure it)

---

## 📊 Side-by-Side Comparison

| Feature                     | `IMemoryCache`               | `IDistributedCache`               |
| --------------------------- | ---------------------------- | --------------------------------- |
| Storage location            | In-memory (in-process)       | External store (Redis, SQL, etc.) |
| Performance                 | ✅ Fastest                    | ❌ Slower (network involved)       |
| Availability across servers | ❌ No (per-instance only)     | ✅ Yes (shared/distributed)        |
| Scenarios                   | Single-server, local caching | Load-balanced apps, microservices |
| Configuration required      | None                         | Yes (external provider setup)     |
| Data type                   | Any object                   | `byte[]` or `string`              |
| Expiration policies         | ✅ Built-in                   | ❌ Must be handled manually        |

---

## 🧪 When to Use What?

| Scenario                              | Use This            |
| ------------------------------------- | ------------------- |
| Local-only, fast in-memory cache      | `IMemoryCache`      |
| Distributed app, multiple servers     | `IDistributedCache` |
| Caching ViewModels, small blobs       | `IMemoryCache`      |
| Session or token caching in web farms | `IDistributedCache` |

---

## ✅ Summary

* `IMemoryCache` is best for **simple, fast, per-instance caching**.
* `IDistributedCache` is best for **shared cache across multiple servers** (e.g., **Redis** in the cloud).
* Use **both** together if needed: fast access with `IMemoryCache`, backup consistency with `IDistributedCache`.

---

# ✅ 35. Explain How HybridCache / FusionCache Works

## 🧠 What Is a Hybrid Cache?

A **hybrid cache** combines the speed of **in-memory caching** (`IMemoryCache`) with the scalability of **distributed caching** (`IDistributedCache`).

> ✅ Goal: Get **fast local access** and **shared consistency across multiple instances**.

---

## 🚀 What Is FusionCache?

**FusionCache** is a high-performance .NET caching library that provides:

* ✅ **Hybrid caching** (memory + distributed)
* ✅ **Fail-safe** caching
* ✅ **Background factory refresh**
* ✅ **Cache stampede prevention**
* ✅ Strong `Task`-based API with timeout handling

It's available as a NuGet package:

```bash
dotnet add package ZiggyCreatures.FusionCache
```

---

## ⚙️ How Does It Work?

When using FusionCache with a distributed backend (e.g. Redis), the logic is:

1. Check **local memory cache** first → fastest
2. If not found or expired:

   * Try **distributed cache** (e.g. Redis)
3. If still not found:

   * Call the **factory method** (fetch from DB or API)
   * Store in both **memory** and **distributed** cache

> 🔁 This reduces pressure on Redis and avoids expensive DB/API calls.

---

## 🧱 FusionCache Setup in ASP.NET Core

### 1️⃣ Register in `Program.cs`

```csharp
builder.Services.AddMemoryCache();

builder.Services.AddFusionCache()
    .WithDefaultEntryOptions(new FusionCacheEntryOptions
    {
        Duration = TimeSpan.FromSeconds(30),
        IsFailSafeEnabled = true
    })
    .WithDistributedCache(new RedisCache(new RedisCacheOptions
    {
        Configuration = "localhost:6379"
    }));
```

---

### 2️⃣ Use in Services or Endpoints

```csharp
public class ProductService
{
    private readonly IFusionCache _cache;

    public ProductService(IFusionCache cache)
    {
        _cache = cache;
    }

    public async Task<Product> GetProductByIdAsync(int id)
    {
        return await _cache.GetOrSetAsync<Product>(
            $"product:{id}",
            async _ =>
            {
                // Simulate DB call
                await Task.Delay(100);
                return new Product { Id = id, Name = "Apple" };
            },
            TimeSpan.FromMinutes(5)
        );
    }
}
```

---

## 🧩 Key Features of FusionCache

| Feature                | Description                                                 |
| ---------------------- | ----------------------------------------------------------- |
| ✅ Hybrid cache         | Uses both in-memory and distributed cache                   |
| 🧠 Fail-safe           | Returns stale data if backend is unavailable (configurable) |
| 🔄 Auto-refresh        | Refreshes entries in background on expiration               |
| 🚫 Stampede protection | Prevents multiple concurrent calls for the same key         |
| 🔁 `GetOrSetAsync`     | Smart wrapper around common caching pattern                 |
| 🧪 Manual invalidation | Supports `RemoveAsync` and `SetAsync`                       |

---

## 🔄 What Problem Does It Solve?

| Problem                            | How FusionCache Helps                         |
| ---------------------------------- | --------------------------------------------- |
| Slow distributed cache lookups     | Uses in-memory cache for fast access          |
| Cache stampedes (multiple reloads) | Locks and deduplicates concurrent cache calls |
| Downtime in backend APIs           | Fail-safe mode returns stale cached data      |
| Consistency across instances       | Syncs with distributed cache (e.g., Redis)    |

---

## ✅ Summary

| Feature             | FusionCache                                    |
| ------------------- | ---------------------------------------------- |
| Based on            | `IMemoryCache` + `IDistributedCache`           |
| Use case            | Fast and resilient hybrid caching              |
| Key methods         | `GetOrSetAsync`, `TryGet`, `Remove`, etc.      |
| Providers supported | Memory, Redis, SQL Server, custom              |
| Ideal for           | APIs, microservices, high-traffic applications |

---

# ✅ 36. What Caching Patterns Do You Know?

## 🧠 What Is a Caching Pattern?

A **caching pattern** defines how and when to **store** and **retrieve** data from cache to improve performance, reduce latency, and prevent unnecessary processing or database access.

---

## 🔁 Common Caching Patterns

### 1️⃣ **Cache-Aside** (Lazy Loading)

#### 📚 Description

* The app **checks the cache first**
* If not found, it **loads from DB or source**, stores the result in cache, and returns it
* Most popular and flexible pattern

#### ✅ Use Case

* Database queries
* External API calls

#### 🧱 Example

```csharp
public async Task<Product> GetProductAsync(int id)
{
    var cacheKey = $"product:{id}";
    var cached = await _cache.GetStringAsync(cacheKey);

    if (cached != null)
        return JsonSerializer.Deserialize<Product>(cached);

    var product = await _db.Products.FindAsync(id); // Load from DB
    await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(product));

    return product;
}
```

---

### 2️⃣ **Write-Through Cache**

#### 📚 Description

* Data is **written to both** cache and database at the same time
* Ensures cache is **always up-to-date**

#### ✅ Use Case

* Read-heavy systems where cache must always be valid

#### ⚠️ Trade-off

* Slightly slower writes due to caching step

---

### 3️⃣ **Write-Behind Cache** (Deferred Write)

#### 📚 Description

* Data is written to **cache first**, and database updates are **delayed or queued**
* Useful for high-write environments where DB writes can be batched

#### ⚠️ Risk

* Potential data loss if cache is lost before DB write

---

### 4️⃣ **Read-Through Cache**

#### 📚 Description

* Cache is responsible for **loading** the data if it's missing
* Common in **distributed cache systems** (e.g., Redis with loaders)

> FusionCache's `GetOrSetAsync` behaves like a read-through cache.

---

### 5️⃣ **Refresh-Ahead (Proactive Refresh)**

#### 📚 Description

* Data is refreshed in the background **before** it expires
* Prevents latency spikes when cache expires

#### ✅ Use Case

* Time-sensitive data (stock prices, dashboards)

#### 🚀 Example with FusionCache

```csharp
new FusionCacheEntryOptions
{
    Duration = TimeSpan.FromMinutes(5),
    AllowBackgroundRefreshed = true
}
```

---

### 6️⃣ **Cache Invalidation**

#### 📚 Description

* Cache is **cleared/updated** when data changes
* Prevents serving stale data

#### ✅ Techniques

* Manual removal (`RemoveAsync`)
* Eviction by tag (e.g., in FusionCache)
* Pub/sub invalidation (e.g., Redis with events)

---

### 7️⃣ **Sliding vs Absolute Expiration**

| Strategy                | Description                                   |
| ----------------------- | --------------------------------------------- |
| **Absolute Expiration** | Item expires after a fixed time               |
| **Sliding Expiration**  | Item expires if not accessed in a time window |

```csharp
_memoryCache.Set("key", value, new MemoryCacheEntryOptions
{
    SlidingExpiration = TimeSpan.FromMinutes(5)
});
```

---

### 8️⃣ **Cache Stampede Prevention**

#### 📚 Description

* Prevents multiple threads from hitting the DB simultaneously on cache miss
* Solved by **locking**, **deduplication**, or libraries like **FusionCache**

---

## 🧩 Summary Table

| Pattern             | Description                              | Suitable For                    |
| ------------------- | ---------------------------------------- | ------------------------------- |
| Cache-Aside         | Load-on-demand, stores in cache          | Most general purpose use        |
| Write-Through       | Write to cache and DB simultaneously     | Strict consistency requirements |
| Write-Behind        | Delay DB writes                          | High-write scenarios            |
| Read-Through        | Cache handles data loading automatically | External caching services       |
| Refresh-Ahead       | Pre-warms data before expiration         | Realtime dashboards, analytics  |
| Invalidation        | Manual or event-based cache clearing     | Data updates or deletes         |
| Stampede Prevention | Deduplicates parallel cache misses       | High-concurrency endpoints      |

---

## ✅ Best Practice

* 🔁 **Use Cache-Aside** + **Expiration**
* ⚠️ **Never cache sensitive or user-specific data** unless scoped properly
* 📦 Use libraries like **FusionCache** for advanced patterns out of the box

---

# ✅ 37. What Is Rate Limiting Used For and What Types Do You Know?

## 🧠 What Is Rate Limiting?

**Rate Limiting** is a technique used to **control the number of requests** a client can make to an API within a specific time window.

> ✅ It helps **protect APIs** from abuse, denial-of-service attacks, and excessive load — ensuring **fair usage** among users.

---

## 🔒 Why Use Rate Limiting?

| Purpose                   | Description                                    |
| ------------------------- | ---------------------------------------------- |
| 🛡️ Protect the server    | Prevent overload or abuse                      |
| ⚖️ Fair usage enforcement | Ensure no single client consumes all resources |
| 🔐 Security               | Deter brute force or spam attacks              |
| 💰 Cost control           | Avoid excessive usage of paid services/APIs    |

---

## ⚙️ Types of Rate Limiting

ASP.NET Core (starting .NET 7) supports built-in **Rate Limiting middleware** via `Microsoft.AspNetCore.RateLimiting`.

### 1️⃣ **Fixed Window**

* ✅ Allows N requests per fixed time window (e.g., 100 requests every 1 minute)
* Simple and efficient

```csharp
options.AddPolicy("fixed", context =>
    RateLimitPartition.GetFixedWindowLimiter(
        partitionKey: context.Connection.RemoteIpAddress?.ToString(),
        factory: _ => new FixedWindowRateLimiterOptions
        {
            PermitLimit = 5,
            Window = TimeSpan.FromSeconds(10),
            QueueProcessingOrder = QueueProcessingOrder.OldestFirst,
            QueueLimit = 2
        }));
```

---

### 2️⃣ **Sliding Window**

* ✅ Smoother than fixed window
* Tracks requests within a sliding time window (e.g., last 60 seconds)

```csharp
options.AddPolicy("sliding", context =>
    RateLimitPartition.GetSlidingWindowLimiter(
        context.Connection.RemoteIpAddress?.ToString(),
        _ => new SlidingWindowRateLimiterOptions
        {
            PermitLimit = 5,
            Window = TimeSpan.FromSeconds(30),
            SegmentsPerWindow = 3
        }));
```

---

### 3️⃣ **Token Bucket**

* ✅ Allows bursts of traffic while controlling the average rate
* Each request **uses a token** from a “bucket”; tokens are replenished over time

```csharp
options.AddPolicy("token", context =>
    RateLimitPartition.GetTokenBucketLimiter(
        context.Connection.RemoteIpAddress?.ToString(),
        _ => new TokenBucketRateLimiterOptions
        {
            TokenLimit = 10,
            TokensPerPeriod = 1,
            ReplenishmentPeriod = TimeSpan.FromSeconds(1)
        }));
```

---

### 4️⃣ **Concurrency Limit**

* ✅ Controls how many **concurrent requests** can be processed at once
* Useful to prevent thread starvation or DB overload

```csharp
options.AddPolicy("concurrent", context =>
    RateLimitPartition.GetConcurrencyLimiter(
        context.Connection.RemoteIpAddress?.ToString(),
        _ => new ConcurrencyLimiterOptions
        {
            PermitLimit = 2,
            QueueLimit = 1,
            QueueProcessingOrder = QueueProcessingOrder.OldestFirst
        }));
```

---

## 🚀 How to Use Rate Limiting in ASP.NET Core

### ✅ 1. Add the middleware

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter = RateLimitPartition.GetFixedWindowLimiter(...);
    options.RejectionStatusCode = 429;
});
```

### ✅ 2. Add `UseRateLimiter` to the pipeline

```csharp
app.UseRateLimiter();
```

### ✅ 3. Apply to endpoints

```csharp
app.MapGet("/limited", () => "OK")
   .RequireRateLimiting("fixed");
```

---

## 📊 Summary Table

| Type              | Description                                  | Best For                         |
| ----------------- | -------------------------------------------- | -------------------------------- |
| Fixed Window      | N requests per fixed period                  | Simple limits                    |
| Sliding Window    | More accurate than fixed, reduces bursts     | Smoother traffic                 |
| Token Bucket      | Allows bursts, replenishes over time         | APIs with spiky usage            |
| Concurrency Limit | Limits number of parallel in-flight requests | DB-heavy or CPU-bound operations |

---

## 🛡️ Real-World Tips

* 🔑 Use **IP address** or **user ID** as partition key
* ⚠️ Always return `HTTP 429 Too Many Requests`
* ⏱️ Include `Retry-After` headers if possible
* 🔐 Combine with **authentication** and **API keys** for better control

---

# ✅ 38. How to Invalidate Data in `OutputCache` (ASP.NET Core)?

## 🧠 What Is Output Cache Invalidation?

When data changes (e.g., product updated or deleted), the **cached response** might become outdated.

**Invalidating** the output cache ensures:

* The next request generates fresh content
* You don't serve stale data

---

## 📦 Built-in `OutputCache` API (since .NET 7 / improved in .NET 8)

You can invalidate cache **manually** using:

```csharp
IOutputCacheStore
```

---

## ✅ Common Invalidation Strategies

---

### 1️⃣ Invalidate by Cache **Tag**

You can **tag** cached responses, and later **evict by tag**.

### 🧱 Define cache with a tag:

```csharp
app.MapGet("/products", GetProducts)
   .CacheOutput(p => p
       .Expire(TimeSpan.FromMinutes(10))
       .Tag("products"));
```

### 🧼 Evict by tag:

```csharp
public class ProductService
{
    private readonly IOutputCacheStore _cache;

    public ProductService(IOutputCacheStore cache)
    {
        _cache = cache;
    }

    public async Task UpdateProductAsync(Product p)
    {
        // Update DB logic here...

        await _cache.EvictByTagAsync("products", CancellationToken.None);
    }
}
```

---

### 2️⃣ Invalidate by Cache **Key**

If you know the exact cache key, you can evict it directly.

```csharp
await _cache.EvictAsync("product-detail-123", CancellationToken.None);
```

You must assign the key explicitly:

```csharp
.CacheOutput(p => p.SetCacheKey("product-detail-123"))
```

---

### 3️⃣ Use Short Expiration (when frequent updates)

If invalidation is too complex, you can set **shorter expiration windows**:

```csharp
.CacheOutput(p => p.Expire(TimeSpan.FromSeconds(30)));
```

---

## 🔄 Rebuild After Invalidate?

No — output cache will rebuild itself **on the next request** to the endpoint.

---

## 📌 Summary

| Method              | When to Use                                |
| ------------------- | ------------------------------------------ |
| `EvictByTagAsync()` | Invalidate all entries tagged (bulk clear) |
| `EvictAsync(key)`   | Invalidate a specific entry                |
| Short expiration    | If data changes too frequently             |

---

## 🧠 Tip

✅ Prefer **tags** for grouped resources like:

* `products`
* `user-profile:{id}`
* `invoices`

Use them consistently in `.CacheOutput()` to simplify invalidation.

---

# ✅ 39. How to Implement API Versioning in ASP.NET Core?

## 🧠 Why API Versioning?

API versioning allows you to **evolve your API without breaking existing clients**.

> ✅ Clients can choose the version they support, while you ship new features in newer versions.

---

## 🔧 1. Install NuGet Package

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Versioning
```

---

## 🧱 2. Register Versioning in `Program.cs`

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.ReportApiVersions = true;
});
```

---

## 🧩 3. Choose a Versioning Strategy

ASP.NET Core supports multiple methods:

| Method           | Description                      | Example                        |
| ---------------- | -------------------------------- | ------------------------------ |
| **URL Segment**  | Version is in route path         | `/api/v1/products`             |
| **Query String** | Version is in query param        | `/api/products?api-version=1`  |
| **Header**       | Version in custom request header | `api-version: 1.0`             |
| **Media Type**   | Version via `Accept` header      | `Accept: application/json;v=1` |

You can configure it like so:

```csharp
options.ApiVersionReader = ApiVersionReader.Combine(
    new QueryStringApiVersionReader("api-version"),
    new HeaderApiVersionReader("x-api-version")
);
```

---

## 🧱 4. Create Versioned Controllers

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0")]
public class ProductsV1Controller : ControllerBase
{
    [HttpGet]
    public IActionResult Get() => Ok("Product list - v1");
}
```

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("2.0")]
public class ProductsV2Controller : ControllerBase
{
    [HttpGet]
    public IActionResult Get() => Ok("Product list - v2");
}
```

> 🧠 Use route constraints: `v{version:apiVersion}`

---

## 🔄 Optional: Use `MapToApiVersion` for Method-Level Versioning

```csharp
[HttpGet]
[MapToApiVersion("1.0")]
public IActionResult GetV1() => Ok("v1");

[HttpGet]
[MapToApiVersion("2.0")]
public IActionResult GetV2() => Ok("v2");
```

---

## 🔍 5. Versioning in Minimal APIs

As of .NET 8, there's **experimental** or **manual** support. You can use different route groups:

```csharp
app.MapGroup("/api/v1").MapGet("/products", GetV1Handler);
app.MapGroup("/api/v2").MapGet("/products", GetV2Handler);
```

> For now, controllers offer the most structured versioning support.

---

## 📦 Output Example (with `ReportApiVersions = true`)

```http
GET /api/products?api-version=1.0

Response Headers:
api-supported-versions: 1.0, 2.0
api-deprecated-versions: 1.0
```

---

## 📌 Best Practices

| Tip                                 | Why                                          |
| ----------------------------------- | -------------------------------------------- |
| Version breaking changes only       | Minor fixes should not require a new version |
| Avoid too many versions             | Maintainability decreases with each version  |
| Use header or media-type versioning | Keeps URLs clean and RESTful                 |
| Deprecate old versions gradually    | Give clients time to migrate                 |

---

## ✅ Summary

| Step                     | Task                                    |
| ------------------------ | --------------------------------------- |
| Install package          | `Microsoft.AspNetCore.Mvc.Versioning`   |
| Configure services       | `AddApiVersioning` in `Program.cs`      |
| Choose strategy          | URL, Query, Header, or MediaType        |
| Version your controllers | Use `[ApiVersion]` and versioned routes |
| Test responses           | Verify with `api-version` and headers   |

---

# ✅ 40. How to Add API Versioning Without Changing the URL?

## 🧠 Context

In some cases, you're not allowed to version the API via the **URL path** (`/api/v1/products`).
So you must **preserve existing routes** like:

```
GET /api/products
```

But still version the API under the hood.

---

## 🧩 Supported Alternatives (URL-Stable Versioning)

ASP.NET Core allows **non-URL versioning methods**:

| Strategy         | Where the version is provided    | Example                          |
| ---------------- | -------------------------------- | -------------------------------- |
| **Query String** | `?api-version=1.0`               | `/api/products?api-version=1.0`  |
| **Header**       | Custom header                    | `x-api-version: 1.0`             |
| **Media Type**   | Via `Accept` content-type header | `Accept: application/json;v=1.0` |

---

## ✅ 1. Configure API Versioning

### In `Program.cs`:

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.ReportApiVersions = true;

    options.ApiVersionReader = ApiVersionReader.Combine(
        new QueryStringApiVersionReader("api-version"),
        new HeaderApiVersionReader("x-api-version"),
        new MediaTypeApiVersionReader()
    );
});
```

> 👆 This allows the client to specify the version **outside of the URL**.

---

## ✅ 2. Use Same Route for Multiple Versions

```csharp
[ApiController]
[Route("api/products")]
[ApiVersion("1.0")]
public class ProductsV1Controller : ControllerBase
{
    [HttpGet]
    public IActionResult Get() => Ok("Products v1");
}
```

```csharp
[ApiController]
[Route("api/products")]
[ApiVersion("2.0")]
public class ProductsV2Controller : ControllerBase
{
    [HttpGet]
    public IActionResult Get() => Ok("Products v2");
}
```

🧪 Example requests:

```http
GET /api/products?api-version=1.0
GET /api/products (with header x-api-version: 2.0)
GET /api/products (with Accept: application/json;v=2.0)
```

---

## 🧪 3. Example with Media-Type Versioning

```http
Accept: application/json;v=2.0
```

The version is read from the `Accept` header.

---

## 🔍 Optional: Method-Level Versioning

Instead of separate controllers, you can use `[MapToApiVersion]`:

```csharp
[HttpGet]
[MapToApiVersion("1.0")]
public IActionResult GetV1() => Ok("v1 result");

[HttpGet]
[MapToApiVersion("2.0")]
public IActionResult GetV2() => Ok("v2 result");
```

Same route, but responds differently based on the version.

---

## 🧠 Summary

| Requirement                | ✅ Solution                                         |
| -------------------------- | -------------------------------------------------- |
| Don't change URL structure | Use header, query string, or media-type versioning |
| Support multiple versions  | Use `[ApiVersion]` with same route                 |
| Client version control     | Clients choose version via query/header/media      |
| Future-ready versioning    | Easy to extend without breaking existing clients   |

---

## ✅ Recap

To version an existing API without touching the URL:

* Use `QueryStringApiVersionReader`, `HeaderApiVersionReader`, or `MediaTypeApiVersionReader`
* Keep `[Route("api/resource")]` the same in all versions
* Decorate each controller with `[ApiVersion("x.y")]`

---

# ✅ 41. What Is Swagger Used For?

## 🧠 Definition

**Swagger** is a set of **open-source tools** built around the **OpenAPI specification** that allows developers to:

* 📖 **Document** RESTful APIs
* ✅ **Test** endpoints interactively
* 🚀 **Explore** available operations and models
* 📦 **Generate clients** and SDKs in multiple languages

> In ASP.NET Core, Swagger is typically integrated via the `Swashbuckle.AspNetCore` package.

---

## 🔧 What Does Swagger Provide?

| Feature                   | Description                                                                                     |
| ------------------------- | ----------------------------------------------------------------------------------------------- |
| 📝 API Documentation      | Auto-generates docs from controllers and models                                                 |
| 🧪 Interactive UI         | Test endpoints from browser (via Swagger UI)                                                    |
| 🔄 OpenAPI JSON Export    | Exports spec as machine-readable `.json`                                                        |
| 📦 Client Code Generation | Tools like NSwag or Swagger Codegen use the spec to generate SDKs in C#, TypeScript, Java, etc. |

---

## 📦 How to Set Up Swagger in ASP.NET Core

### ✅ 1. Install NuGet Package

```bash
dotnet add package Swashbuckle.AspNetCore
```

---

### ✅ 2. Configure in `Program.cs`

```csharp
builder.Services.AddEndpointsApiExplorer(); // Required for Minimal APIs
builder.Services.AddSwaggerGen();
```

```csharp
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(); // Optional config: c => c.SwaggerEndpoint(...)
}
```

---

### ✅ 3. Run and Access UI

After running the app:

```
https://localhost:5001/swagger
```

This shows an **interactive UI** for all your endpoints and schemas.

---

## 🧪 Example Swagger Output

```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "DogWalk API",
    "version": "v1"
  },
  "paths": {
    "/api/users": {
      "get": {
        "summary": "Returns list of users",
        "responses": {
          "200": {
            "description": "Success"
          }
        }
      }
    }
  }
}
```

---

## 🧩 Common Use Cases

| Use Case                  | Description                                       |
| ------------------------- | ------------------------------------------------- |
| 📚 API documentation      | Show methods, inputs, responses                   |
| 👨‍💻 Developer testing   | Test endpoints directly via Swagger UI            |
| 🔐 Token-based testing    | Authorize with JWT and test secured routes        |
| 🔄 SDK generation         | Generate client code with tools like NSwag        |
| 📤 Shareable API contract | Provide frontend teams a contract for integration |

---

## 🔐 Add JWT Support to Swagger UI

```csharp
builder.Services.AddSwaggerGen(c =>
{
    c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        In = ParameterLocation.Header,
        Description = "Enter 'Bearer {token}'",
        Name = "Authorization",
        Type = SecuritySchemeType.ApiKey
    });

    c.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference { Type = ReferenceType.SecurityScheme, Id = "Bearer" }
            },
            Array.Empty<string>()
        }
    });
});
```

---

## ✅ Summary

| Feature          | Benefit                                    |
| ---------------- | ------------------------------------------ |
| 📝 Documentation | Automatically shows all API endpoints      |
| 🧪 Testing       | Try requests with real data in the browser |
| 🔐 Auth Testing  | Test secure endpoints (JWT, OAuth2, etc.)  |
| 🔄 Spec Export   | Export OpenAPI `.json` file for codegen    |

---

# ✅ 42. How to Add Documentation of Endpoints, Models and Fields in Swagger?

> 🔍 Swagger (via OpenAPI spec) supports rich documentation directly from your code using:

* XML Comments
* Attributes
* Fluent API configuration

---

## 🧱 1. Documenting **Endpoints** (Controllers, Actions)

### ✅ Add XML Comments

First, enable XML comments in your `.csproj` file:

```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
  <NoWarn>$(NoWarn);1591</NoWarn> <!-- optional: ignore warnings for missing comments -->
</PropertyGroup>
```

Then, update Swagger configuration in `Program.cs`:

```csharp
builder.Services.AddSwaggerGen(c =>
{
    var xmlPath = Path.Combine(AppContext.BaseDirectory, "YourApi.xml");
    c.IncludeXmlComments(xmlPath);
});
```

### ✏️ Add summary to your controller/action:

```csharp
/// <summary>
/// Returns all users in the system.
/// </summary>
[HttpGet]
public IActionResult GetUsers()
{
    return Ok(_userService.GetAll());
}
```

This appears in Swagger UI under the `GET /users` endpoint.

---

## 🧱 2. Documenting **Models**

### ✅ Use XML Comments in your classes:

```csharp
/// <summary>
/// Represents a user in the system.
/// </summary>
public class UserDto
{
    /// <summary>
    /// The user's unique ID.
    /// </summary>
    public Guid Id { get; set; }

    /// <summary>
    /// The user's full name.
    /// </summary>
    public string FullName { get; set; }
}
```

These comments will appear in the Swagger UI model schema.

---

## 🧩 3. Using Data Annotations for Metadata

While XML comments describe purpose, attributes describe **constraints**:

```csharp
public class ProductDto
{
    [Required]
    [MaxLength(50)]
    public string Name { get; set; }

    [Range(0, 9999)]
    public decimal Price { get; set; }
}
```

Swagger will show:

* Required fields
* Value constraints
* Data types

---

## 🧪 4. Add Descriptions Manually via Fluent API (Advanced)

In `SwaggerGen` config, you can also customize descriptions:

```csharp
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "DogWalk API",
        Version = "v1",
        Description = "API for connecting dog owners and sitters"
    });
});
```

---

## 🔐 5. Document Authorization Requirements

Use `[Authorize]` or `AllowAnonymous`, and configure Swagger to display auth info (see Q41 for full setup).

You can also decorate endpoints with custom attributes:

```csharp
[Authorize(Roles = "Admin")]
/// <summary>
/// Deletes a user (Admin only).
/// </summary>
[HttpDelete("{id}")]
public IActionResult DeleteUser(Guid id) { ... }
```

---

## ✅ Summary

| What You’re Documenting | How To Do It                            |
| ----------------------- | --------------------------------------- |
| **Endpoints**           | XML comments on controller methods      |
| **Models**              | XML comments + DataAnnotations          |
| **Fields**              | `[Required]`, `[Range]`, `[MaxLength]`  |
| **API Info**            | `SwaggerDoc()` in `AddSwaggerGen`       |
| **Authentication**      | `[Authorize]` + `AddSecurityDefinition` |

---

# ✅ 43. How to Get a Connection String from Configuration?

## 📦 1. Define It in `appsettings.json`

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb;User Id=sa;Password=your_password;"
  }
}
```

---

## 🧱 2. Access It in `Program.cs` (or Startup)

```csharp
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
```

This retrieves the value from:

```json
"ConnectionStrings": {
  "DefaultConnection": "..."
}
```

---

## 🔧 3. Register the DB Context with the Connection String

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"))
);
```

> ✅ This is the most common pattern when using EF Core.

---

## 📦 4. Access from a Service or Class

### ✅ Option A: Inject `IConfiguration`

```csharp
public class MyService
{
    private readonly string _connString;

    public MyService(IConfiguration config)
    {
        _connString = config.GetConnectionString("DefaultConnection");
    }
}
```

---

### ✅ Option B: Bind to a Settings Class

#### In `appsettings.json`:

```json
"MyAppSettings": {
  "DefaultConnection": "Server=localhost;Database=MyDb;..."
}
```

#### POCO Class:

```csharp
public class MyAppSettings
{
    public string DefaultConnection { get; set; }
}
```

#### Register in `Program.cs`:

```csharp
builder.Services.Configure<MyAppSettings>(
    builder.Configuration.GetSection("MyAppSettings"));
```

#### Inject with `IOptions`:

```csharp
public class MyService
{
    private readonly string _connString;

    public MyService(IOptions<MyAppSettings> options)
    {
        _connString = options.Value.DefaultConnection;
    }
}
```

---

## ✅ Summary

| Method                        | Description                         |
| ----------------------------- | ----------------------------------- |
| `GetConnectionString("Name")` | Shortcut for `"ConnectionStrings"`  |
| `GetSection("...").Value`     | Use for custom structure            |
| `IOptions<T>` binding         | Strong-typed config binding         |
| Works with secrets/env vars   | Yes, supports layering and override |

---

Here’s your answer to question 43 in **Markdown**, explaining how to **get a connection string** from the `appsettings.json` configuration file in **ASP.NET Core**.

---

# ✅ 43. How to Get a Connection String from Configuration?

## 📦 1. Define It in `appsettings.json`

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb;User Id=sa;Password=your_password;"
  }
}
```

---

## 🧱 2. Access It in `Program.cs` (or Startup)

```csharp
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
```

This retrieves the value from:

```json
"ConnectionStrings": {
  "DefaultConnection": "..."
}
```

---

## 🔧 3. Register the DB Context with the Connection String

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"))
);
```

> ✅ This is the most common pattern when using EF Core.

---

## 📦 4. Access from a Service or Class

### ✅ Option A: Inject `IConfiguration`

```csharp
public class MyService
{
    private readonly string _connString;

    public MyService(IConfiguration config)
    {
        _connString = config.GetConnectionString("DefaultConnection");
    }
}
```

---

### ✅ Option B: Bind to a Settings Class

#### In `appsettings.json`:

```json
"MyAppSettings": {
  "DefaultConnection": "Server=localhost;Database=MyDb;..."
}
```

#### POCO Class:

```csharp
public class MyAppSettings
{
    public string DefaultConnection { get; set; }
}
```

#### Register in `Program.cs`:

```csharp
builder.Services.Configure<MyAppSettings>(
    builder.Configuration.GetSection("MyAppSettings"));
```

#### Inject with `IOptions`:

```csharp
public class MyService
{
    private readonly string _connString;

    public MyService(IOptions<MyAppSettings> options)
    {
        _connString = options.Value.DefaultConnection;
    }
}
```

---

## ✅ Summary

| Method                        | Description                         |
| ----------------------------- | ----------------------------------- |
| `GetConnectionString("Name")` | Shortcut for `"ConnectionStrings"`  |
| `GetSection("...").Value`     | Use for custom structure            |
| `IOptions<T>` binding         | Strong-typed config binding         |
| Works with secrets/env vars   | Yes, supports layering and override |

---

# ✅ 44. How Can You Deploy an ASP.NET Core Application?

ASP.NET Core is **cross-platform**, so you can deploy it on:

* 🐧 Linux (Ubuntu, Debian, etc.)
* 🪟 Windows Server
* ☁️ Cloud services (Azure, AWS, GCP, etc.)
* 🐳 Docker containers

---

## 🚀 1. Deployment Targets

| Target                              | Description                              |
| ----------------------------------- | ---------------------------------------- |
| **Azure App Service**               | PaaS for hosting .NET apps in minutes    |
| **IIS on Windows**                  | Host on-prem apps on Windows Server      |
| **Linux + Nginx**                   | Host via reverse proxy (Kestrel + Nginx) |
| **Docker**                          | Containerized deployment                 |
| **Kubernetes**                      | Scalable container orchestration         |
| **GitHub Pages / Vercel / Netlify** | Only for frontend apps (e.g., React)     |

---

## 🛠️ 2. Deployment Options

### ✅ Option A: **Self-Contained Deployment (SCD)**

Publishes the app with the entire .NET runtime (no need to install .NET on host).

```bash
dotnet publish -c Release -r linux-x64 --self-contained true -o ./publish
```

### ✅ Option B: **Framework-Dependent Deployment (FDD)**

Smaller, but requires .NET runtime on the server.

```bash
dotnet publish -c Release -o ./publish
```

---

## ☁️ 3. Azure App Service (Most Common)

1. Go to Azure Portal → Create App Service
2. Select **Runtime Stack: .NET 8**
3. Deploy via:

   * Visual Studio / GitHub Actions / Azure CLI / FTP / WebDeploy
4. Optional: Configure **CI/CD** with GitHub Actions

```bash
az webapp up --name MyAspApp --resource-group MyGroup --runtime "DOTNET|8.0"
```

---

## 🧭 4. Linux + Nginx + Kestrel

1. Publish your app:

```bash
dotnet publish -c Release -o /var/www/myapp
```

2. Create a systemd service for the app:

```ini
[Unit]
Description=My ASP.NET Core App

[Service]
WorkingDirectory=/var/www/myapp
ExecStart=/usr/bin/dotnet MyApp.dll
Restart=always
User=www-data

[Install]
WantedBy=multi-user.target
```

3. Configure **Nginx** reverse proxy:

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass         http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## 🐳 5. Deploy via Docker

### 🧱 Create Dockerfile:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

### 🚀 Build and run:

```bash
docker build -t myapp .
docker run -d -p 8080:80 myapp
```

---

## 🧪 6. Test Your Deployment

* Test with browser: `http://localhost:5000` or `http://yourdomain.com`
* Check logs: `journalctl -u myapp` or `docker logs`
* Use reverse proxies like Nginx or Apache for production use

---

## ✅ Summary

| Option            | Best For                                 |
| ----------------- | ---------------------------------------- |
| Azure App Service | Fast deployment with scaling + CI/CD     |
| IIS on Windows    | Enterprises using Windows infrastructure |
| Linux + Nginx     | Lightweight and flexible                 |
| Docker            | Cloud-native and reproducible            |
| Kubernetes        | Microservices or large-scale apps        |

---

# ✅ 45. How to Configure Logging in ASP.NET Core?

## 🧠 What Is Logging?

Logging is the practice of recording events, errors, warnings, and diagnostic messages during the execution of your application.

ASP.NET Core provides a **built-in logging framework** that is:

* Lightweight and extensible
* Supports **structured logging**
* Works with **multiple providers** (Console, Debug, File, Seq, Serilog, etc.)

---

## 🧱 1. Built-in Logging Providers

| Provider      | Description                        |
| ------------- | ---------------------------------- |
| Console       | Logs to terminal/console           |
| Debug         | Logs to Visual Studio Debug output |
| EventSource   | Windows EventSource                |
| EventLog      | Windows Event Log                  |
| Azure Monitor | Cloud logging                      |

> ✔️ You can also add third-party loggers like **Serilog**, **NLog**, **Seq**, etc.

---

## ⚙️ 2. Configure Logging in `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

// Clear default providers (optional)
builder.Logging.ClearProviders();

// Add desired providers
builder.Logging.AddConsole();
builder.Logging.AddDebug();

// Set minimum level globally
builder.Logging.SetMinimumLevel(LogLevel.Information);
```

---

## 📝 3. Configure Logging Levels in `appsettings.json`

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "MyAppNamespace": "Debug"
    }
  }
}
```

> 📌 You can fine-tune logging per namespace or class.

---

## 🧑‍💻 4. Use `ILogger<T>` in Your Services/Controllers

```csharp
public class UserService
{
    private readonly ILogger<UserService> _logger;

    public UserService(ILogger<UserService> logger)
    {
        _logger = logger;
    }

    public void CreateUser(User user)
    {
        _logger.LogInformation("Creating user: {Name}", user.Name);

        try
        {
            // Logic
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating user");
        }
    }
}
```

> ✅ Supports structured logging with `{}` placeholders.

---

## 🔍 5. Logging Levels

| Level         | Description                       |
| ------------- | --------------------------------- |
| `Trace`       | Most detailed (diagnostic)        |
| `Debug`       | Development-only info             |
| `Information` | High-level application events     |
| `Warning`     | Unexpected but recoverable issues |
| `Error`       | Failed operations                 |
| `Critical`    | System is unusable                |
| `None`        | No logging                        |

---

## 🧪 6. Add Third-Party Logging (e.g. Serilog)

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
```

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .CreateLogger();

builder.Host.UseSerilog();
```

---

## 📦 7. Example of Structured Log Output

```csharp
_logger.LogInformation("User {UserId} logged in at {Time}", user.Id, DateTime.UtcNow);
```

➡️ Output:

```
info: UserService[0]
      User 42 logged in at 2025-06-24T14:00:00Z
```

---

## ✅ Summary

| Task                      | How to Do It                                 |
| ------------------------- | -------------------------------------------- |
| Add basic logging         | Use `ILogger<T>` and `AddConsole()`          |
| Configure per environment | Use `appsettings.{Environment}.json`         |
| Customize per category    | Set `LogLevel` in `appsettings.json`         |
| Use structured logging    | Use `{placeholder}` format                   |
| Add file/cloud providers  | Use Serilog, NLog, Seq, Application Insights |

---





