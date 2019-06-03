
Qubic is a multivariate tangle which is communicated over the IOTA tangle through serialized graph vertices.

It begins with a genesis transaction of a committee, in which the configuration of the committee is expressed.

This configuration includes:

- The duration of an epoch (a complete cycle in Qubic)
- The duration of the testing phase
- The time at which the first epoch will begin
- A list of how stake is allocated to oracles

The list of stake allocation parameters is defined by a map of hashes referencing test definition transactions 
to a positive integer which represents the amuont of stake associated to that test.

These test definitions have one function to generate the resource test to publish, 
and another to validate those published and discovered. 
The output of a resource generation function is published directly to the qubic tangle - referencing all other valid resource tests leading to the configuration transaction - along with the id of the participant (the oracle), the value of the result, and the environment of the validating function.
The result of the validating function outputs an id (of the generator of the test) and a number indicating its score.

During the resource test phase, only valid resource tests are referenced.

Each participant and consumer of the committee will validate and tally the final weight assigned to each oracle id
in the resource test phase to measure weights assigned to votes in the processing phase.

In the processing phase, a `qubic` can be attached - this is akin to a smart contract in the common parlance. Its actual code consists of Abra dataflow branches attached to environments (or system-level environment extensions).

A qubic interacts with the Qubic tangle by flagging an environment in the attachment metadata to output for quorum, or to input only when a quorum has been reached.

When an output is flagged for quorum, Qubic (which has been configured by the owner with its private keys) will compute the `hash(effect)` to be commited to, as well as a salted hash `hash(effect||salt)`, which is the address of the key for the one-time signature used to sign the commitment (in a merkle signature scheme). These both are published as the data for a new vertex in the Qubic tangle for the oracle's committee.

When evaluating subtangles in Qubic, it is valid for commitment transactions to reference those of other commitments of identical effect hash or valid reveal transactions. The rule of thumb is to reference all visible, valid unreferenced Qubic subtangles, which includes valid reveals, commits of identical hash, finalized votes on rule changes, and qubic attachments.

A reveal transaction is valid if a sufficient quorum is reached in the commitments it references. This means that below it - for the given effect hash - there must be greater than 50% of the voting weight (from stake assigned in the resource test phase) on the committed value.


**Separate qubics (composed of many entities) only affect each other through quorum results or through shared higher-level environmental interactions (such as system-level extensions).

