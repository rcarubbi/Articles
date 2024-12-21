# 📚 How to Add Support for Multiple API Versions with Swagger in ASP.NET Core 🔄

Managing multiple versions of an API might seem challenging, but it’s essential when you need to maintain compatibility with older clients while continuing to evolve your system. In this article, we’ll explore how to set up support for multiple API versions in ASP.NET Core using Swagger (via Swashbuckle). Let’s dive right in!

## 1. Configuring API Versioning

First, add the required package:

```powershell
Install-Package Microsoft.AspNetCore.Mvc.Versioning
```

Then, configure versioning in `Program.cs`:

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1);
    options.ReportApiVersions = true;
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new HeaderApiVersionReader("X-Api-Version"));
}).AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'V";
    options.SubstituteApiVersionInUrl = true;
});
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddTransient<IConfigureOptions<SwaggerGenOptions>, ConfigureSwaggerOptions>();
```

**What’s happening here?**

- **DefaultApiVersion:** Sets the default API version (1.0).
- **ReportApiVersions:** Adds version information to the HTTP response header.
- **AssumeDefaultVersionWhenUnspecified:** Uses the default version if none is specified.
- **ApiVersionReader:** Allows reading the API version via URL or HTTP header.
- **GroupNameFormat:** Groups versions in Swagger using the format `v1`, `v2`, etc.
- **SubstituteApiVersionInUrl:** Correctly substitutes the `{version}` placeholder in the URL.

In the end, you’ll have an API capable of elegantly managing multiple versions.

---

## 2. Creating Controllers with Version Support

Let’s add a controller supporting two versions:

```csharp
[ApiVersion(1)]
[ApiVersion(2)]
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
public class WeatherForecastController : ControllerBase
{
    private static readonly string[] Summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };

    private readonly ILogger<WeatherForecastController> _logger;

    public WeatherForecastController(ILogger<WeatherForecastController> logger)
    {
        _logger = logger;
    }

    [MapToApiVersion(1)]
    [HttpGet(Name = "GetWeatherForecast")]
    public IEnumerable<WeatherForecast> Get()
    {
        return Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)]
        }).ToArray();
    }

    [MapToApiVersion(2)]
    [HttpGet(Name = "GetWeatherForecast")]
    public IEnumerable<WeatherForecast> GetV2()
    {
        return Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            TemperatureC = Random.Shared.Next(0, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)]
        }).ToArray();
    }
}
```

**What’s happening here?**

- **ApiVersion:** Declares that the controller supports multiple versions.
- **MapToApiVersion:** Maps specific methods to specific API versions.
- **Route:** The version is passed in the URL using `{version:apiVersion}`.

**Important:** In a real versioning architecture, endpoints sharing the same route across different versions might have:

- **Different Parameters:** A version might require an additional parameter or use a different type.
- **Different Responses:** The DTO format or fields might vary between versions.
- **Different Behavior:** The logic behind the same endpoint might be altered.

In this example, both versions return the same response type, but in real-world scenarios, this could change.

---

## 3. Configuring Swagger for API Versions

Next, we need to ensure Swagger displays versions correctly. For that, we’ll create a custom class:

```csharp
public class ConfigureSwaggerOptions : IConfigureOptions<SwaggerGenOptions>
{
    private readonly IApiVersionDescriptionProvider _provider;

    public ConfigureSwaggerOptions(IApiVersionDescriptionProvider provider)
    {
        _provider = provider;
    }

    public void Configure(SwaggerGenOptions options)
    {
        foreach (var description in _provider.ApiVersionDescriptions)
        {
            options.SwaggerDoc(description.GroupName, CreateInfoForApiVersion(description));
        }
    }

    private OpenApiInfo CreateInfoForApiVersion(ApiVersionDescription description)
    {
        return new OpenApiInfo
        {
            Title = "Weather API",
            Version = description.ApiVersion.ToString(),
            Description = description.IsDeprecated ? "This version has been deprecated." : "Active API version"
        };
    }
}
```

Each API version gets its own Swagger documentation.

---

## 4. Configuring Swagger Middleware

Finally, we add Swagger endpoints to the middleware:

```csharp
app.UseSwaggerUI(options =>
{
    var descriptions = app.DescribeApiVersions();
    foreach (var description in descriptions)
    {
        options.SwaggerEndpoint($"/swagger/{description.GroupName}/swagger.json", description.GroupName.ToUpperInvariant());
    }
});
```

---

## 5. Testing with Swagger

Run your application and access `https://localhost:<port>/swagger`. There, you’ll see each version listed separately. Click and test!

---

## Conclusion

Now you have multiple API versions running in ASP.NET Core with full Swagger support. This setup ensures you can maintain compatibility with legacy clients while continuing to evolve your APIs.

For more details, check out the official repository: [WebApiVersioningDemo](https://github.com/rcarubbi/WebApiVersioningDemo).

