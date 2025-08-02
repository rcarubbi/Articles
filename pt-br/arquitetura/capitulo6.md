# Capítulo 6 – Use Cases, Command Handlers e MediatR: o que separa clareza de confusão

Vamos ser sinceros: CQRS virou buzzword.

Todo mundo fala, pouca gente entende, e a maioria que tenta aplicar já começa pensando em event sourcing, projections, Kafka, e uma avalanche de complexidade que não faz o menor sentido pra um CRUD de contas a pagar.

Mas a ideia central é simples.  
Tão simples que, quando bem aplicada, parece que nem tem arquitetura.

---

## A essência do CQRS

CQRS significa: **Command Query Responsibility Segregation**.  
Em português claro: **separar quem lê de quem escreve**.

É só isso.

- **Command:** você altera o estado.  
  Exemplo: `CreateUser`, `TransferFunds`, `ApprovePayment`.
- **Query:** você consulta o estado.  
  Exemplo: `GetUserDetails`, `ListTransactions`.

O benefício? **Clareza.**

- Comandos têm efeitos colaterais.
- Queries não têm efeitos colaterais.

Com isso, você separa os dois mundos e evita aquela zona cinzenta onde um `Get()` também faz `Update()` por “conveniência”.

---

## Mas o que CQRS resolve de verdade?

- Evita efeitos colaterais acidentais em métodos de leitura.
- Deixa claro o que é leitura e o que é escrita.
- Permite otimizar cada lado separadamente:
  - Leitura pode usar queries diretas, projections ou Dapper.
  - Escrita pode ter regras complexas de negócio e persistência transacional.
- Escala de forma diferente: leitura costuma ser mais frequente e pode ser cacheada com agressividade.

---

## CQRS não exige Event Sourcing

Esse é o maior mito.

Você pode aplicar CQRS com Entity Framework e banco relacional **sem mexer na sua infraestrutura**.  
Não precisa de tópicos Kafka, não precisa versionar eventos, não precisa projetar nada.

Olha um exemplo básico:

```csharp
// Command
public record CreateTradeCommand(decimal Price, decimal Quantity);

// Command Handler
public class CreateTradeHandler
{
    public async Task Handle(CreateTradeCommand command)
    {
        var trade = new Trade(command.Price, command.Quantity);
        _repository.Add(trade);
        await _unitOfWork.Commit();
    }
}

// Query
public class GetTradeDetailsHandler
{
    public async Task<TradeDto> Handle(Guid tradeId)
    {
        return await _db.Trades
            .Where(t => t.Id == tradeId)
            .Select(t => new TradeDto(t.Id, t.Price, t.Quantity))
            .SingleAsync();
    }
}
```

Repare: usamos o mesmo banco, só separamos as responsabilidades.

---

## Mas não é overkill para sistemas pequenos?

Se você acha que precisa de CQRS completo pra tudo, **sim**, é overkill.

Mas aplicar a **ideia central** — comandos não retornam dados; queries não causam efeito colateral — já resolve 80% dos problemas.

O restante você evolui com o tempo.

Você pode usar EF no começo. Quando surgir necessidade real, extrair leitura para Dapper, usar projeções, ou até separar bancos.

**CQRS não é tudo ou nada. É uma evolução.**

---

## Onde o CQRS pode complicar

- Você exagera nos Command/Query Handlers e cria uma classe pra cada botão do sistema.
- Começa a criar pipelines complexos (cache, log, retry, metrics, etc) para cada handler.
- Usa MediatR com tanto comportamento global que você nem sabe mais onde o código executa.
- Faz a separação de leitura/escrita até no banco (write database vs. read replica) antes de precisar.

**CQRS mal aplicado vira arquitetura cerimonial.**

O segredo é não aplicar tudo de uma vez. **Vá separando quando o acoplamento começar a doer**.

---

## MediatR: amigo ou inimigo?

MediatR é um bom facilitador para CQRS. Ele te força a pensar em comandos e queries como classes distintas. Ele permite usar pipelines. Ele ajuda a manter handlers pequenos.

Mas o problema **não é o MediatR** — é como você usa.

Se você precisa de um diagrama pra entender o fluxo de um comando, você já passou do ponto.

Use MediatR **se ele ajudar**.  
Abandone se estiver escondendo complexidade com “mágica”.

---

## E como tudo isso se conecta?

- **Controllers** disparam comandos/queries
- **Command Handlers** implementam os casos de uso orquestrando domínio e infraestrutura
- **Entidades** encapsulam regras
- **Queries Handlers** leem direto de onde for melhor (EF, Dapper, Redis…)
- **Eventos** são publicados no final da transação via outbox, se necessário

---

## Conclusão

CQRS não é um framework. É uma forma de pensar.

**Separar leitura de escrita é um passo simples que traz clareza, previsibilidade e testabilidade.**

Se aplicado com moderação, CQRS te ajuda a crescer com o sistema, sem amarrar seus pés antes de começar a correr.

---

**Próximo capítulo:**  
Vamos falar sobre os **repositórios** — e por que muita gente ainda usa `GenericRepository<T>` achando que está aplicando DDD, quando na verdade está só criando um DAL com nome bonito.
