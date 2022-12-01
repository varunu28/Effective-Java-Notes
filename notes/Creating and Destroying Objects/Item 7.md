# Item 7: Eliminate obsolete object references

Java is a language with in-built garbage collection where objects which are no longer are automatically reclaimed. But this doesn't means that you as a programmer doesn't needs to think about memory ever. 

A basic example where memory leaks can happen is when we are using collections of large size where each element in collection is a reference to another object. It also has a method to add elements into the collection.

```
public class MemoryLeak {
    private ExpensiveObject[] expensiveObjects;
    private int idx;

    public MemoryLeak() {
        this.expensiveObjects = new ExpensiveObject[/* initialCapacity= */ 4];
        this.idx = 0;
    }

    public void add(ExpensiveObject expensiveObject) {
        this.expensiveObjects[idx++] = expensiveObject;
    }
}
```

Now once we have added enough elements to the collection so that it reaches the `initialCapacity`, we will resize the collection to double it's size in order to accomodate more elements. So a collection of size 4 ends up becoming of size 8 and so on.
```
private void resize() {
    this.expensiveObjects = Arrays.copyOf(this.expensiveObjects, this.expensiveObjects.length * 2);
}
```
Now if we provide a functionality to remove the last element, one way of doing it can be to just update the idx.
```
public ExpensiveObject deleteLastEntry() {
    return this.expensiveObjects[idx--];
}
```
The problem with above approach is that the reference to `ExpensiveObject` is still present and will not be removed by garbage collector. And if the collection keeps on shrinking then we will end up with a large number of such dangling instances. Example a collection of size 32 might be shrinked to 2 with 30 dangling instances.

Also in above case, the `ExpensiveObject` itself can hold references to other objects which also end up becoming ineligible for garbage collection. The main problem here is that, testing for such scenarios is very hard. All the unit tests and integration tests for above class are going to pass and we are going to see the memory leak in our application when it is too late. Even then it is very difficult to debug about the code from where memory leak is generating.

One way to solve the above problem is to specifically `null` out the reference of obsolete elements.
```
public ExpensiveObject deleteLastEntry() {
    ExpensiveObject returnObject = this.expensiveObjects[idx];
    this.expensiveObjects[idx--] = null;
    return returnObject;
}
```
It also solves a logical error where now if we somehow reference the element at previous index, we will get a `null` whereas previously it was returning an obsolete object.

But this is not the only and not the most cleanest way to solve for memory leaks. If we start nulling out objects then our codebase will soon end up becoming messy. A better way is to define an object in as much limited scope as possible. By doing this, it will become eligible for garbage collection as soon as it's scope is reached.

In above example where class was managing it's own memory as it only has the information of what elements are in and out of scope, it becomes responsibility of programmer to handle memory.

Another common cases of memory leaks are cache and callback objects.