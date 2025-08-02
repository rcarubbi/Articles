# Capítulo 3 – A falsa promessa do “service pattern” genérico

Depois que você percebe que o modelo anêmico está te limitando, a reação mais comum é criar um _service_.  
A lógica precisa ir para algum lugar, certo?

Aí nasce o `UserService`. Depois o `AccountService`, o `PaymentService`, o `FooHandler`, o `BarManager`.  
Cada um carregando validações, regras de negócio, orquestração — e, com sorte, alguns comentários explicando o caos.

No começo, tudo parece resolvido. A lógica está “centralizada”. Você acha que está sendo organizado.  
Mas, na prática, você só mudou o problema de lugar.

---

## Services genéricos: o nome bonito do _god object_ disfarçado

Vamos direto ao ponto: a maioria desses _services_ não tem coesão.  
Eles fazem coisas demais e não representam nenhuma ação do domínio de forma clara.

```csharp
public class AccountService
{
    public void CreateAccount(...) { ... }
    public void CloseAccount(...) { ... }
    public void TransferFunds(...) { ... }
    public void ChangePrimaryContact(...) { ... }
    public void EnableNotifications(...) { ... }
}
```

Esse `AccountService` faz tudo. Ele parece saber demais sobre o mundo e vira um ponto único de acoplamento para qualquer funcionalidade relacionada a "account".  
Mas qual é o real **caso de uso** que ele está representando?  
**Nenhum em específico.**

---

## Cheiros clássicos de um service genérico

Você sabe que está lidando com um _service_ fora de controle quando:

- Ele tem mais de 300 linhas (ou muito mais…)
- Contém métodos que nem se conhecem (ex: `ExportToCsv()` convivendo com `ApproveTransaction()`)
- Recebe um número obsceno de dependências no construtor
- O mesmo método valida, transforma DTOs, persiste no banco e dispara notificações
- Você evita alterar algo porque “pode quebrar outra parte que não sei direito”

---

## O que você perde mantendo esse padrão

- **Coesão**: seu código não representa uma ação clara do domínio. Os nomes mentem.
- **Testabilidade**: é difícil isolar comportamentos porque o _service_ virou um Frankenstein de responsabilidades.
- **Evolução**: adicionar novos fluxos exige mexer em lugares já carregados, com risco alto de regressão.
- **Refatoração impossível**: qualquer mudança simples se espalha por múltiplas dependências.

O que começa como uma tentativa de organização, vira um **anti-pattern estrutural** que atrasa todo o time.

---

## A alternativa: serviços voltados para casos de uso

Em vez de criar um _service_ genérico para "tudo que envolve Conta", pense em **ações específicas**.  
Em vez de `AccountService.CloseAccount`, você pode ter:

```csharp
public class CloseAccountHandler
{
    public Task Handle(CloseAccountCommand command)
    {
        // regras de negócio específicas para fechamento
    }
}
```

Isso tem nome, escopo e foco. Representa um **caso de uso real**.  
Você pode testá-lo isoladamente.  
E ele não tem ambições de ser o centro do universo.  
É só mais uma peça de um domínio bem particionado.

Esse é o ponto de virada onde muitos times começam a adotar **Command Handlers** e **CQRS** — que vamos explorar com mais profundidade nos próximos capítulos.

---

## “Mas se eu quebrar os services grandes, vão sobrar centenas de classes…”

Sim. E isso é bom.

Ter 200 classes pequenas, cada uma responsável por um único caso de uso,  
é melhor do que ter 5 classes gigantes com mil responsabilidades embaralhadas.

Arquitetura saudável é modular.  
E isso exige granularidade.

---

## “Mas o domínio já está encapsulado, então qual o problema de ter um orquestrador gordo?”

O problema é que esses orquestradores viram **campos minados**.  
Você nunca sabe ao certo o impacto de alterar um pedaço, porque tudo está interconectado.

Quando você separa cada caso de uso em uma classe dedicada, você **isola a complexidade**  
e cria um caminho natural para aplicar pipelines, validações, logs, transações e eventos de forma clara.

---

## Conclusão

Services genéricos podem parecer uma forma natural de agrupar lógica,  
mas acabam virando buracos negros de responsabilidade.

A alternativa — casos de uso pequenos, bem nomeados e testáveis —  
é o caminho para um sistema que respira e escala.

---

**Próximo capítulo:**  
Mas onde está a lógica que os services genéricos deveriam conter?  
No próximo capítulo, vamos olhar de perto para as **entidades** —  
e por que elas precisam deixar de ser apenas objetos com `get; set;` para se tornarem guardiãs do seu próprio estado.
