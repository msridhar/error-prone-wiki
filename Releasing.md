# Keeping the build green
We run a [continuous build](https://travis-ci.org/google/error-prone).

[![Build Status](https://travis-ci.org/google/error-prone.svg?branch=master)](https://travis-ci.org/google/error-prone)

If the build is broken, then we can't do a release. Keep the build green!

# Release to Maven Central

Prerequisite: we sign the released artifacts with GnuPG, so you'll need gpg on your path, and also a default key installed (run `gpg --gen-key`). You also need to upload your ASCII-armored public key to a common repo like http://pgp.mit.edu. You also need to have permission with sonatype, which requires following instructions like steps 2 & 3 of the  here: https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide. (more info on sonatype and google: http://code.google.com/p/google-maven-repository/wiki/GuidelinesForGoogleMavenProjects).

Here is an example ticket to grant publish rights: 
https://issues.sonatype.org/browse/OSSRH-7782

You will need to set up a settings.xml file for maven in your ~/.m2 directory. if you don't have that file already, get the template at http://maven.apache.org/settings.html#Quick_Overview. Then add various server options: https://issues.sonatype.org/browse/OSSRH-3462?page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel&focusedCommentId=162066#comment-162066

Currently we build the output JARs with JDK7. You can check what JDK version Maven is using by typing:
<pre>
$ mvn --version
</pre>

Look at the pom.xml file to see what version is being snapshotted, eg. 0.6-SNAPSHOT means we want to release 0.6

<pre>
$ mvn release:prepare -Dtag=v0.x
# accept the default suggestions
$ mvn release:perform -Darguments=-Dgpg.passphrase=[XXX] 
</pre>

Now the release is "staged" at Sonatype.
Go to https://oss.sonatype.org and login, then take a look at the staged artifacts. Check that the right classes appear in the right jars, and the sizes are reasonable.

Then, "close" and then "release" the staging repository to make the release public. More instructions in case you run into trouble:
https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide#SonatypeOSSMavenRepositoryUsageGuide-8.ReleaseIt

If you get a key error, you might need to upload this output to http://pgp.mit.edu:
`gpg -a --export`

*Update installation instructions*

Update the version of the error-prone-core dependency in the example pom.xml file here:
https://code.google.com/p/error-prone/wiki/InstallingErrorProne#Maven

*Update submodule versions*

The release:update-versions plugin only updates the versions of submodules that were active during the release, so manually bump the versions of the jdk6 and jdk8 submodules.

Example: https://code.google.com/p/error-prone/source/detail?r=cc09335b3a7f300cd6a41d997c76e78d44331e6a

Check if http://jira.codehaus.org/browse/MNG-624 is fixed yet.

# Update IDEA plugin

(note: we'd like the plugin to be owned by JetBrains at some point, so they'd take responsibility for keeping it working. Alex has a thread with Sergey Simonchek to follow up on.)

1. Sync your working copy to a tagged release:
<pre>
$ git checkout v1.0.8
</pre>

2. Edit idea-plugin/META-INF/plugin.xml : version tag to match the synced VCS tag.
3. Make sure your Compiler -> Java Compiler has Project bytecode version 1.6 (makes -target 1.6)
4. In IDEA, select the idea-plugin module and use menus: Build -> Prepare plugin module 'idea-plugin' for Deployment. It builds and points you to the JAR file.
5. Verify that the resulting JAR will work when IDEA is run on JRE 1.6:
<pre>
$ javap -classpath ~/Projects/error-prone/out/production/idea-plugin/ -verbose com.google.errorprone.intellij.ErrorProneIdeaCompiler | grep version:
  minor version: 0
  major version: 50
</pre>
6. Go to http://plugins.jetbrains.com/plugin/edit?pluginId=7349 (logged in as Alex?) and upload the new JAR.

# Update documentation

The [bug pattern documentation](http://errorprone.info/bugpatterns) and [javadoc](http://errorprone.info/api/latest) is automatically updated when the continuous build passes by [this script](https://github.com/google/error-prone/blob/master/util/generate-latest-docs.sh).