## Keeping the build green

We run a [continuous build](https://travis-ci.org/google/error-prone).

[![Build Status](https://travis-ci.org/google/error-prone.svg?branch=master)](https://travis-ci.org/google/error-prone)

If the build is broken, then we can't do a release. Keep the build green!

## Prerequisites:

### GnuPG

We sign the released artifacts with GnuPG, so you'll need gpg on your path, and also a default key installed (run `gpg --gen-key`). You also need to upload your ASCII-armored public key to a common repo like http://keys.gnupg.net.

If you want a chance to remember your gpg key before starting `mvn release`, you can use the following incantation. (Courtesy of [SO](http://stackoverflow.com/a/11484411).)

```
$ gpg -o /dev/null --local-user "${USERNAME}@google.com" -as <(echo 1234) && echo "The correct passphrase was entered for this key"
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
      <id>sonatype-nexus-snapshots</id>
      <username>username</username>
      <password>password</password>
    </server>
    <server>
      <id>sonatype-nexus-staging</id>
      <username>username</username>
      <password>password</password>
    </server>
    <server>
      <id>google-releases</id>
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

### Build and Stage the Release

First, supply your SSH passphrase to avoid getting asked while running Maven commands: 

```shell
ssh-add
```

Sync to the commit from which you want to build the release (if necessary).

```shell
git checkout -b release-branch <commit-hash>
```

Look at the pom.xml file to see what version is being snapshotted, e.g. 2.0.1-SNAPSHOT means we want to release 2.0.1.

As long as we're using the JUnit 13 SNAPSHOT, pass `-DignoreSnapshots=true` to `release:prepare`.

```shell
mvn release:prepare -Dtag=v2.X.X -DignoreSnapshots=true
# accept the default suggestions
mvn release:perform
```

Now the release is "staged" at Sonatype.

#### Troubleshooting

If you get any error messages, make sure you have your ssh key and gpg key set up properly.

If the release build is interrupted by an error, you will need to clean up the changes that maven already made to set up the release before trying again.

```shell
git add -A
git reset --hard
```

### Release to Sonatype

Go to https://oss.sonatype.org and login, then take a look at the [staged artifacts](https://oss.sonatype.org/#stagingRepositories). Check that the right classes appear in the right jars, and the sizes are reasonable.

Then, "close" and then "release" the staging repository to make the release public. ([more instructions](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide#SonatypeOSSMavenRepositoryUsageGuide-8.ReleaseIt
) in case you run into trouble.)

#### Troubleshooting

If you get a key error, you might need to upload this output to http://keys.gnupg.net: `gpg -a --export` (Allow a few minutes to propagate). If it still doesn't work, try uploading the key to another keyserver. You can search your keyserver for the hash in the Sonatype error message to see if it uploaded yet.

### Update installation instructions

Update the Error Prone version in the installation instructions:
https://github.com/google/error-prone/blob/gh-pages/docs/installation.md

> Tip: Search the page for the old version and replace with the new version.

### Update examples

Update the version of the error_prone_core dependency in the various examples to the new version.  Suppose the previous version was 2.0.0, the new version is 2.0.1, and the new snapshot version is 2.0.2-SNAPSHOT.

```shell
OLD="2\.0\.0"; NEW="2\.0\.0"
sed -i "s/$OLD/$NEW/g" examples/ant/build.xml examples/maven/pom.xml examples/plugin/bazel/WORKSPACE examples/plugin/gradle/build.gradle examples/plugin/gradle/sample_plugin/build.gradle examples/plugin/maven/hello/pom.xml examples/plugin/maven/sample_plugin/pom.xml examples/refaster/README.md
OLD_SNAPSHOT="2\.0\.1-SNAPSHOT"; NEW_SNAPSHOT="2\.0\.2-SNAPSHOT"
sed -i "s/$OLD_SNAPSHOT/$NEW_SNAPSHOT/g" examples/ant/build.xml examples/maven/pom.xml examples/plugin/bazel/WORKSPACE examples/plugin/gradle/build.gradle examples/plugin/gradle/sample_plugin/build.gradle examples/plugin/maven/hello/pom.xml examples/plugin/maven/sample_plugin/pom.xml examples/refaster/README.md
```

Sample commit: https://github.com/google/error-prone/commit/739c6c155827209fcbee7f056381415e31094768

> TIP: Do this in the Google-internal version, not Github, to avoid two-way syncing issues.

## Write release notes

Write release notes and update the git tag with those release notes.  Look at the commits since the last release to help you write them. Use this command: `git log v2.0.9..HEAD`

TODO(eaftan): Script writing the release notes.

> Tip: Extract interesting relnotes with `git log v2.2.0..HEAD | grep RELNOTES | grep -vi "n/a\|none"`

> Tip: Find issues fixed with `git log v2.2.0..HEAD | grep -o "#[0-9]\+" | sort | tr '\n' ',' | sed 's/,/, /'g`

Example release notes: https://github.com/google/error-prone/releases/tag/v2.3.0

## Tell the world

Email `error-prone-announce@googlegroups.com` and `error-prone-discuss@googlegroups.com` and point them to the release notes.  

## Optional

### Update IDEA plugin

TODO(cushon): update the plugin, and fix these instructions

An [earlier version](https://github.com/google/error-prone/wiki/Releasing/229bf30bb8da26dc745379dd313b197203050a7a) of this page had instructions that were wildly out of date. The plugin hasn't been updated in a while: [error-prone/issues/374](https://github.com/google/error-prone/issues/374)