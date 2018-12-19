
## Abra

Abra describes recursive trinary dataflow. It is a dataflow-oriented computation language.
Abra is composed of lookup tables and trit vectors (and constants), which are combined to create functions and branches, which may be recursive.
Branches attached to environments are called entities.
The inputs to and outputs from entities are called effects.
Effects which are sent to environments defined by entity metadata affect other entities belonging to those environments.
Branches are defined as a list of sizes of inputs, and a list of dataflow sites which may be marked as stateful and output.

#### Goals

Abra should be trivial to execute by an IoT device. Thus, the encoding
given here should modify only to reduce the complexity for an IoT device
to run it. Functionality of higher-level languages should be transformed
and reduced to lower-level constructs to make for simpler execution.

#### Terms

```
 Knot: an invocation of a branch which outputs to a site in the dataflow of a branch

 Lookup Table: 3-input, 1-output, which only returns a non-null value where defined

 Entity: an entrypoint branch which receives input effects from many environments, sends output effects to many environments

 Environment: an address to which effects are sent. Effects are length-extended or cropped, depending on Entity input size.

 Effect: a non-null trit vector sent between effects

 Branch: much like a function in traditional programming paradigms. It can have state, and can recursively invoke itself. Recursive invocations of branches are new instances of the branch. If it is stateful, then it is a new state in each recursive path. It has an exact input size and an exact return size. Its output is serialized as each site marked as "output" in the order it appears.

 Site: a vertex in our dataflow graph within a branch, representing a constant, a result of a branch invocation, or a merging of other sites.
 
 Memory Latch: a stateful site whose new value will be usable in the next invocation of the same branch.
 
 Merge: in a water analogy, a wye terminating in one output. Multiple sites can be selected from, where the merging site's value will be the unique non-null vector of the many site inputs to the merger.
```

#### Trit encoding spec

Attachment metadata:

We have a list of entities and their attachment metadata. Incoming, they have a `limit`, per belonging environment, on the number of times they may be invoked in a time `quant` (positive integer). They have a `delay` in number of `quants`, per outgoing environment, before the effects will affect the entities of that environment. They have a maximum `depth` of recursion, counted as the maximum number of branches which may be attached. A depth of `0` will not traverse any branches.

Attachment transactions are kept separate from the dataflow definition transactions for purposes of reusability, to keep entities (instantiations) separate from branches (definitions).

```
Entity attachment:
[ code hash (243 trits)
, number of attachments (positive integer)
, attachments...
]

Attachment:
[ branch block index (positive integer)
, maximum recursion depth (positive integer)
, number of input environments (positive integer)
, input environment data...
, number of output environments (positive integer)
, output environment data...
]

input environment data:
[ environment hash
, limit (positive integer)
]

output environment data:
[ environment hash
, delay (positive integer)
]


code:
[ tritcode version (positive integer [0])
, number of lookup table blocks (positive integer)
, 35-trit lookup tables (27 nullable trits in bct)...
, number of branch blocks (positive integer)
, branch block definitions ...
, number of external blocks (positive integer)
, external block definitions...
]

block (whether external, lut, or branch):
[ number of trits in block definition (positive integer)
, value...
]

branch:
[ number of inputs (positive integer)
, input lengths (positive integers)...
, number of body sites (positive integer)
, number of output sites (positive integer)
, number of memory latch sites (positive integer)
, body site definitions...
, output site indices (positive integers)...
, memory latch site indices (positive integers)...
]

site:
[ merge / knot? 1 trit (1/-)
, value...
]

merge:
[ number of input sites (positive integer)
, input site indices (positive integers)...
]

knot:
[ number of input sites (positive integer)
, input site indices (positive integers)...
, block index
]

external block:
[ code hash
, number of blocks to import (positive integer)
, block indices (positive integers)...
]
```

#### Merge / Knot / Sites

A merge has all input sites of identical length; but a knot (a branch invocation) would simply take all vectors as little-endian-packed input, as `b(xxxxyyyyzzzzzz)`.

A knot's definition (as opposed to inputs) may be defined by any of the blocks listed in the definition - branch, lookup table, or externally imported block (which may be lookup table or branch).

In a branch, because these are packed to little-endian, n input vectors could be concatenated or shuffled to select, concatenate, or rearrange by ordering of sites which each merge one input index, and marking the site as output to the branch. In a concatenation example, it uses no lookup tables. A branch which selects a range of trits at an offset which changes dependent on one of its inputs (an index), however, may have many input sites of 1-trit vectors, and use many lookup tables and mergers.

#### Importing branch definitions from external sites

An external site import will occupy n indices for the declared number of imported sites. When a branch is used in a site (knot), these external sites will occupy that portion.

##### Inputs to knot/merge

A site in the dataflow graph is wired feed-forward. However, memory latches may be used anywhere within a branch, and as they are declared first, they may use other sites for the definition of their new value.
With the exception of memory latches, whose value previously defined is used, any index declared as an input to a merge or branch must be less than the current site index (starting from the first memory latch, through input sites, body sites, and output sites)

If the site being defined is a memory latch, it may use any other index from the rest of the branch as an input, including other memory latches. This is because the resulting value will only be set on a memory latch after the branch is invoked.

All other sites (body, output) may only use site indices less than their current index, such that there is no possibility of cycles (which may possibly not settle) in a branch.

#### Encoding
Positive integers (as listed above) are encoded as binary.1/-, little endian, terminated with 0.

Site indices may be positive or negative, so the minimum number of trits to encode the site is given first (positive integer), followed by the site value. `0` indicates both 0 trits and the value `0`. `101` encodes `1`, `10-` encodes `minus 1`, `1101--` encodes `minus 11`.

##### To determine
Changes to this spec may be necessary to determine fitness for the following:

1. Distributed Computing - may be defined in branches and metadata
2. Parallel Computing
3. Concurrent Computing
4. Real-time Computing
5. Lockstep Computing
6. Virtualization
7. ANN-friendliness
