# Ballerina Code to Cloud Specification

## Introduction

The main objective of this specification is to simplify the developer experience of developing and deploying
 ballerina code in the cloud. C2C enables developers who have no in-depth knowledge about cloud native
  technologies to concentrate on writing the code and deploying the code on cloud with no additional configuration.
   Cloud orchestration systems such as Kubernetes require developers to have extensive knowledge about the technology in order to be able to deploy their code. 
   The Code2Cloud functionality built into Ballerina would allow the developers to generate the deployment artifacts necessary for deploying their code in these platforms with minimal configurations.

## Project Structure

The deployment artifacts will be generated in the target directory of the project depending on the provider selected.

``` 
.
├── Cloud.toml                               <- Used to override default configerations used to generate artifacts.
├── Ballerina.lock
├── Ballerina.toml
├── entry.bal                                <- Entry point of the project.
└── target
    ├── bala
    │   └── module-0.0.1.bala
    ├── bin
    │   └── module.jar
    ├── docker
    │   └── module
    │       └── Dockerfile                        <- Generated Dockerfile according to the ballerina project
    └── kubernetes
        └── module
            └── c2c-sample-module-0.0.1.yaml      <- Generated Kuberenetes deployment YAML artifact for the ballerina project

```

#### Ballerina.toml

Used to enable code to cloud in the project. This file also contains metadata about the ballerina project. This file is 
used to fetch defaults for deployment artifacts generation.

#### entry.bal

Represents any .bal file that has an entry point. Compiler will be using the file to retrieve service related
 information for the deployment artifacts.

#### Cloud.toml

An optional configuration file used to override the default values generated by C2C. 

#### target/docker/

Contains the Docker artifacts generated from C2C. These artifacts will be required to build the Docker image.

#### target/kubernetes/

Contains the Kubernetes artifacts generated from C2C. These artifacts will be required to deploy the ballerina
 application in kubernetes.

## Building Cloud Artifacts

Code to cloud artifacts can be built executing following commands in the ballerina project directory. Code to cloud 
behaves differently depending on the type of the project.
 
#### For Single File Projects

Execute the following command to build the file

###### Docker
```shell
bal build --cloud=docker filename.bal
```
###### Kubernetes
```shell
bal build --cloud=k8s filename.bal
```

#### For Ballerina Projects
Add the following field in Ballerina.toml

###### Docker
```toml
[build-options]
cloud = "docker"
```
###### Kubernetes
```toml
[build-options]
cloud = "k8s"
```

Execute the build command inside the project directory.

```shell
bal build
```

## Features

### Container Creation

#### Basic Container Creation

C2C enables developers to create containers using the ballerina code without any additional configuration. 

``` yaml
[container.image]
repository="wso2"
name="sample-c2c" 
tag="v0.1.1" 

```

#### Exposing Services

C2C supports exposing containers via listeners. It also allows developers to set the internal domain and external
 accessibility.

``` ballerina
import ballerina/http;

service /hello on new http:Listener(9090) {

    resource function sayHello() returns string {
        return "Hello World!";
    }
}

```

``` yaml
[cloud.deployment]
internal_domain_name="myfoo" 
external_accessible=true  
```

#### Resource Limits and Autoscaling

Enabling Resource limits prevents containers from crashing and ensures healthy containers. C2C generates artifacts
 required for the orchestrator to limit the resource taken from one
 container. It also generates autoscaling artifacts to smoothly scale when containers as overloaded.

``` yaml
[cloud.deployment]
min_memory="100Mi" 
max_memory="256Mi" 
min_cpu="1000m" 
max_cpu="1500m" 

[cloud.deployment.autoscaling]
min_replicas=2
max_replicas=5
cpu=50
memory=80
```

### Configuration 

Ballerina Code to Cloud provides ability for developers to provide configurations in the Cloud.toml.

``` yaml
[[cloud.config.files]]
    file="resource/file.txt"
    mount_dir="/home/ballerina/foo"
[[cloud.secret.files]]
    file="./resource1"
    mount_dir="./resource1"

```


### Cronjobs

When the developer wants to execute a task periodically, the most optimum way is to let the orchestrator handle the
 scheduling and execution rather than having a process running continuously. The following behavior can be achieved
  using Task annotation in ballerina/cloud library.

``` ballerina
import ballerina/io;
import ballerina/cloud;

@cloud:Task {
  schedule: {
        minutes: "*",
        hours: "*",
        dayOfMonth: "*",
        monthOfYear: "*",
        daysOfWeek: "*"
  }
}

public function main(string... args) {
    io:println("hello world");
}
```

### Probes

Container orchestrators uses probes as a way of ensuring the readiness and health of the container. Ballerina Code to
 Cloud makes it easier for the developers to define probes.

``` yaml
[cloud.deployment.probes.readiness]
port=9091
path="/readyz"

[cloud.deployment.probes.liveness]
port=9091
path="/healthz" 
```

## Cloud.toml Properties

Ballerina Code to Cloud supports number of properties that is required to deploy the code in the cloud. All of these
 properties contains suitable default values derived from the code. These values can be overridden using the optional
  Cloud.toml file.

#### [container.image]

Contains container image related properties.

| Identifier   	 | Description	                                                    | Default Value   	           |
|----------------|-----------------------------------------------------------------|-----------------------------|
| name   	       | Name of the container image                                     | $USER-$MODULE_NAME          |
| repository   	 | Container repository to host the container                      | ""   	                      |
| tag   	        | Tag of the container   	                                        | "latest"   	                |
| base   	       | Base container of the container image   	                       | "ballerina/jvm-runtime:2.0" |
| entrypoint   	 | Instruction, which is executed when the container starts up   	 | -                           |

    

