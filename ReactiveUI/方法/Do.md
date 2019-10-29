Invokes an action for each element in the observable sequence, and propagates  all observer messages through the result sequence. This method can be used for  debugging, logging, etc. of query behavior by intercepting the message stream  to run arbitrary actions for messages on the pipeline.

为可观察序列中的每个元素调用一个操作，并通过结果序列传播所有观察者消息。该方法可以通过拦截消息流来调试、记录查询行为，从而在管道上为消息运行任意操作。


Observable.Do<TSource> Method (IObservable<TSource>, Action<TSource>, Action<Exception>, Action)

Invokes an action for each element in the observable sequence, and invokes an action upon graceful or exceptional termination of the observable sequence.
为可观察序列中的每个元素调用一个动作，并在可观察序列的正常或异常终止时调用一个动作。