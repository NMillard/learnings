# Openshift

## OC maven plugin
You can build and push images as well as creating kubernetes deployment and service yaml files using the `oc` plugin.
This is quite useful in CI/CD pipelines, where, all you need is a bit of configuration and invoking a series of `oc` maven goals.

```xml
<!-- in pom.xml -->
<properties>
    <jkube.build.strategy>jib</jkube.build.strategy> <!-- use jib plugin to build the image -->
    <jkube.generator.name>${image-name}:${build.version}</jkube.generator.name>
    <jkube.docker.registry>${env.DOCKER_SERVER}</jkube.docker.registry>
    <jkube.docker.username>${env.DOCKER_USENAME}</jkube.docker.username>
    <jkube.docker.password>${env.DOCKER_PASSWORD}</jkube.docker.password>
</properties>

<plugins>
    <plugin>
        <groupId>com.google.cloud.tools</groupId>
        <artifactId>jib-maven-plugin</artifactId>
        <version>3.3.1</version>
    </plugin>

    <plugin>
        <groupId>org.eclipse.jkube</groupId>
        <artifactId>openshift-maven-plugin</artifactId>
        <version>1.10.1</version>
    </plugin>
</plugins>
```

### Commands
Some of the useful maven oc plugin commands to run are:
- `oc:build`: build the image tar files. This creates a folder in target named after the image name and places the tar there.
- `oc:push`: pushes the built image to the registry.
- `oc:resource`: creates the deployment and service files in the `target/classes/META-INF/jkube/openshift.yml`.
- `oc:apply`: applies the deployment yaml file to the kubernetes cluster.

Some of these commands need a bit more details before they work, as outlined in the preconditions section below.

### Preconditions
Make sure to run the `oc login` CLI on the machine that needs to apply the deployment specification. This stores the login information in `~/.kube/config.json`, which contains the authentication token along with some cluster information.

Or, they can be set programmatically in the `pom.xml`.
```xml
<properties>
    <kubernetes.master>${env.OPENSHIFT_URL}</kubernetes.master>
    <kubernetes.auth.token>${env.OPENSHIFT_NAMESPACE}</kubernetes.auth.token>
    <kubernetes.namespace>${env.OPENSHIFT_TOKEN}</kubernetes.namespace>
</properties>    
```

Also, if you're using a private docker registry, then the Openshift instance needs to have a special docker registry secret created.

### Creating docker registry secret on openshift
Openshift needs to have the docker registry login information in order to pull images.

```
oc create secret docker-registry name_of_the_secret \
--docker-server=your_private_registry \
--docker-username=your_username \
--docker-password=your_token_or_password \
--docker-email=an_email
```

Then, the secret needs to be connected to your service account to pull images used for pods.
`oc secrets link default <pull_secret_name> --for=pull`

## Links
- https://github.com/fabric8io/docker-maven-plugin/blob/master/src/main/asciidoc/inc/_authentication.adoc
- https://github.com/eclipse/jkube/tree/master/quickstarts/maven/spring-boot/
- https://github.com/eclipse/jkube/tree/master/quickstarts/maven/spring-boot-with-jib/
- https://docs.openshift.com/container-platform/4.5/openshift_images/managing_images/using-image-pull-secrets.html#images-allow-pods-to-reference-images-from-secure-registries_using-image-pull-secrets
- https://developers.redhat.com/articles/2022/03/01/package-and-run-your-java-maven-application-openshift-seconds#deploy_the_application_to_openshift
- https://docs.openshift.com/container-platform/4.9/authentication/using-service-accounts-in-applications.html