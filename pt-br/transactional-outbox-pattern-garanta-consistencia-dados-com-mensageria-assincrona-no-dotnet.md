# ✉️ Transactional Outbox Pattern: Garanta a Consistência de Dados com Mensageria Assíncrona no .NET 📬

Neste artigo, vamos explorar o **Transactional Outbox Pattern**, um padrão essencial para garantir a consistência de dados em sistemas que utilizam mensageria. Esse padrão é amplamente utilizado para situações onde é preciso garantir que uma mudança no banco de dados seja replicada com segurança em outros sistemas, como filas de mensagens. Neste exemplo, utilizaremos o **Amazon SQS** e a biblioteca **MassTransit** para implementar o Outbox Pattern. Vale lembrar que esse padrão é agnóstico ao serviço de mensageria e pode ser utilizado com RabbitMQ, Azure Service Bus, entre outros.

---

### O Problema

Imagine que você tem uma API de clientes. Ao criar um novo cliente, talvez seja necessário enviar um e-mail de confirmação ou acionar outras funcionalidades. A implementação ingênua seria adicionar essas chamadas diretamente ao código que cria o cliente. Isso pode tornar o código lento e complexo, especialmente em sistemas escaláveis ou distribuídos, pois criar um cliente e acionar processos dependentes simultaneamente pode comprometer o desempenho e a robustez da aplicação.

Uma solução eficiente é utilizar uma fila de mensagens para processar essas ações de maneira assíncrona. Isso permite adicionar o cliente ao banco de dados e retornar a resposta ao cliente imediatamente, enquanto as outras ações ocorrem em segundo plano.

### O Desafio de Consistência

Embora o uso de filas de mensagens resolva o problema de complexidade, ele introduz um novo desafio: **garantir que tanto o cliente quanto a mensagem sejam persistidos de forma consistente**. O grande problema é que a maioria dos bancos de dados e sistemas de mensageria não permite transações distribuídas ou commits em duas fases, o que torna difícil garantir a consistência entre as operações.

### Solução: Transactional Outbox Pattern

O Transactional Outbox Pattern resolve o problema de consistência ao dividir a operação em duas etapas:

1. **Criar o cliente e registrar o evento de criação** em uma tabela de outbox no banco de dados, numa única transação.
2. Um serviço secundário lê a tabela de outbox e publica os eventos na fila de mensagens.

Dessa forma, temos uma **transação atômica** entre a criação do cliente e a inserção do evento na tabela de outbox, garantindo que qualquer falha no envio para a fila seja tratada pelo próprio serviço de outbox.

### Implementação do Outbox Pattern com MassTransit e Amazon SQS

#### Configuração Inicial

Primeiro, adicionamos o **MassTransit** ao projeto. O MassTransit é uma biblioteca que abstrai o uso de diferentes filas e mecanismos de publicação/assinatura de mensagens. Ele permite trabalhar com RabbitMQ, SQS, Azure Service Bus e outros de forma agnóstica. No caso do SQS, o MassTransit cria automaticamente as filas necessárias.

```csharp
// Configuração do MassTransit no Program.cs
services.AddMassTransit(x =>
{
    x.UsingAmazonSqs((context, cfg) => 
    {
        cfg.Host("us-east-1");
    });
});
```

#### Criando o Outbox Pattern

Para implementar o Outbox Pattern, adicionamos uma nova tabela ao banco de dados chamada `Outbox`. Essa tabela armazena eventos que serão posteriormente enviados para a fila.

1. Modifique o `DbContext` para incluir as tabelas `InboxState`, `OutboxState` e `OutboxMessage`, que são necessárias para o funcionamento do Outbox Pattern.

```csharp
public class YourDbContext :
    DbContext
{
    public YourDbContext(DbContextOptions<YourDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.AddInboxStateEntity();
        modelBuilder.AddOutboxMessageEntity();
        modelBuilder.AddOutboxStateEntity();
    }
}
```

2. No serviço de criação de clientes, registre o evento de criação do cliente na tabela `Outbox`, ao invés de enviar diretamente para a fila. Assim, o processo de criação de cliente e o registro do evento são realizados numa transação única.

```csharp
public async Task AddCustomerAsync(Customer customer)
{
    await _dbContext.Customers.AddAsync(customer);
    await _dbContext.OutboxMessages.AddAsync(new OutboxMessage { /* dados do evento */ });
    await _dbContext.SaveChangesAsync();
}
```

3. Configure o MassTransit para ler a tabela `Outbox` e publicar os eventos na fila.

```csharp
services.AddMassTransit(x =>
{
    x.AddEntityFrameworkOutbox<YourDbContext>(cfg =>
    {
        cfg.UsePostgres();
        cfg.UseBusOutbox(); // Ativa o Outbox Pattern no MassTransit
    });
});
```

4. Execute as migrações para criar as novas tabelas.

```bash
dotnet ef migrations add AddOutbox
dotnet ef database update
```

#### Consumo dos Eventos

Finalmente, configure um worker para consumir as mensagens publicadas na fila. Esse worker escuta a fila e processa os eventos de forma independente.

---

### Vantagens do Outbox Pattern

1. **Consistência**: Garante que o cliente seja criado e o evento publicado sem depender de transações distribuídas.
2. **Resiliência**: Em caso de falhas na publicação, o evento pode ser reenviado posteriormente, mantendo a consistência dos dados.
3. **Desempenho**: O Outbox Pattern permite que as operações dependentes sejam processadas de maneira assíncrona, melhorando o desempenho do sistema.

### Considerações Finais

O Transactional Outbox Pattern é uma solução prática para aplicações distribuídas que necessitam de consistência entre banco de dados e fila de mensagens. Com o uso de ferramentas como MassTransit, é possível implementar o padrão de maneira eficiente, permitindo maior escalabilidade e resiliência na aplicação.
