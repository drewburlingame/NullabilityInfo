# <img src="/src/icon.png" height="30px"> NullabilityInfo

[![Build status](https://ci.appveyor.com/api/projects/status/636i70gvxfuwdq38?svg=true)](https://ci.appveyor.com/project/SimonCropp/NullabilityInfo)
[![NuGet Status](https://img.shields.io/nuget/v/NullabilityInfo.svg)](https://www.nuget.org/packages/NullabilityInfo/)

Code-only package that exposes top-level nullability information from reflection.

This feature is [coming in net6](https://github.com/dotnet/runtime/issues/29723). This package exposes the APIs to lower runtime. It supports `netstandard2.0` and up.

Designed to be compatible for libraries that are targeting multiple frameworks including `net6`.


## Copyright / Licensing

The csproj and nuget config that builds the package is under MIT.

The cs files inside the nuget are also MIT but are [Copyright (c) .NET Foundation and Contributors](https://github.com/dotnet/runtime/blob/main/LICENSE.TXT)


## NuGet package

https://nuget.org/packages/NullabilityInfo/


## Usage


### Example target class

Given the following class

<!-- snippet: Target.cs -->
<a id='snippet-Target.cs'></a>
```cs
class Target
{
    public string? StringField;
    public string?[] ArrayField;
    public Dictionary<string, object?> GenericField;
}
```
<sup><a href='/src/Tests/Target.cs#L1-L6' title='Snippet source file'>snippet source</a> | <a href='#snippet-Target.cs' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->


### NullabilityInfoContext

<!-- snippet: Usage -->
<a id='snippet-usage'></a>
```cs
[Fact]
public void Test()
{
    var type = typeof(Target);
    var arrayField = type.GetField("ArrayField");
    var genericField = type.GetField("GenericField");

    var context = new NullabilityInfoContext();

    var arrayInfo = context.Create(arrayField);

    Assert.Equal(NullabilityState.NotNull, arrayInfo.ReadState);
    Assert.Equal(NullabilityState.Nullable, arrayInfo.ElementType.ReadState);

    var genericInfo = context.Create(genericField);

    Assert.Equal(NullabilityState.NotNull, genericInfo.ReadState);
    Assert.Equal(NullabilityState.NotNull, genericInfo.GenericTypeArguments[0].ReadState);
    Assert.Equal(NullabilityState.Nullable, genericInfo.GenericTypeArguments[1].ReadState);
}
```
<sup><a href='/src/Tests/Samples.cs#L6-L29' title='Snippet source file'>snippet source</a> | <a href='#snippet-usage' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->


### NullabilityInfoExtensions

`NullabilityInfoExtensions` provides static and thread safe wrapper around <see cref="NullabilityInfoContext"/>. It adds three extension methods to each of ParameterInfo, PropertyInfo, EventInfo, and FieldInfo.

 * `GetNullabilityInfo`: returns the `NullabilityInfo` for the target info.
 * `GetNullability`: returns the `NullabilityState` for the state (`NullabilityInfo.ReadState` or `NullabilityInfo.WriteState` depending on which has more info) of target info.
 * `IsNullable`: given the state (`NullabilityInfo.ReadState` or `NullabilityInfo.WriteState` depending on which has more info) of the info:
   * Returns true if state is `NullabilityState.Nullable`.
   * Returns false if state is `NullabilityState.NotNull`.
   * Throws an exception if state is `NullabilityState.Unknown`.


## API

```
namespace System.Reflection
{
    public sealed class NullabilityInfoContext
    {
        public NullabilityInfo Create(ParameterInfo parameterInfo);
        public NullabilityInfo Create(PropertyInfo propertyInfo);
        public NullabilityInfo Create(EventInfo eventInfo);
        public NullabilityInfo Create(FieldInfo parameterInfo);
    }
}
```

<!-- snippet: NullabilityInfo.cs.pp -->
<a id='snippet-NullabilityInfo.cs.pp'></a>
```pp
#nullable enable
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.

using System.Collections.ObjectModel;

namespace System.Reflection
{
    /// <summary>
    /// A class that represents nullability info
    /// </summary>
    sealed class NullabilityInfo
    {
        internal NullabilityInfo(Type type, NullabilityState readState, NullabilityState writeState,
            NullabilityInfo? elementType, NullabilityInfo[] typeArguments)
        {
            Type = type;
            ReadState = readState;
            WriteState = writeState;
            ElementType = elementType;
            GenericTypeArguments = typeArguments;
        }

        /// <summary>
        /// The <see cref="System.Type" /> of the member or generic parameter
        /// to which this NullabilityInfo belongs
        /// </summary>
        public Type Type { get; }
        /// <summary>
        /// The nullability read state of the member
        /// </summary>
        public NullabilityState ReadState { get; internal set; }
        /// <summary>
        /// The nullability write state of the member
        /// </summary>
        public NullabilityState WriteState { get; internal set; }
        /// <summary>
        /// If the member type is an array, gives the <see cref="NullabilityInfo" /> of the elements of the array, null otherwise
        /// </summary>
        public NullabilityInfo? ElementType { get; }
        /// <summary>
        /// If the member type is a generic type, gives the array of <see cref="NullabilityInfo" /> for each type parameter
        /// </summary>
        public NullabilityInfo[] GenericTypeArguments { get; }
    }

    /// <summary>
    /// An enum that represents nullability state
    /// </summary>
    enum NullabilityState
    {
        /// <summary>
        /// Nullability context not enabled (oblivious)
        /// </summary>
        Unknown,
        /// <summary>
        /// Non nullable value or reference type
        /// </summary>
        NotNull,
        /// <summary>
        /// Nullable value or reference type
        /// </summary>
        Nullable
    }
}
```
<sup><a href='/src/NullabilityInfo/NullabilityInfo.cs.pp#L1-L65' title='Snippet source file'>snippet source</a> | <a href='#snippet-NullabilityInfo.cs.pp' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->


## How this is built

On every build the `NullabilityInfo.csproj` downloads `NullabilityInfo.cs` and `NullabilityInfoContext.cs`. This ensures that the bundled files are always up to date on each release.


## Icon

[Reflection](https://thenounproject.com/term/reflection/4087162/) designed by [Yogi Aprelliyanto](https://thenounproject.com/yogiaprelliyanto/) from [The Noun Project](https://thenounproject.com).
