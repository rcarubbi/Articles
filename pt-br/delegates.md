# 🤝 Explorando a Evolução dos Delegates no C#: Da Versão 1.0 à Atualidade ⏳

Delegates em C# são estruturas poderosas que permitem tratar métodos como variáveis, armazenando referências para métodos com assinaturas específicas. Assim como `int` armazena números e `string` armazena texto, um `delegate` armazena um bloco de código que pode ser invocado sempre que necessário. Ao longo das versões do C#, novos recursos relacionados a delegates foram introduzidos, cada um trazendo melhorias significativas.

Vamos ver como esta funcionalidade evoluiu através das versões da linguagem:

## C# 1.0 - Delegates
Os delegates foram introduzidos no C# 1.0 como uma maneira de encapsular métodos. Eles permitem que métodos sejam armazenados como variáveis e passados como parâmetros.

### Exemplo Básico de Delegate:
```csharp
public delegate void NotifyDelegate(string message);

public class Notifier
{
    public static void SendEmail(string message)
    {
        Console.WriteLine("Email sent: " + message);
    }

    public static void ExecuteNotification(NotifyDelegate notify, string message)
    {
        notify(message);
    }

    static void Main()
    {
        ExecuteNotification(SendEmail, "You have a new message!");
    }
}
```

Os delegates também suportam múltiplas referências, criando os chamados **Multicast Delegates**:
```csharp
NotifyDelegate notifier = SendEmail;
notifier += SendSMS;

notifier("You have a new notification!");

public static void SendSMS(string message)
{
    Console.WriteLine("SMS sent: " + message);
}
```

---

## C# 2.0 - Eventos e Delegates Anônimos
No C# 2.0, os **eventos** e os **delegates anônimos** foram introduzidos. Eventos encapsulam delegates e garantem que apenas a classe que os declara pode invocá-los. Já os delegates anônimos permitem criar funções inline sem a necessidade de métodos nomeados.

### Exemplo de Evento:
```csharp
using System;

public class DownloadEventArgs : EventArgs
{
    public string File { get; set; }
}

public class DownloadManager
{
    public event EventHandler<DownloadEventArgs> DownloadCompleted;

    public void DownloadFile(string file)
    {
        Console.WriteLine($"Downloading {file}...");
        System.Threading.Thread.Sleep(1000);
        OnDownloadCompleted(file);
    }

    protected virtual void OnDownloadCompleted(string file)
    {
        DownloadCompleted?.Invoke(this, new DownloadEventArgs { File = file });
    }
}

class Program {

    static void Main() {
        var manager = new DownloadManager();
        manager.DownloadCompleted += manager_DownloadCompleted;

        void manager_DownloadCompleted(object? sender, DownloadEventArgs e)
        {
            Console.WriteLine($"{e.File} dowload completed");
        }

        manager.DownloadFile("test.png");
    }
}
```

### Exemplo de Delegate Anônimo:
```csharp
NotifyDelegate notifier = delegate(string message)
{
    Console.WriteLine("Anonymous: " + message);
};
notifier("Hello from Anonymous Delegate!");
```

---

## C# 3.0 - Expressões Lambda, Func, Action e Predicate
O C# 3.0 trouxe **Expressões Lambda** e os delegates genéricos pré-definidos **Func**, **Action** e **Predicate**, que simplificaram ainda mais o uso de delegates.

### Expressões Lambda:
```csharp
NotifyDelegate notifier = message => Console.WriteLine("Lambda: " + message);
notifier("Hello from Lambda!");
```

### Func, Action e Predicate:
- **Func**: Para métodos que retornam valores onde o ultimo tipo é o tipo de retorno e os demais são os tipos dos parâmetros de entrada.
- **Action**: Para métodos que não retornam valores.
- **Predicate**: Para métodos que retornam booleanos.

Exemplo:
```csharp
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(5, 3));

Action<string> log = message => Console.WriteLine("Log: " + message);
log("Log entry");

Predicate<int> isGreaterThanTen = number => number > 10;
Console.WriteLine(isGreaterThanTen(15));
```

### Árvores de Expressão (Expression Trees)
A árvore de expressão é uma representação de um código em formato de árvore. Elas são muito usadas pelo **LINQ** e permitem manipulação e avaliação dinâmica de expressões em tempo de execução.

