# Item 1: Consider static factory methods instead of constructors

 - Class can provide clients a static factory method for instance creation in place-of or in addition to a public constructor.
 - Advantage of having a static factory method is that they have logical names. Example `createNewEmployee`
 
## Use cases of factory methods over constructors

<details>
<summary>Better readability for arbitary constructor parameters</summary>
<br>
A place where factory methods are helpful is when a class can be constructed using multiple permutations of constructor parameters. Most common solution to deal with such a case is by having multiple constructors for each case. 

Consider the following example where a `User` class has three attributes i.e. `name`, `screeName` & `zipcode` where `name` is a required attribute. Having multiple constructors for each combination soon ends up becoming confusing and the client has to look into the actual implementation for correct usage. This can soon become a reason for bugs if the source code is not available and constructors are not well documented.
 ```
 public class User {

  private final String name;
  private String screeName;
  private Integer zipcode;
  
  public User(String name) {
    this.name = name;
  }

  public User(String name, String screeName) {
    this.name = name;
    this.screeName = screeName;
  }

  public User(String name, Integer zipcode) {
    this.name = name;
    this.zipcode = zipcode;
  }

  public User(String name, String screeName, Integer zipcode) {
    this.name = name;
    this.screeName = screeName;
    this.zipcode = zipcode;
  }
}
 ```
Now if we make use of well-named factory methods then the above class becomes more readable as we name each method to describe what kind of `User` are we constructing.
```
public class User {

  private final String name;
  private String screeName;
  private Integer zipcode;

  private User(String name, String screeName, Integer zipcode) {
    this.name = name;
    this.screeName = screeName;
    this.zipcode = zipcode;
  }

  public static User createUser(String name) {
    return new User(name, null, null);
  }

  public static User createUserWithScreeName(String name, String screeName) {
    return new User(name, screeName, null);
  }

  public static User createUserWithZipcode(String name, Integer zipcode) {
    return new User(name, null, zipcode);
  }

  public static User createUserWithScreeNameAndZipcode(String name, String screeName,
      Integer zipcode) {
    return new User(name, screeName, zipcode);
  }
}
```
</details>

<details>
<summary>Allow instantiating classes as Singleton</summary>
<br>
Classes such as database wrappers which take expensive computation to be constructed can make use of static factory methods. If a public constructor is not exposed then the class author controls, how many instances of the class can be constructed. This avoids heavy work and allows for re-usage of class instances.

```
public class DatabaseWrapper {
  
  private static DatabaseWrapper instance;

  private DatabaseWrapper() {
    // Heavy computation
  }
  
  public static DatabaseWrapper getInstance() {
    if (instance == null) {
      instance = new DatabaseWrapper();
    }
    return instance;
  }
}
```
</details>

<details>
<summary>Return objects of any subtype</summary>
<br>
For cases where we want to return different object types based upon the input, factory methods are great for abstracting the underlying class implementation. We can return the object of type interface and hide the implementation as private classes.

For example in below `EmployeeManagement` class, we want to return `Employee` based upon the `EmployeeType`. 

```
enum EmployeeType {
  ENGINEERING_MANAGER, SOFTWARE_ENGINEER, PRODUCT_MANAGER;
}
```
But our client is not concerned with internals of class being returned. They want the `performTask` method to work according to the `EmployeeType` they provide. 
```
interface Employee {

  void performTask();
}
```
This fits our use case perfectly and we can leverage factory methods to return concrete implementations and at the same time hide the concrete classes as part of `EmployeeManagement` class.

```
public class EmployeeManagement {

  private EmployeeManagement() {}

  public static Employee createNewEmployee(EmployeeType employeeType) {
    return switch (employeeType) {
      case ENGINEERING_MANAGER -> new EngineeringManager();
      case SOFTWARE_ENGINEER -> new SoftwareEngineer();
      case PRODUCT_MANAGER -> new ProductManager();
    };
  }

  private static class EngineeringManager implements Employee {

    @Override
    public void performTask() {
      System.out.println("Working as engineering manager");
    }
  }

  private static class SoftwareEngineer implements Employee {

    @Override
    public void performTask() {
      System.out.println("Working as software engineer");
    }
  }

  private static class ProductManager implements Employee {

    @Override
    public void performTask() {
      System.out.println("Working as product manager");
    }
  }
}
```
</details>

<details>
<summary>Return object of class based upon function of input parameters</summary>
<br>
Using static factory methods allows us to return sub-classes of the return class type. Client just sees the return class type and the underlying functionality changes can be abstracted based upon the input parameters. This way we can remove a sub-class without impacting the client side contract.

For example `InventoryContainer` provides a data structure for storage based upon the `size` parameter. Now for `size` greater than 100, we want the container to be of type `long` and for `size` less than 100, it should be of type `int`. We achieve this by creating two subclasses that provide container of above two types respectively. 
```
private static class SmallInventoryContainer extends InventoryContainer {
    private int[] container;

    public SmallInventoryContainer(int size) {
        this.container = new int[size];
    }
}

private static class BigInventoryContainer extends InventoryContainer {
    private long[] container;

    public BigInventoryContainer(int size) {
        this.container = new long[size];
    }
}
```

Now we can return any subtype of `InventoryContainer` and client is unaware about the internal workings of the class. This is done by putting a static factory method inside `InventoryContainer`
```
public class InventoryContainer {

  private InventoryContainer() {}

  public static InventoryContainer getInventoryContainer(int size) {
    return size > 100 ? new BigInventoryContainer(size) : new SmallInventoryContainer(size);
  }
}
```
</details>

<details>
<summary>Class of return type need not exist when class containing the method is written</summary>
<br>
This forms the foundation of service provider framework. For example JDBC. Here the framework provides implementation of a service and the system makes the implementation available to client.

```
public static MyServiceInterface getMyService() {
    // Load instance and serve to the client
}
```
</details>

## Limitations of static factory methods
 - Classes without `public` or `protected` constructor cannot be subclassed.
 - Static factory methods don't form the Javadoc for a class hence they can be difficult to find. This can be overcome by using common naming conventions like `from`, `instance` or `getInstance`.
