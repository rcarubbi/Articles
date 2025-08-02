# Capítulo 8 – Domínio expressivo: Value Objects, Invariantes e Eventos de Domínio

Chegamos ao ponto em que o código já tem cara de domínio: entidades fazem sentido, repositórios não cheiram mais a genérico, e handlers finalmente parecem scripts de orquestração. Mas falta algo...

Falta o domínio falar.

Falta abrir uma classe e saber de cara o que ela representa, o que ela permite, e o que ela nunca aceitará.

Falta o modelo ser mais que uma coleção de `public string`, `public decimal`, `public bool`.

É aqui que entram os **Value Objects**, as **invariantes** e os **eventos de domínio**.

---

## Value Objects: encapsular é libertar

Vamos começar por onde a maioria erra: o `decimal Amount`.

Toda aplicação financeira tem isso. E quase toda trata como um tipo primitivo em toda parte.

Mas pense comigo: Amount é só um número?

- Ele precisa ser positivo?
- Ele pode ter casas decimais?
- Ele representa o valor em qual moeda?
- Ele pode ser somado com outro valor de moeda diferente?

Se essas regras existem… por que o `Amount` ainda é um `decimal`?

A resposta: porque você ainda não modelou o domínio.

---

## Um bom Value Object carrega significado, invariantes e semântica

```csharp
public record Money
{
    public decimal Value { get; }
    public string CurrencyCode { get; }

    public Money(decimal value, string currencyCode)
    {
        if (value < 0)
            throw new ArgumentException("Value must be non-negative");

        if (string.IsNullOrWhiteSpace(currencyCode))
            throw new ArgumentException("Currency is required");

        Value = value;
        CurrencyCode = currencyCode.ToUpperInvariant();
    }

    public Money Add(Money other)
    {
        if (other.CurrencyCode != CurrencyCode)
            throw new InvalidOperationException("Cannot add amounts with different currencies");

        return new Money(Value + other.Value, CurrencyCode);
    }
}
```

Agora seu domínio não é só mais seguro.  
**Ele explica sua regra com o próprio código.**

---

## Invariantes: as regras que nunca mudam

Toda entidade tem regras que não podem ser quebradas. Jamais.  
Se elas forem violadas, o sistema está corrompido.

Essas são as invariantes.

Exemplos:

- Uma conta não pode ser fechada se ainda tiver saldo.
- Um trade não pode ser aprovado duas vezes.
- Um pagamento não pode ser marcado como “paid” sem uma instrução de liquidação válida.

Você pode deixar essas regras espalhadas em `ifs` ao longo do código...

Ou você pode colocá-las no domínio, guardadas a sete chaves:

```csharp
public void Approve()
{
    if (Status != TradeStatus.PendingApproval)
        throw new DomainException("Only pending trades can be approved.");

    Status = TradeStatus.Approved;
}
```

Agora, não importa se o comando vem da UI, de um worker, de um ETL ou de um fluxo de eventos.  
**A regra é a mesma. Está no lugar certo.**

---

## Eventos de domínio: reações naturais de um modelo vivo

Outro ponto negligenciado: **eventos de domínio**.

Se o modelo representa comportamento e estado, ele também reage a mudanças.  
E essas reações precisam ser capturadas.

Exemplo: ao aprovar um pagamento, talvez você precise:

- Notificar compliance
- Gerar instrução de liquidação
- Disparar alerta de risco

Essas não são responsabilidades diretas da entidade.  
Mas a entidade sabe que algo relevante aconteceu.

Logo, ela emite um evento:

```csharp
public void Approve()
{
    if (Status != PaymentStatus.Pending)
        throw new DomainException("Only pending payments can be approved.");

    Status = PaymentStatus.Approved;

    _domainEvents.Add(new PaymentApproved(Id, Amount, AccountNumber));
}
```

E depois, no seu pipeline de Unit of Work ou MediatR, esses eventos são despachados.

Isso torna seu sistema mais reativo, coeso e observável — **sem acoplamento**.

---

## 🧪 Testando tudo isso

Com **Value Objects** e **invariantes** bem definidos:

- Testar o domínio fica fácil: você instancia e interage.
- Testar regras de negócio vira algo natural.
- Você evita mocks e stubs em 80% dos testes.
- Você começa a se sentir confortável com mudanças, porque o modelo está blindado contra inputs inválidos.

---

## Conclusão

Não subestime o poder de modelar bem.

- **Value Objects** encapsulam significado.
- **Invariantes** protegem a integridade.
- **Eventos de domínio** tornam o modelo expressivo e reativo.

Quando seu domínio fala com clareza,  
a aplicação vira uma orquestra — e não um monte de banda tocando sozinha.

---

**Próximo capítulo:**  
E se vamos falar de expressividade e contexto, o próximo capítulo precisa ser sobre **Bounded Contexts** —  
como separar responsabilidades, linguagens e modelos em sistemas complexos sem se perder.
