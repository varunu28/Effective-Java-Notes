# Item 3: Enforce the singleton property with a private constructor or enum type

A singleton is a class that is instantiated only once. Though making a class singleton makes it difficult for client to test as it cannot be replaced with a mock implementation unless the singleton itself implements an interface.

Let's see two approaches using which we can implement a singleton

## Member instance field is final and public
In this approach the instance field is `public`, `final` and `static`. The constructor is kept private and is invoked only once when the instance field is created.
```
public class CustomParser {
  
  public static final CustomParser INSTANCE = new CustomParser();
  
  private CustomParser() {
    System.out.println("Instantiating parser");
    // Instantiation logic
  }
  
  public void parse() {
    // parsing logic
  }
}
```
Now if we try to access the parser twice as demo,
```
System.out.println(CustomParser.INSTANCE);
System.out.println(CustomParser.INSTANCE);
```
We will notice that, the constructor is invoked only once by seeing the print statement which is part of constructor only once. Output of above demo will be as following:
```
Instantiating parser
effectivejava.singleton.CustomParser@4617c264
effectivejava.singleton.CustomParser@4617c264
```
Using a public field gives an indication to the client that the class is a singleton and also makes it easier to use as we are just accessing a class parameter.

## Using a static factory method
In this approach we keep the instance as a `private` field and use a `static` factory method to return the instance. The above described class implementation gets updated as follows:
```
public class CustomParser {

  private static final CustomParser INSTANCE = new CustomParser();

  private CustomParser() {
    System.out.println("Instantiating parser");
    // Instantiation logic
  }

  public static CustomParser getInstance() {
    return INSTANCE;
  }
}
```
Advantage of using `static` factory method is that it gives you the flexibility of removing singleton functionality in future without affecting the client. Another advantage is that the static method can be used as a supplier like `CustomParser::getInstance`.

One caveat to singleton classes is in regards of serializing and deserializing. If we just implement the `Serializable` interface, new instances will be created during each deserialization. To prevent this, we have to add a `readResolve` method to our singleton class that return the instance. This method is used for replacing the object read from the stream.
```
private Object readResolve() {
    return INSTANCE;
}
```

## Singleton using enums
Singleton functionality can also be achieved by using a sinle-element enum.
```
enum CustomParser {
    INSTANCE;
}
```
This provides serialization functionality for free and we don't need the `readResolve` mechanism.