# 🛠️ Explorando a Arquitetura Limpa: Um Guia Prático 📖

A Arquitetura Limpa (Clean Architecture) é uma abordagem de design de software que visa separar as responsabilidades do sistema em camadas bem definidas. Essa estrutura modular torna o sistema mais fácil de manter e escalar, promovendo a **separação de preocupações** (separation of concerns) e facilitando a compreensão e modificações no código. Vamos detalhar como cada camada funciona e sua importância.

## Principais Conceitos da Arquitetura Limpa

1. **Arquitetura com Separação de Preocupações**:
   A Clean Architecture distribui as responsabilidades da aplicação em camadas distintas, o que ajuda a reduzir dependências e facilita a manutenção e escalabilidade. Cada camada é responsável por uma parte específica da aplicação, tornando o código mais organizado e previsível.

2. **Foco no Domínio**:
   A camada de domínio é o coração do sistema, encapsulando a **lógica de negócios** e os elementos essenciais, como entidades e regras de negócio. Ela é independente de outras camadas, permitindo maior aderência aos requisitos de negócio e facilitando a criação de testes unitários.

3. **CQRS (Command Query Responsibility Segregation)**:
   A implementação do padrão CQRS separa as operações de leitura e escrita de dados, otimizando o desempenho para cada tipo de operação. Esse padrão torna o código mais claro e melhora o gerenciamento de recursos, especialmente em aplicações de alta carga.

4. **Flexibilidade da Infraestrutura**:
   A camada de infraestrutura gerencia integrações externas, como banco de dados e sistemas de mensageria, escondendo os detalhes de implementação do resto da aplicação. Essa flexibilidade permite modificar tecnologias ou integrações sem impactar a lógica de negócio.

5. **Camada de Apresentação para Interação do Usuário**:
   A camada de apresentação serve como interface para o usuário interagir com o sistema, geralmente por meio de APIs RESTful ou gRPC. Ela deve delegar a lógica de negócio para a camada de aplicação, mantendo-se fina e focada na interação do usuário.

6. **Injeção de Dependência (Dependency Injection)**:
   O uso de injeção de dependência é crucial para manter a integridade da arquitetura. Ele ajuda a controlar as dependências entre camadas e permite que o sistema seja configurado de forma flexível, favorecendo testes e manutenção.

### Exemplo Prático: Camadas e Funcionalidades

A seguir, detalhamos cada camada com exemplos práticos:

### 1. Camada de Domínio

No centro da arquitetura, a **Camada de Domínio** define as entidades de negócio e as regras fundamentais. É nela que encapsulamos a lógica essencial, como no exemplo abaixo com uma entidade `Webinar`:

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

Aqui, também definimos **interfaces de repositórios** e **exceções personalizadas**:

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

### 2. Camada de Aplicação

A **Camada de Aplicação** orquestra as regras de negócio e implementa os **casos de uso** do sistema. É aqui que aplicamos o padrão **CQRS**, separando comandos e consultas para gerenciar operações de escrita e leitura de forma mais eficiente.

#### Exemplo: Criar Webinar com Comando e Manipulador

**Comando `CreateWebinarCommand`:**

```csharp
public class CreateWebinarCommand
{
    public string Name { get; set; }
    public DateTime ScheduledOn { get; set; }
}
```

**Manipulador `CreateWebinarCommandHandler`:**

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

#### Benefícios do CQRS

O uso de CQRS na camada de aplicação permite que o sistema lide separadamente com operações de leitura e escrita, o que melhora o desempenho e facilita o entendimento do fluxo de dados.

### 3. Camada de Infraestrutura

A **Camada de Infraestrutura** lida com as integrações externas, como bancos de dados e sistemas de mensageria. Ela encapsula a complexidade de interação com esses sistemas, permitindo que a lógica de negócio permaneça desacoplada.

#### Exemplo: Repositório para Acesso ao Banco de Dados

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

Ao isolar a lógica de acesso ao banco de dados, a camada de infraestrutura facilita a modificação ou substituição de tecnologias externas sem impactar o código central do sistema.

### 4. Camada de Apresentação

A **Camada de Apresentação** oferece uma interface para que os usuários e outros sistemas possam interagir com a aplicação, frequentemente via APIs REST. Essa camada deve ser leve, delegando a lógica de negócio para a camada de aplicação.

#### Exemplo: API para Interação com Usuários

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

Ao delegar a lógica para a camada de aplicação, essa estrutura assegura que a camada de apresentação se concentre em **tratar requisições e enviar respostas**.

### Tratamento de Erros com Middleware

Para um sistema robusto, é importante centralizar o tratamento de erros. Usar um `ExceptionHandlingMiddleware` permite capturar todas as exceções e retornar uma resposta consistente para o cliente:

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

Esse middleware captura exceções e retorna respostas padronizadas, melhorando a experiência do usuário e a segurança.

### Registro de Dependências no `Startup.cs`

Para configurar as dependências de cada camada, podemos adicionar cada serviço necessário no `Startup.cs`:

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Camada de Apresentação
        services.AddControllers()
                .AddApplicationPart(typeof(WebinarsController).Assembly);

        // Camada de Domínio e Aplicação
        services.AddScoped<IWebinarRepository, WebinarRepository>();
        services.AddScoped<CreateWebinarCommandHandler>();
        services.AddScoped<GetWebinarByIdQueryHandler>();

        // Registro do Mediator (para CQRS)
        services.AddMediatR(typeof(CreateWebinarCommandHandler).Assembly);

        // Camada de Infraestrutura - Registro de serviços externos, como DB context
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

        // Registro do Middleware de Tratamento de Erros
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

        // Adicionando o Middleware de Tratamento de Exceções
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

### Explicação de Cada Registro

1. **Camada de Apresentação**:
   - `AddApplicationPart(typeof(WebinarsController).Assembly)`: Adiciona o assembly onde os controladores estão definidos, necessário caso estejam em um projeto separado.

2. **Camada de Domínio e Aplicação**:
   - `AddScoped<IWebinarRepository, WebinarRepository>()`: Registra a implementação do repositório para a interface `IWebinarRepository`, mantendo o isolamento da camada de domínio.
   - `AddScoped<CreateWebinarCommandHandler>()` e `AddScoped<GetWebinarByIdQueryHandler>()`: Registra os manipuladores de comandos e consultas que implementam os casos de uso da aplicação.

3. **Registro do Mediator**:
   - `AddMediatR(typeof(CreateWebinarCommandHandler).Assembly)`: Registra o Mediator, que é usado para implementar o padrão CQRS. Ele localiza automaticamente os manipuladores de comando e consulta, gerenciando a execução de cada um.

4. **Camada de Infraestrutura**:
   - `AddDbContext<AppDbContext>(...)`: Configura o contexto do banco de dados, permitindo que a camada de infraestrutura faça a persistência de dados usando o Entity Framework Core.

5. **Middleware de Tratamento de Exceções**:
   - `UseMiddleware<ExceptionHandlingMiddleware>()`: Adiciona o middleware personalizado para tratamento centralizado de erros, garantindo uma resposta consistente em caso de exceções.

### Conclusão

A Arquitetura Limpa promove uma estrutura de software modular e sustentável. Com camadas bem definidas, um sistema é mais fácil de manter e menos suscetível a bugs e erros de design. A separação de preocupações permite uma evolução contínua do sistema, garantindo flexibilidade e robustez ao longo do tempo.

Implementar a Arquitetura Limpa é um passo importante para desenvolver software de alta qualidade, especialmente em projetos complexos que exigem escalabilidade e facilidade de manutenção.
