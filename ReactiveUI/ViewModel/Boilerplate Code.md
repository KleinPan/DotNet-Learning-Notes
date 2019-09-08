# Boilerplate Code

If you are tired of writing boilerplate code for property change notifications, you can try either PropertyChanged.Fody or ReactiveUI.Fody. These libraries are both based on Fody - an extensible tool for weaving .NET assemblies, and they'll inject INotifyPropertyChanged code into properties at compile time for you. We recommend using ReactiveUI.Fody package that also handles ObservableAsProperyHelper properties.

如果您厌倦了为属性更改通知编写样板代码，可以尝试使用PropertyChanged.Fody或ReactiveUI.Fody。 这些库都基于Fody（一种用于编织.NET程序集的可扩展工具），它们会在编译时将INotifyPropertyChanged代码注入属性中。 我们建议使用也处理ObservableAsProperyHelper属性的ReactiveUI.Fody包。

## Read-write properties

Typically properties are declared like this:

```C#
private string name;

public string Name
{
    get { return name; }
    set { this.RaiseAndSetIfChanged(ref name, value); }
}
```

With ReactiveUI.Fody, you don't have to write boilerplate code for getters and setters of read-write properties - the package will do it automagically for you at compile time.

使用ReactiveUI.Fody，您不必为读写属性的getter和setter编写样板代码 - 程序包将在编译时自动为您执行。

```C#
[Reactive]
public string Name { get; set; }
```

All you have to do is annotate the property with the [Reactive] attribute.

## ObservableAsPropertyHelper properties

Similarly, to declare output properties, the code looks like this:

```C#
ObservableAsPropertyHelper<string> firstName;

public string FirstName
{
    get { return firstName.Value; }
}
```

Then the helper is initialized with a call to ToProperty:

```C#
...
.ToProperty(this, x => x.FirstName, out firstName);
```

With ReactiveUI.Fody, you can simply declare a read-only property using the `[ObservableAsProperty]` attribute:

`public string FullName { [ObservableAsProperty]get; }`

The field will be generated and the property implemented at compile time.

Because there is no field for you to pass to .ToProperty, you should use the .ToPropertyEx extension method provided by this library:

```C#
...
.ToPropertyEx(this, x => x.FullName);
```

This extension will assign the auto-generated field for you rather than relying on the out parameter.
