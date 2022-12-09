# Item 8: Avoid finalizers and cleaners

## Why not to use `finalizer` or `cleaner`?

`Finalizers` or `cleaners`(Java9+) are unpredictable mechanisms for claiming memory as there is no guarantee of when they will run. This can lead to making incorrect assumptions as a programmer which will result in bugs.

There is no ensurance that a `finalizer` will be executed immediately or even when they will get executed. So anything time-critical shouldn't be done as part of `finalizers`. When a `finalizer` is executed depends upon the underlying garbage-collection algorithm which varies with implementation. Hence what works immediately with one GC algorithm might be delayed when the same code runs with another GC algorithm.

The specifications also don't provide any guarantee that the `finalizers` and `cleaners` will run at all. Hence any state-update such as releasing a lock shouldn't be added as part of a `finalizer` as it can lead to program getting stuck if the `finalizer` never ends up running.

Methods such as `System.gc` and `System.runFinalization` increase the chances of executing the `finalizers` but even they don't provide hundred percent guarantee that the `finalizer` will run.

An additional issue with using `finalizer` is that any uncaught exception that occurs during the `finalizer` is not propogated back to the program and this can lead to unpredictable system state as now you don't know the effect of the exception that was thrown.

`finalizer` are also slow in terms of performance as they depend upon the underlying garbage collection for resource cleanup. Classes implementing `finalizer` are also prone to `finalizer attack` where a sub-class can invoke methods on an incompletely instantiated super class. To avoid this for non-final class, add a `final` `finalize` method that is empty so that the subclass is unable to override the `finalize` method.

## What to use instead?
Just have your class implement the `AutoCloseable` interface and require the clients to invoke the `close` method.

`finalizer` or `cleaners` can be used as a safety net for a case when the client forgets to implement the `close` method of the interface. Even then it should be used only for non-critical resources due to the reasons discussed above.