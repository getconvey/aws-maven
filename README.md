AWS Maven is a [Maven Wagon][wagon] for [Amazon S3][s3].

Convey's fork is based on [Main Street Hub's fork](https://github.com/mainstreethub/aws-maven) and was further customized for use for all Convey projects, most specifically allowing this extension to be built and used with Java 12+, as well as being easily released to Maven Central.

[wagon]: http://maven.apache.org/wagon/
[s3]: http://aws.amazon.com/s3/

# Usage
To publish Maven artifacts to S3 a build extension must be defined in a project's `pom.xml`.

  <project>
    ...
    <build>
      ...
      <extensions>
        ...
        <extension>
          <groupId>com.getconvey.oss</groupId>
          <artifactId>aws-maven</artifactId>
          <version>5.0.0.RELEASE.MSH.1.CONVEY.3</version>
        </extension>
        ...
      </extensions>
      ...
    </build>
    ...
  </project>

Once the build extension is configured distribution management repositories can be defined in the `pom.xml` with an `s3://` scheme.

  <project>
    ...
    <distributionManagement>
      <repository>
        <id>aws-release</id>
        <name>AWS Release Repository</name>
        <url>s3://distribution.bucket/release</url>
      </repository>
      <snapshotRepository>
        <id>aws-snapshot</id>
        <name>AWS Snapshot Repository</name>
        <url>s3://distribution.bucket/snapshot</url>
      </snapshotRepository>
    </distributionManagement>
    ...
  </project>

Finally the appropriate AWS credentials need to be provided through the credentials chain.

# Releasing

## Initial setup
In order to release this POM to Maven Central you'll need to do some setup
the very first time.

### Find the Convey oss.sonatype.org account
We'll be releasing to Maven Central via oss.sonatype.org using the 
sonatype@getconvey.com account.

### Create a GPG signing key
One of the requirements for releasing binaries to oss.sonatype.org is that
they are signed with a GPG key.  This ensures that it's more difficult for
a hacker to change binaries should maven central or sonatype ever be
compromised. *You should use your own email address*. Instructions for how 
to create a GPG key and to publish it can be found 
[here](http://central.sonatype.org/pages/working-with-pgp-signatures.html).

### Create a $HOME/.m2/settings.xml file.
Now that you have a oss.sonatype.org account, you'll need to tell maven
your username and password (so that it can upload files).  We'll do this
using a `settings.xml` file in your `$HOME/.m2` directory.  Its contents
should look something like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/Settings/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/Settings/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>oss.sonatype.org</id>
      <username>sonatype@getconvey.com</username>
      <password>YOUR PASSWORD HERE</password>
    </server>
  </servers>
</settings>
```

Don't put these credentials or this file anywhere other than your laptop!

## Creating a new release

To release this POM to the Maven Central Profile you'll need to activate
the `sonatype` profile and deploy the POM as you would otherwise normally
do.

```bash
# Activate and enable the sonatype profile, prepare and then perform the release.
mvn -P sonatype release:prepare release:perform
```

If the above command fails with "gpg: problem with the agent: Inappropriate
ioctl for device", run the following: `export GPG_TTY=$(tty)`.

Now at this point your pom should have been uploaded to oss.sonatype.org and
is waiting to be released. Log in to the Sonatype console, find the POM's bundle
in the staging repository, close it and then release it.
