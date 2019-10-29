# Examples

In the following example, we convert a cold observable sequence source to a hot one using the Publish operator, which returns an IConnectableObservable<T> instance we name hot. The Publish operator provides a mechanism to share subscriptions by broadcasting a single subscription to multiple subscribers. hot acts as a proxy and subscribes to source, then as it receives values from source, pushes them to its own subscribers. To establish a subscription to the backing source and start receiving values, we use the IConnectableObservable.Connect() method. Since IConnectableObservable inherits IObservable, we can use Subscribe to subscribe to this hot sequence even before it starts running. Notice that in the example, the hot sequence has not been started when subscription1 subscribes to it. Therefore, no value is pushed to the subscriber. After calling Connect, values are then pushed to subscription1. After a delay of 3 seconds, subscription2 subscribes to hot and starts receiving the values immediately from the current position (3 in this case) until the end. The output looks like this:

在下面的示例中，我们使用Publish操作符将一个冷的可观察序列源转换为一个热的，该操作符返回一个IConnectableObservable<T>实例，我们将其命名为hot。Publish操作符通过向多个订阅者广播单个订阅来提供共享订阅的机制。hot充当代理并订阅源，然后当它从源接收值时，将它们推送给自己的订阅者。要建立对支持源的订阅并开始接收值，我们使用IConnectableObservable.Connect()方法。因为IConnectableObservable继承了IObservable，所以我们可以使用Subscribe来订阅这个热序列，甚至在它开始运行之前。请注意，在本例中，subscription1订阅热序列时没有启动它。因此，不会向订阅服务器推送任何值。调用Connect之后，值被推送到subscription1。延迟3秒后，subscription2订阅hot，并立即从当前位置(本例中为3)开始接收值，直到结束。输出是这样的

Current Time: 6/1/2011 3:38:49 PM

Current Time after 1st subscription: 6/1/2011 3:38:49 PM

Current Time after Connect: 6/1/2011 3:38:52 PM

Observer 1: OnNext: 0

Observer 1: OnNext: 1

Current Time just before 2nd subscription: 6/1/2011 3:38:55 PM 

Observer 1: OnNext: 2

Observer 1: OnNext: 3

Observer 2: OnNext: 3

Observer 1: OnNext: 4

Observer 2: OnNext: 4

```C#
Console.WriteLine("Current Time: " + DateTime.Now);
var source = Observable.Interval(TimeSpan.FromSeconds(1));   //creates a sequence

IConnectableObservable<long> hot = Observable.Publish<long>(source);  // convert the sequence into a hot sequence

IDisposable subscription1 = hot.Subscribe(     // no value is pushed to 1st subscription at this point
                            x => Console.WriteLine("Observer 1: OnNext: {0}", x),
                            ex => Console.WriteLine("Observer 1: OnError: {0}", ex.Message),
                            () => Console.WriteLine("Observer 1: OnCompleted"));
Console.WriteLine("Current Time after 1st subscription: " + DateTime.Now);
Thread.Sleep(3000);  //idle for 3 seconds
hot.Connect();       // hot is connected to source and starts pushing value to subscribers 
Console.WriteLine("Current Time after Connect: " + DateTime.Now);
Thread.Sleep(3000);  //idle for 3 seconds
Console.WriteLine("Current Time just before 2nd subscription: " + DateTime.Now);
IDisposable subscription2 = hot.Subscribe(     // value will immediately be pushed to 2nd subscription
                            x => Console.WriteLine("Observer 2: OnNext: {0}", x),
                            ex => Console.WriteLine("Observer 2: OnError: {0}", ex.Message),
                            () => Console.WriteLine("Observer 2: OnCompleted"));
Console.ReadKey();
```
