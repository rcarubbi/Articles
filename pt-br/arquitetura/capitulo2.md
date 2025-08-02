# Capítulo 2 – O modelo anêmico vai te trair (e como saber quando fugir dele)

Se tem uma armadilha que parece inofensiva, mas que já derrubou muito dev bom, é o tal do **modelo anêmico**.  
Sabe aquela entidade que tem `Id`, `Name`, `CreatedAt`… e nada mais? Parece um DTO de banco de dados disfarçado de domínio.  
E é exatamente isso que ele é.

Eu sei que você já escreveu algo assim. Eu também. Às vezes por pressa. Às vezes por convenção. E muitas vezes por não querer “complicar demais” no começo do projeto.

Só que o barato sai caro.

---

## O que é um modelo anêmico — e por que ele é um problema

No fundo, o modelo anêmico é uma negação do paradigma orientado a objetos.  
Ele é um recipiente de dados que depende de uma outra classe (Service) para executar qualquer coisa de relevante.  
É como ter um carro que precisa de um mecânico do lado pra ligar o motor.

```csharp
public class PaymentRequest
{
    public Guid Id { get; set; }
    public decimal Amount { get; set; }
    public PaymentStatus Status { get; set; }
}
```

Você bate o olho e pensa: “Ok, simples, direto”.

Mas aí começa a surgir isso aqui:

```csharp
public class PaymentRequestService
{
    public void Approve(PaymentRequest request)
    {
        if (request.Amount <= 0)
            throw new InvalidOperationException("Invalid amount");

        request.Status = PaymentStatus.Approved;
    }
}
```

Veja o problema: nada impede outro pedaço de código de fazer isso diretamente:

```csharp
request.Status = PaymentStatus.Approved; // sem validação, sem regra
```

Você perdeu o controle. Toda a lógica de domínio ficou solta, espalhada, e pior: vulnerável a ser ignorada.

---

## Sintomas clássicos de um modelo anêmico

Se você identificar esses sinais no seu código, acenda o alerta:

- Entidades com apenas getters e setters públicos
- Regras de negócio implementadas em services genéricos (`XyzService`)
- Poucos ou nenhum método nas entidades
- Validações feitas via `if` em todo lugar
- Testes unitários testando services e não o domínio
- Entities sem construtores ou com construtores vazios

> Se parece com um DTO, age como um DTO, e precisa de services para funcionar… é um DTO.  
> E seu “modelo de domínio” está morto por dentro.

---

## “Mas assim é mais simples…”

Sim. No começo.

Projetos pequenos sobrevivem bem com modelo anêmico.  
O problema aparece quando:

- O sistema cresce
- A regra de negócio muda
- Mais gente começa a mexer no código
- Você precisa garantir integridade em fluxos assíncronos
- Começa a duplicar lógica em vários services “porque ficou mais fácil copiar do que generalizar”

Você entra no ciclo de código frágil.  
Refatorar se torna perigoso.  
Reutilizar, impossível.  
E bugs surgem por bypassar regras silenciosamente.

---

## Qual o caminho, então?

Você não precisa cair direto no DDD hardcore.  
O primeiro passo é colocar a lógica onde ela pertence.  
E ela pertence à entidade.

```csharp
public class PaymentRequest
{
    public Guid Id { get; private set; }
    public decimal Amount { get; private set; }
    public PaymentStatus Status { get; private set; }

    public PaymentRequest(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");

        Id = Guid.NewGuid();
        Amount = amount;
        Status = PaymentStatus.Pending;
    }

    public void Approve()
    {
        if (Status != PaymentStatus.Pending)
            throw new InvalidOperationException("Cannot approve payment in current state");

        Status = PaymentStatus.Approved;
    }
}
```

Agora a regra está clara, encapsulada e testável.

```csharp
var payment = new PaymentRequest(1000);
payment.Approve(); // ✅

payment.Approve(); // ❌ throws exception
```

Esse é o começo de um **modelo rico**, onde a entidade protege sua consistência.

---

## Testes mais naturais, código mais coeso

Testar uma entidade rica é direto:

```csharp
[Fact]
public void Should_Throw_When_Approving_Twice()
{
    var payment = new PaymentRequest(1000);
    payment.Approve();

    Should.Throw<InvalidOperationException>(() => payment.Approve());
}
```

- Você não precisa de mock.
- Não depende de infraestrutura.
- E pode evoluir o domínio com confiança.

---

## Quando o modelo anêmico ainda pode ser aceitável?

Nem todo projeto precisa de um domínio expressivo.  
Cenários onde o modelo anêmico é tolerável:

- APIs CRUD sem regras de negócio
- Ferramentas internas de curto prazo
- Sistemas onde a fonte de verdade é um sistema externo e você só espelha dados

Mesmo nesses casos, **isolar o domínio da infra ainda é uma boa prática**.

---

## Conclusão

O modelo anêmico é confortável.  
Mas conforto é diferente de segurança.

Se você está construindo um sistema com regras complexas, com múltiplos times e onde bugs custam caro — você vai pagar esse preço se continuar tratando seu domínio como um DTO com nome bonito.

---

**Próximo capítulo:**  
Vamos olhar com lupa para os “services” genéricos que nascem do modelo anêmico e entender por que eles acabam virando depósitos de lógica sem identidade.

Spoiler: `XyzService` é um cheiro. E dos fortes.
