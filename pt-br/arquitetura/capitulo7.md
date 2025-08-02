# Capítulo 7 – Repositórios que respeitam o domínio (e evitam vazamento de infra)

Se tem um padrão que apanha muito na prática é o tal do Repository.  
Ele aparece em toda arquitetura “DDD-like”, geralmente como um `IRepository<T>` com meia dúzia de métodos (`Add`, `Update`, `Delete`, `GetById`, `GetAll`).

Mas vamos ser honestos: isso é um **DAL com outro nome**.

Se o seu repositório é só um wrapper genérico em cima do Entity Framework, ele não está servindo ao domínio. Está servindo à infraestrutura.  
E pior: está vazando detalhes da persistência para fora da camada que deveria estar protegida.

---

## O problema do `GenericRepository<T>`

Essa abstração parece útil. Reaproveitável. DRY.  
Mas o custo disso é altíssimo.

Imagine um `GenericRepository<Trade>` com método `FindByCondition(Expression<Func<Trade,bool>> condition)`.  
O que impede o application service de escrever uma query diretamente ali dentro, baseada em EF Core?

Nada.

O domínio passa a depender de expressões de LINQ, tipagem de EF, `DbSet`, `AsNoTracking`, `Include`, etc.

Resultado?  
Você **abstraiu o `DbContext`, mas espalhou EF para o domínio inteiro.**

---

## Repositório é um contrato do domínio, não um DSL de banco

Um bom repositório tem uma **interface específica para o agregado** que representa.  
E a assinatura dos métodos fala a **linguagem do domínio**.

Veja o contraste:

```csharp
// Interface genérica (anti-padrão)
public interface IRepository<T>
{
    Task<T?> GetByIdAsync(Guid id);
    Task<List<T>> FindByCondition(Expression<Func<T, bool>> condition);
}
```

```csharp
// Interface rica e explícita (preferível)
public interface ITradeRepository
{
    Task<Trade?> GetPendingByIdAsync(Guid tradeId);
    Task<List<Trade>> GetByFirmAndStatus(string firm, TradeStatus status);
    void Add(Trade trade);
}
```

Percebe a diferença?

O repositório agora **conhece o negócio**. Ele existe para **atender casos de uso específicos**.

E com isso, você ganha:

- Intellisense mais útil
- Testes mais fáceis (mock de métodos claros)
- Menos vazamento de infra
- Evolução mais segura (você não quebra tudo ao mudar um campo no banco)

---

## Repositório é para agregados, não para entidades avulsas

No DDD, o repositório representa um **Aggregate Root**.

Isso quer dizer:  
Você **não precisa de repositório para tudo.**  
Você não precisa de um `CustomerRepository`, `AddressRepository`, `PhoneRepository`...

Você precisa de `CustomerRepository` se **Customer é um Aggregate Root** que contém `Address` e `Phone`.

A operação é sempre no agregado inteiro. Nunca em pedaços.

```csharp
var customer = await _customerRepository.GetByEmail(email);
customer.UpdatePhone(newPhone);
await _unitOfWork.Commit();
```

Isso é o modelo funcionando como o DDD propõe.

---

## Como testar repositórios?

Você pode manter o repositório com EF por baixo. Mas:

- Evite deixar ele exposto no domínio (ex: **não injete** `_dbContext` em handler)
- Escreva testes unitários para os métodos do repositório (mockando o DbContext ou usando EF in-memory)
- Escreva testes de integração para garantir que as queries refletem o modelo real

> Mas não caia na armadilha de mockar LINQ com `Expression<Func<T,bool>>`.  
> Teste as queries de verdade ou mantenha-as como **privadas no repositório**.

---

## Dica prática: agrupe queries em ReadModels quando fizer sentido

Nem tudo precisa ir para o repositório do domínio.

Se sua query serve só para exibir dados na tela (ex: um dashboard), crie um `ITradeReadModel` separado.  
Nele, você pode usar Dapper, LINQ, joins — o que quiser — **sem poluir o modelo de domínio**.

É o espírito do CQRS:

- O **domínio cuida da escrita**,
- Os **ReadModels cuidam da leitura**.

---

## Conclusão

Repositórios não são para reutilização genérica.  
Eles são **contratos do domínio com a persistência**, e devem **falar a linguagem da sua aplicação.**

Ao projetar bem o repositório, você:

- Protege o domínio de detalhes de infra
- Ganha confiança ao refatorar
- Reduz o acoplamento implícito
- E prepara o terreno para evoluir sem dor

---

**Próximo capítulo:**  
E se estamos falando de proteger o modelo, então precisamos falar de um dos maiores aliados nessa missão: os **Value Objects**.
