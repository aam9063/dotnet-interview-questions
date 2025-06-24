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





