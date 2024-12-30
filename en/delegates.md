# 🤝 Exploring the Evolution of Delegates in C#: From Version 1.0 to the Present ⏳

Delegates in C# are powerful structures that allow you to treat methods as variables, storing references to methods with specific signatures. Just like `int` stores numbers and `string` stores text, a `delegate` stores a block of code that can be invoked whenever needed. Over the versions of C#, new delegate-related features have been introduced, each bringing significant improvements.

Let’s see how this functionality has evolved throughout the language versions:

## C# 1.0 – Delegates
Delegates were introduced in C# 1.0 as a way to encapsulate methods. They allow methods to be stored as variables and passed around as parameters.

### Basic Delegate Example

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

Delegates also support multiple references, creating what are known as **Multicast Delegates**:

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

## C# 2.0 – Events and Anonymous Delegates

In C# 2.0, **events** and **anonymous delegates** were introduced. Events encapsulate delegates and ensure that only the class that declares them can invoke them. Meanwhile, anonymous delegates allow you to create inline functions without the need for named methods.

### Event Example

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

class Program
{
    static void Main()
    {
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

### Anonymous Delegate Example

```csharp
NotifyDelegate notifier = delegate(string message)
{
    Console.WriteLine("Anonymous: " + message);
};
notifier("Hello from Anonymous Delegate!");
```

---

## C# 3.0 – Lambda Expressions, Func, Action, and Predicate

C# 3.0 introduced **Lambda Expressions** and the predefined generic delegates **Func**, **Action**, and **Predicate**, which further simplified the use of delegates.

### Lambda Expressions

```csharp
NotifyDelegate notifier = message => Console.WriteLine("Lambda: " + message);
notifier("Hello from Lambda!");
```

### Func, Action, and Predicate

- **Func**: For methods that return values, where the last type is the return type and the preceding types are the input parameter types.  
- **Action**: For methods that do not return values.  
- **Predicate**: For methods that return booleans.

**Example:**

```csharp
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(5, 3));

Action<string> log = message => Console.WriteLine("Log: " + message);
log("Log entry");

Predicate<int> isGreaterThanTen = number => number > 10;
Console.WriteLine(isGreaterThanTen(15));
```

### Expression Trees

Expression trees are a representation of code in a tree format. They are widely used by **LINQ** and allow for dynamic manipulation and evaluation of expressions at runtime.

### Practical Example with Dynamic Filter

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

## C# 6.0 – Expression-Bodied Members

**Expression-bodied members** were introduced in C# 6.0 to simplify the definition of methods and properties with simple returns. Instead of using full code blocks, you can define the behavior right after the `=>`, reducing the use of `{}` for simple methods.

### Basic Example of Expression-Bodied Members

```csharp
public class MathHelper
{
    public int Square(int x) => x * x;
    public string Greet(string name) => $"Hello, {name}!";
}
```

---

## C# 7.0 – Local Functions

### Local Functions

**Local functions** allow you to define methods inside other methods. This helps encapsulate logic that does not need to be exposed outside the current scope.

### Local Function Example

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

## C# 9.0 – Static Lambdas

In C# 9.0, the ability to define **static lambdas** was introduced. Static lambdas do not capture variables from the outer scope, which can improve performance and reduce memory consumption. When you use the `static` modifier in a lambda expression, you prevent it from accessing local variables or instance members of the outer scope. This makes the lambda lighter because it does not need to create a reference to the external context.

### Static Lambda Example

```csharp
Func<int, int> square = static x => x * x;
Console.WriteLine(square(5));
```

In the example above, the lambda `static x => x * x` cannot access any variables from the outer scope. This avoids unnecessary captures and improves performance.

---

## C# 10.0 – Type Inference and Attributes in Lambdas

C# 10.0 introduced significant improvements for lambda expressions, including **return type inference** and **attribute support**.

### Type Inference in Lambdas

Before C# 10.0, lambdas had to have their return type inferred based on the delegate or the `Func/Action/Predicate` interface. In C# 10.0, the compiler can infer the return type directly.

#### Type Inference Example

```csharp
var square = (int x) => x * x;
Console.WriteLine(square(4));
```

The compiler infers that `square` is of type `Func<int, int>`.

### Attributes in Lambdas

It is now possible to add attributes directly to lambda expressions.

#### Example with Attribute

```csharp
var lambda = [Obsolete("Use another method")] (int x) => x * x;
Console.WriteLine(lambda(4));
```

This allows you to provide additional metadata for the lambdas, such as deprecation warnings, validations, or descriptions.

---

## Conclusion

Delegates have evolved significantly since their introduction in C# 1.0. With the arrival of lambdas, expression trees, local functions, attributes, and type inference, they have become an incredibly flexible and powerful tool. Each new version of C# has brought improvements that make code more readable, efficient, and expressive.

By mastering delegates, lambdas, and their modern variations, you will be prepared to tackle complex challenges and create elegant solutions in the .NET ecosystem.

Practice the examples shown, explore different scenarios, and keep learning!