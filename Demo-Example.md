## INTRODUCTION
In this example we will show how to implement a few simple types of tests against a trading system. The example contains a simplified sim module to emulate the trading system we are going to test (Demo Exchange).

**The example will contain:**

* [Simple Test algorithm illustrating sending orders and verifying responses](https://github.com/th2-net/th2-documentation/wiki/_new#use-case-1-lineal-test-automation)
* [Autonomous real-time reconciliation algorithm comparing multiple data streams during tests](https://github.com/th2-net/th2-documentation/wiki/_new#use-case-2-rule-based-checking)
* [Simplified emulation of the trading system](https://github.com/th2-net/th2-documentation/wiki/_new#use-case-3-simulation-of-multiple-endpoints)

![](https://github.com/th2-net/th2-documentation/blob/master/images/th2architecture/th2_HL_schema.png)

## PREREQUISITES
1. Core Kubernetes components;
2. Kubernetes namespace;
3. Your git repository;
4. Python environment 3.7+

To learn more about how to prepare your system follow the [link](https://github.com/th2-net/th2-documentation/wiki/Getting-Started). Please make sure that your system corresponds to the [technical requirements](https://github.com/th2-net/th2-documentation/wiki/Technical-Requirements). 

## THEORY OF TH2
Let’s consider the components which we have in the demo configuration. 
1. We have a set of components which are responsible for the storage of information in the data lake and to display them into the GUI: `estore`, `mstore`, `rpt-data-provider`, `rpt-data-viewer`.
2. A set of components which are responsible for creating outcoming messages and for checking messages: `act`, `check1`, `recon`, `util`.
3. We also have a set of components which receives messages from a remote system or other sources and convert them into a format that is understandable to the users and to other th2 components: `demo-conn1`, `demo-conn2`, `demo-dc1`, `demo-dc2`, `codec`, `read-log`.
4. A set of components that simulates another system (the provided set of messages is limited within the current demo script): `fix-demo-server1`, `fix-demo-server2`, `dc-demo-server1`, `dc-demo-server2`, `sim-demo`.

In addition we have a dictionary with the FIX protocol version, which is used by our components `conn` and `codec`. The `codec` will be used to encode/decode messages while the dictionary contains the description of version specific protocol messages. The `conn` component is used to communicate with the target system. A description with the connections between these components is represented in the diagram below:

![](https://github.com/th2-net/th2-documentation/blob/master/images/th2architecture/th2_schema.png)


## DEMO EXAMPLE RUN

If you installed th2 using the [Getting Started](https://github.com/th2-net/th2-documentation/wiki/Getting-Started/) instructions, you already have [th2-demo-script](https://github.com/th2-net/th2-demo-script) and can run this example from your PC. If not, please refer to the [GET AND RUN DEMO SCRIPT](https://github.com/th2-net/th2-documentation/wiki/Getting-Started#get-and-run-demo-script) guide first.


The script represents the set of sending messages to the system and getting the responses from the system.

When sending the message, script sends a grpc request to the `act` component with instructions which message in which connector have to be sent. Act transfers the message to the `conn` client component. Then, based on the used grpc call, it starts to find the message which will be the response from the system on the message we’ve sent.

The `conn` client component gets the th2 message from the `act`, forms the FIX message based on a dictionary and then sends it to the `conn` server on FIX protocol. 

The `sim` gets this message from the `conn` server and creates a response on it, simulating remote system behavior. 

The response returns on the `conn` server and then transfers to the `conn` client on FIX protocol. Then response goes to the `codec`, where it’s decoded into human readable th2 format which is also clear to the other components.
From the codec all the messages come to the `act`, to the `check1` for verifying on requests from script and to the `recon` for passive verification.

When checking, the script sends a grpc request to `check1` with instructions on messages verification. This instructions content expected result on each message we want to verify. 

Also component `recon` performs the passive verification during all the env work.

## USE CASE #1: Lineal test automation
Use case is based on the following th2 components :

**th2 infra and core** - th2 core and infra components are required for all th2 custom implementations, more details can be found in the [th2 installation guide](https://github.com/th2-net/th2-sim/blob/master/README.md) 

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

Simulator is the service for simulating different logic. All logic contains in a Rules. You can turn on/off rules for different connections or some rules for one connection.
Simulator is a flexible instrument, which allows to simulate different systems. 

To learn more about how to install and set up your Simulator follow the [link](https://github.com/th2-net/th2-sim).

## USE CASE #4: Results
All raw messages and test events are stored in the centralized data lake and then shown in a web-based GUI, which helps to manage and analyze test data. The GUI is divided into two parts: Events and Messages.

On the Event tab you can see the tree of all executed test cases (act events, check1 events, check2-recon events, conn events). Events may contain different data: information about sending messages, incoming messages, comparison tables, where you can check that expected and actual results match or don’t match.

The Message tab is the list of outcoming and incoming messages. It is linked with Events. When you choose an event with a message, this message is displayed on the list in the Message tab. Also you can navigate through the message list without reference to any event for extra analysis.

Events and messages are stored in `estore` and `mstore` without time limits, so you can return to your test scenarios anytime.

![](https://raw.githubusercontent.com/th2-net/th2-documentation/master/images/th2architecture/demo1_gui1.gif)

![](https://raw.githubusercontent.com/th2-net/th2-documentation/master/images/th2architecture/demo1_gui2.gif)

![](https://raw.githubusercontent.com/th2-net/th2-documentation/master/images/th2architecture/demo1_gui3.gif)
