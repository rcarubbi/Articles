# Capítulo 5 – A camada de aplicação não é um lixo eletrônico

A essa altura, já deixamos para trás os services genéricos e começamos a construir entidades que se comportam como o domínio espera.  
Mas então vem aquela dúvida: **onde orquestro tudo isso?** Onde coloco o código que não pertence nem à entidade nem ao controller?

A resposta quase sempre é: _"no Application Service"_.

E está certo. Em teoria.

Na prática, esse cara acaba virando um novo buraco negro: tudo que você não sabe onde colocar, vai parar ali.

---

## `ApplicationService` virou o novo `utils`

Quantas vezes você viu algo assim?

```csharp
public class TradeApplicationService
{
    public async Task ApproveTrade(Guid tradeId)
    {
        var trade = await _repository.Get(tradeId);

        if (trade.Status == TradeStatus.PendingApproval)
        {
            trade.Approve();
            await _repository.Save(trade);
        }

        if (trade.Type == TradeType.Forex)
        {
            _logger.LogInfo("Forex trade approved");
        }

        if (trade.Amount > 1_000_000)
        {
            await _alertService.NotifyRisk();
        }
    }
}
```

Essa classe não está só orquestrando.  
Ela está **validando, decidindo, persistindo, logando e disparando alertas**.

É o mesmo problema do `XyzService`, agora disfarçado de Application Service — com uma cara mais “arquiteturalmente correta”.

---

## O que realmente pertence ao Application Service?

Application Service é **orquestrador de caso de uso**.  
Ele chama o domínio. Ele **não implementa as regras**.

Imagine o caso de uso “fechar uma conta bancária”.  
A responsabilidade do Application Service seria:

1. Receber o comando (DTO ou Command)
2. Carregar a entidade (`Account`)
3. Chamar um método do domínio (`Close()`)
4. Persistir as mudanças
5. (Opcional) Publicar evento de domínio ou de integração

```csharp
public class CloseAccountHandler
{
    public async Task Handle(CloseAccountCommand command)
    {
        var account = await _repository.Get(command.AccountId);

        account.Close(); // a regra está no domínio

        await _repository.Save(account);

        await _eventPublisher.Publish(new AccountClosed(account.Id));
    }
}
```

Só isso.  
Sem `if`.  
Sem `switch`.  
Sem validação de regra.  
Sem log de negócio.  
Sem business decisions.

---

## “Mas e quando o domínio precisa de algo externo?”

Nem tudo pode ser encapsulado dentro da entidade. Às vezes, você precisa consultar um serviço externo (como um `FxRateService`) para tomar uma decisão.

Mas isso não significa que você deve entupir o handler com lógica.

A solução:

- Delegue a lógica para orquestradores especializados
- Ou use objetos de domínio que recebem colaboradores

Em DDD, chamamos isso de **Domain Services**.  
Mas você pode dar nomes mais específicos como `FundsAvailabilityChecker` ou `SettlementInstructionResolver`.

A ideia é manter o **Application Service fino** e dar **nomes às responsabilidades**.

---

## Você não quer mais que um dev leia 300 linhas de um handler

Você quer:

- Um comando que descreve a intenção (`ApprovePaymentCommand`)
- Um handler que invoca 2 ou 3 colaboradores claros
- Entidades que cuidam do próprio estado
- Regras explícitas e isoladas em serviços bem nomeados

Quando você lê o handler, deve entender **o que acontece**,  
não **como acontece**.

---

## Application Service como glue code... com responsabilidade

Sim, Application Services são cola.  
Mas até cola tem qualidade.

- Se estiver espalhada, gruda tudo e quebra fácil.
- Se for modular, você troca peças com confiança.

Não tenha medo de criar:

- Pequenas classes auxiliares
- Handlers finos
- Objetos de comando
- Serviços de domínio

**Isso não é overengineering. É clareza.**

---

## Conclusão

Application Services **não devem conter lógica de negócio**.  
Eles devem invocar comportamentos do domínio, persistir mudanças e coordenar fluxos. Só.

Se você trata essa camada como dumping ground, você está apenas adiando a entropia.

---

**Próximo capítulo:**  
Agora que temos entidades ricas e application services enxutos, o próximo passo natural é refinar a forma como lidamos com ações e comandos.  
É aí que o padrão **CQRS** entra: separar leitura de escrita, clarear intenções e reduzir acoplamentos invisíveis.
