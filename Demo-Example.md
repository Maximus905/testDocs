## INTRODUCTION
In this example we will show how to implement a few simple types of tests against a trading system. The example contains a simplified sim module to emulate the trading system we are going to test (Demo Exchange).

**The example will contain:**

* [Simple Test algorithm illustrating sending orders and verifying responses](https://github.com/th2-net/th2-documentation/wiki/_new#use-case-1-lineal-test-automation)
* [Autonomous real-time reconciliation algorithm comparing multiple data streams during tests](https://github.com/th2-net/th2-documentation/wiki/_new#use-case-2-rule-based-checking)
* [Simplified emulation of the trading system](https://github.com/th2-net/th2-documentation/wiki/_new#use-case-3-simulation-of-multiple-endpoints)

## PREREQUISITES

## THEORY OF TH2
Let’s consider the components which we have in the demo configuration. 
1. We have a set of components which are responsible for the storage of information in the data lake and to display them into the GUI: `estore`, `mstore`, `rpt-data-provider`, `rpt-data-viewer`.
2. A set of components which are responsible for creating outcoming messages and for checking messages: `act`, `check1`, `recon`, `util`.
3. We also have a set of components which receives messages from a remote system or other sources and convert them into a format that is understandable to the users and to other th2 components: `demo-conn1`, `demo-conn2`, `demo-dc1`, `demo-dc2`, `codec`, `read-log`.
4. A set of components that simulates another system (the provided set of messages is limited within the current demo script): `fix-demo-server1`, `fix-demo-server2`, `dc-demo-server1`, `dc-demo-server2`, `sim-demo`.

In addition we have a dictionary with the FIX protocol version, which is used by our components `conn` and `codec`. The `codec` will be used to encode/decode messages while the dictionary contains the description of version specific protocol messages. The `conn` component is used to communicate with the target system. A description with the connections between these components is represented in the diagram below:

![](https://github.com/th2-net/th2-documentation/blob/master/th2_schema.png)


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
* User1 submit buy order with Price=x and Size=30 - Order1
* User1 receives an Execution Report with ExecType=0
* User1 submit buy order with Price=x+1 and Size=10 - Order2
* User1 receives an Execution Report with ExecType=0
* User2 submit sell IOC order with price=x-1 and Size=100 - Order3
* User1 receives an Execution Report with ExecType=F on trade between Order2 and Order3
* User2 receives an Execution Report with ExecType=F on trade between Order3 and Order2
* User1 receives an Execution Report with ExecType=F on trade between Order1 and Order3
* User2 receives an Execution Report with ExecType=F on trade between Order3 and Order1
* User2 receives an Execution Report with ExecType=C on expired Order3

## USE CASE #2: Rule-based checking
Use case is based on the following th2 components :

**th2 building blocks**
* th2-read-log
* th2-codec
* th2-conn
* th2-util

**th2 custom** 
* th2-check2-recon

>> to be added about RECON

 
## USE CASE #3: Simulation of multiple endpoints
Use case is based on the following th2 components :

**th2 building blocks**

* th2-codec
* th2-conn
* th2 custom 
* th2-sim