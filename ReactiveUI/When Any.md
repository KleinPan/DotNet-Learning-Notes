# When Any

In interactive UI applications, state is continually changing in response to user actions and application events. ReactiveUI enables you to express changes to application state as streams of values and combine and manipulate them using the powerful Reactive Extensions library.

在交互式UI应用程序中，状态会不断变化以响应用户操作和应用程序事件。 ReactiveUI使您能够将应用程序状态的更改表示为值流，并使用功能强大的Reactive Extensions库来组合和操作它们。

The motivation is intuitive enough when you think about it. It's not hard to imagine that changes to a property can be considered events - that's how INotifyPropertyChanged works. From there, the same argument for using Rx over events applies. In the context of MVVM application design specifically, modelling property changes as observables leads to several advantages:

当你想到它时，动机足够直观。 不难想象，对属性的更改可以被视为事件 - 这就是`INotifyPropertyChanged`的工作原理。 从那里开始，相同论点也适用于在事件应用上使用Rx。 特别是在MVVM应用程序设计的上下文中，将属性更改建模为可观察对象会带来以下几个优点：

* The logic of an application can be defined in terms of changes to properties
* 可以根据属性的更改来定义应用程序的逻辑
* This logic can be composed and expressed declaratively, using the power of Rx operators
* 可以使用Rx运算符的功能以声明方式组合和表达该逻辑
* Concepts like time and asynchronicity become easier to reason about, due to their first-class treatment in an Observable context.
* 由于在可观察的上下文中对时间和异步性进行了一流的处理，这类概念变得更容易推理。

ReactiveUI provides several variants of WhenAny to help you work with properties as an observable stream.

## What is WhenAny

WhenAny is a set of extension methods starting the prefix WhenAny that allow you to get notifications when property on objects change.

when any是一组启动前缀when any的扩展方法，当对象上的属性发生更改时，这些方法允许您获得通知。

You can think of the WhenAny set of extension methods notifying you when one or many property values have changed.

当一个或多个属性值发生更改时，您可以考虑任何一组扩展方法通知您的时间。

WhenAny supports multiple property types including INotifyPropertyChanged, DependencyProperty and BindableProperty.

支持多种属性类型，包括`INotifyPropertyChanged`、`DependencyProperty`和`BindableProperty`。

It will check the property for support for each of those property types, and when you Subscribe() it will subscribe to the events offered by the applicable property notification mechanism.

它将检查属性是否支持这些属性类型，当您Subscribe()时，它将订阅适用的属性通知机制提供的事件。

WhenAny by default is just a wrapper around these property notification events, and won't store any values before a Subscribe. You can use techniques such as Publish, Replay() to get it to store these values.

默认情况下any只是这些属性通知事件的包装器，在订阅之前不会存储任何值。可以使用Publish、Replay()等技术使其存储这些值。

You can also wrap the WhenAny in a `Observable.Defer` to avoid the value being calculated until a Subscribe has happened. This is useful for `ObservableAsPropertyHelper` when you using the defer feature.

你也可以将WhenAny封装在一个可观察对象中。延迟以避免在发生订阅之前计算值。当您使用deferred特性时，这对于`ObservableAsPropertyHelper`非常有用。

## Basic syntax

The following examples demonstrate simple uses of `WhenAnyValue`, the `WhenAny` variant you are likely to use most frequently.

下面的示例演示了简单的WhenAnyValue用法，即您可能最常用的WhenAny变量。

### Watching single property

This returns an observable that yields the current value of Foo each time it changes:
这将返回一个可观察到的值，该值在Foo每次发生变化时都会产生当前值:

`this.WhenAnyValue(x => x.Foo)`

### Watching a number of properties

This returns an observable that yields a new Color with the latest RGB values each time any of the properties change. The final parameter is a selector describing how to combine the three observed properties:
这将返回一个可观察值，该值将在每次属性更改时生成带有最新RGB值的新颜色。最后一个参数是一个选择器，描述了如何组合这三个观察到的属性:

```C#
this.WhenAnyValue(x => x.Red, x => x.Green, x => x.Blue,
                  (r, g, b) => new Color(r, g, b));
```

### Watching a nested property

WhenAny variants can observe nested properties for changes, too:

`this.WhenAnyValue(x => x.Foo.Bar.Baz);`

## Idiomatic usage `习惯用法`

Naturally, once you have an observable of property changes you can Subscribe to it in order to perform actions in response to the changed values. However, in many cases there may be a Better Way to achieve what you want. Below are some typical usages of the observables returned by the WhenAny variants:
自然地，一旦您拥有一个可观察到的属性更改，您就可以订阅它，以便对更改的值执行响应操作。然而，在很多情况下，可能有更好的方法来实现你想要的。下面是WhenAny变异体返回的可观测值的一些典型用法:

### Exposing 'calculated' properties

In general, using `Subscribe` on a `WhenAny` observable (or any observable, for that matter) just to set a property is likely a code smell. Idiomatically, the ToProperty operator is used to create a 'read-only' calculated property that can be exposed to the rest of your application, only settable by the WhenAny chain that preceded it:
通常，使用Subscribe on a WhenAny observable(或任何可观察到的)来设置属性很可能是一种代码气味。通常情况下，ToProperty操作符用于创建一个“只读”的计算属性，该属性可以公开给应用程序的其他部分，只有当它之前的任何链:

```C#
this.WhenAnyValue(x => x.SearchText, x => x.Length)
    .ToProperty(this, x => x.SearchTextLength, out _searchTextLength);
```

