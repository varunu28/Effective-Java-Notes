# Item 2: Consider a builder when faced with many constructor parameters

Classes that consist of multiple optional attributes suffer from the problem of instantiating their classes without lot of boilerplate code. An approach which is common is to overload the constructor with all possible combination of class parameters. This approach soon ends up becoming hard to read and difficult to extend. This is called *telescoping pattern*.

Another approach is forcing the client to pass default values for attributes they don't want to set. This in turn makes the class difficult to understand and also forces the clients to have the knowledge of default values for each parameter.

**JavaBeans pattern** is another approach where the client instantiates the class with an empty constructor and then uses **setter** methods to set the parameters required. Though this approach can turn out to be error-prone as the instance construction is not an atomic process and is spread across multiple lines of code. This can lead to instance getting used when it is not completely instantiated. 
This also adds difficulty if we want the class to immutable. Then we will need to check if the `setter` for an attribute is not called more than once.

**Builders** are great for such use cases. In case of builder, client calls uses the builder object and sets the attributes similar to the JavaBeans pattern. Finally they call a `build` method which in-turn creates an instance of the class.

Consider the below `Pizza` class. It has a required parameter of `bread` and then the client can choose to add multiple types of `cheese` and `meat`. They can even choose to not add any `cheese` or `meat` and just set the `bread` parameter. 
```
public class Pizza {
  private final String bread;
  private final Set<Cheese> cheeses;
  private final Set<Meat> meats;

  enum Cheese {
    MOZZARELLA, PROVOLONE, CHEDDAR;
  }

  enum Meat {
    BACON, CHICKEN, SAUSAGE;
  }
}
```

Now if we start adding constructors for each combination of `cheese` and `meat`, our code will soon become bloated. Also pushing the responsibility of passing list of `cheese` and `meat` to client side also makes it usage difficult as now the client who doesn't want any `cheese` or `meat`, they are also forced to pass a default value. Also what happens when we introduce let's say a new attribute such as `sauce`. Now all the client code needs to be refactored. 

So we use the builder pattern to tackle this challenge. A builder class is a static inner class, which client uses to set the attributes of `Pizza` class something like below:
```
  public static class PizzaBuilder {
    private final String bread;
    private final Set<Cheese> cheeses;
    private final Set<Meat> meats;

    public PizzaBuilder(String bread) {
      this.bread = bread;
      this.cheeses = new HashSet<>();
      this.meats = new HashSet<>();
    }

    public PizzaBuilder addMeat(Meat meat) {
      this.meats.add(meat);
      return this;
    }

    public PizzaBuilder addCheese(Cheese cheese) {
      this.cheeses.add(cheese);
      return this;
    }

    public Pizza build() {
      return new Pizza(this);
    }
  }
```
We make the constructor of `Pizza` private and instantiate the `Pizza` class using `PizzaBuilder` as an input.
```
  private Pizza(PizzaBuilder pizzaBuilder) {
    this.bread = pizzaBuilder.bread;
    this.cheeses = pizzaBuilder.cheeses;
    this.meats = pizzaBuilder.meats;
  }
```

Client uses the `PizzaBuilder` to form `Pizza` as follows:
```
    Pizza pizza = new PizzaBuilder("sourdough")
        .addCheese(Cheese.PROVOLONE)
        .addCheese(Cheese.CHEDDAR)
        .addMeat(Meat.SAUSAGE)
        .addMeat(Meat.CHICKEN)
        .build();
```

 Input validation can also be done as part of the builder class.

 Builder pattern is also useful for class hierarchies. An `abstract` class like `Pizza` can have an `abstract` builder and classes extending this class like `NYPizza` add the concrete implementation of the builder. 
 All the extended classes return their own class instance as part of the `build` method. This is called `covariant return typing` and removes the need for casting.

 So, a builder pattern is a nice approach when we want to design classes with more than a few attributes and also when we are dealing with classes with multiple optional attributes.