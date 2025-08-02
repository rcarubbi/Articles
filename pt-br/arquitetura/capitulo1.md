# Capítulo 1 – Os caminhos que a arquitetura pode tomar

Se você é como eu, provavelmente já passou por aquele momento de desconforto olhando para um serviço gigante cheio de `if`, validações repetidas e queries SQL pingando direto do controller. A boa notícia é: você não está sozinho. A má notícia? Isso provavelmente é (ou já foi) um Transaction Script disfarçado.

Quando comecei a desenvolver em .NET — ainda nos tempos de ASP clássico migrando para WebForms — minha arquitetura era movida a "deu certo, então tá bom". Mas com o tempo, fui percebendo que existiam formas diferentes de organizar código. E mais importante: algumas escalavam bem, outras viravam monstros com o tempo.

Neste capítulo, quero fazer um panorama real e direto de quatro abordagens arquiteturais que influenciam fortemente como pensamos o design de uma aplicação: **Transaction Script**, **Modelo Anêmico**, **Modelo Rico** e **DDD (Domain-Driven Design)**. Sem floreio. Só o que realmente importa.

---

## Transaction Script – O procedural disfarçado de OOP

O Transaction Script é simples. Funciona bem para sistemas pequenos. A lógica de negócio está basicamente em métodos: um por operação. Criar cliente? Tem um método. Fazer um pagamento? Outro método. Tudo acontece dentro do service — ou pior, do controller.

```csharp
public void Transfer(string fromAccount, string toAccount, decimal amount)
{
    var debitAccount = _db.Accounts.Single(a => a.Number == fromAccount);
    var creditAccount = _db.Accounts.Single(a => a.Number == toAccount);

    if (debitAccount.Balance < amount)
        throw new Exception("Insufficient funds");

    debitAccount.Balance -= amount;
    creditAccount.Balance += amount;

    _db.SaveChanges();
}
```

Funciona? Sim. Escala? Não. Essa abordagem acopla tudo: validações, persistência e regras de negócio. À medida que o sistema cresce, o número de scripts explode, e manter coerência entre eles vira um inferno.

---

## Modelo Anêmico – O DTO vestido de entidade

É aqui que muitos de nós fomos treinados a cair: o tal do Modelo Anêmico. Você tem uma classe `Account`, mas ela só tem propriedades. Toda a lógica está no service.

```csharp
public class Account
{
    public string Number { get; set; }
    public decimal Balance { get; set; }
}
```

A regra de negócio? Está num `AccountService`. Ou pior: num `AccountManager`, `AccountHandler`, ou algum outro nome genérico.

O problema é claro: violação do encapsulamento. As entidades não se protegem. Qualquer parte do sistema pode alterar o `Balance` diretamente. Se alguém esquecer de validar antes de debitar, o sistema aceita — e o bug aparece em produção, claro.

---

## Modelo Rico – Entidades com responsabilidade

Aqui a coisa melhora. No Modelo Rico, a entidade é responsável por manter sua integridade. Regras de negócio vivem dentro da própria entidade.

```csharp
public class Account
{
    public string Number { get; private set; }
    public decimal Balance { get; private set; }

    public void TransferTo(Account destination, decimal amount)
    {
        if (Balance < amount)
            throw new InvalidOperationException("Insufficient funds");

        this.Balance -= amount;
        destination.Balance += amount;
    }
}
```

É um passo além. A lógica agora está onde deveria estar. Testar essa entidade é fácil. Ela tem coesão e encapsula comportamento. Só que, em sistemas mais complexos, com múltiplos bounded contexts, integrações, consistência eventual e orquestração de regras, isso ainda pode não ser suficiente.

---

## Domain-Driven Design (DDD) – Modelo Rico com propósito

O Domain-Driven Design vai além. Ele começa com a premissa de que software é uma ferramenta para resolver problemas de domínio — e que o conhecimento de domínio deve guiar o design do sistema.

Com DDD, não se trata apenas de ter entidades ricas. Trata-se de entender o contexto em que elas vivem, de dividir a aplicação em _bounded contexts_, de criar uma _ubiquitous language_ entre devs e stakeholders.

Você não implementa DDD só criando um `AggregateRoot` ou usando MediatR. DDD é sobre conversar com o time de negócio e refletir essas conversas no código.

É aqui que entram termos como:

- Entities com identidade persistente;
- Value Objects imutáveis que encapsulam regras;
- Aggregates que mantêm consistência;
- Domain Events que refletem mudanças importantes;
- Repositories que respeitam os limites do domínio;
- Application Layer que orquestra casos de uso sem violar o domínio.

Sim, é mais complexo. Mas em sistemas que lidam com regras complicadas (como finanças, logística, compliance), é aí que a clareza e a resiliência aparecem.

---

## Conclusão

Eu vejo essas abordagens como estágios de maturidade. Todas têm seu lugar — e não existe bala de prata. Já usei Transaction Script para ferramentas simples, Modelo Rico para APIs limpas, e DDD para sistemas onde o domínio precisava respirar no código.

O importante é saber **por que** você está escolhendo uma abordagem.  
E mais importante ainda: saber **quando mudar**.

---

**Próximo capítulo:**  
Vamos explorar as limitações do Modelo Anêmico em projetos reais — e como ele pode nos trair quando menos esperamos.
