# 🛠️ Exploring Clean Architecture: A Practical Guide 📖

Clean Architecture is a software design approach that aims to separate system responsibilities into well-defined layers. This modular structure makes the system easier to maintain and scale, promoting **separation of concerns** and facilitating code comprehension and modifications. Let's break down how each layer works and its importance.

## Key Concepts of Clean Architecture

1. **Architecture with Separation of Concerns**:
   Clean Architecture distributes application responsibilities into distinct layers, which helps reduce dependencies and makes maintenance and scalability easier. Each layer is responsible for a specific part of the application, making the code more organized and predictable.

2. **Focus on Domain**:
   The domain layer is the heart of the system, encapsulating **business logic** and essential elements like entities and business rules. It is independent of other layers, allowing better adherence to business requirements and making unit testing easier.

3. **CQRS (Command Query Responsibility Segregation)**:
   Implementing the CQRS pattern separates data reading and writing operations, optimizing performance for each type of operation. This pattern makes the code clearer and improves resource management, especially in high-load applications.

4. **Infrastructure Flexibility**:
   The infrastructure layer manages external integrations, such as databases and messaging systems, hiding implementation details from the rest of the application. This flexibility allows modifying technologies or integrations without impacting business logic.

5. **Presentation Layer for User Interaction**:
   The presentation layer serves as the interface for users to interact with the system, usually through RESTful APIs or gRPC. It should delegate business logic to the application layer, remaining thin and focused on user interaction.

6. **Dependency Injection**:
   Using dependency injection is crucial to maintain architecture integrity. It helps control dependencies between layers and allows the system to be configured flexibly, facilitating testing and maintenance.

### Practical Example: Layers and Functionality

Below, we detail each layer with practical examples:

### 1. Domain Layer

At the core of the architecture, the **Domain Layer** defines business entities and fundamental rules. This is where we encapsulate essential logic, such as in the example below with a `Webinar` entity:

```csharp
public class Webinar
{
    public Guid Id { get; private set; }
    public string Name { get; private set; }
    public DateTime ScheduledOn { get; private set; }

    public Webinar(string name, DateTime scheduledOn)
    {
        Id = Guid.NewGuid();
        Name = name;
        ScheduledOn = scheduledOn;
    }

    public void Reschedule(DateTime newDate)
    {
        ScheduledOn = newDate;
    }
}
```

Here, we also define **repository interfaces** and **custom exceptions**:

```csharp
public interface IWebinarRepository
{
    Webinar GetById(Guid id);
    void Add(Webinar webinar);
}

public class WebinarNotFoundException : Exception
{
    public WebinarNotFoundException(Guid webinarId)
        : base($"Webinar with ID {webinarId} was not found.")
    { }
}
```

### 2. Application Layer

The **Application Layer** orchestrates business rules and implements **use cases** of the system. This is where we apply the **CQRS** pattern, separating commands and queries to handle write and read operations more efficiently.

#### Example: Creating a Webinar with a Command and Handler

**`CreateWebinarCommand` Command:**

```csharp
public class CreateWebinarCommand
{
    public string Name { get; set; }
    public DateTime ScheduledOn { get; set; }
}
```

**`CreateWebinarCommandHandler` Handler:**

```csharp
public class CreateWebinarCommandHandler
{
    private readonly IWebinarRepository _repository;

    public CreateWebinarCommandHandler(IWebinarRepository repository)
    {
        _repository = repository;
    }

    public Guid Handle(CreateWebinarCommand command)
    {
        var webinar = new Webinar(command.Name, command.ScheduledOn);
        _repository.Add(webinar);
        return webinar.Id;
    }
}
```

#### Benefits of CQRS

Using CQRS in the application layer allows the system to handle read and write operations separately, improving performance and making data flow easier to understand.

### 3. Infrastructure Layer

The **Infrastructure Layer** handles external integrations, such as databases and messaging systems. It encapsulates the complexity of interacting with these systems, keeping business logic decoupled.

#### Example: Repository for Database Access

