# Item 6: Avoid creating unnecessary objects

There are use cases where reusing an object instead of creating a new object is appropriate. **An object is always reusable if it is immutable.**

So instead of doing
```
String s = new String("just a string");
```
do
```
String s = "just a string";
```
When we create a `String` using a constructor, we end up creating a new instance of `String` class. Whereas creating the `String` by instantiation, creates a single copy of class which can be reused as it is immutable.

`static` factory methods can be used to avoid creating unnecessary objects of a class as now the control for object creation is abstracted away and is not within reach of the user. This is also useful when the object creation is expensive such as a database connection and it is preferrable to cache such objects instead of creating a new instance everytime.

Another way in which we end up creating unneccessary objects is `auto-boxing` i.e. when we mix `primitive` types such as `int` and `boxed primitives` such as `Integer`. If we are not careful, we can seriosly degrade the performance of our code. Consider the following:
```
Integer sum = 0;
for (int i = 0; i < 100; i++) {
    sum += i;
}
```
In the above code we end up creating 100 instances of `Integer` class as sum is declared as `Integer` instead of `int`. Hence, **prefer primitives to boxed primitives and always be on the lookout for unintentional autoboxing.**