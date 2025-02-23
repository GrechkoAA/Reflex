<p align="center">
  <img src="Graphics\logo-crimson.png" width="250">
</p>
<p align="center">
  <img src="Graphics\text-crimson.png" width="300">
</p>

Reflex is an [Dependency Injection](https://stackify.com/dependency-injection/) framework for [Unity](https://unity.com/). Making your classes independent of its dependencies, granting better separation of concerns. It achieves that by decoupling the usage of an object from its creation. This helps you to follow SOLID’s dependency inversion and single responsibility principles. Making your project more **readable, testable and scalable.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![](https://github.com/gustavopsantos/reflex/workflows/Tests/badge.svg)
[![PullRequests](https://img.shields.io/badge/PRs-welcome-blueviolet)](http://makeapullrequest.com)
[![Releases](https://img.shields.io/github/release/gustavopsantos/reflex.svg)](https://github.com/gustavopsantos/reflex/releases)
[![openupm](https://img.shields.io/npm/v/com.gustavopsantos.reflex?label=openupm&registry_uri=https://package.openupm.com)](https://openupm.com/packages/com.gustavopsantos.reflex/)
[![Unity](https://img.shields.io/badge/Unity-2021+-black.svg)](https://unity3d.com/pt/get-unity/download/archive)

## Features 
- **Fast:** ~3x faster than VContainer, ~10x faster than Zenject.
- **GC Friendly:** ~2x less allocations than VContainer, ~12x less allocations than Zenject.
- **AOT Support:** Basically theres no runtime no Emit, so it works fine on IL2CPP builds.

## Contract Table
Contract table allows you to define multiple contracts for a single binding, allowing you to then retrieve multiple bindings that implements a given contract.

|                    | PrefabManager | BundleManager |
|--------------------|---------------|---------------|
| **IManager**       | ✅           | ✅      	    |
| **IDisposable**    | ✅     		   | ✅      	    |
| **IPrefabManager** | ✅        	 | ❌          	|
| **IBundleManager** | ❌	         | ✅	          |

### Installing

```csharp
public class ProjectInstaller : MonoBehaviour, IInstaller
{
    public void InstallBindings(ContainerDescriptor descriptor)
    {
        descriptor.AddSingleton(concrete: typeof(BundleManager), contracts: new[]
        {
            typeof(IBundleManager), typeof(IManager), typeof(IDisposable)
        });
        
        descriptor.AddSingleton(concrete: typeof(PrefabManager), contracts: new[]
        {
            typeof(IPrefabManager), typeof(IManager), typeof(IDisposable)
        });
    }
}
```
### Retrieving
```csharp
    public void InitializeManagers(Container container)
    {
        foreach (var manager in container.All<IManager>())
        {
            manager.Initialize();
        }
    }
```



## Performance
> Resolving ten thousand times a transient dependency with four levels of chained dependencies. See [NestedBenchmarkReflex.cs](Assets/Reflex.Benchmark/NestedBenchmarkReflex.cs).

### Android

<table>
<tr><th>Mono</th><th>IL2CPP</th></tr>
<tr><td>

|           | GC    | Time |
|-----------|------:|-----:|
| Reflex    |  54KB | 10ms
| Zenject   | 464KB | 73ms
| VContainer| 128KB | 51ms

</td><td>

|           | GC    | Time |
|-----------|------:|-----:|
| Reflex    |  70KB | 15ms
| Zenject   | 480KB | 77ms
| VContainer| 128KB | 18ms

</td></tr> </table>

### Windows

<table>
<tr><th>Mono</th><th>IL2CPP</th></tr>
<tr><td>

|           | GC    | Time |
|-----------|------:|-----:|
| Reflex    | 109KB | 1ms
| Zenject   | 900KB | 7ms
| VContainer| 257KB | 3ms

</td><td>

|           | GC    | Time |
|-----------|------:|-----:|
| Reflex    | 140KB | 1ms
| Zenject   | 900KB | 7ms
| VContainer| 257KB | 2ms

</td></tr> </table>

> The performance on `IL2CPP (AOT)` backend is not so good because the expressions are actually interpreted, unlike `Mono (JIT)`, where they are actually compiled.

> I'm investigating whether dealing with IL Reweaving is worth the complexity it brings.

## Installation

*Requires Unity 2021+*

### Install via UPM (using Git URL)
```
https://github.com/gustavopsantos/reflex.git?path=/Assets/Reflex/#4.0.0
```

### Install manually (using .unitypackage)
1. Download the .unitypackage from [releases](https://github.com/gustavopsantos/reflex/releases) page.
2. Import Reflex.X.X.X.unitypackage

## Getting Started

### Installing Bindings

Create your IInstaller implementation to install your bindings in the desired context (eg. ProjectContext or SceneContext), and remember to attach this component directly or as a child of your context prefab. See [SceneContext](Assets/Reflex.Sample/Reflex.Sample.unity) gameobject for reference.

```csharp
public class ProjectInstaller : MonoBehaviour, IInstaller
{
    public void InstallBindings(ContainerDescriptor descriptor)
    {
        descriptor.AddInstance(42);
        descriptor.AddTransient(typeof(BundleManager), typeof(IBundleManager));
        descriptor.AddTransient(typeof(PrefabManager), typeof(IPrefabManager));
    }
}
```

### MonoBehaviour Injection

> Be aware that fields and properties with [Inject] are injected only into pre-existing MonoBehaviours within the scene after the SceneManager.sceneLoaded event, which happens after Awake and before Start. See [MonoInjector.cs](Assets/Reflex/Scripts/Injectors/MonoInjector.cs).  

> If you want to instantiate a MonoBehaviour/Component at runtime and wants injection to happen, use the `Instantiate` method from Container.

```csharp
public class MonoBehaviourInjection : MonoBehaviour
{
    [Inject] private readonly Container _container;
    [Inject] public IDependencyOne DependencyOne { get; private set; }

    [Inject]
    private void Inject(Container container, IDependencyOne dependencyOne)
    {
        var dependencyTwo = container
            .Resolve(typeof(IDependencyTwo));
    }

    private void Start()
    {
        var dependencyTwo = _container
            .Resolve(typeof(IDependencyTwo));

        var answerForLifeTheUniverseAndEverything = _container
            .Resolve<int>();
    }
}
```

### Non MonoBehaviour Injection

```csharp
public class NonMonoBehaviourInjection
{
    private readonly Container _container;
    private readonly IDependencyOne _dependencyOne;
    private readonly int _answerForLifeTheUniverseAndEverything;

    public NonMonoBehaviourInjection(Container container, IDependencyOne dependencyOne, int answerForLifeTheUniverseAndEverything)
    {
        _container = container;
        _dependencyOne = dependencyOne;
        _answerForLifeTheUniverseAndEverything = answerForLifeTheUniverseAndEverything;
    }
}
```

## Order of Execution when a Scene is Loaded

| Events                                               |
|:----------------------------------------------------:|
| MonoBehaviour.Awake                                  |
| ↓                                                    |
| Reflex.Injectors.SceneInjector.Inject                |
| ↓                                                    |
| MonoBehaviour.Start                                  |

> `Reflex.Injectors.SceneInjector.Inject` injects fields, properties and methods decorated with [Inject] attribute.

## Contexts

### Project Context
A single prefab named `ProjectContext` that should live inside a `Resources` folder and should contain a `ProjectContext` component attached
> Non-Obligatory to have

### Scene Context
A single root gameobject per scene that should contain a `SceneContext` component attached
> Non-Obligatory to have

## Configuration Asset
Its a `ReflexConfiguration` scriptable object instance, named `ReflexConfiguration` that should live inside a `Resources` folder.  
It can be created by menu item Reflex → Configuration → Create Configuration.  
> Non-Obligatory to have but projects without it will fallback using following default configuration
### Default Configuration
- LogLevel: Default (Info, everything gets logged by default)

### Properties
#### LogLevel
Used by our internal logger so developers can define desired logging verbosity level.

## Bindings

### Bind Function
Binds a function to a type. Every time resolve is called to this binding, the function binded will be invoked.

### Bind Instance
Binds a object instance to a type. Every time resolve is called, this instance will be returned.
> Instances provided by the user, since not created by Reflex, will not be disposed automatically, we strongly recomend using BindSingleton.

### Bind Transient
Binds a factory. Every time the resolve is called, a new instance will be provided.
> Instances will be disposed once the container that provided the instances are disposed.

### Bind Singleton
Binds a factory. Every time the resolve is called, the same instance will be provided.
> The instance will be disposed once the container that provided the instance are disposed.

## Debugging Window
![Preview](Graphics/reflex-debugging-window.png)  
To access the debugging window on unity menu bar click `Reflex` → `Debugger`  
Through the debugging window you can visualize:
- Container hierarchy
- Bindings
- Resolution count

## Scripting Restrictions/IL2CPP Support
```csharp
public sealed class NumberManager
{
    public IEnumerable<int> Numbers { get; }

    public NumberManager(IEnumerable<int> numbers)
    {
        Numbers = numbers;
    }
        
    // In some extreme cases, you'll need to hint compiler so your code doesn't get stripped
    // https://docs.unity3d.com/Manual/ScriptingRestrictions.html
    [Preserve] private static void UsedOnlyForAOTCodeGeneration()
    {
        Array.Empty<object>().Cast<int>();
        throw new Exception("This method is used for AOT code generation only. Do not call it at runtime.");
    }
}
```

## Author
[![Twitter](https://img.shields.io/twitter/follow/codinggustavo.svg?label=Follow&style=social)](https://twitter.com/intent/follow?screen_name=codinggustavo)  

[![LinkedIn](https://img.shields.io/badge/Linkedin-%230077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/codinggustavo)  


## License

Reflex is licensed under the MIT license, so you can comfortably use it in commercial applications (We still love contributions though).
