# ➗ Simplify Calculations with Units of Measure in .NET Using UnitsNet 🧮

The [UnitsNet](https://github.com/angularsen/UnitsNet) library is an excellent option for developers who want to handle units of measure in .NET projects in a precise, practical, and cleaner way. This article will explore the features of UnitsNet, highlighting how it can simplify conversions and calculations with units of measure, while also avoiding common errors related to manual conversion.

## What is UnitsNet?

UnitsNet is an open-source library for .NET designed to make working with units of measure easier, offering a wide variety of predefined units (such as meters, liters, seconds, etc.) and automatic conversion between compatible units. The library is often used in scientific, engineering, and commercial applications that require high precision and clarity when working with measurements.

## Installing UnitsNet

To start using UnitsNet, you can install it via NuGet with the following command:

```bash
Install-Package UnitsNet
```

Or, if you prefer the .NET CLI:

```bash
dotnet add package UnitsNet
```

Once installed, you can begin using the features offered by the library.

## Key Features

### 1. Creating Objects with Units

UnitsNet allows you to create objects representing a specific unit of measure. Suppose you need to work with a distance in meters; you can instantiate the `Length` class like this:

```csharp
using UnitsNet;

Length distance = Length.FromMeters(100);
Console.WriteLine($"Distance: {distance}");
```

This code creates a `Length` object representing 100 meters. The same concept applies to many other units, such as `Mass`, `Temperature`, `Volume`, `Speed`, and more.

### 2. Automatic Unit Conversion

One of the most useful features of UnitsNet is automatic conversion between units. For example, you can easily convert meters to miles or kilometers:

```csharp
Length distanceInMiles = distance.ToUnit(UnitsNet.Units.LengthUnit.Mile);
Console.WriteLine($"Distance in miles: {distanceInMiles}");
```

Additionally, UnitsNet automatically converts between different unit systems (metric, imperial, etc.), making it easier to adapt applications for users in different regions.

### 3. Mathematical Operations with Units

UnitsNet allows direct mathematical operations between units. For example, you can add two distances, multiply an area by a length to get volume, and more:

```csharp
Length distance1 = Length.FromMeters(50);
Length distance2 = Length.FromMeters(30);
Length totalDistance = distance1 + distance2;

Console.WriteLine($"Total distance: {totalDistance.Meters} meters");
```

In cases where operations involve different units of measure, UnitsNet internally performs the necessary conversions, ensuring the correct value.

### 4. Unit Comparison

The library also makes it easy to compare units of measure, allowing you to check which value is greater or smaller with intuitive code:

```csharp
if (distance1 > distance2)
{
    Console.WriteLine("The first distance is greater.");
}
else
{
    Console.WriteLine("The second distance is greater.");
}
```

### 5. Working with Composite Physical Units

In addition to simple units, UnitsNet supports composite units. For example, you can work with `Area` and `Volume`, which are combinations of other units (square meters, cubic meters, etc.):

```csharp
Area area = Area.FromSquareMeters(20);
Length height = Length.FromMeters(3);

Volume volume = area * height;
Console.WriteLine($"Volume: {volume.CubicMeters} cubic meters");
```

This feature allows you to express physical relationships directly in code, improving readability and maintainability.

### 6. Formatting and Localization

UnitsNet enables you to display values formatted according to the local measurement system. This is useful for global applications that need to adapt units and formatting for different audiences.

```csharp
Temperature temperature = Temperature.FromDegreesCelsius(30);
Console.WriteLine(temperature.ToString("c", CultureInfo.InvariantCulture));  // Output: 30 °C
```

With this, you can easily adapt your system to work with standard formats, like Fahrenheit for the United States and Celsius for Brazil and Europe.

### 7. Working with Temperatures

Temperature is a unit of measure with different scales (Celsius, Kelvin, Fahrenheit). UnitsNet provides convenient conversions between them:

```csharp
Temperature tempCelsius = Temperature.FromDegreesCelsius(25);
Temperature tempFahrenheit = tempCelsius.ToUnit(UnitsNet.Units.TemperatureUnit.Fahrenheit);

Console.WriteLine($"Temperature in Fahrenheit: {tempFahrenheit}");
```

Besides supporting conversion operations, UnitsNet helps prevent common errors that occur when trying to do these conversions manually.

## Practical Example: Currency Converter

UnitsNet can be easily combined with other APIs to create a currency converter, calculating the value of a unit based on an exchange rate. Although UnitsNet does not directly support monetary values, you can use the library to calculate proportional rates. An example would be calculating the cost of a liter of fuel in another currency using an exchange rate:

```csharp
decimal conversionRate = 5.0m; // Example of a fictitious rate
Length length = Length.FromMeters(100);
decimal finalValue = length.Meters * conversionRate;
Console.WriteLine($"Converted value: {finalValue}");
```

## Conclusion

UnitsNet simplifies complex operations with units, avoiding manual conversions and reducing the risk of errors in code. It’s a practical, flexible, and easy-to-use tool, especially useful for developers working with physical measurements, whether basic or composite.

With UnitsNet, you gain clarity and precision, standardizing the use of units in your .NET project, be it scientific, engineering, or commercial.

## Comments

The Microsoft Bot Framework has a really cool feature called FormFlow, which is a convenient way to build interactive forms in the bot, perfect for when you need to collect structured information like name, email, preferences, and such without having to code each question manually. It guides the user through a conversation, making the experience feel more natural.

In Chatbot Studio, I implemented the Form block to simplify using FormFlow. Just add the block and configure the fields the bot should ask for, and you’re done! Additionally, you can perform custom validations by calling external APIs, allowing you to check user responses as needed. You can even suppress questions with specific logic injected via API, making the form highly adaptable and intelligent. At the end, you can display a summary of all responses and ask the user to confirm or even edit any response.
