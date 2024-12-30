# 📟 Introdução ao Spectre.Console: Criando Interfaces de Linha de Comando Modernas e Elegantes 📦

No desenvolvimento de ferramentas de linha de comando (CLI) com .NET, frequentemente nos deparamos com desafios relacionados à apresentação de informações de forma clara e amigável. Felizmente, o **Spectre.Console** surgiu como uma solução poderosa para simplificar a criação de CLIs interativas, modernas e visualmente atraentes.

Neste artigo, exploraremos as principais funcionalidades do **Spectre.Console**, como ele pode melhorar suas ferramentas de linha de comando e como começar a utilizá-lo em seus projetos .NET.

---

## O que é Spectre.Console?

**Spectre.Console** é uma biblioteca open-source para .NET que permite criar interfaces de linha de comando (CLI) robustas, interativas e esteticamente agradáveis. Ele oferece suporte para renderização avançada de texto, tabelas, barras de progresso, prompts interativos e muito mais.

Desenvolvido para ser flexível e fácil de usar, Spectre.Console se destaca por:

- **Interface rica:** Suporte para tabelas, gráficos e destaques.
- **Interatividade:** Prompts para entrada de dados e seleção de opções.
- **Barras de progresso:** Para acompanhar tarefas demoradas.
- **Comandos estruturados:** Organização modular de comandos.

---

## Principais Recursos do Spectre.Console

### 1. **Saída de Texto com Estilo**

Spectre.Console permite formatar texto de forma avançada, usando estilos como cores, negrito, itálico e sublinhado.

```csharp
AnsiConsole.Markup("[green]Olá, mundo![/]");
```

### 2. **Criação de Tabelas**

Você pode exibir dados tabulares de forma elegante:

```csharp
var table = new Table();
table.AddColumn("[yellow]Nome[/]");
table.AddColumn("[blue]Idade[/]");

table.AddRow("Alice", "30");
table.AddRow("Bob", "25");

AnsiConsole.Write(table);
```

### 3. **Barras de Progresso**

Exiba o progresso de tarefas demoradas:

```csharp
AnsiConsole.Progress()
    .Start(ctx => {
        var task = ctx.AddTask("[green]Processando...[/]");
        while (!ctx.IsFinished)
        {
            task.Increment(10);
            Thread.Sleep(500);
        }
    });
```

### 4. **Interatividade com Prompts**

Receba entrada de forma intuitiva:

```csharp
var name = AnsiConsole.Ask<string>("Qual é o seu [green]nome[/]?");
AnsiConsole.MarkupLine($"Olá, [blue]{name}[/]!");
```

### 5. **Sistema de Comandos Estruturado**

Spectre.Console também inclui um sistema robusto para criação de comandos hierárquicos, ideal para ferramentas CLI complexas.

```csharp
using Spectre.Console.Cli;

public class GreetCommand : Command<GreetCommand.Settings>
{
    public class Settings : CommandSettings
    {
        [CommandOption("--name")]
        public string Name { get; set; }
    }

    public override int Execute(CommandContext context, Settings settings)
    {
        AnsiConsole.MarkupLine($"Olá, [green]{settings.Name}[/]!");
        return 0;
    }
}
```

### 6. **Renderização de Árvores Hierárquicas**

Exiba dados hierárquicos de forma clara:

```csharp
var root = new Tree("Raiz");
var node1 = root.AddNode("Nó 1");
node1.AddNode("Subnó 1.1");
node1.AddNode("Subnó 1.2");
var node2 = root.AddNode("Nó 2");
node2.AddNode("Subnó 2.1");

AnsiConsole.Write(root);
```

### 7. **Gráficos de Barras Horizontais**

Crie representações visuais de dados:

```csharp
var chart = new BarChart()
    .Width(60)
    .Label("[green bold]Vendas por Mês[/]")
    .AddItem("Janeiro", 40, Color.Green)
    .AddItem("Fevereiro", 30, Color.Yellow)
    .AddItem("Março", 25, Color.Red);

AnsiConsole.Write(chart);
```

### 8. **Renderização de Imagens ASCII**

Crie representações gráficas simples:

```csharp
var canvas = new Canvas(20, 10);
canvas.SetPixel(5, 5, Color.Red);
canvas.SetPixel(10, 5, Color.Green);
canvas.SetPixel(15, 5, Color.Blue);

AnsiConsole.Write(canvas);
```

### 9. **Manipulação de Layouts com Grid**

Organize saídas em colunas:

```csharp
var grid = new Grid();
grid.AddColumn();
grid.AddColumn();

grid.AddRow("Nome", "Raphael");
grid.AddRow("Idade", "35");
grid.AddRow("Profissão", "Desenvolvedor");

AnsiConsole.Write(grid);
```

### 10. **Exibição de Regras Horizontais**

Separe visualmente seções:

```csharp
AnsiConsole.Write(new Rule("[yellow]Seção 1[/]"));
AnsiConsole.WriteLine("Conteúdo da seção 1");

AnsiConsole.Write(new Rule("[green]Seção 2[/]"));
AnsiConsole.WriteLine("Conteúdo da seção 2");
```

---

## Conclusão

O **Spectre.Console** é uma ferramenta poderosa para qualquer desenvolvedor .NET que queira criar CLIs modernas e funcionais. Seja para exibir tabelas organizadas, manipular entradas do usuário ou acompanhar progresso com barras elegantes, Spectre.Console facilita todas essas tarefas.

Se você ainda não experimentou essa biblioteca, agora é a hora! Explore, experimente e leve suas ferramentas CLI para o próximo nível.

---

**Links Úteis:**
- Repositório GitHub: [Spectre.Console](https://github.com/spectreconsole/spectre.console)
- Documentação Oficial: [Spectre.Console Docs](https://spectreconsole.net/)

Pronto para começar sua jornada com Spectre.Console? 🚀

