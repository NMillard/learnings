## General
As a DevOps "engineer", you wouldn't want to make code changes to have the code built and deployed.

## GitHub
Generate a github access token and add it to Jenkins with the Jenkins Credentials Provider, as Secret Text kind

## Scripted pipelines
- Configure through code
- Changes tracked through Source Control
- Pipelines can be parameterized and reused

A Stage is a specific section of the build pipeline.
Stages contains steps that represent the instructions within a stage.

## Agent

## Jenkins in Docker
Run `docker run -d --name jenkins -p 8080:8080 jenkins/jenkins:lts`.
It's also possible to map the agent port 5000, e.g. `-p 5000:5000`.
For easy debugging, map a volume to the jenkins container.

```sh
docker run -d --name jenkins -p 8080:8080 \
-v /Users/nicklasmillard/docker/volumes/jenkins:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/local/bin/docker:/usr/bin/docker \
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

