# Capítulo 4 – Entidades ricas: encapsulamento é mais que `private set;`

Quando você começa a criar handlers dedicados para cada caso de uso, logo percebe que precisa de entidades que saibam se comportar.

E aí vem o choque: `private set;` não garante encapsulamento real.

Ele impede que código externo altere o valor diretamente — mas não impede que métodos internos mal projetados ou construtores expostos deixem o objeto em um estado inválido.

Encapsular estado não é esconder o `set`.  
**É proteger o invariável.**

---

## 🛡️ A entidade não é um dicionário com nome bonito

Essa aqui é direta: se sua entidade só tem propriedades públicas e nenhum comportamento, ela não é uma entidade. É um dicionário tipado.

```csharp
public class Trade
{
    public decimal Quantity { get; set; }
    public decimal Price { get; set; }
    public string Status { get; set; }
}
```

O que impede `Status = "Settled"` mesmo quando ainda não houve clearing?  
**Nada.**

Agora, compare com isso:

```csharp
public class Trade
{
    public decimal Quantity { get; private set; }
    public decimal Price { get; private set; }
    public TradeStatus Status { get; private set; }

    public Trade(decimal quantity, decimal price)
    {
        if (quantity <= 0 || price <= 0)
            throw new ArgumentException("Invalid trade values");

        Quantity = quantity;
        Price = price;
        Status = TradeStatus.Pending;
    }

    public void Settle()
    {
        if (Status != TradeStatus.Cleared)
            throw new InvalidOperationException("Cannot settle a trade that hasn't cleared");

        Status = TradeStatus.Settled;
    }

    public void Clear() => Status = TradeStatus.Cleared;
}
```

A regra mora dentro.  
Você não precisa lembrar em cada handler o que pode ou não pode.  
A entidade se protege. O domínio fala por si.

---

## Entidade rica ≠ entidade gorda

A primeira crítica ao modelo rico é que ele “engorda a entidade”.  
Só que isso é um falso dilema.

**Entidade rica é aquela que tem comportamento e se protege.**  
Ela não precisa saber de infraestrutura, de logging, de `HttpClient`, nem de que botão o usuário apertou no front-end.

Ela só sabe reagir a comandos e manter seu estado consistente.

Se o domínio tem lógica complexa, a entidade pode delegar para métodos privados, Value Objects, ou até helpers internos —  
mas ela continua como ponto de entrada e guardiã de sua própria integridade.

---

## E onde ficam as validações?

A resposta honesta: **depende**.

Mas as validações que dizem respeito ao **estado da entidade** pertencem à entidade.

- “Campo obrigatório”, “número positivo”, “formato de e-mail”? Pode viver em validators externos ou DTOs.
- Mas...

```plaintext
“Não pode aprovar pagamento rejeitado”
“Trade só pode liquidar se já tiver sido clearado”
“Conta só pode ser fechada se não houver saldo pendente”
```

Essas são **invariantes**.  
E invariantes pertencem ao domínio.

---

## Testes como ferramenta de feedback

Quando você move lógica para dentro da entidade, seus testes mudam.

Em vez de verificar se um handler chamou o `repository.Update(...)`, você passa a testar **regras de negócio diretamente**.

```csharp
[Fact]
public void Cannot_Settle_Trade_Without_Clearing()
{
    var trade = new Trade(10, 100);
    Should.Throw<InvalidOperationException>(() => trade.Settle());
}
```

Mais rápidos. Mais claros. Menos dependências externas.

Você pode testar o domínio **em memória**.  
E o domínio vira seu **contrato de negócio real**.

---

## Dica prática: entenda o ciclo de vida da entidade

Nem toda entidade precisa ser um Aggregate Root. Nem conter toda a lógica do sistema.

Algumas serão simples. Outras mais complexas.

Mas se você está modelando um conceito com múltiplas regras, estados e transições —  
você **precisa modelar o ciclo de vida.**

Imagine uma `Invoice`:

```plaintext
Criada → Rascunho
Finalizada → Enviada
Paga ou Cancelada
Se paga, gera baixa; se cancelada, emite nota de crédito
```

É esse tipo de fluxo que você representa em métodos da entidade.

E com isso, você blinda o domínio contra uso incorreto.

---

## Conclusão

Encapsular não é esconder o `set`.  
É evitar que o estado da sua entidade seja corrompido por uso indevido.

Um modelo rico **protege a si mesmo**.  
Ele declara intenções e se recusa a violar as regras do negócio.

---

**Próximo capítulo:**  
Agora que temos entidades mais robustas, fica evidente que a camada de aplicação não pode continuar sendo apenas um **dumping ground** de lógica acoplada.

No próximo capítulo, vamos focar exatamente nisso: como os **Application Services** devem ser tratados como orquestradores — e não como novas classes `XyzService` disfarçadas.

_Nota:_ “**Dumping ground**” (literalmente, “depósito de lixo” ou “aterro”) é uma expressão idiomática em inglês que significa:
Um lugar onde você joga tudo que não sabe onde colocar. No contexto de código ou arquitetura, quando dizemos que
“a camada de aplicação virou um dumping ground”, queremos dizer que ela acumulou responsabilidades demais.