This initialises the SearchTextLength property (an ObservableAsPropertyHelper property) as a property that will be updated with the current search text length every time it changes. The property cannot be set in any other manner and raises change notifications, so can itself be used in a WhenAny expression or a binding.
这会将SearchTextLength属性（ObservableAsPropertyHelper属性）初始化为一个属性，每次更改时都会使用当前搜索文本长度进行更新。 该属性不能以任何其他方式设置并引发更改通知，因此本身可以在WhenAny表达式或绑定中使用。

See the ObservableAsPropertyHelper section for more information on this pattern.

### Supporting validation as a CanExecute criteria `支持将验证作为一个可以执行的标准`

## Variants of WhenAny `变异的WhenAny`

Several variants of WhenAny exist, suited for different scenarios.
WhenAny存在的几个变体，适用于不同的场景。

### WhenAny vs WhenAnyValue

WhenAnyValue covers the most common usage of WhenAny, and is a useful shortcut in many cases. The following two statements are equivalent and return an observable that yields the updated value of SearchText on every change:
WhenAnyValue涵盖了WhenAny的最常见用法，在许多情况下，它是一种有用的快捷方式。下面两个语句是等价的，并返回一个可观察的结果，该结果在每次更改时生成SearchText的更新值:

`this.WhenAny(x => x.SearchText, x => x.Value)`
`this.WhenAnyValue(x => x.SearchText)`

When needing to observe one or many properties for changes, `WhenAnyValue` is quick to type and results in simpler looking code. Working with `WhenAny` directly gives you access to the `ObservedChange<,>` object that ReactiveUI produces on each property change. This is typically useful for framework code or extension methods. `ObservedChange` exposes the following properties:
当需要观察一个或多个属性的更改时，WhenAnyValue可以快速键入并生成更简单的代码。使用WhenAny直接让您访问ObservedChange<，>对象，该对象是ReactiveUI对每个属性更改生成的。这对于框架代码或扩展方法非常有用。ObservedChange公开了以下属性:

* `Value` - the updated value
* `Sender` - the object whose has property changed
* `Expression` - the expression that changed.

At the risk of extreme repetition - use `WhenAnyValue` unless you know you need `WhenAny`.
使用WhenAnyValue除非你知道你需要使用WhenAny.

### WhenAnyObservable
WhenAnyObservable acts a lot like the Rx operator CombineLatest, in that it watches one or multiple observables and allows you to define a projection based on the latest value from each. WhenAnyObservable differs from CombineLatest in that its parameters are expressions, rather than direct references to the target observables. The impact of this difference is that the watch set up by WhenAnyObservable is not tied to the specific observable instances present at the time of subscription. That is, the observable pointed to by the expression can be replaced later, and the results of the new observable will still be captured.
当任何可观察对象的行为非常类似于Rx操作符CombineLatest时，它会监视一个或多个可观察对象，并允许您根据每个对象的最新值定义投影。当任何可观测对象与CombineLatest的不同之处在于，它的参数是表达式，而不是对目标可观测对象的直接引用。这种差异的影响是，WhenAnyObservable设置的手表不与订阅时出现的特定observable实例绑定。也就是说，表达式所指向的可观测对象可以在以后替换，新的可观测对象的结果仍然会被捕获。

An example of where this can come in handy is when a view wants to observe an observable on a viewmodel, but the viewmodel can be replaced during the view's lifetime. Rather than needing to resubscribe to the target observable after every change of viewmodel, you can use WhenAnyObservable to specify the 'path' to watch. This allows you to use a single subscription in the view, regardless of the life of the target viewmodel.
当一个视图想要在一个视图模型上观察一个可观察对象，但是视图模型可以在视图的生命周期内被替换时，这个功能就派上了用场。您可以使用WhenAnyObservable来指定要监视的“路径”，而不是在每次更改viewmodel之后都需要重新订阅目标observable。这允许您在视图中使用单个订阅，而不管目标视图模型的生命周期如何。

## Additional Considerations

Using `WhenAny` variants is fairly straightforward. However, there are a few aspects of their behaviour that are worth highlighting.
使用WhenAny非常简单。然而，他们的行为有几个方面值得强调。

### INotifyPropertyChanged is required

Watched properties must implement ReactiveUI's `RaiseAndSetIfChanged` or raise the standard `INotifyPropertyChanged` events. If you attempt to use `WhenAny` on a property without either of these in place, `WhenAny` will produce the current value of the property upon subscription, and nothing thereafter. Additionally, a warning will be issued at run time (ensure you have registered a service for `ILogger` to see this).
被监视的属性必须实现ReactiveUI的RaiseAndSetIfChanged或提高标准INotifyPropertyChanged事件。如果您试图在没有上述任何一种方法的情况下对属性使用WhenAny，那么WhenAny将在订阅时生成属性的当前值，之后什么也不生成。此外，将在运行时发出警告(确保您已经为ILogger注册了一个服务来查看此信息)。

### WhenAny only notifies on change of the output value


### null propogation inside WhenAnyValue

WhenAnyValue is not directly able to perform null propogation due to the fact Expression's don't support this feature yet.
WhenAnyValue不能直接执行空传播，因为事实表达式还不支持该特性。

You can simulate null propogation support by chaining your WhenAnyValue() call to each property along the way. Here's an example:

```C#
this.WhenAnyValue(x => x.Foo, x => x.Foo.Bar, x => x.Foo.Bar.Baz, (foo, bar, baz) => foo?.Bar?.Baz)
    .Subscribe(x => Console.WriteLine(x));
```
