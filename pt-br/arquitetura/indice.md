# Arquitetura de Software em C# no Mundo Real

Essa série de artigos nasceu da frustração — e da prática.

Se você já tentou organizar um sistema .NET complexo, sabe que não é fácil: cada time tem uma visão diferente, os tutoriais se contradizem, e os “padrões” muitas vezes atrapalham mais do que ajudam. Entre um `GenericRepository<T>` e um `PaymentServiceHandlerProcessorFactory`, fica difícil saber o que realmente importa.

Aqui, a proposta é simples: falar sobre **arquitetura real**, com um olhar crítico, direto e sem medo de questionar modinhas. Vamos explorar estilos, padrões, anti-padrões e armadilhas comuns que se acumulam ao longo dos anos — e mostrar alternativas que funcionam fora do slide de apresentação.

Essa série é para desenvolvedores que já sujaram as mãos. Que já herdaram legado, quebraram produção, e tentaram explicar DDD em reunião de grooming.

Sem fórmulas mágicas. Sem purismo. Só código, contexto e experiência real.

---

## Índice da Série

### **Capítulo 1 – Os caminhos que a arquitetura pode tomar**

> Um panorama direto e sem rodeios sobre Transaction Script, Modelo Anêmico, Modelo Rico e DDD.

### **Capítulo 2 – O modelo anêmico vai te trair (e como saber quando fugir dele)**

> Quando usar um modelo anêmico parece mais simples, mas te afunda em bugs e complexidade acidental.

### **Capítulo 3 – A falsa promessa do “service pattern” genérico**

> `AccountService`, `UserManager`, `FooHandler`: nomes que escondem baixa coesão e design acoplado.

### **Capítulo 4 – Entidades ricas: encapsulamento é mais que `private set;`**

> Como construir entidades com comportamento real e proteger sua integridade.

### **Capítulo 5 – A camada de aplicação não é um lixo eletrônico**

> Application Services devem orquestrar, não conter regras de negócio nem virar dumping ground de dependências.

### **Capítulo 6 – Use Cases, Command Handlers e MediatR: o que separa clareza de confusão**

> Quando aplicar CQRS traz benefícios reais — e quando só adiciona complexidade desnecessária.

### **Capítulo 7 – Repositórios que respeitam o domínio (e evitam vazamento de infra)**

> Um repositório não é só um wrapper de `DbContext`. É um contrato do domínio com a persistência.

### **Capítulo 8 – Domínio expressivo: Value Objects, Invariantes e Eventos de Domínio**

> Modelando regras com segurança e intenção. O domínio deve falar por si.

### **Capítulo 9 – Dividindo para conquistar: a importância dos Bounded Contexts**

> DDD real exige saber onde termina um modelo e começa outro. E como manter essa separação.

### **Capítulo 10 – Anti-patterns arquiteturais que todo dev .NET já viu (ou cometeu)**

> Um olhar honesto sobre os maiores pecados que já cometemos em nome da “arquitetura”.

### **Capítulo 11 – Mitos e Más Interpretações na Arquitetura .NET**

> Por que MVC não é arquitetura, repositório não é para toda entidade, padrão não é enfeite, e SOLID não é carta branca pra travar refatoração.
