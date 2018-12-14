#### Concatenation

You want to take two or more sites, which are vectors of trits, 
and combine them to a single site which is a concatenated combination of the two.

Create a branch (following the spec)

```
[ 0 // number of memory latches
, [] // there are no memory latches
, -10 // number of input sites is 2
, [110, -110] // the first site is of length 3, the second is of length 6
, 0 // there are no body sites
, -10 // there are 2 output sites
, [ { 1 // site merges other sites
    , 10 // it merges one site
    , 10 // it 'merges' the first site
    }
  , { 1 // site merges other sites
    , 10 // it merges one site
    , -10 // it 'merges' the second input site
    }
  ]
]
```

To wrap up this branch, we would see its definition as `0-10110-1100-1011010110-10`. 
When we want to merge n sites of combined length 9, we can use this branch.
We could shorten this, since we know that the combined length is 9.

```
[ 0 // number of memory latches
, [] // there are no memory latches
, 10 // number of input sites is 1
, [1--10] // the first site is of length 3, the second is of length 6
, 0 // there are no body sites
, 10 // there are 2 output sites
, [ { 1 // site merges other sites
    , 10 // it merges one site
    , 10 // it 'merges' the first site
    }
  ]
]
```
Resulting in: `01011--1001011010`

We can see that this is implicit - concatenation as a usage of selected input sites or output sites are automatically used in the order they were declared, and so this example is only useful for instructive purposes.

#### Reordering

Let's imagine now that we want to reorder a site of length 9, and put the 3 first trits at the end. 
We can take our less efficient example from the first concatenation, 
and modify it to reverse the order of the two output sites.

```
[ 0 // number of memory latches
, [] // there are no memory latches
, -10 // number of input sites is 2
, [110, -110] // the first site is of length 3, the second is of length 6
, 0 // there are no body sites
, -10 // there are 2 output sites
, [ { 1 // site merges other sites
    , 10 // it merges one site
    , -10 // it 'merges' the second input site
    }
  , { 1 // site merges other sites
    , 10 // it merges one site
    , 10 // it 'merges' the first site
    }

  ]
]
// result: 0-10110-1100-10110-1011010
```

#### Select

We can select a static portion from within a vector, to return the 5 middle trits offset from the first 3 trits.

```
[ 0 // number of memory latches
, [] // there are no memory latches
, 110 // number of input sites is 2
, [110, 1-10, 10] // the first site is of length 3, the second is of length 5, the third is of length 1
, 0 // there are no body sites
, -10 // there are 2 output sites
, [ { 1 // site merges other sites
    , 10 // it merges one site
    , -10 // it 'merges' the second input site
    }
  ]
]
// result: 01101101-101010-10110-10
```
