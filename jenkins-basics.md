## General
As a DevOps engineer, you wouldn't want to make code changes to have the code built and deployed. Use declarative pipelines instead of click-ops.

You can install the Blue Ocean plugin for a modern look and feel.

## GitHub
Generate a github access token and add it to Jenkins with the credentials manager as Username and Password secret. The username can be whatever but the password has to be the token.

## Scripted pipelines
- Configure through code
- Changes tracked through Source Control
- Pipelines can be parameterized and reused

A Stage is a specific section of the build pipeline.
Stages contains steps that represent the instructions within a stage.

## Agent

## Jenkins in Docker
Run `docker run -d --name jenkins -p 8080:8080 jenkins/jenkins:lts`.
Notice that you'll have a hard time install additional binaries on the Jenkins server due to permission restrictions.

It's also possible to map the agent port 5000, e.g. `-p 50000:50000`.
For easy debugging, map a volume to the jenkins container.

```sh
# mac
docker run -d --name jenkins -p 8080:8080 \
-v /Users/nicklasmillard/docker/volumes/jenkins:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/local/bin/docker:/usr/bin/docker \
jenkins/jenkins:lts
```

```sh
# win
docker run -d --name jenkins -p 8090:8080 \
-v F:\\docker-volumes\\jenkins:/var/jenkins_home \
-v //var/run/docker.sock:/var/run/docker.sock \
-v //usr/local/bin/docker:/usr/bin/docker \
-e DOCKER_HOST=tcp://host.docker.internal:2375 \
jenkins/jenkins:lts
```

### Get admin password
For first time set up you'll need the admin password.
Exec into the running container and cat the password file.
1. `docker exec -it jenkins bash`
2. `cat /var/jenkins_home/secrets/initialAdminPassword`

## Pipeline syntax
```jenkins
pipeline { // complete script from begin to end
    agent any // required block
    stages {  // the different stages of the run
        stage('Hello, World') {
            steps {
                echo "Hello, World"
            }
        }
    }
}
```

## Use another Java SDK version
You can set the java version a pipeline should use by first installing that version onto the Jenkins server, and then refer to it in the pipeline itself.

In Global Tool Configuration, add a new JDK and give it a sensible name, such as "adoptopenjdk17", and use the "Extract zip" installer. The download URL can be e.g. `https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.6%2B10/OpenJDK17U-jdk_x64_linux_hotspot_17.0.6_10.tar.gz`, and sub directory extracted to will then be `jdk-17.0.6+10`.

The pipeline then refers to this tool like this:
```gradle
pipeline {
  agent any
  tools {
    jdk 'adoptopenjdk-17'
  }
}
```

## Fixing Permission Denied
If you use shell scripts in the pipeline, you might run into issues. This can e.g. be fixed by granting execute on all files within the workspace.

```gradle
pipeline {
  agent any
  tools {
    jdk 'adoptopenjdk-17'
  }
  stages {
    stage('error') {
      steps {
        sh "chmod +x -R ${env.WORKSPACE}" // <- this
        sh './build.sh'
      }
    }

  }
}
```

## Reading environment variables
Install the Jenkins Pipeline Utility Steps plugin. This provides lots of functionality to read external files, that is, files external to the source code, and even `pom.xml` properties, which can be quite useful.

However, it's not recommended to read the `pom.xml` using `readMavenPom`.

With maven projects, you can get properties from the `pom.xml` by running the `mvn help:evaluate` goal. 

Say you need the build numer as an environment variable, then you can do something like below.

```gradle
environment {
  BUILD = sh script: './mvnw help:evaluate -Dexpression=build.version -q -DforceStdout', returnStdout: true
}
```

## Using credentials
You can set credentials on Jenkins in the "Manage Jenkins -> Manage Credentials" section.
Say you setup a credentials username and password set with the credentials ID=`artifactory-registry`
Using this in the pipeline would look like this:

```gradle
pipeline {
  agent any
  tools {
    jdk 'adoptopenjdk-17'
  }
  environment {
    ARTIFACTORY_CREDS = credentials('artifactory-registry')
    DOCKER_REGISTRY_SERVER = "nmillarddemo.jfrog.io"
    DOCKER_USERNAME = "$ARTIFACTORY_CREDS_USR"
    DOCKER_PASSWORD = "$ARTIFACTORY_CREDS_PSW"
  }
}
```

Notice how the username and password are referenced by prepending `_USR` and `_PSW` to the credentials variable.

Read more about [credentials here](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#usernames-and-passwords).


## Running SonarQube
To use a quality gate step like sonarqube, you need to install the SonarQube for Jenkins plugin.

In a Spring based project, you'll then need the sonarqube maven plugin. There's really not much setup to do besides knowing that the sonarqube plugin looks for a few variables:
- sonar.host.url: where the sonarqube server is located (e.g. http://localhost:9000 or some docker address like http://172.17.0.7:9000)
- sonar.projectKey: the project id in sonarqube
- sonar.login: a secret token, typically set in your `~/.m2/setting.xml` or in the environment.

```gradle
stage('Quality check') {
    steps {
        withSonarQubeEnv("SonarQube") {
            sh "./quality-check.sh $SONAR_URL"
        }
    }
}
stage ("Wait for quality check") {
    steps {
        timeout(time: 1, unit: 'HOURS') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```


```xml
<!--In the pom.xml-->
<properties>
    <java.version>17</java.version>
    <sonar.projectKey>${project.artifactId}</sonar.projectKey>
    <sonar.host.url>http://localhost:9000</sonar.host.url>
    <sonar.login>${env.SONAR_TOKEN}</sonar.login>
</properties>

<plugin>
    <groupId>org.sonarsource.scanner.maven</groupId>
    <artifactId>sonar-maven-plugin</artifactId>
    <version>3.9.1.2184</version>
</plugin>
```

Then, in the Jenkins dashboard, go to Manage Jenkins -> Configure System and set the SonarQube Servers name and Server URL with an authentication token.

On the SonarQube side, you need to set up a webhook that sends the run report back to Jenkins. The webhook url is: `<your_jenkins_instance>:port/sonarqube-webhook/`