### Exemplo Prático com Filtro Dinâmico:
```csharp
using System;
using System.Linq.Expressions;
using System.Collections.Generic;
using System.Linq;

class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}

class Program
{
    static void Main()
    {
        List<Product> products = new List<Product>
        {
            new Product { Name = "Product A", Price = 10 },
            new Product { Name = "Product B", Price = 20 },
            new Product { Name = "Product C", Price = 30 }
        };

        ParameterExpression param = Expression.Parameter(typeof(Product), "product");
        Expression property = Expression.Property(param, "Price");
        Expression constant = Expression.Constant(20M);
        Expression filter = Expression.GreaterThan(property, constant);

        var lambda = Expression.Lambda<Func<Product, bool>>(filter, param);
        var func = lambda.Compile();

        var results = products.Where(func);
        foreach (var product in results)
        {
            Console.WriteLine(product.Name);
        }
    }
}
```

---

## C# 6.0 - Expressões de Membros

As **Expressões de Membros** foram introduzidas no C# 6.0 para simplificar a definição de métodos e propriedades com retornos simples. Em vez de usar blocos de código completos, você pode definir o comportamento diretamente após a seta `=>` reduzindo o uso de blocos `{}` para métodos simples.

### Exemplo Básico de Expressões de Membros:
```csharp
public class MathHelper
{
    public int Square(int x) => x * x;
    public string Greet(string name) => $"Hello, {name}!";
}
```

---

## C# 7.0 - Funções Locais e Correspondência de Padrões

### Funções Locais
As **Funções Locais** permitem definir métodos dentro de outros métodos. Isso ajuda a encapsular lógica que não precisa ser exposta fora do escopo atual.

### Exemplo de Função Local:
```csharp
class Program
{
    static void Main()
    {
        void DisplaySquare(int number)
        {
            int Square(int x) => x * x;
            Console.WriteLine($"Square of {number} is {Square(number)}");
        }

        DisplaySquare(5);
    }
}
```

---

## C# 9.0 - Lambdas Estáticas

No C# 9.0, foi introduzida a capacidade de definir **Lambdas Estáticas**. Lambdas estáticas não capturam variáveis do escopo externo, o que pode melhorar o desempenho e reduzir o consumo de memória. Quando você usa o modificador `static` em uma expressão lambda, você impede que ela acesse variáveis locais ou membros da instância do escopo externo. Isso torna a lambda mais leve, pois não precisa criar uma referência ao contexto externo.

### Exemplo de Lambda Estática:
```csharp
Func<int, int> square = static x => x * x;
Console.WriteLine(square(5));
```

No exemplo acima, a lambda `static x => x * x` não pode acessar nenhuma variável do escopo externo. Isso evita capturas desnecessárias e melhora o desempenho.

---

## C# 10.0 - Inferência de Tipo e Atributos em Lambdas

O C# 10.0 trouxe melhorias significativas para expressões lambda, incluindo **Inferência de Tipo de Retorno** e **Suporte a Atributos**.

### Inferência de Tipo em Lambdas
Antes do C# 10.0, lambdas precisavam ter seu tipo de retorno inferido com base no delegate ou na interface Func/Action/Predicate. No C# 10.0, o compilador pode inferir o tipo de retorno diretamente.

### Exemplo de Inferência de Tipo:
```csharp
var square = (int x) => x * x;
Console.WriteLine(square(4));
```

O compilador infere que `square` é do tipo `Func<int, int>`.

### Atributos em Lambdas
Agora é possível adicionar atributos diretamente às expressões lambda.

### Exemplo com Atributo:
```csharp
var lambda = [Obsolete("Use another method")] (int x) => x * x;
Console.WriteLine(lambda(4));
```

Isso permite que você forneça metadados adicionais às lambdas, como advertências de obsolescência, validações ou descrições.

---

## Conclusão

Delegates evoluíram muito desde sua introdução no C# 1.0. Com a chegada de lambdas, árvores de expressão, funções locais, atributos e inferência de tipos, eles se tornaram uma ferramenta incrivelmente flexível e poderosa. Cada nova versão do C# trouxe melhorias que tornam o código mais legível, eficiente e expressivo.

Ao dominar delegates, lambdas e suas variações modernas, você estará preparado para enfrentar desafios complexos e criar soluções elegantes no ecossistema .NET.

Pratique os exemplos apresentados, explore diferentes cenários e continue aprendendo!