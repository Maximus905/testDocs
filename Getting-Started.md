## INTRODUCTION
In this guide we will go through the simple th2 demo example. If you are already familiar with the th2 paradigm and interested in installing th2 for your business needs, refer to our [th2 technical requirements](https://github.com/th2-net/th2-documentation/wiki/Technical-Requirements) page to check which th2 configuration will be suitable for you.

These steps require you to have a cluster running a compatible version of Kubernetes. It's possible to use any supported platform, for example Minikube or install it locally, guide for kubernetes installation on Centos-7 is available in the [FAQ section](https://github.com/th2-net/th2-documentation/wiki/Centos-7-kubernetes-and-cassandra-installation-guide).

More information regarding the use cases covered in the demo example, it's configuration, execution process and results' verification can be found on our [th2 demo example page](https://github.com/th2-net/th2-documentation/wiki/Demo-Example).

## GETTING STARTED WITH th2
**Follow these steps to get started with th2:**
1. Install the required software
2. Download th2 to your Git repository
   - Get [th2-infra](https://github.com/th2-net/th2-infra) for core platform
   - Get [th2-infra-schema](https://github.com/th2-net/th2-infra-demo-configuration) for custom components
   - Get **th2 custom** _(this step is required if you would like to try these components and modify them based on your business logic)_, components from the demo example:
     - [th2-recon-template](https://github.com/th2-net/th2-check2-recon-template)
     - [th2-grpc-sim](https://github.com/th2-net/th2-grpc-sim-template)
     - [th2-sim](https://github.com/th2-net/th2-sim-template)
     - [th2-grpc-act](https://github.com/th2-net/th2-grpc-act-template)
     - [th2-act-template](https://github.com/th2-net/th2-act-template-j)
5. Set up your cluster and Install th2
6. Create your first th2 custom environment in a dedicated namespace
7. Get [th2-script-demo](https://github.com/th2-net/th2-demo-script)
8. Run [th2-script-demo](https://github.com/th2-net/th2-demo-script)

