## Keeping the build green

We run a [continuous build](https://travis-ci.org/google/error-prone).

[![Build Status](https://travis-ci.org/google/error-prone.svg?branch=master)](https://travis-ci.org/google/error-prone)

If the build is broken, then we can't do a release. Keep the build green!

## Prerequisites:

### GnuPG

We sign the released artifacts with GnuPG, so you'll need gpg on your path, and also a default key installed (run `gpg --gen-key`). You also need to upload your ASCII-armored public key to a common repo like http://pgp.mit.edu.

If you want a chance to remember your gpg key before starting `mvn release`, you can use the following incantation. (Courtesy of [SO](http://stackoverflow.com/a/11484411).)

```
$ echo "1234" | gpg --no-use-agent -o /dev/null --local-user "Your Name <you@example.com>" -as - && echo "The correct passphrase was entered for this key"
```

### Sonatype

First, you will need a Sonatype account.  Create one at https://issues.sonatype.org.  **Use a unique, throwaway password.** You will need to save it in cleartext in a later step.

Second, you will need permission to upload to the Error Prone Sonatype repository.  Create a JIRA ticket requesting this permission. It should look just like this example: https://issues.sonatype.org/browse/OSSRH-20124

More info on Sonatype and Google: http://code.google.com/p/google-maven-repository/wiki/GuidelinesForGoogleMavenProjects

### Maven settings

Create a settings.xml file for maven in your ~/.m2 directory.  The file should look like this, where username and password are your Sonatype username and password:

```xml
<settings>
<servers>
  <server>
    <id>google-releases</id>
    <username>username</username>
    <password>password</password>
  </server>
  <server>
    <id>sonatype-nexus-snapshots</id>
    <username>username</username>
    <password>password</password>
  </server>
  <server>
    <id>sonatype-nexus-staging</id>
    <username>username</username>
    <password>password</password>
   </server>
</servers>
</settings>
```

Debugging notes: [related discussion](https://issues.sonatype.org/browse/OSSRH-3462?page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel&focusedCommentId=162066#comment-162066)).
 
### JDK8

Currently we build the output JARs with JDK8. You can check what JDK version Maven is using with `mvn --version`.

## Releasing

Look at the pom.xml file to see what version is being snapshotted, e.g. 2.0.1-SNAPSHOT means we want to release 2.0.1.

    $ mvn release:prepare -Dtag=v2.X.X
    # accept the default suggestions
    $ mvn release:perform

Now the release is "staged" at Sonatype.

Go to https://oss.sonatype.org and login, then take a look at the staged artifacts. Check that the right classes appear in the right jars, and the sizes are reasonable.

Then, "close" and then "release" the staging repository to make the release public. ([more instructions](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide#SonatypeOSSMavenRepositoryUsageGuide-8.ReleaseIt
) in case you run into trouble.)

If you get a key error, you might need to upload this output to http://pgp.mit.edu: `gpg -a --export`

### Update installation instructions

Update the version of the error_prone_core dependency in the example pom.xml file here:
https://github.com/google/error-prone/blob/gh-pages/docs/installation.md

### Update examples

Update the version of the error_prone_core dependency in the various examples to the new version.  Do
this for:

* `examples/ant/ant_fork/build.xml`
* `examples/gradle/build.gradle`
* `examples/maven/pom.xml`
* `examples/plugin/bazel/WORKSPACE`
* `examples/plugin/gradle/build.gradle`
* `examples/plugin/gradle/sample_plugin/build.gradle`
* `examples/plugin/maven/hello/pom.xml`
* `examples/plugin/maven/sample_plugin/pom.xml`

Sample commit: https://github.com/google/error-prone/commit/739c6c155827209fcbee7f056381415e31094768

## Update documentation

This part is automatic: the [bug pattern documentation](http://errorprone.info/bugpatterns) and [javadoc](http://errorprone.info/api/latest) is automatically updated when the continuous build passes by [this script](https://github.com/google/error-prone/blob/master/util/generate-latest-docs.sh).

## Tell the world

Write release notes (see [this example](https://groups.google.com/d/msg/error-prone-announce/-f6Cv6jKvig/cFCdhYuC5lwJ)) and email them to `error-prone-announce@googlegroups.com` and `error-prone-discuss@googlegroups.com`.  

Look at the commits since the last release to help you write the release notes. Use this command: `git log v2.0.9..HEAD`

## Optional

### Update IDEA plugin

TODO(cushon): update the plugin, and fix these instructions

An [earlier version](https://github.com/google/error-prone/wiki/Releasing/229bf30bb8da26dc745379dd313b197203050a7a) of this page had instructions that were wildly out of date. The plugin hasn't been updated in a while: [error-prone/issues/374](https://github.com/google/error-prone/issues/374)