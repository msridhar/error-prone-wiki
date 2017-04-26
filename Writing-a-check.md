### Tutorial: Writing a BugChecker

Once you know what you want to check for, you can write a BugChecker to do it
automatically. In this tutorial we will suppose that we want to ban the use of
`return null;` statements.

#### Avoiding `return null;` statements

It is well known that misuse of `null` can cause a staggering variety of bugs.
You can find more [details in this document]
(https://github.com/google/guava/wiki/UsingAndAvoidingNullExplained). Let us
suppose that we agree with the arguments in favor of avoiding null, so as a
first step we decide to avoid any `return null;` statements. Put plainly, we
want to avoid code like this:

```java
public Person findUser(String name) {
  if (database.contains(name)) {
      return new Person(name);
  }
  return null; // Code to avoid
}
```

Suppose that some of our colleagues have strong arguments against this policy. A
lot of our code would break if we just decided to ban `return null;` statements!
After much discussion, we all agree that if a developer explicitly annotates a
method with `@Nullable`, the `return null;` statements will be allowed:

```java
@Nullable
public Person findUser(String name) {
  if (database.contains(name)) {
    return new Person(name);
  }
  return null; // this is okay because of the @Nullable annotation
}
```

We now have a policy that we can attempt to enforce: **The use of `return null;`
is banned, except in methods annotated with `@Nullable`.**

#### Create a BugChecker

How do we enforce our policy? By writing a BugChecker! Here's some skeleton code
that will get us started in writing our BugChecker.

```java
package com.google.errorprone.bugpatterns;

import static com.google.errorprone.BugPattern.Category.JDK;
import static com.google.errorprone.BugPattern.MaturityLevel.EXPERIMENTAL;
import static com.google.errorprone.BugPattern.SeverityLevel.SUGGESTION;

import com.google.errorprone.BugPattern;
import com.google.errorprone.VisitorState;
import com.google.errorprone.bugpatterns.BugChecker;
import com.google.errorprone.bugpatterns.BugChecker.MethodTreeMatcher;
import com.google.errorprone.matchers.Description;

import com.sun.source.tree.MethodTree;

/**
 * Bug checker to detect usage of {@code return null;}.
 */
@BugPattern(
  name = "DoNotReturnNull",
  summary = "Do not return null.",
  category = JDK,
  severity = SUGGESTION,
  maturity = EXPERIMENTAL)
public class DoNotReturnNull extends BugChecker implements MethodTreeMatcher {
  // your code here

  @Override
  public Description matchMethod(MethodTree tree, VisitorState state) {
    // your code here
    return Description.NO_MATCH;
  }
}
```

First, the annotation on the BugChecker contains the name, a short summary, the
category, severity of the bug, and the maturity of the bug. We have chosen to
fill these out for you; since our check is new, the maturity will be
`EXPERIMENTAL` and since we don't want to necessarily force everyone to use this
check, the severity has been listed as `SUGGESTION`.

Next, the BugChecker itself is an implementation of `MethodTreeMatcher` because
we want to match *methods* that return `null`. Lastly, the logic in
`matchMethod` should return a match if the method tree does **not** have a
`@Nullable` annotation and the method body contains a `return null;` statement.
However, nothing has been implemented! Let's fix that, making liberal use of
utility methods from the `Matchers` class.

#### Exercise 1

Let's start with writing code for a matcher that will determine whether a method
is annotated with @Nullable. We want to fill in the right hand side of the
following statement:

```java
...
import javax.annotation.Nullable;
...

private static final Matcher<Tree> HAS_NULLABLE_ANNOTATION = //your code goes here
```

Write code for a matcher that will determine whether a method has a @Nullable
annotation.

#### Exercise 1 Solution

```java
import static com.google.errorprone.matchers.Matchers.hasAnnotation;
...
import javax.annotation.Nullable;
...
private static final Matcher<Tree> HAS_NULLABLE_ANNOTATION =
    hasAnnotation(Nullable.class.getCanonicalName());
```

#### Exercise 2

Now that we can detect when a method has a `@Nullable` annotation, we want to
write a matcher that returns true when it encounters a method with a `return
null;` statement and returns false otherwise. Below is the definition of the
matcher class that we need to implement.

```java
private static final Matcher<Tree> CONTAINS_RETURN_NULL = //your code goes here
...
private static class ReturnNullMatcher implements Matcher<Tree> {
  @Override
  public boolean matches(Tree tree, VisitorState state) {
    // your code goes here
  }
}
```

Again, the `com.google.errorprone.matchers.Matchers` class will have the methods
we need. Let's fill in the right hand side of the second statement, with the
assumption that `ReturnNullMatcher` will match `return null;` statements once
implemented:

```java
...
private static final Matcher<Tree> RETURN_NULL = new ReturnNullMatcher();
private static final Matcher<Tree> CONTAINS_RETURN_NULL = //your code goes here
...
```

#### Exercise 2 Solution

```java
import static com.google.errorprone.matchers.Matchers.contains;
...
private static final Matcher<Tree> RETURN_NULL = new ReturnNullMatcher();
private static final Matcher<Tree> CONTAINS_RETURN_NULL = contains(RETURN_NULL);
...
```

#### Exercise 3

Now, let's implement `ReturnNullMatcher`. Remember that you will need to
override the `matches` method.

> **Note**: There is more than one way to complete this exercise, but in search
> of the simplest solution, you can take a look at
> `com.sun.source.tree.Tree.Kind.NULL_LITERAL`, as that will allow you to
> directly match a null literal expression instead of having to match using
> `LiteralTree`.

```java
public class DoNotReturnNull extends BugChecker implements MethodTreeMatcher {
...

  private static class ReturnNullMatcher implements Matcher<Tree> {
    @Override
    public boolean matches(Tree tree, VisitorState state) {
    // your code goes here
    }
  }
}
```

#### Exercise 3 Solution

```java
...
import static com.sun.source.tree.Tree.Kind.NULL_LITERAL;
...
import com.sun.source.tree.ReturnTree;
...
private static class ReturnNullMatcher implements Matcher<Tree> {
  @Override
  public boolean matches(Tree tree, VisitorState state) {
    if (tree instanceof ReturnTree) {
      return ((ReturnTree) tree).getExpression().getKind() == NULL_LITERAL;
    }
    return false;
  }
}
```

#### Exercise 4

We're almost there! Now that we have both the ability to check if a method has a
`@Nullable` annotation as well as the ability to check whether a method contains
a`return null;` statement, all that's left is to put the two together to create
a matcher that will match methods that satisfy both of our criteria. Our code
should now look something like this:

```java
...
import static com.google.errorprone.matchers.Matchers.contains;
import static com.google.errorprone.matchers.Matchers.hasAnnotation;
...
import static com.sun.source.tree.Tree.Kind.NULL_LITERAL;
...
import com.sun.source.tree.ReturnTree;
...
import javax.annotation.Nullable;
...

@BugPattern(...)
public class DoNotReturnNull extends BugChecker implements MethodTreeMatcher {
  private static final Matcher<Tree> HAS_NULLABLE_ANNOTATION =
      hasAnnotation(Nullable.class.getCanonicalName());
  private static final Matcher<Tree> RETURN_NULL = new ReturnNullMatcher();
  private static final Matcher<Tree> CONTAINS_RETURN_NULL = contains(RETURN_NULL);

  @Override
  public Description matchMethod(MethodTree tree, VisitorState state) {
    ...
  }

  /**
   * Matches any Tree that represents return of a null.
   */
  private static class ReturnNullMatcher implements Matcher<Tree> {

    @Override
    public boolean matches(Tree tree, VisitorState state) {
      if (tree instanceof ReturnTree) {
        return ((ReturnTree) tree).getExpression().getKind() == NULL_LITERAL;
      }
      return false;
    }
  }
}
```

Let's now fill in `matchMethod` that will complete our BugChecker. If the method
does not have the `@Nullable` annotation and the method body contains `return
null;`, we want to return `describeMatch(tree)`, but in all other cases we want
to return `Description.NO_MATCH`.

#### Exercise 4 Solution

```java
@Override
public Description matchMethod(MethodTree tree, VisitorState state) {
   if (HAS_NULLABLE_ANNOTATION.matches(tree, state)
       || !CONTAINS_RETURN_NULL.matches(tree.getBody(), state)) {
     return Description.NO_MATCH;
   }
   return describeMatch(tree);
}
```

Once you've completed your BugChecker, it's time to move on to testing it.

### Testing a BugChecker

When writing tests for a BugChecker, you should check for both *positive* and
*negative* cases. Positive cases are those that will trigger the check. In our
example, any method that includes a `return null;` statement is a positive case.
Negative cases are those that will not trigger the check.

To check for positive cases, create a valid java file that contains violations
of the policy being checked and place the file in a `testdata` subdirectory. In
our case this means that the file should include at least one `return null;`
statement. As in the example that follows, add a comment before each AST node
that should be identified by a BugChecker as a match.

> **Note**: Remember that the BugChecker will match a method that returns null.
> Thus, the diagnostic comment should immediately precede the method definition,
> not the return expression.

```java
public class DoNotReturnNullPositiveCases {
  // BUG: Diagnostic contains: Do not return null.
  public Object returnsNull() {
    return null;
  }
}
```

For negative cases, you just need to create a valid java file that does not
contain a violation of the policy being checked, and again place it in a
`testdata` subdirectory.

Now, you can use `com.google.errorprone.CompilationTestHelper` to test your
BugChecker by creating an ordinary JUnit suite and adding a test for each
positive and negative case you want to check.

> **Note**: As per the example, the helper expects a comment in the
> following
> format: `// BUG: Diagnostic contains: <SUMMARY>`, where `<SUMMARY>`
> is the
> value of the `summary` parameter of the BugChecker's `BugPattern`
> annotation.

The whole JUnit test then looks like this:

```java
package com.google.errorprone.bugpatterns;

import com.google.errorprone.CompilationTestHelper;
import com.google.testing.testsize.MediumTest;
import com.google.testing.testsize.MediumTestAttribute;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.JUnit4;

/** Unit tests for {@link DoNotReturnNull}. */
@RunWith(JUnit4.class)
@MediumTest(MediumTestAttribute.FILE)
public class DoNotReturnNullTest {

  private CompilationTestHelper compilationHelper;

  @Before
  public void setup() {
    compilationHelper = CompilationTestHelper.newInstance(DoNotReturnNull.class, getClass());
  }

  @Test
  public void doNotReturnNullPositiveCases() {
    compilationHelper.addSourceFile("DoNotReturnNullPositiveCases.java").doTest();
  }

  @Test
  public void doNotReturnNullNegativeCases() {
    compilationHelper.addSourceFile("DoNotReturnNullNegativeCases.java").doTest();
  }
}
```

#### Exercise 5.

Write tests in `DoNotReturnNullPositiveCases.java` and `DoNotReturnNullNegativeCases.java`.

#### Exercise 5 Positive Cases Example Solution

```java
package com.google.errorprone.bugpatterns.testdata;

public class DoNotReturnNullPositiveCases {

  // BUG: Diagnostic contains: Do not return null.
  public Object returnsNull() {
    return null;
  }

  // BUG: Diagnostic contains: Do not return null.
  public Object sometimesReturnsNull(boolean random) {
    if (random) {
      return null;
    }
    return new Object();
  }
}
```

#### Exercise 5 Negative Cases Example Solution

```java
package com.google.errorprone.bugpatterns.testdata;

import javax.annotation.Nullable;

public class DoNotReturnNullNegativeCases {

  public Object doesNotReturnNull() {
    return new Object();
  }

  @Nullable
  public Object returnsNullButAnnotatedWithNullable() {
    return null;
  }
}
```
