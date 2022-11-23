# Item 5: Prefer dependency injection to hardwiring resources

Classes in real-life depend upon other classes for performing their use cases. For example consider the below `FancyEditor` class. It depends upon

```
public class FancyEditor {

  private final FileFormatter fileFormatter = new WordDocFormatter();
  
  public void update(Document document) {
    
  }
  
  public void save(Document document) {
    fileFormatter.format(document);
  }
}
```

The `FancyEditor` is used by clients to edit a `Document` and it depends upon `FileFormatter` for formatting the file after it has been updated. 

Now in above implementation of `FancyEditor`, we have added a concrete implementation of `FileFormatter` as `WordDocFormatter`. What happens if we need to change it with a `PdfDocFormatter`. Or what happens if we need two different types of `FancyEditor`. One for word document and one for pdfs. The current implementation will require refactoring and even lead to duplication of code.

This can be solved by providing the flexibilty of choosing the underlying dependency to the client of `FancyEditor`. This approach is known as **dependency injection**. Here the `FancyEditor` only knows that it has a dependency upon some form of implementation of `FileFormatter` interface.
```
interface FileFormatter {

  void format(Document document);
}
```

And now the client is responsible for passing in the concrete implementation, decoupling the `FancyEditor` from hard-coded implementation.
```
public class FancyEditor {

  private FileFormatter fileFormatter;

  public FancyEditor(FileFormatter fileFormatter) {
    this.fileFormatter = fileFormatter;
  }

  public void update(Document document) {
    // Updating...
    fileFormatter.format(document);
  }
}
```
Now client can instantiate two different `FancyEditor` for `WordDocFormatter` & `PdfDocFormatter` as below:
```
FancyEditor fancyEditorForWordDoc = new FancyEditor(new WordDocFormatter());

FancyEditor fancyEditorForPdf = new FancyEditor(new PdfFormatter());
```

This also promotes immutability as now the `FancyEditor` can use the same instance of underlying dependency. Factory methods can also be used as part of `FancyEditor` which provides various formatter instances based upon the use case.

With increase in number of dependencies, constructor based dependency injection can end up becoming verbose. Because of this there are various frameworks such as Guice, Spring that provide dependency injection in a readable manner.