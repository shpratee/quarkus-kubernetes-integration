# quarkus-kubernetes-integration
Quarkus and kubernetes integration

Quarkus provides extensions for building and optionally pushing container images. At the time of writing, the following container build strategies are supported:

1. Google Jib
Jib builds Docker and OCI container images for your Java applications without a Docker daemon (Dockerless). This makes it perfect for building Docker images when running the process inside a container because you avoid the hassle of the Docker-in-Docker (DinD) process.

Using jib with Quarkus caches all dependencies in a different layer than the actual application, making rebuilds fast and small

a. To build and push a container image using Jib, add jib extension
```
./mvnw quarkus:add-extensions -Dextensions="quarkus-container-image-jib”
```
b. Set the properties in application.properties (or system properties or environment variables)                   
```
quarkus.container-image.group=com.demo.api.developers
quarkus.container-image.registry=<registry co-ordinates> --> docker.io is going to be used by default if not provided.
quarkus.container-image.username=<username>
quarkus.container-image.password=<password>
```

c. Control the build and push behaviour using below properties:
```
quarkus.container-image.build=true
quarkus.container-image.push=true --> mark it false if you do not need to push the image. Though Docker is not required for jib, Docker daemon is still going to be needed in this case.
```

d. Package the build in JVM mode
```
./mvnw clean package -Dquarkus.container-image.build=true -Dquarkus.container-image.push=false
```

e. Package the build in native mode
```
./mvnw clean package -Pnative -Dquarkus.container-image.build=true -Dquarkus.container-image.push=false
```

2. Docker
Using the Docker strategy builds container images using the docker binary, which is installed locally and by default uses Dockerfiles located under src/main/docker to build the images

To build and push a container image using docker
```
./mvnw quarkus:add-extensions -Dextensions="quarkus-container-image-docker”
```

3. Source-to-Image (S2I)
The Source-to-Image (S2I) strategy uses s2i binary builds to perform container builds inside an OpenShift cluster. S2I builds require creating a BuildConfig and two ImageStream resources

To build and push a container image using docker
```
./mvnw quarkus:add-extensions -Dextensions="quarkus-container-image-s2i”
```

Generating Kubernetes Resources

1. To enable the generation of Kubernetes resources, you need to register the quarkus-kubernetes extension:
```
./mvnw quarkus:add-extension -Dextensions="quarkus-kubernetes"
```

2. Package the build to see two new files being generated in target/kubernetes directory - named "kubernetes.json" and "kubernetes.yml", each containing object definitions of kind "Deployment" and "Service"
```
./mvnw clean package
```

