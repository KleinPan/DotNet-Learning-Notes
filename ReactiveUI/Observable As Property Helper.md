# Observable As Property Helper

## Example

First, we need to be able to declare an Output Property, using a class called ObservableAsPropertyHelper<T>:

首先，我们需要能够使用名为`ObservableAsPropertyHelper <T>`的类来声明输出属性：

```C#
    readonly ObservableAsPropertyHelper<string> firstName;
    public string FirstName => firstName.Value;
```

Similar to read-write properties, this code should always be 100% boilerplate. Next, we'll use a helper method ToProperty to initialize firstName in the constructor. ToProperty has two overloads, one where it directly returns the OAPH and another where it is returned in a out parameter:

与读写属性类似，此代码应始终为100％样板。 接下来，我们将使用辅助方法ToProperty来初始化构造函数中的firstName。 ToProperty有两个重载，一个直接返回OAPH，另一个返回out参数：

```C#
this.WhenAnyValue(x => x.Name)
    .Select(name => name.Split(' ')[0])
    .ToProperty(this, x => x.FirstName, out firstName);
```

or

```C#
firstName = this
    .WhenAnyValue(x => x.Name)
    .Select(name => name.Split(' ')[0])
    .ToProperty(this, x => x.FirstName);
```

Here, ToProperty creates an ObservableAsPropertyHelper instance which will signal that the "FirstName" property has changed. ToProperty is an extension method on IObservable<T> and semantically acts like a "Subscribe".

在这里，ToProperty创建一个ObservableAsPropertyHelper实例，该实例将发出“FirstName”属性已更改的信号。 ToProperty是IObservable <T>上的扩展方法，在语义上就像“订阅”。

## Performance considerations

### nameof() instead of using default Index

For performance based solutions you can also use the nameof() operator override of ToProperty() which won't use the Expression.

```C#
firstName = this
    .WhenAnyValue(x => x.Name)
    .Select(name => name.Split(' ')[0])
    .ToProperty(this, nameof(FirstName));
```
