# 📘 Capítulo 11 – Mitos e Más Interpretações na Arquitetura .NET

> Ou: o que a gente aprende errado, repete sem pensar, e ensina pior ainda.

Se tem uma coisa que a internet ensinou pra gente nos últimos anos é que saber o nome das coisas não é o mesmo que saber usá-las.  
E arquitetura de software virou esse mar de termos citados fora de contexto, padrões aplicados por osmose, e discussões técnicas que mais parecem briga de quem decora mais sigla.

Neste capítulo, vamos encarar de frente alguns vícios conceituais que se espalharam pelo mundo .NET — alguns herdados da era pré-Core, outros alimentados por treinamentos, cursos rasos e diagramas didáticos que nunca passaram por uma base de código real.

---

## 1. Achar que MVC é DAL + BLL + UI com nomes diferentes

Esse erro é mais comum do que parece — principalmente entre devs que migraram de aplicações em 3 camadas clássicas (DAL, BLL, UI) e foram parar num projeto ASP.NET MVC ou Web API.

A confusão é mais ou menos assim:

```
Controller → BLL
Model     → DAL
View      → UI
```

Só que isso **não é** MVC.

MVC é um **padrão de separação de responsabilidade dentro da camada de apresentação**.  
Ele **não dita nada** sobre persistência, domínio ou infraestrutura.

Na verdade:

- Assume que você já tem um modelo de domínio,
- Supõe que o controller só vai orquestrar e preparar dados para a view,
- E que a view é apenas um refletor dos dados que recebeu.

Essa visão distorcida cria sistemas onde:

- Toda lógica fica nos controllers (às vezes com mais de 500 linhas),
- O “Model” são só os EF Entities,
- E o domínio? Não existe — só um grande vácuo entre o controller e o banco.

**MVC não substitui arquitetura.**  
É apenas um padrão de UI, usado dentro de algo muito maior.

---

## 2. Criar um repositório pra cada entidade

Outro clássico:

> “Temos 32 entidades, então criamos 32 repositórios.”

Isso não é DDD, nem Clean Architecture.  
Isso é um ORM disfarçado de arquitetura.

Você só precisa de repositório onde existe um **Aggregate Root**, ou seja:

- Uma entidade que possui controle de consistência sobre outras.
- Um ponto de entrada para transações no domínio.

Você **não precisa** de `EmailRepository`, `PhoneRepository`, `AddressRepository`...

Você precisa de `CustomerRepository` se `Customer` **encapsula tudo isso**.

**Regra prática:**  
Se a entidade não tem identidade própria e regras de consistência internas, **não merece um repositório.**

---

## 3. Estudar design patterns como se fosse faculdade teórica — ou fingir que eles não existem

Tem desenvolvedor que leu todos os livros do GoF, sabe explicar `Strategy` no papel, recita `Factory`, `Observer`, `Decorator`… mas nunca aplicou nenhum de verdade.

Na prática:

- Escreve `if/else` encadeado onde um `Strategy` resolveria com elegância.
- Ou pior: ignora a complexidade, porque “dá pra resolver rápido assim mesmo”.

Por outro lado, tem o dev que **despreza padrões**:

> “Esses padrões aí só complicam.”  
> “Prefiro fazer do meu jeito, mais direto.”

Aí o que ele faz?

Cria uma tabela de outbox… mas chama de `SyncMessageBuffer`, `TempQueue`, `SomethingManager`.

O problema aqui é **não usar o vocabulário comum da engenharia de software**.

Padrões não são modinhas.  
São **atalhos cognitivos**.  
São **acordos de comunicação** entre desenvolvedores.

Você não precisa forçar um padrão onde não cabe.  
Mas também não precisa reinventar o que já tem nome, forma e prática consolidada.

---

## 4. Usar SOLID como dogma sem contexto

Esse é clássico: alguém joga uma sigla na conversa como se fosse carta de autoridade.

> “Isso aí tá quebrando o OCP.”  
> “Tá violando SRP.”  
> “Tem que ter interface, por causa do DIP.”

Só que:

- O “OCP” citado é mal interpretado — você só estava refatorando uma classe gorda, não alterando regra de negócio.
- O SRP virou “uma classe por responsabilidade”, mesmo que a responsabilidade tenha 5 linhas.
- O DIP virou “injeção de dependência sem propósito”.

E pior: vira uma conversa onde ninguém mais discute o problema real — só defesas baseadas em siglas, ego e medo de parecer que não conhece o acrônimo da vez.

**SOLID é um guia prático, não um código penal.**  
Aplicar os princípios exige **contexto**, **propósito** e **bom senso** — não repetição automática.

---

## Conclusão

Esses erros não acontecem por falta de capacidade técnica.  
Eles acontecem porque, no dia a dia, a teoria vira ruído — e o time só quer entregar a feature e fechar o ticket.

Mas é aí que mora o perigo:

- Quando você cita SOLID como escudo, e não como guia.
- Quando rejeita padrões de projeto por “estilo próprio”, e não por decisão técnica consciente.
- Quando lê sobre arquitetura, mas não enxerga os padrões na prática.
- Quando transforma boas práticas em dogmas, ignorando o contexto.

O resultado disso tudo é sempre o mesmo:  
**Código difícil de entender, difícil de evoluir, e impossível de explicar.**

Você não precisa aplicar todos os padrões. Nem deve.  
Mas também não pode ignorar o conhecimento coletivo da engenharia de software **em nome do “meu jeito”.**

Saber programar bem é mais que saber sintaxe.  
É tomar decisões conscientes, com base em propósito — e não em siglas ou alergia a abstração.

---

## Fim da jornada (por enquanto)

Essa série não é um manual de regras.  
É um convite ao **senso crítico**.

Modelar software é um trabalho criativo, iterativo e profundamente humano.

Você vai errar. Eu erro até hoje.

Mas quanto mais seu domínio fala, mais o código respira.  
E quando o código respira, o time evolui junto.

Nos próximos artigos, podemos explorar temas como:

- Integrações entre contextos (com anticorruption layers)
- Event Sourcing
- Técnicas para refatorar legado sem quebrar tudo
- Como evoluir arquitetura mesmo quando o time resiste

Se você chegou até aqui:  
**obrigado pela companhia.**  
Nos vemos na próxima refatoração. 👊🏼
