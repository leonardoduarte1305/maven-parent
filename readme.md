# Maven Parent

## Goal

Provide a `pom.xml` with [intelligent defaults](https://en.wikipedia.org/wiki/Convention_over_configuration) for projects. The default maven configuration is still based on [Java 5](https://maven.apache.org/plugins/maven-compiler-plugin/compile-mojo.html#source) and platform dependent builds.

It is not a goal to force anyone to use this `pom.xml`.


## KeyFacts
* java 11 (lts version)
* [junit jupiter](http://junit.org/junit5/docs/current/user-guide/) (junit 4 is [still supported](http://junit.org/junit5/docs/current/user-guide/#running-tests-junit-platform-runner) for legacy projects)
* fixed versions for common plugins (org.apache.maven.plugins & org.codehaus.mojo)
* default configurations, e.g. java 11, junit 5 provider, display compiler warnings, ...
* added maven flag **maven.deploy.only** to skip repackaging for **mvn deploy** if already packaged
* configured maven reports and site (see [versions-rules.xml](src/main/versions/versions-rules.xml) for ignored versions)
* this parent contains very liberal checkstyle rules from Hermma team: [checkstyle](src/main/checkstyle)


## Usage

Just add the following code to your `pom.xml` and insert the version:

```
<parent>
	<groupId>dev.leoduarte</groupId>
	<artifactId>maven-parent</artifactId>
	<version>${version}</version>
	<relativePath />
</parent>
```

For deployment and release build add the following lines to your `pom.xml` and replace the variables with your values:
```
<scm>
	<connection>scm:git:https://bitbucket.dev.leonardo/scm/{PROJECT}/{REPO}.git</connection>
	<url>https://bitbucket.dev.leonardo/projects/{PROJECT}/repos/{REPO}/browse</url>
</scm>
<distributionManagement>
	<repository>
		<id>{PROJECT}-maven-releases</id>
		<url>https://repo.dev.leonardo/artifactory/{PROJECT}-maven-releases</url>
	</repository>
	<snapshotRepository>
		<id>{PROJECT}-maven-snapshots</id>
		<url>https://repo.dev.leonardo/artifactory/{PROJECT}-maven-snapshots</url>
	</snapshotRepository>
</distributionManagement>
```

For a proper Maven site add to your `pom.xml`:
```
<url>https://teamweb.leonardo/x/{SHORTLINK}</url>
<issueManagement>
	<system>JIRA</system>
	<url>https://agile.leonardo/projects/{PROJECT}</url>
</issueManagement>
<distributionManagement>
	<site>
		<id>{PROJECT}-maven-site</id>
		<url>file:///${user.home}/.m2/site/</url>
	</site>
</distributionManagement>
```

## Key Agreements

* All changes should undergo `create branch -> pull request -> reviews and approvals -> document changes -> merge`
* **Platform Architechture Team** should be at lease a reviewer of this change(s).

## Profiles

### release

Release is active by default:

* add source jar (no javadoc, no test-sources)
* test instrumentation with jacoco for recording test coverage
* use [tidy-maven-plugin](http://www.mojohaus.org/tidy-maven-plugin/) to check pom structure
* enable minimal code style rules (see [checkstyle](src/main/checkstyle))
* enforce correct java version (accepts all versions equal or above property **maven.compiler.source**)

### windows | linux | osx

Sets the property `project.build.os` to the current os.

### notest

Skips all tests by setting `skipTests` and `maven.test.skip.exec` to `true`.

## Version updates

Display dependency updates:
```
mvn versions:display-property-updates -U
```

Update dependencies:
```
mvn versions:update-properties -DgenerateBackupPoms=false -U
```

Or build site:
```
mvn site -U
firefox target/site/index.html
```

## Dependency-Track

### Goal

Provide a profile consisting of configuration and plugins for creation of SBOM from all used dependencies and upload this BOM to Dependency-Track. By default, the SBOM creation will happen in the ``package`` phase of the build lifecycle.

### Licenses

* CycloneDX Maven plugin: [Apache License 2.0](https://github.com/CycloneDX/cyclonedx-maven-plugin/blob/master/LICENSE)
* dependency-track-maven-plugin: [Apache License 2.0](https://github.com/pmckeown/dependency-track-maven-plugin/blob/master/LICENSE)

### Setup without using maven-parent

The dependency-track profile can also be found in the ``pom.deptrack.include.xml`` file. You can add this profile to your local ``pom.xml`` of your project.

### Configuration

Configuration only needs the Dependency-Track API Key and your project name within Dependency-Track. Those two variables can be set in the properties of the ``pom.xml`` of your project or in your ``settings.xml``. An environment variable can be used to inject the API key e.g. from a Jenkins secret.

```xml
<properties>
	<dependencyTrackApiKey>${env.dependencyTrackApiKey}</dependencyTrackApiKey>
	<dependencyTrack.projectName>${project.name}</dependencyTrack.projectName>
</properties>
```

Optionally, the server URL of the Dependency-Track instance can be changed using the variable as follows:

```xml
<properties>
	<dependencytrack.serverurl>https://dependencytrack.cloudinfra.dev.leonardo</dependencytrack.serverurl>
</properties>
```

### Local Scan

The upload to Dependency-Track happens in the verify phase. So, a local scan can look like this:

```bash
dependencyTrackApiKey=aStrongKey mvn clean verify -P dependency-track
```

## issues Artifact

You can let maven generate an artifact that contains all Jira issues mentioned in the commits since the last tagged version. Just enable the
plugins like this:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.codehaus.gmaven</groupId>
      <artifactId>groovy-maven-plugin</artifactId>
      <executions>
        <execution>
          <id>generate-changelog</id>
          <phase>prepare-package</phase>
        </execution>
      </executions>
    </plugin>
    <plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>build-helper-maven-plugin</artifactId>
      <executions>
        <execution>
          <id>attach-changelog</id>
          <phase>package</phase>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

The Jira issues are matched by the regex in the property `changelog.issue.pattern`. The default value
is `((?<!([A-Za-z]{1,10})-?)[A-Z]+-\\d+)`.

The resulting artifact has the category `issues` and the extension `txt`.