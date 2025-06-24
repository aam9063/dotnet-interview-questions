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

Perfect! Here's everything from question 2 in **English** and **Markdown format**, including a practical example and a flashcard at the end.

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

✅ For example, you must call `UseAuthentication()` **before** `UseAuthorization()` — otherwise, the user won’t be authenticated when authorization runs.

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

Perfect, here’s the full explanation in **English** and **Markdown format**, without flashcards.

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

  * You’re building a **REST API**
  * Your app has **complex routing**
  * You prefer **separation of concerns**

* ✅ Use **Razor Pages** if:

  * You’re building a **simple web UI**
  * You want a **quick and clean page-based model**
  * You’re creating **form-based pages** (CRUD)

---

Here’s the full response in **Markdown format** with examples in **C#**, answering:

> **"Name 3 ways to create middleware"**

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

Here's your answer in **Markdown format**, fully in English, with practical examples to explain:

> **"Explain how appsettings.json configuration layering works"**

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

## 🔧 How It’s Loaded in `Program.cs`

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

Here’s the full explanation of the difference between **Singleton**, **Scoped**, and **Transient** services in ASP.NET Core, written in **Markdown with C# examples**.

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
* You don’t need to share state at all.

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

Great question — this is a common topic in advanced ASP.NET Core interviews. Here’s the full explanation in **Markdown**, with practical code examples.

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

Here’s the full answer to question 9 in **Markdown format**, with examples in **ASP.NET Core**:

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



