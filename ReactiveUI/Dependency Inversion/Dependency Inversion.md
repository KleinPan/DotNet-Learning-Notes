# Dependency Inversion

## Dependency Injection

Dependency resolution is very useful for moving logic that would normally have to be in platform-specific code, into the shared platform code. First, we need to define an Interface for something that we want to use - this example isn't a Best Practice, but it's illustrative.

依赖项解析对于将逻辑(通常必须在特定于平台的代码中)转移到共享平台代码中非常有用。首先，我们需要为我们想要使用的东西定义一个接口——这个例子不是一个最佳实践，但它是说明性的。

Dependency resolution is a feature built into the core framework, which allows libraries and ReactiveUI itself to use classes that are in other libraries without taking a direct reference to them. This is quite useful for cross-platform applications, as it allows portable code to use non-portable APIs, as long as they can be described via an Interface.

依赖项解析是构建在核心框架中的一个特性，它允许库和ReactiveUI本身使用其他库中的类，而不直接引用它们。
这对于跨平台应用程序非常有用，因为它允许可移植代码使用非可移植api，只要它们可以通过接口进行描述。

ReactiveUI's use of dependency resolution can more properly be called the Service Location pattern. If you service location doesn't fit your situation then have a look at how to do composition root.

ReactiveUI对依赖项解析的使用可以更恰当地称为服务位置模式。
如果您的服务位置不适合您的情况，那么看看如何进行组合根。

Since ReactiveUI 6, Splat is used by ReactiveUI for service location and dependency injection. Earlier versions included a RxUI resolver. If you come across samples for RxUI versions earlier than 6, you should replace references to RxApp.DependencyResolver with Locator.Current and RxApp.MutableResolver with Locator.CurrentMutable.

从ReactiveUI 6开始，ReactiveUI使用Splat进行服务定位和依赖注入。 早期版本包括一个RxUI解析器。 如果您遇到6之前的RxUI版本的示例，则应使用Locator.Current替换对RxApp.DependencyResolver的引用，并使用Locator.CurrentMutable替换对RxApp.MutableResolver的引用。

### Why Splat ?

"I generally think a lot of the advice around IoC/DI is pretty bad in the domain of 'cross-platform mobile applications', because you have to remember that a lot of their ideas were written for web apps, not mobile or desktop apps.

“我通常认为，在‘跨平台移动应用程序’领域，很多关于IoC/DI的建议都非常糟糕，因为你必须记住，他们的很多想法都是针对web应用程序而写的，而不是针对移动或桌面应用程序。

For example, the vast majority of popular IoC containers concern themselves solely with resolution speed on a warm cache, while basically completely disregarding memory usage or startup time - this is 100% fine for server applications, because these things don't matter; but for a mobile app? Startup time is huge.

例如，绝大多数流行的IoC容器只关心热缓存上的解析速度，而基本上完全不考虑内存使用或启动时间——这对于服务器应用程序来说是100%正确的，因为这些事情并不重要;但对于移动应用程序呢?启动时间很长。

Splat's Service Location solves a number of issues for RxUI:

Splat的服务位置解决了RxUI的许多问题：

Service Location is fast, and has almost no overhead to set up. It encapsulates several different common object lifetime models (i.e. 'create new every time', 'singleton', 'lazy'), just by writing the Func differently It's Mono Linker friendly (generally) Service Location allows us to register types in platform-specific code, but use them in PCL code."

服务定位速度快，而且几乎没有需要设置的开销。它封装了几个不同的公共对象生命周期模型(即'每次创建新'，'单例'，'懒惰')，只是通过编写不同的函数，它是Mono链接友好(通常)的服务位置允许我们注册类型在特定于平台的代码，但使用它们在PCL代码。

Anaïs Betts @ http://stackoverflow.com/a/26924067/496857

