# Item 4: Enforce noninstantiability with a private constructor

Some classes are meant to be non-instantiable. These can be classes containing `static`, `final` fields or static methods. Users of these classes can misuse the class by creating instances of them when it is not required.

Even though as class author, you might not provide a constructor explictly, compiler provides a `public` default constructor. Hence to avoid such misuse, it is important to explicitly prevent instantiation. 

This is done by adding a `private` constructor which cannot be invoked outside the class and in-turn prevents the users from creating instances of the class.

```
public class NonInstantiableClass {

  private NonInstantiableClass() {}
  
  public static void someMethod() {

  }
}
```

This will also prevent `NonInstantiableClass` from being subclassed as the subclass won't be able to find a `default` constructor.