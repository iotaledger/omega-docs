## Serialization extension

The serialization extension should allow for the user to take an arbitrary data set and a list of keys, and encode it as a bundle fragment.
This is the principle of one-data-many-keys. 

* The user is an author of subsequent modules which publish unique data to the tangle.
* The keys should be addressable by index.
* There should be some mechanism for checking the class of serialized data
* There should be a way of indicating the total size of the data payload

### Keys

A key is some value, either of 243-trit length, which could be a pointer to some larger data (depending on context). 
The keys should occupy portions of the bundle essence, such as the extra data digest and address.

### Data

The data should probably occupy the message portion of a transaction, 
and there should be as many transactions necessary to encode the entire data portion.

### Class

Given a transaction, one should be able to check if it is of a particular serialization class.
One way of doing this may be to modify the bundle nonce of the last transaction such that it m
ay be used as a checksum against the inner state of the bundle hash at that point, 
which would then modify some number of trits such as to match the class of serialized data.
One way to create a unique identifier for the class of the serialized data is to name it as well as each of its keys, 
and to compute the hash of this.