#### [container.image.user]

Container image user account related properties

| Identifier   	 | Description	                    | Default Value   	 |
|----------------|---------------------------------|-------------------|
| run_as   	     | Name of the container image   	 | "ballerina"   	   |


#### [[container.copy.files]]

Copy the files to the container image

| Identifier   	 | Description	                          | Default Value   	               |
|----------------|---------------------------------------|---------------------------------|
| sourceFile  	  | Path to the external file   	         | "./data/data.txt"               |
| target  	      | Path of the file within the container | "/home/ballerina/data/data.txt" |


#### [[cloud.config.envs]]

Contains the environment variables required for the application.

| Identifier     | Description	                                  | Example Value |
|----------------|-----------------------------------------------|---------------|
| key_ref   	    | Key of the environment variable               | "FOO"   	     |
| name(optional) | Name of the env if its different from the Key | "foo"   	     | 
| config_name    | Name of the config config map                 | "module-foo"  |

#### [[cloud.config.secrets]]

Contains the secrets required for the application.

| Identifier   	 | Description	                         | Example Value   	   |
|----------------|--------------------------------------|---------------------|
| file   	       | file path to the Config.toml locally | "Config.toml"       |
| name(optional) | Name of the secret created	          | "config-secret1"  	 |


#### [[cloud.config.files]]

Contains config toml files required for the code.

| Identifier     | Description                  | Example Value        |
|----------------|------------------------------|----------------------|
| file           | Path of the external file    | "conf/Config.toml"   |
| name(optional) | Name field for the configmap | "sample5-config-map" |


#### [[cloud.config.maps]]

Used to mount file/directory as config maps in kubernetes.

| Identifier | Description                            | Example Value              |
|------------|----------------------------------------|----------------------------|
| file       | Path of the file neededs to be mounted | "resource/file.txt"        |
| mount_dir  | Directory of the file in the container | "/home/ballerina/resource" |

#### [[cloud.secret.files]]

Used to mount file/directory as secrets in kubernetes.

| Identifier | Description	                             | Example Value                |
|------------|------------------------------------------|------------------------------|
| file   	   | Path of the file neededs to be mounted 	 | "./resource/data.txt"	       |
| mount_dir  | Directory of the file in the container	  | "/home/ballerina/resource" 	 |

#### [cloud.deployment]

Contains the properties related to the deployment.

| Identifier           | Description                                    | Default Value      |
|----------------------|------------------------------------------------|--------------------|
| internal_domain_name | Name of the internal domain (k8s service name) | ${MODULE_NAME}-svc |
| min_memory           | Minimum memory allocated to the container      | "100Mi"            |
| max_memory   	       | Maximum memory allocated to the container  	   | "256Mi"   	        |
| min_cpu   	          | Minimum CPU allocated to the container 	       | "1000m"   	        |
| max_cpu   	          | Maximum CPU allocated to the container  	      | "1500m"   	        |

#### [cloud.deployment.autoscaling]

Contains the matrices to auto scale the container

| Identifier   | Description	                                                      | Default Value |
|--------------|-------------------------------------------------------------------|---------------|
| enable   	   | Status of autoscaling  	                                          | true  	       |
| min_replicas | Minimum number of replicas of the container alive at a given time | 2   	         |
| max_replicas | Maximum number of replicas of the container alive at a given time | 3   	         |
| cpu   	      | CPU Utilization threshold for spawning a new instance 	           | 50   	        |
| memory   	   | Memory Utilization threshold for for spawning a new instance 	    | 80   	        |

#### [cloud.deployment.probes.readiness]

Probe to indicate whether the container is ready to respond to requests. No readiness probe will be generated if not
 specified.
 
| Identifier | Description	                         | Example Value |
|------------|--------------------------------------|---------------|
| port   	   | Port of the Readiness probe endpoint | 9091  	       |
| path   	   | Endpoint of the Readiness probe   	  | "/readyz"   	 |
  

#### [cloud.deployment.probes.liveness]

Probe to indicate whether the container is running. No liveness probe will be generated if not specified.

| Identifier | Description	                        | Example Value   	 |
|------------|-------------------------------------|-------------------|
| port   	   | Port of the Liveness probe endpoint | 9091  	           |
| path   	   | Endpoint of the Liveness probe   	  | "/healthz"   	    |

#### [[cloud.deployment.storage.volumes]]

Contains the volume definitions of the application. No default volumes will be generated if not specified.

| Identifier | Description	               | Example Value |
|------------|----------------------------|---------------|
| name   	   | Name of the volume  	      | "volume1"  	  |
| local_path | Path of the volume   	     | "files"   	   |
| size   	   | Maximum size of the Volume | "2Gi"   	     |


#### [[cloud.secret.envs]]

Properties to mount environment variables as secrets

| Identifier  | Description	                                 | Example Value    |
|-------------|----------------------------------------------|------------------|
| key_ref   	 | Name of the secret component in the cluster	 | "b7a-password" 	 |
| name        | Name of the environment variable   	         | "b7a-password" 	 |
| secret_name | Name of the key in the secret component      | "B7A_PASSWORD"	  |


#### [[settings]]
Contains the settings related to artifacts generation.

| Identifier     | Description	                                       | Example Value |
|----------------|----------------------------------------------------|---------------|
| singleYAML   	 | Generate a single YAML file with all the artifacts | true  	       |
| buildImage     | Build Docker image  	                              | true      	   |
| thinJar        | Uses thin jars instead of fat jar in the container | true      	   |
