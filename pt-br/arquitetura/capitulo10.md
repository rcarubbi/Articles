# Capítulo 10 – Anti-patterns arquiteturais que todo dev .NET já viu (ou cometeu)

Se tem algo que a experiência me ensinou é que “arquitetura elegante” é um conceito traiçoeiro.  
Muitas vezes, o que parece limpo, genérico e reutilizável…  
… é, na verdade, um código ilegível, acoplado e difícil de evoluir.

Neste capítulo, vamos encarar de frente os maiores anti-patterns que assolam codebases .NET — principalmente aquelas que dizem seguir DDD, mas que na prática só adicionam camadas e abstrações inúteis.

> Se doeu, é porque você já passou por isso (eu também 😅).

---

## 1. Application Service obeso (ou o novo Transaction Script disfarçado)

Você abre um `PaymentRequestService` e vê 700 linhas de código.  
Todo comportamento está lá: validação, autorização, enrich, log, chamadas externas, controle de transação, envio de evento…

Isso não é DDD.  
**Isso é um Transaction Script metido a besta.**

**Sinais de alerta:**

- Métodos com nomes genéricos como `ProcessAsync`.
- `if`s de todos os tipos de pagamento dentro da mesma classe.
- Injeta 12 serviços diferentes no construtor.

**🩹 Como evitar:**

- Use Application Services como orquestradores.
- Mova lógica para o domínio (entidades, VO, domain services).
- Separe comportamentos por estratégia.

---

## 2. Repositório genérico com LINQ exposto

Já falamos disso no Capítulo 7, mas vale repetir aqui:  
`GenericRepository<T>` com `.Find(Expression<Func<T,bool>>)` é um tiro no pé.

Você não protegeu nada — só criou um falso senso de encapsulamento.

**Sinais de alerta:**

- Controllers usando `.Find(x => x.IsActive && x.TenantId == ...)`
- Testes quebrando quando muda um campo no banco.
- Time de produto perguntando “por que a tela ficou vazia hoje?”

**Como evitar:**

- Crie repositórios específicos para cada agregado.
- Faça queries no domínio falarem a linguagem do negócio.

---

## 3. CQRS em tudo (até onde não precisa)

Separar leitura e escrita é útil.  
Mas CQRS full-blown com command bus, event sourcing e projection até pra alterar um nome de usuário é exagero.

**Sinais de alerta:**

- Criar um `CreateUserCommandHandler` para um CRUD básico.
- Três projetos pra fazer `GetUserById`.
- Queries que só servem para retornar 1 linha, mas envolvem pipeline de 5 steps.

**Como evitar:**

- Use CQRS onde o modelo de leitura e escrita diverge.
- Não force onde um CRUD direto resolve.

---

## 4. “Infra disfarçada de domínio”

Você olha uma entidade `Trade`, e encontra dentro dela:

```csharp
public ILogger Logger { get; set; }
public HttpClient Client { get; set; }
```

Sim. Já vi isso em produção.

**Sinais de alerta:**

- Entidades com lógica de envio de email.
- Value Object chamando API externa.
- Uso de `DateTime.Now` ou `Guid.NewGuid()` diretamente no domínio.

**Como evitar:**

- O domínio **não conhece** infraestrutura.
- Deixe que Application Services e handlers cuidem da execução externa.

---

## 5. “Arquitetura em camadas” com apenas uma camada útil

Você jura que está aplicando Clean Architecture.  
Tem os projetos: Domain, Application, Infrastructure, Api.

Mas 90% da lógica está no Application.  
O domínio é só POCO.  
E a infra só tem `DbContext`.

Isso não é Clean Architecture.  
**É um n-tier com nomes novos.**

**Como evitar:**

- Use o domínio para centralizar regras de negócio.
- Separe comportamentos, não apenas pastas.

---

## 6. Abstrações sem propósito

`IPaymentRequestService`, `IPaymentRequestServiceHandler`, `IPaymentRequestServiceProcessorHandlerFactory`…

Pra quê?

**Sinais de alerta:**

- Interface com apenas uma implementação.
- Camadas de abstração que não são substituídas em nenhum lugar.
- Testes que precisam de 10 mocks só pra instanciar uma classe.

**Como evitar:**

- Abstraia comportamento que varia.
- Prefira código direto a abstrações genéricas que ninguém entende.

---

## 7. “Orquestração” em controller ou worker

Controller que chama três serviços, faz enrich, transforma DTO, chama outro serviço e decide fluxo.

Esse cara não é controller. É um **orquestrador perdido**.

**Sinais de alerta:**

- Controller tem `try/catch`, `foreach`, `if`, `.Select(...)`, log, e `SaveChanges()`.
- Worker que processa direto do banco e envia email.

**Como evitar:**

- Crie Application Services para orquestrar.
- Deixe o controller simples.
- Use MediatR se quiser modularizar.

---

## Conclusão: Errar faz parte. Persistir no erro, não.

A maioria desses anti-patterns surge com boas intenções: DRY, SOLID, testabilidade…  
Mas sem entender o **porquê** das camadas e padrões, só estamos trocando complexidade explícita por complexidade acidental.

É melhor um sistema simples que funciona do que uma arquitetura "perfeita" que ninguém entende.

DDD, Clean Architecture, CQRS — tudo isso são ferramentas.  
Use com propósito. Questione o que não faz sentido.

E o mais importante: **volte e melhore. Sempre.**

**Próximo capítulo:**
Antes de encerrar essa jornada, ainda precisamos fazer um acerto de contas com as interpretações equivocadas que rondam qualquer conversa sobre arquitetura .NET.

No próximo capítulo, vamos falar sobre os **mitos** mais repetidos — e mais perigosos:

- Que MVC é uma arquitetura de 3 camadas.
- Que repositório é obrigatório pra toda entidade.
- Que design patterns devem ser usados como receita de bolo.
- E que citar SOLID basta para encerrar qualquer discussão.

Se você já ouviu (ou defendeu) alguma dessas ideias, o próximo capítulo é pra você.
