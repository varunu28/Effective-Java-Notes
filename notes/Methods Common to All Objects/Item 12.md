# Item 12: Always override toString

The default implementation of the `toString` method is not very user friendly. Consider the following `Person` class
```
class Person {
    private final String firstName;
    private final String lastName;

    public Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```
If we create an instance of this class and invoke `toString` for that instance, the result we get does not tell us anything meaningful about that class. It consists class name, an `@` sign and unsigned hexadecimal representation of hash code.
```
Person person = new Person("John", "Wick");
System.out.println(person); // Returns Person@36baf30c. Not helpful
```

Providing meaningful result as part of `toString` method helps the user understand what the instance represents and further helps during the debugging process. This also helps when the instances of an object are passed to a collection in order to distinguish among multiple instances.
```
@Override
public String toString() {
    return "Person{" +
            "firstName='" + firstName + '\'' +
            ", lastName='" + lastName + '\'' +
            '}';
}

System.out.println(person); // Returns "Person{firstName='John', lastName='Wick'}"
```

Also note that the `toString` method is directly invoked when the instance is passed to any output, concatenation or assert operation. 

One important use case is that the result returned by the `toString` method can have a well-defined format(or not). If you decide to expose the format to the user then it is helpful to expose factory method for user to serialize and deserialize the class object. A disadvantage to specifying the format is that the user is now dependent upon this format and you cannot break the contract for the format in future.

Also ensure that the information returned by the `toString` method is also accessible by instance or else the user will need to parse the string representation which is going to be error-prone.

With Java 14 and above, consider using `record` in place of Java class if your attributes are immutable. `record` provides a user-friendly `toString` implementation out of the box. Considering the above example:
```
record Person(String firstName, String lastName) {}

System.out.println(person); // Returns "Person[firstName=John, lastName=Wick]"
```