# Java Coding Guidelines

[Source File Structure](#source-file-structure)  
[Formatting](#formatting)

## Source File Structure

Each source file has four sections, each separated by a single blank line:

1. Licence or copyright information
2. Package statement
3. Import statements
4. Exactly one top-level class

A `package-info.java` file is the same, but without the top-level class. A `module-info.java` file
follows the same structure but does not have a package statement and substitutes a module
declaration for the top-level class.

### Package Statement

The packages statement is not line-wrapped; the column limit does not apply.

### Import Statements

Import statements must observe the following:

+ Static imports (importing members of classes) are not allowed
+ Wildcard imports are not allowed
+ Import statements are not line-wrapped
+ No blank lines should separate import statements

### Top-Level Class

Every top-level class resides in its own source file.

#### Member Ordering

Members of a class that share the same name should be ordered in sequence with no other members in
between.

### Module Declaration

Module directives are ordered as follows:

1. All `requires` directives in a single block
2. All `exports` directives in a single block
3. All `opens` directives in a single block
4. All `users` directives in a single block
5. All `provides` directives in a single block

A single blank line separates each block that is present.

## Formatting

### Braces

#### Optional Braces

Braces for empty or single statement `if`, `else`, `for`, `do`, `while` and other statements are
optional.

#### Non-empty Blocks

Braces follow the Kernighan and Ritchie style for non-empty block-like constructs:

+ No line breaks before the opening brace, except as detailed below
+ Line break after the opening brace
+ Line break before the closing brace
+ Line break after the closing brace, only if that brace terminates a statement or terminates the
  body of a method, constructor, or named class. For example, there is no line break after the brace
  if it is followed by `else` or a comma.

A block of statements inside a method has its opening and closing braces on their own lines.

Examples:

```java
class MyClass {
    public void method() {
        if(condition) {
            try {
                something();
            } catch (Exception e) {
                recover();
            }
        } else if (otherCondition) {
            somethineElse();
        } else {
            lastThing();
        }
        {
            int x = 10;
            print(x);
        }
    }
}
```

#### Empty Blocks

An empty block can follow the [non-empty block](#non-empty-blocks) rules above or may be closed
immediately after it is opened, with no characters or line break in between.

### Indentation

Each time a new block or block-like construct is opened the indent is increased by two spaces. When
the block closes then the indent returns to the previous level.

### One Statement Per Line

Each statement is followed by a line break.

### Column Limit

Code must obey a 100 character line limit, with any line that would exceed this limit wrapped.

Exceptions:

+ Lines where obeying the limit is not possible (for example, a long URL in Javadoc)
+ `package` and `import` statements (see [Package Statement](#package-statement) and
  [Import Statements](#import-statements) above)
+ Very long identifiers, on the rare occasions they are called for, are allowed to exceed the limit

### Line Wrapping

While line wrapping should be applied to lines that would otherwise exceed the column limit, it
may also be applied to lines that would fit within the limit at the author's discretion.

#### Line Breaks

The prime directive of line-wrapping is to have clear code, not necessarily to conserve lines. We
should prefer to break at a higher syntactic level. Also:

1. When a line is broken at a non-assignment operator the break comes before the symbol.
   This also applies to the following "operator-like" symbols:
    + the dot separator (`.`)
    + the two colons of a method reference (`::`)
    + an ampersand in a type bound (`<T extends Foo & Bar>`)
    + a pipe in a catch block (`catch (FooException | BarException e)`).
2. When a line is broken at an assignment operator the break typically comes after the symbol,
   but either way is acceptable. This also applies to the "assignment-operator-like" colon in an
   enhanced `for` ("foreach") statement.
3. A method, constructor, or record-class name stays attached to the open parenthesis that follows
   it.
4. A comma (`,`) stays attached to the token that precedes it.
5. A line is never broken adjacent to the arrow in a lambda or a switch rule, except that a break
   may come immediately after the arrow if the text following it consists of a single unbraced
   expression.

```java
class ToBeDone {
    
}
```

#### Line Wrap Indents

Each line after the first (each continuation line) is indented by at least four spaces from the
original line. In general, two continuation lines use the same indentation level if and only if
they begin with syntactically parallel elements.

### Whitespace

#### Vertical Whitespace

A single blank line always appears between consecutive members (fields, constructors, methods,
nested classes, et al.) of a class except when consecutive fields are logically grouped together.

Blank lines in methods are typically indicators of logical parts of the method doing separate jobs
which violates the single responsibility principle. Empty lines are discouraged for this reason
but are still allowed if they improve readability. Instead, prefer to decompose the method into
smaller methods that focus on a single thing.

#### Horizontal Whitespace

A single ASCII space appears in the following places:

1. Separating any reserved word, such as  `if`, `for` or `catch`, from an open parenthesis `(` that
   follows it on that line
2. Separating any reserved word, such as `else` or `catch`, from a closing curly brace `}` that
   precedes it on that line
3. On both sides of any binary or ternary operator. This also applies to the following
   "operator-like" symbols:
    + the ampersand in a conjunctive type bound: `<T extends Foo & Bar>`
    + the pipe for a catch block that handles multiple exceptions:
      `catch (FooException | BarException e)`
    + the colon `:` in an enhanced `for` statement
    + the arrow in a lambda expression: `(String str) -> str.length()`
    + or switch rule: `case "FOO" -> bar();`

   but not the double colon `::` or dot operator `.`.
4. After `,`, `:`, `;` or the closing parenthesis `)` of a cast
5. Between any content and a double slash `//` which begins a comment. Multiple spaces are
   allowed.
6. Between a double slash `//` which begins a comment and the comment's text. Multiple spaces are
   allowed.
7. Between the type and variable of a declaration: `List<String> list`

#### Horizontal Alignment

Horizontal alignment is the practice of adding a variable number of additional spaces in your code
with the goal of making certain tokens appear directly below certain other tokens on previous
lines. The practice is strongly discouraged as attempts to maintain alignment can cause formatting
updates to otherwise unchanged lines, polluting versioning history.

### Variable Declarations

#### One Variable Per Declaration

Every variable declaration (field or local) declares only one variable: declarations such as
`int a, b;` are not used.

**Exception**: Multiple variable declarations are acceptable in the header of a `for` loop.

#### Declare When Needed

Local variables are not habitually declared at the start of their containing block or block-like
construct. Instead, local variables are declared close to the point they are first used (within
reason), to minimise their scope. Local variable declarations typically have initialisers, or are
initialised immediately after declaration.

### Comments

This section addresses implementation comments as opposed to Javadoc comments.

#### Block Style Comments

Block comments are indented at the same level as the surrounding code. They may be in `/* ... */`
style or `// ...` style. For multi-line `/* ... */` comments, subsequent lines must start with `*`
aligned with the `*` on the previous line.

Comments are not enclosed in boxes drawn with asterisks or other characters.

## Naming

Identifiers use only ASCII letters and digits. Prefixes should never be used to indicate the type
of the identifier.

### Class Names

Class names are written in upper camel case. Avoid using verbs as class names as this encourages
procedural programming instead of object-oriented; class names should describe what they are not
what they do. Use of words such as Manager, Handler, Object, Controller, or Data typically provide
no semantic value and should be avoided. Use nouns or noun phrases. Interfaces may be adjectives
or adjective phrases instead such as `Writeable`. Classes that implement an interface should
explain an implementation such as `DynamicList` which implements `List`. If there is nothing
specific to say about an implementation, name it `Default`, `Simple`, or something similar.

Consider:

+ A `Stream` and a `Record` instead of a `Loader`
+ A `Registry` instead of a `Manager` if that's what it is.
+ An `Organisation` instead of an `Organiser`
+ An `Analysis` instead of an `Analyser`
+ A `Render` instead of a `Renderer`

#### Method Names

Method names are written in lower camel case and are usually verb or verb phrases. If a method
does not return something then the name should describe what it does; if it does return something
its name should explain what it returns.

***Never use getters and setters*** as they expose implementation details, encourage coupling, and
diminish the benefits of encapsulation. Design your classes and methods so that you tell a class
what to do instead of asking it for the information so you (the client) can do it yourself.

#### Constant Names

Constants are static final fields whose contents are completely immutable and whose methods have no
observable side effects. Constant names are written in upper snake case with all uppercase letter
and words separated by underscores `_`.

#### Variable Names

Variable names are written in lower camel case and are usually noun or noun phrases. A variable
name should be long enough to avoid ambiguity in its scope of visibility, but not too long if
possible. A name should be a noun in singular or plural form, or an appropriate abbreviation.
Compound names are a good sign that the scope that the variable exists in is too large and complex
otherwise a simple name would be ambiguous. Instead, decompose the enclosing scope into simpler
pieces where simple names can be used without ambiguity. Look at this class:

```java
class CSV {
   private Path filePath;

   public CSV(String csvFileName) {
       this.filePath = Path(csvFileName);
   }
}
```

The scope of `csvFileName` is the constructor of class `CSV`. It should be clear that the single
argument constructor expects the name of a CSV file, which would make the compound name
`csvFileName` unnecessarily long. Next, the scope of the `filePath` variable is the entire class
`CSV`. Renaming this to path would not introduce any ambiguity. This class could be refactored as
follows:

```java
class CSV {
   private Path path;

   public CSV(String file) {
       this.path = Path(file);
   }
}
```

If you cannot perform this refactoring it may be a good sign that the scope is too big, complex, or
a combination of both.

## Programming Practices

### One Primary Constructor

Each class should only have one primary constructor which initialises its state. Secondary
constructors sit in front of this constructor and eventually call the primary constructor as their
final step. The primary constructor should be defined after all secondary constructors in the
source code. ***This practice eliminates code duplication.***

Each class will have a single point of construction that is easy to find.

# Misc

Empty lines are strongly discouraged in methods as they are typically used to logically group
statements that are doing different things in a method. Instead, prefer to decompose the method
into smaller methods that focus on a single thing.
 
