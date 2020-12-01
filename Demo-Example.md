## INTRODUCTION
In this example we will show how to implement a few simple types of tests against a trading system. The example contains a simplified sim module to emulate the trading system we are going to test (Demo Exchange).

**The example will contain:**

* [Simple Test algorithm illustrating sending orders and verifying responses](https://github.com/th2-net/th2-documentation/wiki/_new#use-case-1-lineal-test-automation)
* [Autonomous real-time reconciliation algorithm comparing multiple data streams during tests](https://github.com/th2-net/th2-documentation/wiki/_new#use-case-2-rule-based-checking)
* [Simplified emulation of the trading system](https://github.com/th2-net/th2-documentation/wiki/_new#use-case-3-simulation-of-multiple-endpoints)

![](https://github.com/th2-net/th2-documentation/blob/master/images/th2architecture/th2_HL_schema.png)

## PREREQUISITES
To start using TH2 you have to configure the Kubernetes cluster. You can find the [hints](https://github.com/th2-net/th2-documentation/wiki/Centos-7-kubernetes-and-cassandra-installation-guide) on how to set up your Kubernetes <here>. 

After installing all the core components in the cluster Kubernetes, you can use the th2 demo configuration of the th2 environment.  

### To set up custom parameters of th2 follow the steps:

1. Copy the content of [repository](https://github.com/th2-net/th2-infra-demo-configuration) to your git repository. For gitlab you can use the ‘Import project’ option to create a new repository.
2. Connect your new repository to `infra-mgr`. Please find the hints <here>.
3. To set up read-log follow next steps:
* Create the directory for storage `read-log` files;
* Specify the path to the directory in `config.yaml` file;
* Place the files from the [th2-read-log/example](https://github.com/th2-net/th2-read-log/tree/master/examples) to your directory.
4. Specify the parameter `k8s-propagation` in `infra-mgr-config.yml` file. Specify it as ‘rule’ or ‘sync’ for the `infra-mgr` to start the installation. Please note that namespace’s name with this configuration in Kubernetes will have the same name as the branch name and have the prefix `schema_`. 
5. To set up all the necessary dependencies, you need to install the packages from requirements.txt. This can be done with `pip install -r requirements.txt`.
> Note that `infra-mgr-config.yml` file content a list of possible parameters for `k8s-propagation` and parameter description.

As the result of these steps `infra-mgr` will create a new namespace in your Kubernetes and rise up all components from current configuration. The related namespace with all the necessary settings will be set up automatically with `infra-mng` after you’ll copy the config to your branch.

## THEORY OF TH2
Let’s consider the components which we have in the demo configuration. 
1. We have a set of components which are responsible for the storage of information in the data lake and to display them into the GUI: `estore`, `mstore`, `rpt-data-provider`, `rpt-data-viewer`.
2. A set of components which are responsible for creating outcoming messages and for checking messages: `act`, `check1`, `recon`, `util`.
3. We also have a set of components which receives messages from a remote system or other sources and convert them into a format that is understandable to the users and to other th2 components: `demo-conn1`, `demo-conn2`, `demo-dc1`, `demo-dc2`, `codec`, `read-log`.
4. A set of components that simulates another system (the provided set of messages is limited within the current demo script): `fix-demo-server1`, `fix-demo-server2`, `dc-demo-server1`, `dc-demo-server2`, `sim-demo`.

In addition we have a dictionary with the FIX protocol version, which is used by our components `conn` and `codec`. The `codec` will be used to encode/decode messages while the dictionary contains the description of version specific protocol messages. The `conn` component is used to communicate with the target system. A description with the connections between these components is represented in the diagram below:

![](https://github.com/th2-net/th2-documentation/blob/master/images/th2architecture/th2_schema.png)


## DEMO EXAMPLE RUN

## USE CASE #1: Lineal test automation
Use case is based on the following th2 components :

**th2 infra and core** - th2 core and infra components are required for all th2 custom implementations, more details can be found in the [th2 installation guide](https://github.com/th2-net/th2-infra) 

**th2 building blocks:**
* th2-check1
* th2-codec
* th2-conn

**th2 custom:**
* th2-act 
* demo Python script 

Trader1 sends to the Demo Exchange system (simulated by the independant th2 module) two Buy Orders with different prices and quantities. After that, Trader2 will send an IOC Sell Order to the system with the price lower than both of the Trader1 Orders’ prices. In this demo scenario th2 verify the response received from the Demo Exchange and trade results. Scenario will be performed in several variations with different parameters.

**Test Scenario:**
* User1 submit buy order with Price=x and Size=30 - **Order1**
* User1 receives an Execution Report with ExecType=0
* User1 submit buy order with Price=x+1 and Size=10 - **Order2**
* User1 receives an Execution Report with ExecType=0
* User2 submit sell IOC order with price=x-1 and Size=100 - **Order3**
* User1 receives an Execution Report with ExecType=F on trade between **Order2** and **Order3**
* User2 receives an Execution Report with ExecType=F on trade between **Order3** and **Order2**
* User1 receives an Execution Report with ExecType=F on trade between **Order1** and **Order3**
* User2 receives an Execution Report with ExecType=F on trade between **Order3** and **Order1**
* User2 receives an Execution Report with ExecType=C on expired **Order3**

## USE CASE #2: Rule-based checking
Use case is based on the following th2 components :

**th2 building blocks**
* th2-read-log
* th2-codec
* th2-conn
* th2-util

**th2 custom** 
* th2-check2-recon

In our demo example we configured recon with two rules: `rule_1` and `rule_2`.

`Rule_1` (displayed as `"demo-conn1 vs demo-conn2"` in GUI) shows the trades between DEMO-CONN1 and DEMO-CONN2 traders. We expect to see one `ExecutionReport` from both `demo-conn1` and `demo-conn2` traders with the certain session_alias. Then if the field values of key field `TrdMatchID` will be matched, the reconciliation will occur.

`Rule_2` (displayed as `"FIX vs DC"` in GUI) compares `ExecutionReports` from  FIX conn and DC conn. The messages are matched by `ClOrdID`, `ExecType` and `ExecID` fields. As the script result will see two `ExecutionReports` for both `Order1` and `Order2` and three `ExecutionReports` for `Order3`.

`th2-read-log` is configured to read log file with market data. The results of it's work will be the market data messages in the format they come from the system. Then we compare market data messages with messages, sent into `demo-conn1` and `demo-conn2`. As a result, we expect that all `NewOrderSingle` messages, sent via the script, will find a pair in the log file.
 
## USE CASE #3: Simulation of multiple endpoints
Use case is based on the following th2 components :

**th2 building blocks**

* th2-codec
* th2-conn
* th2 custom 
* th2-sim

Simulator is the service for simulating different logic. All logic contains in a Rules. You can turn on/off rules for different connections or some rules for one connection. This project is java framework for creating custom the Simulator.

Simulator is a flexible instrument, which allows to simulate different systems. 

To learn more how to install and set up your Simulator follow the [link](https://github.com/th2-net/th2-sim).

## USE CASE #4: Verification
Th2 has a web-based GUI, which helps to manage and analyze test data. The GUI is divided into two parts: Events and Messages.

On the Event tab you can see the list of executed scripts. Each script has a tree of actions which is called events. Events may contain different data: information about sending messages, incoming messages, comparison tables, where you can check that expected and actual results match or don’t match.