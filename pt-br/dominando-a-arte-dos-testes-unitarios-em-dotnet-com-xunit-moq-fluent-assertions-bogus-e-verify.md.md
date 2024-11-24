# 🧪 Dominando a Arte dos Testes Unitários em .NET com xUnit, Moq, Fluent Assertions, Bogus e Verify ⚗️

Testes unitários são uma ferramenta essencial para garantir que o código se comporte como esperado, ao mesmo tempo que promovem confiança nas mudanças e refatorações. Neste artigo, vamos explorar como criar testes unitários eficientes usando alguns pacotes populares no ecossistema .NET. Além disso, veremos como estruturar testes para um projeto que segue a Clean Architecture.

O código completo deste exemplo está disponível no [GitHub](https://github.com/rcarubbi/UnitTestsExample).

---

## Introdução aos testes unitários

Testes unitários são responsáveis por validar o comportamento de pequenas unidades de código (geralmente métodos ou classes). A ideia é isolar a unidade sendo testada, garantindo que ela funcione de forma independente de outras partes do sistema. Com a ajuda de ferramentas como mocks, as dependências externas podem ser substituídas por implementações controladas.

A seguir, apresentaremos os pacotes usados neste artigo e como cada um contribui para o processo de testes.

---

## Pacotes usados neste exemplo

### 1. xUnit
O xUnit é um framework de testes popular no .NET. Ele fornece atributos como `[Fact]` e `[Theory]` para definir testes, além de uma integração simples com ferramentas de execução de testes como o Visual Studio e o CLI do .NET.

Exemplo:
```csharp
[Fact]
public async Task Handle_ShouldCreateCustomer_WhenCommandIsValid()
{
    // Teste aqui!
}
```

### 2. Moq
Moq é uma biblioteca de mocking que permite criar objetos simulados para substituir dependências em testes unitários. Com ele, podemos configurar comportamentos esperados para métodos e verificar se eles foram chamados.

No exemplo, usamos o Moq para simular um repositório:
```csharp
_repositoryMock.Setup(x => x.Add(It.Is<Customer>(c => Compare(c, customer)), CancellationToken.None))
    .ReturnsAsync(true);
```

### 3. Fluent Assertions
O Fluent Assertions é usado para criar asserções legíveis e expressivas nos testes. Ele facilita a validação de valores, objetos e exceções.

Exemplo de comparação:
```csharp
actual.Should().BeEquivalentTo(expected, options => options.Excluding(c => c.Id));
```

Neste projeto, o Fluent Assertions é usado principalmente no `Comparer` para objetos complexos como `Customer`, porque comparações padrão verificariam apenas referências.

### 4. Verify
O Verify é uma biblioteca de snapshot testing. Ele salva uma versão esperada do resultado de um teste (snapshot) e compara com os resultados futuros. Se houver discrepâncias, o teste falha, e uma ferramenta de comparação de arquivos é aberta para revisar e ajustar os snapshots.

Primeira execução:
- Um snapshot é gerado e o teste falha porque ainda não existe um de comparação.
- O desenvolvedor ajusta as diferenças e salva o snapshot.
- Nas próximas execuções, os resultados são comparados com o snapshot salvo.

### 5. Bogus
Bogus é uma biblioteca para gerar dados fictícios. É usada para criar dados de teste consistentes e realistas, como nomes, datas, e-mails, etc.

No exemplo, usamos Bogus para criar instâncias da entidade `Customer`:
```csharp
var customer = new Faker<Customer>()
    .CustomInstantiator(f => new Customer(
        f.Name.FirstName(),
        f.Name.LastName(),
        f.Internet.Email(),
        DateOnly.FromDateTime(f.Date.PastOffset(70, DateTime.Now.AddYears(-18)).Date),
        f.PickRandom<Gender>()
    ))
    .Generate();
```

---

## Exemplo de Código: Criando e Testando um Command Handler

### O Command Handler
No exemplo, seguimos a Clean Architecture, onde os `Command Handlers` ficam encapsulados na camada de aplicação e não são expostos diretamente à API. Eles são chamados pelo Mediator, reduzindo o acoplamento.

Para que os testes acessem classes `internal`, adicionamos a configuração abaixo no projeto de aplicação:
```xml
<ItemGroup>
    <AssemblyAttribute Include="System.Runtime.CompilerServices.InternalsVisibleTo">
        <_Parameter1>Application.Tests</_Parameter1>
    </AssemblyAttribute>
</ItemGroup>
```

### Estrutura do Teste
#### Setup com Moq e Comparer
Como `Customer` é um tipo complexo, precisamos de um comparer para garantir que a comparação seja feita pelas propriedades, e não por referência:
```csharp
private static bool Compare<T>(T actual, T expected) where T : IEntity
{
    actual.Should().BeEquivalentTo(expected, options => options.Excluding(c => c.Id));
    return true;
}
```

#### Testando o Comportamento do Handler
- Configuramos o mock do repositório para esperar um `Customer` equivalente ao comando.
- Usamos `Verify` para validar a resposta.

Exemplo:
```csharp
var sut = new CreateCustomerCommandHandler(_repositoryMock.Object);
var response = await sut.Handle(command, CancellationToken.None);

_repositoryMock.Verify(x => x.Add(It.Is<Customer>(c => Compare(c, customer)), CancellationToken.None), Times.Once);
await Verify(response, verifySettings);
```

### Lidando com Randomização nos Snapshots
Para campos com valores aleatórios, como nomes ou GUIDs, usamos `VerifySettings` para aplicar scrubs:
```csharp
var verifySettings = new VerifySettings();
verifySettings.ScrubMember<CreateCustomerResponse>(f => f.FirstName);
verifySettings.ScrubMember<CreateCustomerResponse>(f => f.LastName);
```

---

### Lidando com Randomização nos Snapshots

Aqui está o fluxo de como um teste unitário é executado, utilizando as ferramentas mencionadas:

[![](https://mermaid.ink/img/pako:eNp9UsGO2jAQ_RXLZxYFloTEh0p0gaqHvRS0hyZ7mMZDsDaxqe20sCxf08OqlfoV-bFOElBRtWqkZOLJe29enn3kuZHIBd-U5nu-BevZep7ZTDO6Vp7W6RqdR_ZRq1yBNI_s5uYdm1kLusD0XAW7M3qjitqCZbPqi0JNFIms4z5e9M7oTuHlA7bge5M_uRcqX1O6BZvjDrVsfmka5tgSSgfuf_w5WSL-e1PULu2eou-10xfaW5DwJn8NtkB_j35rZLrYY1771k7z0xtpGLAV2s59-8sX_jWnj8E5pIT6ItgDWrVROel8QleXLffKfAfqvT9AaSw6tlLVrkTyvyxriqyHKKNd2jfY386bOitofkvK6c60Onsgpc7DIe2L6L6Ahea1-WHY7Bvo5vU6kX_ndrILLc-bvlQaSvV8nUEvfMHxAa_QVqAkHaFji8i432KFGRf0KsE-ZTzTJ8JB7c3qoHMuvK1xwK2piy0XG9piWtU7CR7nCgoL1QWyA_3ZmOslF0e-5yIcRlEySm6jeBom02AUDfiBuvFwMg6CaRxO4skkDMLwNODPnUAwTMbRaBQHt_E0iuIkGZ_-AOcZAtY?type=png)](https://mermaid.live/edit#pako:eNp9UsGO2jAQ_RXLZxYFloTEh0p0gaqHvRS0hyZ7mMZDsDaxqe20sCxf08OqlfoV-bFOElBRtWqkZOLJe29enn3kuZHIBd-U5nu-BevZep7ZTDO6Vp7W6RqdR_ZRq1yBNI_s5uYdm1kLusD0XAW7M3qjitqCZbPqi0JNFIms4z5e9M7oTuHlA7bge5M_uRcqX1O6BZvjDrVsfmka5tgSSgfuf_w5WSL-e1PULu2eou-10xfaW5DwJn8NtkB_j35rZLrYY1771k7z0xtpGLAV2s59-8sX_jWnj8E5pIT6ItgDWrVROel8QleXLffKfAfqvT9AaSw6tlLVrkTyvyxriqyHKKNd2jfY386bOitofkvK6c60Onsgpc7DIe2L6L6Ahea1-WHY7Bvo5vU6kX_ndrILLc-bvlQaSvV8nUEvfMHxAa_QVqAkHaFji8i432KFGRf0KsE-ZTzTJ8JB7c3qoHMuvK1xwK2piy0XG9piWtU7CR7nCgoL1QWyA_3ZmOslF0e-5yIcRlEySm6jeBom02AUDfiBuvFwMg6CaRxO4skkDMLwNODPnUAwTMbRaBQHt_E0iuIkGZ_-AOcZAtY)

## Configurações do Verify no Git

### `.gitignore`
Adicionamos estas entradas para ignorar arquivos temporários criados pelo Verify:
```gitignore
# Verify
*.received.*
*.received/
```

### `.gitattributes`
Adicionamos configurações para normalizar os arquivos de snapshot:
```gitattributes
# Verify
*.verified.txt text eol=lf working-tree-encoding=UTF-8
```

O método `VerifyChecks.Run()` valida essas configurações e sugere ajustes quando necessário:
```csharp
[Fact]
public Task VerifyCheck() => VerifyChecks.Run();
```

---

## Nomeação de Testes
Existem dois padrões principais de nomenclatura:

1. **Given... When... Then...**
   - Exemplo: `GivenValidCommand_WhenHandleIsCalled_ThenCustomerIsCreated`.
2. **Action... Should... When...**
   - Exemplo: `Handle_ShouldCreateCustomer_WhenCommandIsValid`.

Ambos são válidos, e a escolha depende do time ou projeto.

---

## Conclusão
Combinando xUnit, Moq, Fluent Assertions, Verify e Bogus, criamos testes unitários poderosos, legíveis e robustos. Este exemplo mostrou como lidar com desafios como comparação de objetos complexos e valores aleatórios, enquanto aproveitamos o poder do Verify para snapshot testing.

O código completo deste exemplo pode ser encontrado no [GitHub](https://github.com/rcarubbi/UnitTestsExample). Agora é sua vez de colocar a mão na massa e aprimorar seus testes!
