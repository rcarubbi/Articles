# 📚 Como Adicionar Suporte a Múltiplas Versões de API com Swagger no ASP.NET Core 🔄

Gerenciar múltiplas versões de uma API pode parecer um desafio, mas é algo essencial quando você precisa manter compatibilidade com clientes antigos enquanto continua evoluindo seu sistema. Neste artigo, vamos explorar como configurar o suporte a múltiplas versões de API no ASP.NET Core com Swagger (via Swashbuckle). Vamos direto ao ponto!

## 1. Configurando o Versionamento da API

Primeiro, adicione o pacote necessário:

```powershell
Install-Package Microsoft.AspNetCore.Mvc.Versioning
```

Depois, configure o versionamento no `Program.cs`:

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

**O que acontece aqui?**

- **DefaultApiVersion:** Define a versão padrão da API (1.0).
- **ReportApiVersions:** Adiciona informações sobre as versões no cabeçalho HTTP.
- **AssumeDefaultVersionWhenUnspecified:** Usa a versão padrão se nenhuma for informada.
- **ApiVersionReader:** Permite ler a versão da API via URL ou cabeçalho HTTP.
- **GroupNameFormat:** Agrupa as versões no Swagger com o formato `v1`, `v2`, etc.
- **SubstituteApiVersionInUrl:** Substitui corretamente o placeholder `{version}` na URL.

No fim das contas, você terá uma API que sabe lidar com múltiplas versões de forma elegante.

---

## 2. Criando Controladores com Suporte a Versões

Vamos adicionar um controlador com suporte a duas versões:

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

**O que acontece aqui?**

- **ApiVersion:** Define que o controlador suporta múltiplas versões.
- **MapToApiVersion:** Direciona métodos específicos para versões específicas.
- **Rota:** A versão é passada na URL com `{version:apiVersion}`.

**Importante:** Em uma arquitetura de versionamento real, é comum que endpoints da mesma rota em versões diferentes possam ter:

- **Parâmetros diferentes:** Uma versão pode exigir um parâmetro adicional ou usar um tipo diferente.
- **Respostas diferentes:** O formato ou os campos do DTO podem variar entre versões.
- **Comportamento diferente:** A lógica por trás do mesmo endpoint pode ser alterada.

No exemplo acima, ambas as versões retornam o mesmo tipo de resposta, mas é importante lembrar que, em cenários reais, isso pode mudar.

---

## 3. Configurando o Swagger para Versões de API

Agora, precisamos garantir que o Swagger exiba corretamente as versões. Para isso, criamos uma classe personalizada:

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

    private OpenApiInfo CaAgorapiVersionDescription description)
    {
        return new OpenApiInfo
        {
            Title = "Weather API",
            Version = description.ApiVersion.ToString(),
            Description = description.IsDeprecated ? "Esta versão foi descontinuada." : "Versão ativa da API"
        };
    }
}
```

Aqui, cada versão da API recebe sua própria documentação Swagger.

---

## 4. Configurando o Middleware Swagger

Por fim, adicionamos os endpoints Swagger no middleware:

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

### 5. Testando no Swagger

Rode sua aplicação e acesse `https://localhost:<porta>/swagger`. Lá, você verá cada versão listada separadamente. Clique e teste!

---

## Conclusão

Agora você tem múltiplas versões de API rodando no ASP.NET Core com suporte completo no Swagger. Esse setup garante que você possa manter compatibilidade com clientes antigos enquanto continua evoluindo suas APIs.

Para mais detalhes, confira o repositório oficial: [WebApiVersioningDemo](https://github.com/rcarubbi/WebApiVersioningDemo).