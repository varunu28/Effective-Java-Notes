# Item 10: Obey the general contract when overriding equals

Overriding `equals` method should be done carefully as incorrect usage of override can lead to bugs and inconsistent results. If possible overriding `equals` can be avoided altogether. Few scenarios where override can be avoided are:
 - Each instance of class is inherently unique like `Thread`
 - Class will never go through an logical equality test
 - A super class has already overriden the `equals` method and that override is valid for the subclass
 - Class is private and there is no way for someone to invoke the `equals` method. Additional safety in such cases can be to override the `equals` method to throw an exception if the method is invoked

For a class that represents a value class such as an `Integer` or a `String`, overriding `equals` method makes sense. This is important for the class instances to be used as part of a `HashMap` or a `HashSet` which relies on logical equality. Classes that ensure that there can only be one instance of themselves(For example: enums) don't need to override the `equals` method. Properties of equality implemented by the `equals` method are:
 - **Reflexive:** For any non-null instance `x`, `x.equals(x)` should always be true.
 - **Symmetric:** For any non-null references `x` and `y`, `x.equals(y)` can be true only if `y.equals(x)` is also true. Here is an example where this might be easy to miss while overriding `equals` method. In the below example passing an instance of either a `String` or `CaseInsensitiveString` to the `equals` method will return a true. But the reverse of this is not true as the `String` class's `equals` method has no concept of `CaseInsensitiveString`. 
 ```
 class CaseInsensitiveString {
    private String s;

    public CaseInsensitiveString(String s) {
        this.s = s;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj instanceof CaseInsensitiveString) {
            return this.s.equalsIgnoreCase(((CaseInsensitiveString) obj).s);
        }
        if (obj instanceof String) {
            return this.s.equalsIgnoreCase((String) obj);
        }
        return false;
    }
}
 ```
 - **Transitive:** For non-null reference say `x`, `y` and `z` if `x.equals(y)` returns true and `y.equals(z)` returns true then `x.equals(z)` must also return true. This can be possible when we have a class say `Point` defined as:
 ```
 class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Point point)) {
            return false;
        }
        return point.x == this.x && point.y == this.y;
    }
}
 ```
 Now we want to extend this class to represent a `ColorPoint` which has an additional attribute of `color`. 
 ```
 class ColorPoint extends Point {

    private final String color;

    public ColorPoint(int x, int y, String color) {
        super(x, y);
        this.color = color;
    }
}
 ```
 Now if we don't override the `equals` method in `ColorPoint` class then the equality comparision will fallback onto the `Point` class implementation. So two color points with different color can end up being considered equal if they have same `Point` coordinates. 
 
 One attempt to solve this problem is to override the `equals` under `ColorPoint` class also as below:
 ```
 @Override
public boolean equals(Object obj) {
    if (!(obj instanceof Point)) {
        return false;
    }
    if (!(obj instanceof ColorPoint)) {
        return obj.equals(this);
    }
    return super.equals(obj) && ((ColorPoint) obj).color.equals(color);
}
 ```
 With the above implementation, we satisfy the property of **symmetry** as now a `Point` class is equal to a `ColorPoint` class and vice-versa. But the **transitive** property is broken. Consider the below case:
 ```
 ColorPoint c1 = new ColorPoint("Red", 1, 1);
 Point p1 = new Point(1, 1);
 ColorPoint c2 = new ColorPoint("Green", 1, 1);

 c1.equals(p1) // True
 p1.equals(c2) // True
 c1.equals(c2) // False
 ```
 One solution to solve the above problem is to *avoid inheritence and favor composition*. So instead of `ColorPoint` class extending `Point` class, `ColorPoint` should be composed of `Point` instance as below:
 ```
 class ColorPoint {

    private final String color;
    private final Point point;

    public ColorPoint(Point point, String color) {
        this.point = point;
        this.color = color;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof ColorPoint colorPoint)) {
            return false;
        }
        return this.color.equals(colorPoint.color) && this.point.equals(colorPoint.point);
    }
}
 ```
 - **Consistent:** Repeated invokations of `equals` method for two instances should return the same(true or false) result given that there are no changes in the method implementation or object attributes. One of the ways where consistency can be compromised is when the `equals` method relies on external factors in addition to the class attributes such as `time`.
 - For a non-null reference `x`, `x.equals(null)` should always return false. Though this doesn't require an explicit null check as the `instanceof` check returns a false if the underlying instance is null.

## Recipe for high-quality equals method
 - Use `==` operation to check if both the objects point to the same reference. If they both are pointing to same reference then we can directly return true and gain in performance.
 - If object is not of correct type then return false. This is done by using `instanceof` operator. 
 - Cast the object to correct type after `instanceof` check has passed.
 - For each underlying field that describes the class, check for equality of field for both objects.
    - For primitive types `==` operator can be used.
    - For `double` and `float` types, use `compare` method instead of `equals` as it will avoid *autoboxing* and hence will give better performance.
    - For array types, use `Arrays.equals` method.
    - If there is a possibility of having a null type, then use `Object.equals` method to avoid `NullPointerException`.
    - For classes where we want to perform equality comparision based on a canonical form, it is best suited to store the canonical form before-hand so that `equals` method can make use of that instead of building the canonical form on every invocation. For example: if a class wants to compare a string attribute in case-insensitive format then it is better to store the case-insensitive version of that string before-hand.
    - Always start by comparing the fields that are most likely to differ in order to make the `equals` method performant. 
    - Don't compare fields that do not represent the class state. For example locks, database connections etc.
    - Evaluate if you want to compare the derived fields or if the derived field represent the correct state of your class. 
- Always check for three attributes of `equals` method
    - Is it symmetric? 
    - Is it transitive?
    - Is it consistent?
- Always override `hashcode` method when you override `equals` method.
- Don't substitute `Object` class type with another class type for `equals` method. Doing this fails to override the underlying `equals` method.

## Note
Writing and testing `equals` method comes with its own set of challenges. Google's open source framework [AutoValue](https://github.com/google/auto) automatically generates these methods for you by adding a single annotation. This framework is well-tested and overloads all the above mentioned checks and balances to an external library.