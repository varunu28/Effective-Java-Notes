# Item 9: Prefer try-with-resources to try-finally 

The `try-finally` pattern ends up becoming an incorrect approach to close the resources and also increases the level of difficulty for debugging the errors. For example:
```
BufferedReader reader = 
    new BufferedReader(new FileReader("file.txt"));
try {
    System.out.println(reader.readLine());
} finally {
    reader.close();
}
```
Here `readLine` method can throw an error due to failure in physical device after which `close` method will also throw an error due to same reason. But the error due to `readLine` method call will get suppressed and it will make debugging difficult.

Resources implementing `AutoCloseable` resource can be used with `try-with-resources` to solve the above mentioned problems. Above code can be refactored using `try-with-resources` as below:
```
try (BufferedReader reader = 
    new BufferedReader(new FileReader("file.txt"))) {
    System.out.println(reader.readLine());
}
```

Multiple resources can be used as part of `try-with-resource` mechanism given that all of them implement the `AutoCloseable` interface. This approach also makes it easier to debug for errors and also makes the code cleaner and much more easier to read.