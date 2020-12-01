In this article we will look at how check1 works and take a closer look on the verification using `chain_id`.

Let's start with how everything works. The connection creates two queues of messages: FIRST direction for messages from the system and SECOND direction for messages that we send. The **check1** connects to these queues.

![](https://github.com/th2-net/th2-documentation/blob/master/images/chain_verification/chainverification_1.png)

### Checkpoint
When we request the Checkpoint from **check1** (using act methods or directly from script), it finds the last message in the queue and writes down it sequence number.

![](https://github.com/th2-net/th2-documentation/blob/master/images/chain_verification/chainverification_2.png)

Сheckpoint is not stored anywhere, it is a simple number sequence of message which we use as start for the verification in CheckRule or CheckSequenceRule. But in some cases we can miss some unexpected messages when we use checkpoints.

Let’s look at following example:

![](https://github.com/th2-net/th2-documentation/blob/master/images/chain_verification/chainverification_3.png)

We have two checkpoints that are created when we send messages using **act**. Using CheckSequenceRule, we verify the two next messages(highlighted in green) that we expect after first send it. And another CheckSequenceRule to verify other two messages after the second send. And we miss one unexpected message(highlighted in red), because the first CheckSequenceRule stops when it had found two expected messages and the second CheckSequenceRule starts after the unexpected message.


### Chain_id
This problem can be solved with `chain_id` in the CheckSequenceRule request.

![](https://github.com/th2-net/th2-documentation/blob/master/images/chain_verification/chainverification_4.png)

When we request the first CheckSequenceRule, **check1** creates a chain caret (like a typewriter caret) which stops at the last verified message. After that, the next verification with the same `chain_id` will be started from this sequence and the chain caret will be moved on from the last message checked by this verification.

Chain caret stored in check1 for a limited amount of time based on parameters TH2_VERIFIER_CLEANUP_OLDER_THAN and TH2_VERIFIER_CLEANUP_TIME_UNIT. If more time has passed since the last request with this `chain_id` than described in the parameters - `chain_id` will be removed. 

Also we can use several chain carets on a single queue:

![](https://github.com/th2-net/th2-documentation/blob/master/images/chain_verification/chainverification_5.png)

For example if we request two CheckSequenceRuleRequests from checkpoint - the first for verification of two messages on Instrument1 and the second for verification of two messages on Instrument2; then two chain carets will be created, with each of them stopping when desired messages are found. The desired message is searched based on the key fields of each individual filter in CheckSequenceRuleRequest.

**Usage example in *script*:**

```python
...
check_connection = grpc.insecure_channel(check1_addr)
check = verifier_pb2_grpc.VerifierStub(check_connection)
# We can create chain_id for first request, or leave it blank and use it from checkpoint.
chain_id = verifier_pb2.ChainID(id=str(uuid.uuid1()))
# First request starts from checkpoint and initiates a chain.       
ver1_chain = check.submitCheckSequenceRule(CheckSequenceRuleRequest(
            description=f'Trader "{input_parameters["trader1"]}" receives Execution Report. '
            f'The order stands on book in status NEW',
            chain_id=chain_id,
            connectivity_id=ConnectionID(session_alias=input_parameters['trader1_fix']),
            checkpoint=order1_response.checkpoint_id,
            timeout=3000,
            parent_event_id=input_parameters['case_id'],
            pre_filter=PreFilter(fields={'SecurityID': ValueFilter(simple_filter=Instrument)}),
            message_filters=[sf.create_filter_object(msg_type='ExecutionReport',
                                                     fields=er1_parameters,
                                                     key_fields_list=['ClOrdID'])]
        ))
# For second request we can use chain_id from response, or from script if we fill it in first request.
check.submitCheckSequenceRule(CheckSequenceRuleRequest(
            description=f'Trader "{input_parameters["trader1"]}" receives Execution Reports: '
            f'first at Order2 and second on Order1 .',
            connectivity_id=ConnectionID(session_alias=input_parameters['trader1_fix']),
            chain_id=ver1_chain.chain_id,
            timeout=3000,
            parent_event_id=input_parameters['case_id'],
            pre_filter=PreFilter(fields={'SecurityID': ValueFilter(simple_filter=Instrument)}),
            message_filters=[er_2vs3_filter, er_1vs3_filter],
            check_order=True))
# For future requests we can use stored chain_id.
...
```