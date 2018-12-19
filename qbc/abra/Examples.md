#### Concatenation

You want to take two or more sites, which are vectors of trits, 
and combine them to a single site which is a concatenated combination of the two.

Create a branch (following the spec)

```
[ -10 // number of input sites is 2
, [110, -110] // the first site is of length 3, the second is of length 6
, 0 // there are no body sites
, -10 // there are 2 output sites
, 0 // number of memory latches
, [ { 1 // site merges other sites
    , 10 // it merges one site
    , 10 // it 'merges' the first site
    }
  , { 1 // site merges other sites
    , 10 // it merges one site
    , -10 // it 'merges' the second input site
    }
  ]
, [] // there are no memory latches
]
```

To wrap up this branch, we would see its definition as `0-10110-1100-1011010110-10`. 
When we want to merge n sites of combined length 9, we can use this branch.
We could shorten this, since we know that the combined length is 9.

```
[ 10 // number of input sites is 1
, [1--10] // total of 9 trits input (can be used by a combination of a 3 and a 6)
, 0 // there are no body sites
, 10 // there are 2 output sites
, 0 // number of memory latches
, [ { 1 // site merges other sites
    , 10 // it merges one site
    , 10 // it 'merges' the first site
    }
  ]
, [] // there are no memory latches
]
```
Resulting in: `01011--1001011010`

#### Reordering

Let's imagine now that we want to reorder a site of length 9, and put the 3 first trits at the end. 
We can take our less efficient example from the first concatenation, 
and modify it to reverse the order of the two output sites.

```
[ -10 // number of input sites is 2
, [110, -110] // the first site is of length 3, the second is of length 6
, 0 // there are no body sites
, -10 // there are 2 output sites
, 0 // number of memory latches
, [ { 1 // site merges other sites
    , 10 // it merges one site
    , -10 // it 'merges' the second input site
    }
  , { 1 // site merges other sites
    , 10 // it merges one site
    , 10 // it 'merges' the first site
    }

  ]
, [] // there are no memory latches
]
// result: 0-10110-1100-10110-1011010
```

#### Select

We can select a static portion from within a vector, to return the 5 middle trits offset from the first 3 trits.

```
[ 110 // number of input sites is 2
, [110, 1-10, 10] // the first site is of length 3, the second is of length 5, the third is of length 1
, 0 // there are no body sites
, -10 // there are 2 output sites
, 0 // number of memory latches
, [] // no body sites
, [ { 1 // site merges other sites
    , 10 // it merges one site
    , -10 // it 'merges' the second input site
    }
  ]
, [] // there are no memory latches
]
// result: 01101101-101010-10110-10
```

#### Making a 2-or-1 input lookup table

All lookup table definitions are 27 nullable trits, as the majority of
lookup table usages will use 3 inputs. In the case where it is required
to create a 2-input lookup table, one can simply give the same input twice.


#### Constants from higher level languages

Thus, when encoding constants from a higher-level language, these should
propagate down to the lookup table, which either combines with other lookup tables,
or repeats inputs for a constant output.
