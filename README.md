# Array.prototype.uniqBy-Proposal

## Intro to Problem

It would be useful to have the ability to get all unique objects according to some provided definition of equality. 

Additional functionality:
```
Array.prototype.uniqBy
```

## Where the problem is seen

There are many implementations of this function across Github, see [here](https://github.com/search?l=JavaScript&q=uniqBy&type=Code). This shows the desire for such functionality. Systems built around custom implementations of this could be subject to bugs or misunderstanding around the differences in equality methods used. This doesn’t account for other implementations of this method which are implemented using another name.

## What people are doing about it today

Developers are forced to either implement this functionality themselves, or rely on third party libraries such as Lodash, which offers the same interface as proposed here.

[Lodash’s uniqBy](https://www.npmjs.com/package/lodash.uniqby) function alone has almost half a million weekly downloads. This sits about high up the table for their most commonly used functions. It is their 7th most popular out of the 65 base functions the libraries comes with. See the table here.

[Ramda’s uniqBy](https://ramdajs.com/docs/#uniqBy) is another popular function implementation. Their complete library has over 5 million weekly downloads.

## Solutions

There are a couple of ways this functionality could be implemented. Two implementations are listed below.

### Map to a canonical representative of an equivalence class:

One approach is to pass uniqBy a function which maps the objects in the array to a canonical representative of an equivalence class.

#### Examples

```
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy(x => x.a); // [ {a: 1, b: 1} ]
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy(x => x.b); // [ {a: 1, b: 1},  {a: 1, b: 2} ]
[ {a: 1, b: {c: 2}},  {a: 2, b: {c: 2}} ].uniqBy(x => x.b.c); // [ {a: 1, b: {c: 2}} ]
```

Implementation 1.a:
```
Array.prototype.uniqBy = function(generateRepresentative) {
  let representatives = this.map(generateRepresentative);
  let uniqueRepresentatives = Array.from(new Set(representatives));
  return uniqueRepresentatives
    .map(x => representatives.indexOf(x))
    .map(index => this[index]);
}
```

#### Difficulties 

This implementation works well when mapping to a single primitive value representative. However, becomes problematic when you wish to uniqBy a combination of values. 

#### How immutable records/tuples makes this API great

This could be improved by using the [Immutable Records proposal](https://github.com/tc39/proposal-record-tuple). Immutable Records have built-in value equality which could be used in the following way to support uniqBy on multiple values:

```
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy(x => #[ x.a, x.b ]); // [ {a: 1, b: 1},  {a: 1, b: 2} ]
```


### Provide a compare method

An alternative implementation of uniqBy would consume a comparison method which would determine whether two entries in the list should be considered equivalent. This way of implementation is referred to as `uniqWith` by both Lodash and Ramda. 

#### Examples

```
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy((x, y) => x.a === y.a); // [ {a: 1, b: 1} ]
[ {a: 1, b: 1},  {a: 1, b: 2} ].uniqBy((x, y) => x.b === y.b); // [ {a: 1, b: 1},  {a: 1, b: 2} ]
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
#### Inefficiency of this API

This implementation yields an unideal performance time, ~O(n^2). 

#### Usability of this API

The API allows for a comparator that does not actually define a partial ordering, which means a user could use this implementation in unintended ways. There are no safeguards around someone passing `(x, y) => x.a > y.a` to the compare function, which could yield confusing results.


## Comparison of implementations


| Implementation                   | 1.a  | 1.b  | 2.a    |
|----------------------------------|------|------|--------|
| Uniquing on single  property     | X    | X    | X      |
| Uniquing on multiple  properties | -    | X    | X      |
| Possibility for misuse           | -    | -    | X      |
| Performance                      | O(n) | O(n) | O(n^2) |
| Requires other proposal to merge | -    | X    | -      |



Additional information: 

There is a node module, [array-unique](https://www.npmjs.com/package/array-unique), which has similar behavior. This module has over 13 million weekly downloads. However, this implementation only for doing so on a single field.
