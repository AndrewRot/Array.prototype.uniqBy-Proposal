# Array.prototype.uniqBy-Proposal

It would be useful to have the ability to get all unique objects according to some provided definition of equality. 

Additional functionality:
```
Array.prototype.uniqBy
```

Here are two possible ways that this could work:

Using object reduction:

Single Level:
```
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy(x => x.a); // [ {a: 1, b: 1} ]
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy(x => x.b); // [ {a: 1, b: 1},  {a: 1, b: 2} ]
```

Nest Objects:
```
[ {a: 1, b: {c: 2}},  {a: 2, b: {c: 2}} ].uniqBy(x => x.b.c); // [ {a: 1, b: {c: 2}} ]
```

Implementation 1.a:
```
Array.prototype.uniqBy = function(getFields) {
  let reducedObjects = this.map(getFields);
  let set = new Set(reducedObjects);
  return Array.from(set)
    .map(x => reducedObjects.indexOf(x))
    .map(index => this[index]);
}
```

This implementation works well when uniquing by a single field. However, becomes problematic when you wish to uniqBy using multiple fields on an object. This could be improved by using Immutable Records, see [proposal](https://github.com/tc39/proposal-record-tuple). Immutable Records have built-in object equality which could be used in the following way to support uniqBy multiple fields:

Implementation 1.b:
```
Array.prototype.uniqBy = function(getFields) {
  let reducedObjects = this.map(getFields);
  // TODO insert reducedObjects in record
  let records = reducedObjects.map(obj => new ImmutableRecord(obj));
  // TODO find out what ImmutableRecord equality method looks like... (.equals(..))?
  // TODO Or use a compare function with just (x, y) => x.equals(y).... 
  let set = new Set(reducedObjects);
  return Array.from(set)
    .map(x => reducedObjects.indexOf(x))
    .map(index => this[index]);
}
```


Using a compare method:

Single Level:
```
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy((x, y) => x.a === y.a); // [ {a: 1, b: 1} ]
```

Nest Objects:
```
[ {a: 1, b: {c: 2}},  {a: 2, b: {c: 2}} ].uniqBy((x, y) => x.b.c === y.b.c); // [ {a: 1, b: {c: 2}} ]
```

Implementation:
```
Array.prototype.uniqBy = function(compare) {
  let out = [];
  this.forEach(x => {
    if (!out.some(y => compare(x, y))) {
      out.push(x);
    }
  })
  return out;
}
```

Time: ~O(n^2)
Can be optimized to O(log(n)) using function identifiers?


Additional information: 

There is a node module, array-unique, which has similar behavior. This module has over 13 million weekly downloads. Only for doing so on a single field.
