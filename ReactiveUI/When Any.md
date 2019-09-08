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

You can also wrap the WhenAny in a Observable.Defer to avoid the value being calculated until a Subscribe has happened. This is useful for ObservableAsPropertyHelper when you using the defer feature.

你也可以将WhenAny封装在一个可观察对象中。延迟以避免在发生订阅之前计算值。当您使用deferred特性时，这对于ObservableAsPropertyHelper非常有用。
