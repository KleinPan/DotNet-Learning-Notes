# Collections

ReactiveUI recommends the use of DynamicData framework for collection based operations. DynamicData has replaced internally the use of ReactiveList

ReactiveUI建议将DynamicData框架用于基于集合的操作。 DynamicData已在内部替换了ReactiveList的使用

## Overview of Dynamic Data

Dynamic Data is reactive collections based on Reactive Extensions for .NET.
动态数据是基于Reactive Extensions for .NET的反应式集合。

Whenever a change is made to one of Dynamic Data's collections a notification is produced. A notification reflects what has changed in the collection. This notification is represented as a ChangeSet which can contain one or more changes. Each item in the change set is represented as a Change which contains information about each individual change since the last notification.
每当对Dynamic Data的一个集合进行更改时，都会生成通知。 通知反映了集合中发生的变化。 此通知表示为ChangeSet，可以包含一个或多个更改。 更改集中的每个项目都表示为更改，其中包含自上次通知以来每个更改的信息。

The changes sets are published as an IObservable<ChangeSet>.
更改集将作为IObservable <ChangeSet>发布。

This basic signature is the monad of Dynamic Data on which a rich set of Linq operators are provided which enable declarative querying and manipulation of data as it changes, and in a thread safe manner.
这个基本签名是动态数据的monad，在其上提供了丰富的Linq运算符集，它们可以在数据发生变化时以线程安全的方式进行声明性查询和操作。

### Maintaining and consuming data

Dynamic Data provides two specialized IObservable<ChangeSet> producing collections:
动态数据提供两个专门的IObservable <ChangeSet>生成集合：

1. `SourceCache<TObject, TKey>` <font color=red>for items which have a unique key.
2. `SourceList<T>` for items which do not have a unique key.</font>

These objects each provide an API for maintaining data which have typical collection methods such as add and remove. The idea is you maintain data in one of these collections, then use the extensive Linq API to dynamicaly query the data in a similar manner as Linq-to-Objects.
这些对象均提供用于维护数据的API，这些数据具有典型的收集方法，例如添加和删除。 我们的想法是在其中一个集合中维护数据，然后使用广泛的Linq API以与Linq-to-Objects类似的方式动态查询数据。

To convert these collections into an IObservable<ChangeSet> you call Connect() at which point notifications can be observed and the provided Linq operators can be applied. The convention in dynamic data is that any consumer which calls Connect receive a notification of any items which are already in the collection plus any subsequent changes.
要将这些集合转换为`IObservable<ChangeSet>`，您可以调用`Connect()`，在这个时候可以观察到通知，并且可以应用所提供的Linq操作符。动态数据中的约定是，调用Connect的任何使用者都将收到一个通知，通知中包含已经在集合中的任何项以及任何后续更改。

Additionally there are several other means of creating observable changes sets from existing collections which implement INotifyCollectionChanged and IEnumerable<T>.
此外，还有一些其他方法可以从现有集合创建可观察的更改集，这些集合实现了INotifyCollectionChanged和IEnumerable <T>。

### What it is not

Dynamic data collections are not an alternative implementation to ObservableCollection<T>. The architecture of it has been based first and foremost on domain driven concepts. The idea is you load and maintain your data in one of the provided collections which can then use operators to manipulate the data without the complexity of managing collections. It can be used to react to your collections however you want, be it binding to a screen or producing some other kind of notification. The collections can be connected to as many times as required and a single collection can in turn become the source of many other derived collections.
动态数据集合不是ObservableCollection <T>的替代实现。 它的体系结构首先基于域驱动的概念。 我们的想法是在一个提供的集合中加载和维护您的数据，然后可以使用运算符来操作数据，而无需管理集合的复杂性。 它可以用来对你想要的集合作出反应，无论是绑定到屏幕还是产生其他类型的通知。 集合可以根据需要连接多次，单个集合又可以成为许多其他派生集合的源。

### How to use it

If you are already using ObservableCollection<T> the easiest and quickest way to try out dynamic data is to use the extension .ToObservableChangeSet() which produces an observable change set and subsequently enables Dynamic Data operators.
如果您已经在使用`ObservableCollection <T>`，那么尝试动态数据的最简单，最快捷的方法是使用扩展名`.ToObservableChangeSet()`，它生成一个可观察的变更集，然后启用动态数据运算符。

