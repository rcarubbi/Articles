# Capítulo 9 – Dividindo para conquistar: a importância dos Bounded Contexts

Se tem um conceito do DDD que todo mundo cita mas quase ninguém aplica direito, é o tal do **Bounded Context**.

Todo arquiteto já usou esse termo em alguma apresentação.  
Mas na prática, o que a gente mais vê é:

- Um domínio chamado `Core`, cheio de coisa genérica.
- Um monte de projeto `.Shared`.
- Um DTO com 48 propriedades tentando servir três departamentos diferentes.

Ou seja: um modelo tentando falar todas as línguas ao mesmo tempo.

---

## O que é um Bounded Context, afinal?

É uma **fronteira explícita** em torno de um modelo de domínio.

**Dentro dela**:

- Os termos têm significado específico.
- As regras de negócio são coerentes.
- O código é otimizado para um problema só.

**Fora dela**:

- Pode haver termos iguais com significados diferentes.
- As regras podem colidir.
- O modelo é outro.

Não é sobre “módulos”, é sobre **linguagens**.

---

## Mesmo nome, significados diferentes

Você acha que “Conta” significa a mesma coisa para o time de Faturamento e para o time de Compliance?

Spoiler: **não significa.**

- Pra Faturamento, uma “conta” pode ser um cliente inadimplente com boleto vencido.
- Pra Compliance, “conta” pode ser um IBAN validado com KYC, aprovado pelo banco parceiro.

Se você tentar criar um `Account` que sirva pros dois…  
…vai acabar com 300 `if` e nenhuma clareza.

---

## Como definir Bounded Contexts na prática?

- Ouça os times. As palavras mudam conforme o departamento muda. O DDD chama isso de **Ubiquitous Language** — e ela só é válida **dentro de um contexto**.
- Mapeie os fluxos. Entenda quem usa o quê, e onde as regras mudam.
- Delimite os modelos. Um `Trade` no Front Office pode não ter nada a ver com um `Trade` no Back Office.
- Separe os projetos, bancos, APIs, se necessário. Ou, pelo menos, os namespaces.

---

## Os pecados do “modelo compartilhado”

- Um `CustomerDto` com `Status`, `IsEnabled`, `IsActive`, `IsBlocked`, `IsLocked`, `IsOnboarded`… tudo booleano… e ninguém sabe o que é o quê.
- Um único `Order` que precisa servir pedidos do ecommerce, da intranet, da automação de warehouse e do faturamento.
- Uma tabela `Accounts` referenciada por 7 serviços, cada um com sua forma de interpretar os dados.

> Isso não é reaproveitamento.  
> **Isso é acoplamento disfarçado de DRY.**

---

## Como manter os contextos separados

- Evite projetos `.Shared` com tipos do domínio.
- Compartilhe contratos, nunca modelos internos.
- Use DTOs específicos pra cada integração.
- Use mapeamento explícito nas bordas.
- Um adaptador traduz do modelo externo para o modelo interno.
- Não deixe modelos externos vazarem para dentro do seu domínio.
- **Eventos também respeitam contexto.**

O `UserCreatedEvent` do RH não deve ser o mesmo do CRM.  
Compartilhe somente o que for contrato público.

**A regra é clara:**

- “Modelo só é compartilhado se a equipe também for.”

---

## Ferramentas úteis

- **Context Map**: um diagrama que mostra os Bounded Contexts e suas integrações (anticorruption layers, conformist, shared kernel, etc.).
- **OpenAPI**: define o contrato público entre contextos (útil quando há APIs).
- **Code Gen + Mapping**: para evitar vazamentos, gere clientes a partir de contratos, mas mapeie os dados no seu próprio modelo.

---

## Conclusão

Se você não define seus Bounded Contexts, seu sistema vira um **telefone sem fio**.

Todo mundo fala com todo mundo, ninguém se entende, e qualquer mudança quebra tudo.

Definir bem os contextos é o passo mais estratégico do DDD.  
É o que te permite crescer sem bagunça.

---

**Próximo capítulo:**  
E já que falamos em bagunça, chegou a hora da gente expor aquilo que quase todo sistema tem — e ninguém admite: os **anti-patterns disfarçados de boas intenções**.