```csharp
public class WebinarRepository : IWebinarRepository
{
    private readonly AppDbContext _dbContext;

    public WebinarRepository(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public Webinar GetById(Guid id)
    {
        return _dbContext.Webinars.Find(id);
    }

    public void Add(Webinar webinar)
    {
        _dbContext.Webinars.Add(webinar);
        _dbContext.SaveChanges();
    }
}
```

By isolating database access logic, the infrastructure layer makes it easier to modify or replace external technologies without impacting the core system code.

### 4. Presentation Layer

The **Presentation Layer** provides an interface for users and other systems to interact with the application, often via REST APIs. This layer should be lightweight, delegating business logic to the application layer.

#### Example: API for User Interaction

```csharp
[ApiController]
[Route("api/[controller]")]
public class WebinarsController : ControllerBase
{
    private readonly IMediator _mediator;

    public WebinarsController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    public IActionResult CreateWebinar([FromBody] CreateWebinarCommand command)
    {
        var webinarId = _mediator.Send(command);
        return CreatedAtAction(nameof(GetWebinar), new { id = webinarId }, webinarId);
    }

    [HttpGet("{id}")]
    public IActionResult GetWebinar(Guid id)
    {
        var query = new GetWebinarByIdQuery { Id = id };
        var webinar = _mediator.Send(query);
        return Ok(webinar);
    }
}
```

By delegating logic to the application layer, this structure ensures the presentation layer focuses on **handling requests and sending responses**.

### Error Handling with Middleware

For a robust system, centralizing error handling is important. Using an `ExceptionHandlingMiddleware` allows catching all exceptions and returning consistent responses to the client:

```csharp
public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;

    public ExceptionHandlingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            context.Response.StatusCode = (int)HttpStatusCode.InternalServerError;
            await context.Response.WriteAsJsonAsync(new { Error = ex.Message });
        }
    }
}
```

This middleware captures exceptions and returns standardized responses, improving user experience and security.

### Dependency Registration in `Startup.cs`

To configure dependencies for each layer, we can add each necessary service in `Startup.cs`:

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Presentation Layer
        services.AddControllers()
                .AddApplicationPart(typeof(WebinarsController).Assembly);

        // Domain and Application Layer
        services.AddScoped<IWebinarRepository, WebinarRepository>();
        services.AddScoped<CreateWebinarCommandHandler>();
        services.AddScoped<GetWebinarByIdQueryHandler>();

        // Register Mediator (for CQRS)
        services.AddMediatR(typeof(CreateWebinarCommandHandler).Assembly);

        // Infrastructure Layer - External services, like DB context
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

        // Error Handling Middleware Registration
        services.AddTransient<ExceptionHandlingMiddleware>();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        else
        {
            app.UseExceptionHandler("/error");
            app.UseHsts();
        }

        // Adding Exception Handling Middleware
        app.UseMiddleware<ExceptionHandlingMiddleware>();

        app.UseHttpsRedirection();
        app.UseRouting();
        app.UseAuthorization();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
}
```

### Explanation of Each Registration

1. **Presentation Layer**:
   - `AddApplicationPart(typeof(WebinarsController).Assembly)`: Adds the assembly where the controllers are defined, necessary if they are in a separate project.

2. **Domain and Application Layer**:
   - `AddScoped<IWebinarRepository, WebinarRepository>()`: Registers the repository implementation for the `IWebinarRepository` interface, keeping the domain layer isolated.
   - `AddScoped<CreateWebinarCommandHandler>()` and `AddScoped<GetWebinarByIdQueryHandler>()`: Registers command and query handlers that implement the application's use cases.

3. **Mediator Registration**:
   - `AddMediatR(typeof(CreateWebinarCommandHandler).Assembly)`: Registers Mediator, used to implement the CQRS pattern. It automatically locates command and query handlers, managing their execution.

4. **Infrastructure Layer**:
   - `AddDbContext<AppDbContext>(...)`: Configures the database context, allowing the infrastructure layer to handle data persistence using Entity Framework Core.

5. **Error Handling Middleware**:
   - `UseMiddleware<ExceptionHandlingMiddleware>()`: Adds custom middleware for centralized error handling, ensuring consistent responses in case of exceptions.

### Conclusion

Clean Architecture promotes a modular and sustainable software structure. With well-defined layers, a system is easier to maintain and less prone to bugs