For example if you have an existing reactive list `ObservableCollection<T> myList` you can do something like this:

```C#
var myDerivedList = myList
    .ToObservableChangeSet()
    .Filter(t => t.Status == "Something")
    .AsObservableList();
```

And voila you have create a filtered observable list. Or if you specify a key

```C#
var myDerivedCache = myList
    .ToObservableChangeSet(t => t.Id)
    .Filter(t => t.Status == "Something")
    .AsObservableCache();
```

you have a derived observable cache.
这样就创建了一个过滤后的可观察列表。或者如果你指定一个键，你就有一个派生的可观察缓存。

A caveat to this approach is if you are using myList will likely not be thread safe. Assuming myList is bound to a screen, then the observable change set is created and notified on the UI thread which I avoid for all operations except binding. The other approach is to create a data source first and bind later.

这种方法的一个警告是，如果您正在使用myList，那么它可能不是线程安全的。假设myList绑定到一个屏幕，然后在UI线程上创建并通知可观察的更改集，除了绑定之外，我对所有操作都避免这样做。另一种方法是先创建数据源，然后绑定。

```C#
var myList = new SourceList<T>()
var myConnection = myList
    .Connect() // make the source an observable change set
    .\\some other operation
```

or similarly for the observable cache

```C#
var myCache = new SourceCache<T, int>(t => t.Id)
var myConnection = myCache
    .Connect() // make the source an observable change set
    .\\some other operation
```

The advantage of creating your own data sources is that they can be maintained on a background thread which frees up valuable main thread time. Then should there be a need, bind as follows:
创建您自己的数据源的好处是，它们可以在后台线程上维护，从而节省了宝贵的主线程时间。如果有需要，绑定如下:

```C#
ReadOnlyObservableCollection<T> bindingData;
var myBindingOperation = mySource
    .Sort(SortExpressonComparer<T>.Ascending(t => t.DateTime))
    .ObserveOn(RxApp.MainThreadScheduler) // Make sure this is only right before the Bind()
    .Bind(out bindingData)
    .Subscribe();
```

The API for the above is the same for cache and list.

### So what's the difference between a SourceList and a SourceCache

If you have a unique id, you should use an observable cache as it is dictionary based which will ensure no duplicates can be added and it notifies on adds, updates and removes, whereas list allows duplicates and only has no concept of an update. SourceCache has several performance advantages over SourceList, so if possible, always prefer SourceCache over SourceList.
如果你有一个唯一的id，你应该使用一个可观察的缓存，因为它是基于字典的，这将确保没有重复可以被添加，它通知添加，更新和删除，而列表允许重复，只有没有更新的概念。与`SourceList`相比，`SourceCache`有几个性能优势，因此，如果可能的话，总是首选`SourceCache`而不是`SourceList`。

There is another difference. The cache side of dynamic data is much more mature and has a wider range of operators. Having more operators is mainly because I found it easier to achieve good all round performance with the key based operators and do not want to add anything to Dynamic Data which inherently has poor performance.
还有一个不同之处。动态数据的缓存端要成熟得多，操作符的范围也更广。拥有更多的操作符主要是因为我发现使用基于键的操作符更容易实现良好的全面性能，并且不希望向动态数据添加任何性能本来就很差的内容。

## Using DynamicData with ReactiveUI

When building applications with ReactiveUI and DynamicData, you have a choice to work with mutable or with immutable collections. When working with immutable ones, using an ObservableAsPropertyHelper<T> is enough in simple cases. The ObservableAsPropertyHelper<T> represents an Observable<T>, a stream of values over time. You can treat those values as events, and the new values as event arguments. This means if you are using immutable collections, you can treat them as event arguments and update a property with a new collection each time it changes. See ObservableAsPropertyHelper Handbook section to learn how to use this feature. Note, that creating a new colletion for each update degrades performance and should be generally avoided, prefer to use DynamicData instead.
当使用ReactiveUI和DynamicData构建应用程序时，您可以选择使用可变集合或不可变集合。当处理不可变的对象时，在简单的情况下使用`ObservableAsPropertyHelper<T>`就足够了。`ObservableAsPropertyHelper<T>`表示一个可观察的`<T>`，一个随时间变化的值流。您可以将这些值视为事件，并将新值视为事件参数。这意味着，如果使用不可变集合，可以将它们视为事件参数，并在属性每次更改时使用新集合更新属性。请参阅ObservableAsPropertyHelper手册部分了解如何使用该特性。注意，为每次更新创建一个新的集合会降低性能，通常应该避免使用动态数据。

