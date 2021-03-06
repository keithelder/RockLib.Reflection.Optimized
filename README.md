# RockLib.Reflection.Optimized [![Build status](https://ci.appveyor.com/api/projects/status/p3ovl21n6hoih61f?svg=true)](https://ci.appveyor.com/project/RockLib/rocklib-reflection-optimized)

*Extension methods to improve reflection performance.*

##### Table of Contents
- [Installation](#installation)
  - [Nuget](#nuget)
  - [Git submodule / shared project](#git-submodule--shared-project)
- [PropertyInfo extension methods](#propertyinfo-extension-methods)
  - [CreateGetter / CreateSetter](#creategetter--createsetter)
    - [Overloads](#overloads)
    - [Static properties](#static-properties)
  - [CreateStaticGetter / CreateStaticSetter](#createstaticgetter--createstaticsetter)
    - [Overloads](#overloads-1)

------

## Installation

### Nuget

```powershell
PM> Install-Package RockLib.Reflection.Optimized
```

### Git submodule / shared project

Another way to consume this library is as a submodule to an existing git repository. Add the RockLib.Reflection.Optimized submodule to your git repository. Then, add the shared project found in the submodule at `/RockLib.Reflection.Optimized.Shared/RockLib.Reflection.Optimized.Shared.shproj` to your solution. Finally, add a shared project reference to the project that needs optimized reflection.

Consuming this library in this manner is intended for use by other libraries - it eliminates a nuget dependency and the optimized reflection extension methods are not exposed publicly.

## PropertyInfo extension methods

Currently, RockLib.Reflection.Optimized has extension methods for just one type: [`System.Reflection.PropertyInfo`](https://msdn.microsoft.com/en-us/library/system.reflection.propertyinfo.aspx). These extension methods create functions at runtime that access the specified property.

### CreateGetter / CreateSetter

The following code snippet demonstrates usage of the `CreateGetter` and `CreateSetter` extension methods:

```c#
using System.Reflection;
using RockLib.Reflection.Optimized;

public class Foo
{
    public int Bar { get; set; }
}

void Main()
{
    PropertyInfo property = typeof(Foo).GetProperty("Bar");
    
    Action<object, object> setBar = property.CreateSetter();
    Func<object, object> getBar = property.CreateGetter();

    Foo foo = new Foo();

    setBar(foo, 123); // Sets the value of the Bar property
    int bar = getBar(foo); // Gets the value of the Bar property
}
```

#### Overloads

The `CreateGetter` and `CreateSetter` extension methods each have three overloads, allowing the parameters of the resulting delegates to be customized.

| Overload  | Return Type |
| --- | --- |
| `CreateGetter` | `Func<object, object>` |
| `CreateSetter` | `Action<object, object>` |
| `CreateGetter<TPropertyType>` | `Func<object, TPropertyType>` |
| `CreateSetter<TPropertyType>` | `Action<object, TPropertyType>` |
| `CreateGetter<TDeclaringType, TPropertyType>` | `Func<TDeclaringType, TPropertyType>` |
| `CreateSetter<TDeclaringType, TPropertyType>` | `Action<TDeclaringType, TPropertyType>` |

#### Static properties

If a `PropertyInfo` represents a static property, then the first parameter of the functions returned by the `CreateGetter` and `CreateSetter` methods are ignored. When invokine functions that access static properties, the caller can safely pass `null` for the first parameter.

### CreateStaticGetter / CreateStaticSetter

If it is known that a `PropertyInfo` is static, then the `CreateStaticGetter` and `CreateStaticSetter` extension methods can be used, as in the following example:

```c#
using System.Reflection;
using RockLib.Reflection.Optimized;

public static class Foo
{
    public static int Bar { get; set; }
}

void Main()
{
    PropertyInfo property = typeof(Foo).GetProperty("Bar");
    
    Action<object> setBar = property.CreateStaticSetter();
    Func<object> getBar = property.CreateStaticGetter();

    Foo foo = new Foo();

    setBar(123); // Sets the value of the Foo.Bar property
    int bar = getBar(); // Gets the value of the Foo.Bar property
}
```

#### Overloads

Similar to their non-static counterpoints, the `CreateStaticGetter` and `CreateStaticSetter` extension methods each have two overloads, allowing the parameters of the resulting delegates to be customized.

| Overload  | Return Type |
| --- | --- |
| `CreateStaticGetter` | `Func<object>` |
| `CreateStaticSetter` | `Action<object>` |
| `CreateStaticGetter<TPropertyType>` | `Func<TPropertyType>` |
| `CreateStaticSetter<TPropertyType>` | `Action<TPropertyType>` |
