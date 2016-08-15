# Kubernetes Artifacts for WSO2 Enterprise Service Bus #
These Kubernetes Artifacts provide the resources and instructions to deploy WSO2 Enterprise Service Bus on Kubernetes.

## Getting Started
>In the context of this document, `KUBERNETES_HOME`, `DOCKERFILES_HOME` and `PUPPET_HOME` will refer to local copies of [`wso2/kubernetes artifacts`](https://github.com/wso2/kubernetes-artifacts/), [`wso2/dockcerfiles`](https://github.com/wso2/Dockerfiles/) and [`wso2/puppet modules`](https://github.com/wso2/puppet-modules) repositories respectively.

To deploy WSO2 Enterprise Service Bus on Kubernetes, the following steps have to be done.
* Build WSO2 Enterprise Service Bus Docker images
* Copy the images to the Kubernetes Nodes
* Run `deploy.sh` which will deploy the Service and the Replication Controllers

#### 1. Build Docker Images

To manage configurations and artifacts when building Docker images, WSO2 recommends to use [`wso2/puppet modules`](https://github.com/wso2/puppet-modules) as the provisioning method. A specific data set for Kubernetes platform is available in WSO2 Puppet Modules. It's possible to use this data set to build Dockerfiles for WSO2 Enterprise Service Bus for Kubernetes with minimum configuration changes.

Building WSO2 Enterprise Service Bus Docker images using Puppet for Kubernetes:

  1. Clone `wso2/puppet modules` and `wso2/dockerfiles` repositories (alternatively you can download the released artifacts using the release page of the GitHub repository).
  2. Copy the [dependency jars](https://docs.wso2.com/display/KA100/Kubernetes+Membership+Scheme+for+WSO2+Carbon) for clustering to `PUPPET_HOME/modules/wso2esb/files/configs/repository/components/lib` location.
  3. Copy the JDK [`jdk-7u80-linux-x64.tar.gz`](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) to `PUPPET_HOME/modules/wso2base/files` location.
  3. Copy  [`kernel patch0005`](http://product-dist.wso2.com/downloads/carbon/4.4.1/patch0005/WSO2-CARBON-PATCH-4.4.1-0005.zip) to `PUPPET_HOME/modules/wso2esb/files/patches/repository/components/patches` folder.
  4. Copy the [`mysql-connector-java-5.1.36-bin.jar`](http://mvnrepository.com/artifact/mysql/mysql-connector-java/5.1.36) file to `PUPPET_HOME/modules/wso2esb/files/configs/repository/components/lib` location.
  5. Copy the WSO2 Enterprise Service Bus 4.9.0 product pack file to `PUPPET_HOME/modules/wso2esb/files` location (Note that if you use a different product version, the `-v` flag provided to the subsequent scripts have to be changed to match).
  6. Set the environment variable `PUPPET_HOME` pointing to location of the puppet modules in local machine.
  7. Navigate to `wso2esb` directory in the Dockerfiles repository; `DOCKERFILES_HOME/wso2esb`.
  8. Build the Dockerfile with the following command:

    **`./build.sh -v 4.9.0 -s kubernetes`**

  Note that `-s kubernetes` flag denotes the Kubernetes platform, when it comes to selecting the configuration from Puppet.

  This will build the default profile of WSO2 Enterprise Service Bus 4.9.0 for Kubernetes platform, using configuration specified in Puppet.

#### 2. Copy the Images to Kubernetes Nodes/Registry

Copy the required Docker images over to the Kubernetes Nodes (ex: use `docker save` to create a tarball of the required image, `scp` the tarball to each node, and use `docker load` to reload the images from the copied tarballs on the nodes). Alternatively, if a private Docker registry is used, transfer the images there.

You can make use of the `load-images.sh` helper script to transfer images to the Kubernetes nodes. It will search for any Docker images with `wso2` as a part of its name on your local machine, and ask for verification to transfer them to the Kubernetes nodes. `kubectl` has to be functioning on your local machine in order for the script to retrieve the list of Kubernetes nodes. You can optionally provide a search pattern if you want to override the default `wso2` string.

**`load-images.sh -p wso2esb`**

**Usage**
```
Usage: ./load-images.sh [OPTIONS]

Transfer Docker images to Kubernetes Nodes
Options:

  -u	[OPTIONAL] Username to be used to connect to Kubernetes Nodes. If not provided, default "core" is used.
  -p	[OPTIONAL] Optional search pattern to search for Docker images. If not provided, default "wso2" is used.
  -h	[OPTIONAL] Show help text.

Ex: ./load-images.sh
Ex: ./load-images.sh -u ubuntu
Ex: ./load-images.sh -p wso2is
```

#### 3. Deploy Kubernetes Artifacts
  1. Navigate to `wso2esb` directory in kubernetes repository; `KUBERNETES_HOME/wso2esb` location.
  2. run the deploy.sh script:

    **`./deploy.sh`**

      This will deploy the WSO2 Enterprise Service Bus 4.9.0 default profile in Kubernetes, using the image available in Kubernetes nodes, and notify once the intended service starts running on the pod.
      __Please note that each Kubernetes node needs the [`mysql:5.5`](https://hub.docker.com/_/mysql/) Docker image in the node's Docker registry.__

#### 4. Access Management Console
  1. Add an host entry (in Linux, using the `/etc/hosts` file) for `wso2esb-default`, resolving to the Kubernetes node IP.
  2. Access the Carbon Management Console URL using `https://wso2esb-default:32122/carbon/`

#### 5. Undeploying
  1. Navigate to `wso2esb` directory in Kubernetes repository; `KUBERNETES_HOME/wso2esb` location.
  2. run the `undeploy.sh` script:

    **`./undeploy.sh`**

      This will undeploy the WSO2 Enterprise Service Bus specific DB pod (`mysql-mbdb`), Kubernetes Replication Controllers, and Kubernetes services. Additionally if `-f` flag is provided when running `undeploy.sh`, it will also undeploy the shared Governance and User DB pods, Replication Controllers, and Services.
      **`./undeploy.sh -f`**

For more detailed instructions on deploying WSO2 Enterprise Service Bus on Kubernetes, please refer the wiki links under the Documentation section below.

# Documentation
* [WSO2 Kubernetes Artifacts Wiki](https://docs.wso2.com/display/KA100/WSO2+Kubernetes+Artifacts)
