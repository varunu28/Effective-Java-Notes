# Item 11: Always override hashCode when you override equals

Not overriding `hashCode` method while overriding `equals` method leads to unpredictable results when the object is used for collections such as `HashMap` and `HashSet`.

Contract of `hashCode` states that:
 - Repeatedly invoking `hashCode` should return the same value.
 - If two objects are equal while calling the `equals` method, then their `hashCode` should produce the same result.
 - If two objects are unequal, it is not necessary that the value returned from `hashCode` for both objects are different.

Consider the `PhoneNumber` class below:
```
class PhoneNumber {
    private int zipcode;
    private int number;

    public PhoneNumber(int zipcode, int number) {
        this.zipcode = zipcode;
        this.number = number;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof PhoneNumber phoneNumber)) {
            return false;
        }
        return phoneNumber.number == this.number && phoneNumber.zipcode == this.zipcode;
    }
}
```
We have overriden the `equals` method but we have not overriden the `hashCode` method. Now when we use it under a collection such as `HashMap`, we encounter an unexpected behavior. We insert an instance of `PhoneNumber` and then try to retrieve the value associated with it using a new instance with same fields. But we receive a null value. This is because `HashMap` has an optimization where it looks for the hash code of the key and returns a null if it doesn't finds a key with same hash code.
```
Map<PhoneNumber, String> directory = new HashMap<>();
directory.put(new PhoneNumber(12345, 56789), "John");
System.out.println(directory.get(new PhoneNumber(12345, 56789))); // Returns null
```
In order to fix the above problem, we have to ensure that two objects which are equal should generate the same hash code. But do note that a good hash function aims to produce different hash codes for different objects. Having tests to verify the equality using hashcode value acts as a safety mechanism unless you are using `AutoValue`. 

Now let us revisit the above example and see what does the hashcode implementation for `PhoneNumber` looks like:
```
@Override
public int hashCode() {
    int result = Integer.hashCode(zipcode);
    result = result * 31 + Integer.hashCode(number);
    return result;
}
``` 
Google's [Guava](https://guava.dev/releases/21.0/api/docs/com/google/common/hash/Hashing.html) provides a much more advanced implementation of the `Hashing` function. 

**Note 1:** If your class is immutable then you can cache the hashcode value and get an increase in performance as your hashcode is not being computed for each invocation. Also if the class is immutable, you can rather choose `record` instead of a Java class.

**Note 2:** Never provide a specification for the value to be returned by the hashcode function of your class. This will prevent you from changing it in future as clients will start depending upon this implementation.