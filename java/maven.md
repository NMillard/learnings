# Maven

At the very core, maven is "just" a build tool.

## Compiled code

All compiled code ends up in the `target` folder by default.

## Goals

The default goals are just plugins configured during the maven install.

## Local repository

Dependencies are downloaded and stored in the local maven repo, typically located at `~/.m2/repository`.

## Effective pom

The effective pom is the pom that is actually applied and used when building. When opening the project poms, you're not seeing the effective pom.

To see the effective pom, that is, the pom that takes in the different profile settings, run `mvn help:effective-pom`.

This is quite helpful to debug missing values, etc.

## Versioning

SemVer is a popular choice for versioning an libraries since the end consumer can then identify changes and potential compatability issues. The version is bumped when there's a new release.

However, for applications, SemVer isn't a suitable choice. Using something else such as `year-month-releasenumber` might be better.

```xml
<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<imageName>${project.groupId}-${project.artifactId}:${maven.build.timestamp}.${git.commit.id.abbrev}</imageName>
					<image>
						<builder>dashaun/builder:tiny</builder>
					</image>
				</configuration>
			</plugin>

			<plugin>
				<groupId>io.github.git-commit-id</groupId>
				<artifactId>git-commit-id-maven-plugin</artifactId>
				<version>8.0.2</version>
			</plugin>
		</plugins>
	</build>
```

## Multi module

Multi module projects allow you to separate concerns into different projects, much like how it's commonly done .NET where project dependencies are added using `ProjectReference` in the `.csproj` file.

You can do something similar in Java using multi modules.

The important thing is then not to package the projects from the individually, but rather by using the "pom" project, or, reactor, to build the related projects.

You'll need a reactor which is the pom at the root level.
```text
- pom.xml
-- domain
   - pom.xml
-- springapp
   - pom.xml
```

Within the reactor pom, you need to define which modules should be included:

```xml
<modules>
	<module>domain</module>
	<module>springapp</module>
</modules>
```

## BOM

If you're creating e.g. a spring boot app and want to use your parent pom, but at the same time want to leverage all the dependencies you get from the spring boot parent, then you'll need to utilize a BOM (bill of materials) dependency.

In your parent project, add a dependency management section with the bom dependency and set type=pom and scope=import.

```xml
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>${spring-boot.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
```