### An Example

Imagine your application needs a service that will expose a collection mutated by a background worker. You need to get change notifications from it somehow to synchronize it with the user interface. Here DynamicData comes to the rescue. You expose an IObservable<IChangeSet<bool>> from your service to the outer world, and DynamicData takes care of allowing you to observe changes of your mutable SourceList of items. Use the .Connect() operator to turn your SourceList<T> to an observable change set IObservable<IChangeSet<bool>>.
假设您的应用程序需要一个服务，该服务将公开由后台工作人员更改的集合。您需要从它获取更改通知，以便与用户界面同步。在这里，动态数据发挥了作用。您将一个`IObservable<IChangeSet<bool>>`从您的服务公开给外部世界，DynamicData允许您观察您的可变项源列表的变化。使用`.connect()`操作符将源列表`<T>`转换为一个可观察的更改集`IObservable<IChangeSet<bool>>`。

```C#
public class Service
{
    private readonly SourceList<bool> _items = new SourceList<bool>();

    // We expose the Connect() since we are interested in a stream of changes.
    // If we have more than one subscriber, and the subscribers are known,
    // it is recommended you look into the Reactive Extension method Publish().
    public IObservable<IChangeSet<bool>> Connect() => _items.Connect();

    public Service()
    {
        // With DynamicData you can easily manage mutable datasets,
        // even if they are extremely large. In this complex scenario
        // a service mutates the collection, by using .Add(), .Remove(),
        // .Clear(), .Insert(), etc. DynamicData takes care of
        // allowing you to observe all of those changes.
        //使用DynamicData，您可以轻松地管理可变数据集，即使它们非常大。在这个复杂的场景中服务使用. add()、. remove()、. clear()、. insert()等来修改集合。DynamicData允许您观察所有这些更改。
        _items.Add(true);
        _items.RemoveAt(0);
        _items.Add(false);
    }
}
```

DynamicData uses .NET types to expose to the outside world, such as ReadOnlyObservableCollection<T>, rather than exposing their own types. IObservable<IChangeSet<T>> (and IObservable<IChangeSet<TObject, TKey>>) are the two base observables you can create derived based functionality from. IObservable<IChangeSet<T>> indicates what has changed to a collection. The first time you use ToObservableChangeSet() it emits the current state of the collection.
DynamicData使用. net类型向外部世界公开，例如`ReadOnlyObservableCollection<T>`，而不是公开它们自己的类型。`IObservable<IChangeSet<T>>`(和`IObservable<IChangeSet<TObject, TKey>>`)是您可以创建基于功能的派生的两个基本可观察对象。`IObservable<IChangeSet<T>>`表示集合更改了什么。第一次使用`ToObservableChangeSet()`时，它会发出集合的当前状态。

SourceList, SourceCache are multithreaded aware and optimised to create IObservable<IChangeSet<T>> and IObservable<IChangeSet<TObject, TKey>>. Generally SourceList/SourceCache are meant to be private to your classes, and you expose using the Bind() method. You generate the change sets by using the Connect() method on them.
`SourceList`, `SourceCache`是多线程感知和优化，以创建`IObservable<IChangeSet<T>>`和`IObservable<IChangeSet<TObject, TKey>>`。通常`SourceList/SourceCache`对类来说是私有的，您可以使用`Bind()`方法进行公开。您可以通过对它们使用`Connect()`方法生成更改集。

Using the powerful DynamicData operators, you convert the IObservable<IChangeSet<T>> to a ReadOnlyObservableCollection<T> to which you can easily bind the platform-specific user interface. Declaring the read-only collection as a field or as a variable is required for the .Bind() operator to work as it uses out variables.
使用强大的动态数据操作符，您可以将`IObservable<IChangeSet<T>>`转换为`ReadOnlyObservableCollection<T>`，您可以轻松地将平台特定的用户界面绑定到它。`.bind()`操作符在使用完变量时，需要将只读集合声明为字段或变量。

