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