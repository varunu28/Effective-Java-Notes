# Item 79: Avoid excessive synchronization

Excessive synchronization can lead to:
 - Reduced performance
 - Deadlock
 - Non-deterministic behavior

Within a `synchronized` block of code, never invoke a method about which your existing code doesn't have any information about or rather a function which is form of an *alien method*. It can lead to liveness issues, exceptions or data corruption.

Aim to do as little work as possible in the syncrhonized block. This will ensure that threads that are waiting for the synchronized block don't end up being starved due to long running process.

For any mutable implementation, you can take two approaches:
 - Either push the responsibility of synchronization to the client and document it as not thread-safe.
 - Make the class thread-safe with internal synchronization.