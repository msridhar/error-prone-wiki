# Getting started

We use the Maven build system. We are using version 3. 
[Download Maven](http://maven.apache.org/download.html)

We test with OpenJDK 7 and 8, so those are currently recommended. Do whatever
your system requires to use one of those JDKs.

Build the library:
<pre>
$ cd error-prone
$ mvn package
</pre>

You'll also need to configure your IDE to build with a supported JDK.

A Maven plugin for your IDE should setup the project very conveniently, and a
git plugin can simplify source code management. The core developers use IntelliJ
IDEA or Eclipse.

It's very useful and recommended to locate the sources for your JDK and attach
them in the IDE, so you can navigate into the javac libraries when needed.

We generally follow the Sun style guide, and limit lines to 100 cols.

# Write a checker

Let's say you want to write a checker and contribute it to the project. Here are
the steps you should follow.

## Set up your work environment

Follow the Getting Started steps above. Then create a branch in which to do your
work.

You should also add the check to the [Issue Tracker][issue-tracker]
(if it is not already there), comment that you are taking the issue over,
and change the status from "New" to "Accepted".

### IDE Configuration

#### Eclipse

We recommend installing the `m2e-apt` plugin and enabling `preferences > maven >
annotation processing` so [AutoValue][autovalue] is run automatically.

#### Intellij

We provide an intellij code style that conforms to 
[Google Java Style][google-style]:
https://github.com/google/error-prone/blob/master/.idea/GoogleStyle.xml

To install it, copy `GoogleStyle.xml` to your intellij configuration directory
(e.g. `~/.IntelliJIdea14/config/codestyles/` for intellij 14 on linux, or
`~/Library/Preferences/IdeaIC14/codestyles/` for intellij 14 on mac). Then
select 'Google Style' in `Settings > Editor > Code Style`.

## Write your checker

Checkers are in the package com.google.errorprone.bugpatterns. You can follow
the tutorial on writing a check [here][writing-a-check]. Don't forget to
write tests!

## Request a code review

A member of error-prone team needs to review your code and merge it into the
mainline project. We use [github](https://github.com/google/error-prone/pulls)
for code reviews.

[issue-tracker]: https://github.com/google/error-prone/issues
[autovalue]: https://github.com/google/auto/tree/master/value
[google-style]: https://google.github.io/styleguide/javaguide.html
[writing-a-check]: https://github.com/google/error-prone/wiki/Writing-a-check