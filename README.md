# quarkus-kubernetes-integration
Quarkus and kubernetes integration

## Contanerization support
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
quarkus.container-image.push=true --> mark it false(or do not specify this property at all) if you do not need to push the image. Though Docker is not required for jib, Docker daemon is still going to be needed in this case.
```

d. Package the build in JVM mode
```
./mvnw clean package -Dquarkus.container-image.build=true -Dquarkus.container-image.push=false
```

e. Package the build in native mode
```
./mvnw clean package -Pnative -Dquarkus.native.container-build=true -Dquarkus.container-image.build=true -Dquarkus.container-image.push=false
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

##Generating Kubernetes Resources

1. To enable the generation of Kubernetes resources, you need to register the quarkus-kubernetes extension:
```
./mvnw quarkus:add-extension -Dextensions="quarkus-kubernetes"

You can genrate different resources for different platform using:
quarkus.kubernetes.deployment-target property. The values that are allowed are: kubernetes(default), knative and openshift
```

2. Package the build to see two new files being generated in target/kubernetes directory - named "kubernetes.json" and "kubernetes.yml", each containing object definitions of kind "Deployment" and "Service"
```
./mvnw clean package
```
3. You can change the group and name of the image using below properties:
```
quarkus.container-image.group=com.demo.api.developers
quarkus.application.name=developers-api
```

##Enabling Liveness/Readiness probes
By default, health probes are not generated in the output file, but if the "quarkus-smallrye-health" extension is present, the readiness and liveness probes are generated automatically 
```
./mvnw quarkus:add-extension -Dextensions="quarkus-smallrye-health"
```

##Deploying services on Kubernetes
1. For this we are going to use Minikube

a. Setup VirtualBox - https://www.virtualbox.org/wiki/Downloads
b. Install Kubectl - https://kubernetes.io/docs/tasks/tools/install-kubectl/
c. Setup Minikube - https://kubernetes.io/docs/tasks/tools/install-minikube/
d. Setup Docker - https://docs.docker.com/install/linux/ docker-ce/centos/

2. Start Minikube and configure minikube as your docker environment
```
$ minikube start

$ eval $(minikube docker-env)
```

3. If you are using Jib, then your image would have got created already while packaging, otherwise, you can build th image using Dockerfiles present in docker folder.
```
./mvnw package -Dquarkus.container-image.build=true
```

4. Create Kubernetes objects using "kubectl"
```
$ kubectl apply -f target/kubernetes/kubernetes.yml
serviceaccount/developers-api configured
service/developers-api created
deployment.apps/developers-api created

$ kubectl patch svc developers-api --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'
service/developers-api patched

$ curl -i $(minikube service developers-api --url)/developers/hello
HTTP/1.1 200 OK
content-length: 10
content-type: text/plain;charset=UTF-8

Hello All!
```

##Building and deploying Container image automatically
You an build and deploy your application automatically by setting below property:
```
./mvnw clean package -Dquarkus.kubernetes.deploy=true

This automatically sets quarkus.container-image.push to true

You can use this for native compilation as well by adding -Pnative -Dquarkus.native.container-build=true
```

## Configuration with Kubernetes
You can refer properties from Kubernetes ConfigMaps where in, you can override any property already mentioned in application.properties or you can declare new ones and refer them as environment variables.

1. Create a ConfigMap
```
apiVersion: v1
kind: ConfigMap
metadata:
    name: developers-api-config
data:
    developer.greeting.message: Kubernetes learning!

$ kubectl create -f developers-configmap.yaml
```

2. Customize the application.properties to refer developer.greeting.message property from ConfigMap
```
quarkus.kubernetes.env-vars.developer-greeting-message.value=developer.greeting.message --> Kubernetes environment variables to be referred to override the property 
quarkus.kubernetes.env-vars.developer-greeting-message.configmap=developers-api-config --> Load ConfigMap and refer the mentioned property in it
```

3. Package the build
4. Apply the changes in Kubernetes Deployment and Service

## Refer properties loaded from external files or properties in ConfigMap
1. Apply "quarkus-kubernetes-config"
```
./mvnw quarkus:add-extension -Dextensions="quarkus-kubernetes-config"
```

2. Create Kubernetes ConfigMap using the properties file
```
$ kubectl create configmap my-extra-config --from-file=src/main/resources/application.properties
```

3. Customize application.properties to use kuberneter-config and ConfigMap
```
quarkus.kubernetes-config.enabled=true
quarkus.kubernetes-config.config-maps=my-extra-config --> There can be more ConfigMaps which can be referred here as ',' separated list.
```
