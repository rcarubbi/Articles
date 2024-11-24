# 🧪 Mastering the Art of Unit Testing in .NET with xUnit, Moq, Fluent Assertions, Bogus, and Verify ⚗️

Unit testing is an essential tool for ensuring that code behaves as expected while promoting confidence in changes and refactoring. In this article, we will explore how to create efficient unit tests using some popular packages in the .NET ecosystem. Additionally, we’ll look at how to structure tests for a project following Clean Architecture principles.

The complete code for this example is available on [GitHub](https://github.com/rcarubbi/UnitTestsExample).

---

## Introduction to Unit Testing

Unit tests are responsible for validating the behavior of small units of code (usually methods or classes). The goal is to isolate the unit being tested, ensuring it works independently of other parts of the system. With the help of tools like mocks, external dependencies can be replaced with controlled implementations.

Below, we will introduce the packages used in this article and how each contributes to the testing process.

---

## Packages Used in This Example

### 1. xUnit
xUnit is a popular testing framework in .NET. It provides attributes like `[Fact]` and `[Theory]` to define tests, along with seamless integration with test runners such as Visual Studio and the .NET CLI.

Example:
```csharp
[Fact]
public async Task Handle_ShouldCreateCustomer_WhenCommandIsValid()
{
    // Your test code here!
}
```

### 2. Moq
Moq is a mocking library that lets you create simulated objects to replace dependencies in unit tests. With it, you can configure expected behaviors for methods and verify if they were called.

In the example, we use Moq to mock a repository:
```csharp
_repositoryMock.Setup(x => x.Add(It.Is<Customer>(c => Compare(c, customer)), CancellationToken.None))
    .ReturnsAsync(true);
```

### 3. Fluent Assertions
Fluent Assertions is used to create readable and expressive assertions in tests. It simplifies the validation of values, objects, and exceptions.

Example of comparison:
```csharp
actual.Should().BeEquivalentTo(expected, options => options.Excluding(c => c.Id));
```

In this project, Fluent Assertions is mainly used in the `Comparer` for complex objects like `Customer`, as standard comparisons only check references.

### 4. Verify
Verify is a snapshot testing library. It saves an expected version of a test result (snapshot) and compares it against future results. If there are discrepancies, the test fails, and a file comparison tool opens to review and adjust the snapshots.

First run:
- A snapshot is generated, and the test fails because no previous snapshot exists.
- The developer reviews the differences and saves the snapshot.
- In subsequent runs, the results are compared against the saved snapshot.

### 5. Bogus
Bogus is a library for generating fake data. It’s used to create consistent and realistic test data, such as names, dates, emails, etc.

In the example, we use Bogus to create instances of the `Customer` entity:
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

## Code Example: Creating and Testing a Command Handler

### The Command Handler
In this example, we follow Clean Architecture principles, where `Command Handlers` are encapsulated in the application layer and are not directly exposed to the API. They are invoked via a Mediator, reducing coupling.

To allow the tests to access `internal` classes, we add the following configuration to the application project:
```xml
<ItemGroup>
    <AssemblyAttribute Include="System.Runtime.CompilerServices.InternalsVisibleTo">
        <_Parameter1>Application.Tests</_Parameter1>
    </AssemblyAttribute>
</ItemGroup>
```

### Test Structure
#### Setup with Moq and Comparer
Since `Customer` is a complex type, we need a comparer to ensure the comparison is done by properties rather than by reference:
```csharp
private static bool Compare<T>(T actual, T expected) where T : IEntity
{
    actual.Should().BeEquivalentTo(expected, options => options.Excluding(c => c.Id));
    return true;
}
```

#### Testing the Handler's Behavior
- We configure the repository mock to expect a `Customer` equivalent to the command.
- We use `Verify` to validate the response.

Example:
```csharp
var sut = new CreateCustomerCommandHandler(_repositoryMock.Object);
var response = await sut.Handle(command, CancellationToken.None);

_repositoryMock.Verify(x => x.Add(It.Is<Customer>(c => Compare(c, customer)), CancellationToken.None), Times.Once);
await Verify(response, verifySettings);
```

---

## Handling Randomization in Snapshots

For fields with random values, like names or GUIDs, we use `VerifySettings` to apply scrubs:
```csharp
var verifySettings = new VerifySettings();
verifySettings.ScrubMember<CreateCustomerResponse>(f => f.FirstName);
verifySettings.ScrubMember<CreateCustomerResponse>(f => f.LastName);
```

---

## Unit Test Execution Flow Diagram

Here is the flow of how a unit test is executed, using the tools mentioned:

[![](https://mermaid.ink/img/pako:eNqFkk9vozAQxb-K5XMaQRYI-LBSd5PdU7VS0_ZQ6MHCE2IVbNZ_2qRJvnsH3KTRqtIiwfCefvM8NuxprQVQRtetfq033Dhyt6hMpQheK4e6vFcSTbAuaPtErq6-k2tjuGqg_KiMrMCR-z6AS_UijVYdKPd0CvsAx-bDb1BguANyo-tne8Dyt8SbjRoEWUAPSoCqJdj_JCy44wfyQzfeluOThRkG_8vWO24acDfgNlqUyy3UHlOCSYJ7brtEw66tBTySUBh54K0UwxC3YH3rLkYdgTDpSnZ9CwPrAbf6C4tyAZBa2TIY5NP5MuWnHlK25I93vXeY8wBGrndlKIxcixeuajy6AeRGWq3OOf8uOSYulbj4tKg-1w2ZJ4pOaAem41Lgb7IfiIq6DXRQUYavgpvnilbqiBz3Tq92qqbMGQ8TarRvNpSteWtR-X44q4XkjeHd2e25etS6O7WgpGxPt5Sl0ywr4uJbls_TYh7F2YTu0M2nySyK5nma5EmSRml6nNC3MSCaFrMsjnNEZ8msyOPjO39u9MA?type=png)](https://mermaid.live/edit#pako:eNqFkk9vozAQxb-K5XMaQRYI-LBSd5PdU7VS0_ZQ6MHCE2IVbNZ_2qRJvnsH3KTRqtIiwfCefvM8NuxprQVQRtetfq033Dhyt6hMpQheK4e6vFcSTbAuaPtErq6-k2tjuGqg_KiMrMCR-z6AS_UijVYdKPd0CvsAx-bDb1BguANyo-tne8Dyt8SbjRoEWUAPSoCqJdj_JCy44wfyQzfeluOThRkG_8vWO24acDfgNlqUyy3UHlOCSYJ7brtEw66tBTySUBh54K0UwxC3YH3rLkYdgTDpSnZ9CwPrAbf6C4tyAZBa2TIY5NP5MuWnHlK25I93vXeY8wBGrndlKIxcixeuajy6AeRGWq3OOf8uOSYulbj4tKg-1w2ZJ4pOaAem41Lgb7IfiIq6DXRQUYavgpvnilbqiBz3Tq92qqbMGQ8TarRvNpSteWtR-X44q4XkjeHd2e25etS6O7WgpGxPt5Sl0ywr4uJbls_TYh7F2YTu0M2nySyK5nma5EmSRml6nNC3MSCaFrMsjnNEZ8msyOPjO39u9MA)

---

## Verify Configuration for Git

### `.gitignore`
We add these entries to ignore temporary files created by Verify:
```gitignore
# Verify
*.received.*
*.received/
```

### `.gitattributes`
We add configurations to normalize snapshot files:
```gitattributes
# Verify
*.verified.txt text eol=lf working-tree-encoding=UTF-8
```

The `VerifyChecks.Run()` method validates these configurations and suggests adjustments when necessary:
```csharp
[Fact]
public Task VerifyCheck() => VerifyChecks.Run();
```

---

## Test Naming Conventions
There are two main naming patterns:

1. **Given... When... Then...**
   - Example: `GivenValidCommand_WhenHandleIsCalled_ThenCustomerIsCreated`.
2. **Action... Should... When...**
   - Example: `Handle_ShouldCreateCustomer_WhenCommandIsValid`.

Both are valid, and the choice depends on the team or project.

---

## Conclusion
By combining xUnit, Moq, Fluent Assertions, Verify, and Bogus, we create powerful, readable, and robust unit tests. This example demonstrated how to handle challenges like comparing complex objects and random values while leveraging Verify for snapshot testing.

The complete code for this example is available on [GitHub](https://github.com/rcarubbi/UnitTestsExample). Now it’s your turn to get hands-on and improve your tests!