```C#
public class ViewModel : ReactiveObject
{
    private readonly ReadOnlyObservableCollection<bool> _items;
    public ReadOnlyObservableCollection<bool> Items => _items;

    public ViewModel()
    {
        var service = new Service();
        service.Connect()
            // Transform in DynamicData works like Select in
            // LINQ, it observes changes in one collection, and
            // projects it's elements to another collection.
            .Transform(x => !x)
            // Filter is basically the same as .Where() operator
            // from LINQ. See all operators in DynamicData docs.
            .Filter(x => x)
            // Ensure the updates arrive on the UI thread.
            .ObserveOn(RxApp.MainThreadScheduler)
            // We .Bind() and now our mutable Items collection
            // contains the new items and the GUI gets refreshed.
            .Bind(out _items)
            .Subscribe();
    }
}
```

Note
If you are updating an observable list or an observable cache from a background thread, adding .ObserveOn(RxApp.MainThreadScheduler) right before a call to .Bind() might be neccessary, to ensure the updates arrive on the UI thread.
如果要从后台线程更新可观察列表或可观察缓存，可能需要在调用`.Bind()`之前添加`.ObserveOn(RxApp.MainThreadScheduler)`，以确保更新到达UI线程。

ObservableCollectionExtended<T> is a good single threaded collection where you don't need to do derived based functionality. To synchronize two collections in your view model, declare one of your collections as ObservableCollectionExtended<T> and another one as ReadOnlyObservableCollection<T>. Then you apply the .ToObservableChangeSet() operator to your observable collection that turns it to IObservable<IChangeSet<T>>.
`ObservableCollectionExtended<T>`是一个很好的单线程集合，您不需要执行基于派生的功能。要同步视图模型中的两个集合，可以将其中一个集合声明为`ObservableCollectionExtended<T>`，将另一个集合声明为`ReadOnlyObservableCollection<T>`。然后将`.ToObservableChangeSet()`操作符应用于您的可观察集合，该集合将其转换为`IObservable<IChangeSet<T>>`。

```C#
public class SynchronizedCollectionsViewModel : ReactiveObject
{
    private readonly ReadOnlyObservableCollection<bool> _derived;
    public ReadOnlyObservableCollection<bool> Derived => _derived;

    public ObservableCollectionExtended<bool> Source { get; }

    public SynchronizedCollectionsViewModel()
    {
        Source = new ObservableCollectionExtended<bool>();

        // Use the ToObservableChangeSet operator to convert
        // the observable collection to IObservable<IChangeSet<T>>
        // which describes the changes. Then, use any DD operators
        // to transform the collection.
        Source.ToObservableChangeSet()
            .Transform(value => !value)
            // No need to use the .ObserveOn() operator here, as
            // ObservableCollectionExtended is single-threaded.
            .Bind(out _derived)
            .Subscribe();
        // Update the source collection and the derived
        // collection will update as well.
        Source.Add(true);
        Source.RemoveAt(0);
        Source.Add(false);
        Source.Add(true);
    }
}
```

## Tracking Changes in Collections of Reactive Objects

DynamicData supports change tracking for classes that implement the INotifyPropertyChanged interface — ReactiveObjects. For example, if you'd like to do a WhenAnyValue on each element in a collection of changing objects, use the AutoRefresh() DynamicData operator:
DynamicData支持对实现`INotifyPropertyChanged`接口- `ReactiveObjects`的类进行更改跟踪。例如，如果您想对一个不断变化的对象集合中的每个元素执行一个`WhenAnyValue`，请使用`AutoRefresh()` DynamicData操作符:

```C#
var databasesValid = collectionOfReactiveObjects
    .ToObservableChangeSet()
    .AutoRefresh(model => model.IsValid) // Subscribe only to IsValid property changes
    .ToCollection()                      // Get the new collection of items
    .Select(x => x.All(y => y.IsValid)); // Verify all elements satisfy a condition etc.

// Then you can convert that IObservable<bool> to a view model
// property declared as ObservableAsPropertyHelper<bool>.
_databasesValid = databasesValid.ToProperty(this, x => x.DatabasesValid);
```

Note
ToCollection() works pretty differently internally, it re-generates the entire list every time while SourceCache/SourceList Bind() does addition/removals etc. ToCollection() is only meant for aggregation based on operations where you really need a full collection each time as an observable.
`ToCollection()`的内部工作方式非常不同，每次当SourceCache/SourceList `Bind()`执行添加/删除等操作时，它都会重新生成整个列表。`ToCollection()`仅用于基于每次都需要作为可观察对象的完整集合的操作进行聚合。

## Converting ReactiveList to DynamicData 待完善


