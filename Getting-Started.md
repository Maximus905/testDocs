## INTRODUCTION
In this guide we will go through the simple steps that would help to start with th2, based on the demo example. If you are already familiar with the th2 paradigm and interested in installing th2 for your business needs, refer to our [th2 technical requirements](https://github.com/th2-net/th2-documentation/wiki/Technical-Requirements) page to check which th2 configuration will be suitable for you.

These steps require you to have a cluster running a compatible version of Kubernetes. It's possible to use any supported platform, for example Minikube or install it locally, guide for kubernetes installation on Centos-7 is available in the [FAQ section](https://github.com/th2-net/th2-documentation/wiki/Centos-7-kubernetes-and-cassandra-installation-guide).

More information regarding the use cases covered in the demo example, it's configuration, execution process and results' verification can be found on our [th2 demo example page](https://github.com/th2-net/th2-documentation/wiki/Demo-Example).

## GETTING STARTED WITH th2
**Follow these steps to get started with th2:**
1. [Install required software](https://github.com/th2-net/th2-documentation/wiki/Getting-Started/_edit#install-required-software)
2. [Download th2 to your Git repository](https://github.com/th2-net/th2-documentation/wiki/Getting-Started/_edit#download-th2-to-your-git-repository)
3. [Download th2 **Demo Example** configuration to your Git repository](https://github.com/th2-net/th2-documentation/wiki/Getting-Started/_edit#download-th2-demo-example-to-your-git-repository)
3. [Set up your cluster and Install th2](https://github.com/th2-net/th2-documentation/wiki/Getting-Started/_edit#set-up-your-cluster-and-instull-th2)
4. [Create your first th2 custom environment in a dedicated namespace](https://github.com/th2-net/th2-documentation/wiki/Getting-Started/_edit#create-your-first-th2-custom-environment)
5. [Get and Run **th2-script-demo**](https://github.com/th2-net/th2-documentation/wiki/Getting-Started/_edit#get-and-run-demo-script)


## INSTALL REQUIRED SOFTWARE
th2 is a Kubernetes-driven microservices solution.
## DOWNLOAD th2 TO YOUR GIT REPOSITORY
th2 components should be copied to your Git repository if you would like to try th2 custom componets, modify them based on your business logic or evaluate th2 onboarding process. If, at this step, you do not want to make any changes and just run demo example without any modifications this step can be skipped. 

You would need the following:
- [th2-infra](https://github.com/th2-net/th2-infra) for core platform
- **th2 custom**, components from the demo example:
     - [th2-recon-template](https://github.com/th2-net/th2-check2-recon-template)
     - [th2-grpc-sim](https://github.com/th2-net/th2-grpc-sim-template)
     - [th2-sim](https://github.com/th2-net/th2-sim-template)
     - [th2-grpc-act](https://github.com/th2-net/th2-grpc-act-template)
     - [th2-act-template](https://github.com/th2-net/th2-act-template-j)
## DOWNLOAD th2 DEMO EXAMPLE TO YOUR GIT REPOSITORY

Get [th2-infra-schema](https://github.com/th2-net/th2-infra-demo-configuration) for custom components
## SET UP YOUR CLUSTER AND INSTULL th2
## CREATE YOUR FIRST th2 CUSTOM ENVIRONMENT
## GET AND RUN DEMO SCRIPT
6. Run [th2-script-demo](https://github.com/th2-net/th2-demo-script)

