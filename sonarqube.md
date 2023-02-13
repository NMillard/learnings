# SonarQube

## Running locally using Docker
Run the following command to setup a local sonarqube instance:

```
docker run -d --name sonarqube \
-e SONAR_ES_BOOTSTRAP_CHECKS_DISABLED=true \
-p 9000:9000 \
sonarqube:latest
```

## Setting up Spring project
In the pom, add the sonar maven plugin, and set the required properties.

```xml
<!--The required plugin-->
<plugin>
    <groupId>org.sonarsource.scanner.maven</groupId>
    <artifactId>sonar-maven-plugin</artifactId>
    <version>3.9.1.2184</version>
</plugin>

<!--The required properties-->
<properties>
    <sonar.projectKey>${project.artifactId}</sonar.projectKey>
    <sonar.host.url>http://localhost:9000</sonar.host.url>
    <sonar.login>${env.SONAR_TOKEN}</sonar.login>
</properties>
```

Notice the `${env.SONAR_TOKEN}`, this is useful when running on a build server, but less so when wanting to execute the `sonar:sonar` locally.

You can use the `.m2/settings.xml` to override the `sonar.login` locally.

```xml
<!--
    Found in your ~/.m2 folder 
    The settings.xml is not auto generated, so you might have to creatae it yourself.
-->

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">

    <profiles>
        <profile>
            <id>local</id>
            <properties>
                <sonar.login>39d523ba05dca072a6d06b2f392775fb3e69aa1a</sonar.login>
            </properties>
        </profile>
    </profiles>
</settings>
```