I would like to focus on just the `uniqBy` style API in this case. Assuming the proposal for immutable records/tuples makes it in, uniqBy can become quite effective.

Here of some more examples of the powerful functionality `uniqBy` can add in conjunction on its own and with immutable records/tuples.

uniqBy (on its own):
```js
// ✅Works with: single field, top level properties 
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy(x => x.a); // [ {a: 1, b: 1} ]
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy(x => x.b); // [ {a: 1, b: 1},  {a: 1, b: 2} ]

// ✅Works with: single field, nested properties
[ {a: 1, b: {c: 2}},  {a: 2, b: {c: 2}} ].uniqBy(x => x.b.c); // [ {a: 1, b: {c: 2}} ]

// ❌Doesn’t work with multiple fields  
[ {a: 1, b: 1},  {a: 1, b: 1} ].uniqBy(x => { return { 'a': x.a, 'b': x.b }}); // [ {a: 1, b: 1}, {a: 1, b: 1} ]
// Incorrect result because using wrong equality comparison, should yield:  [ {a: 1, b: 1} ]
```

uniqBy (with immutable records/tuples):

```js
// ✅Works with previous examples above
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy(x => x.a); // [ {a: 1, b: 1} ]
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy(x => x.b); // [ {a: 1, b: 1},  {a: 1, b: 2} ]
[ {a: 1, b: {c: 2}},  {a: 2, b: {c: 2}} ].uniqBy(x => x.b.c); // [ {a: 1, b: {c: 2}} ]


// ✅Works with single fields (using immutable records)
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy(x => #[ x.a ]); // [ {a: 1} ]

// ✅Works with multiple fields (using immutable records)
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy(x => #[ x.a, x.b ]); // [ {a: 1, b: 1},  {a: 1, b: 2} ]
```

## Additional comments
In addition, `uniqBy` should throw if the given field is not present in one of the objects in the array to prevent false positives:

```js
Array.prototype.uniqBy = function(generateRepresentative) {
    let representatives = this.map(generateRepresentative);
  if (this.some(x => x === void 0)) { throw new Error(‘a object in the array did not contain the given field’);
  let uniqueRepresentatives = Array.from(new Set(representatives));
  return uniqueRepresentatives
    .map(x => representatives.indexOf(x))
    .map(index => this[index]);
}
```
(open to other approaches for detecting bad uniqBy matching on properties.)

The other option (other than to throw) is to not consider objects in the array where the field in the passed function is not present, but I prefer to throw since it is odd to operate on an array of unlike objects.
