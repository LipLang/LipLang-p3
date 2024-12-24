# LipLang - p3

## operators
- '>>' For ordinary pipe transmission
- '/>' Used for branch operations
- '++' is used to merge data flows
- '->' Used for result assignment
- '@' is used for insert operations without affecting the data flow


## advantage
- clean, simple and intuitive
- flexible and efficient `fork` and `merge`
- be safe without ownership related problem (like `rust`)


## examples
### filter and sum
```
range(10) >> filter(x => x > 5) >> map(x => x * 2) >> sum() -> result
// When the function has only one argument, and its data is transmitted from the pipe, the '_' can be omitted
// (such as the `sum` function above).

// result = sum([x * 2 for x in range(10) if x > 5])

result >> print()
// print(result)
```

### merge data flow
```
range(1, 4) >>         // [1,2,3]
map(x => x * 2) ++     // [2,4,6]
range(4, 7) >>         // [4,5,6]
map(x => x + 1) >>     // [5,6,7]
merge() >>             // [2,4,6,5,6,7]
sort() -> result       // [2,4,5,6,6,7]
```

### moving window
```
range(1, 8) >>                // [1,2,3,4,5,6,7,8]
window(_, size=3, step=1) >>  // [[1,2,3],[2,3,4],[3,4,5]...]
map(sum()) >>                 // [6,9,12,15,18,21]
@ print() -> result           // insert an op, but donot affect the data flow
```

### reduce
```
range(1, 6) >>              // [1,2,3,4,5]
reduce(sum()) >>            // [3,6,10,15]
@ print("Running Sum", _) >>
get(_, -1) -> result        // 15
```

### 2d data frame and another fork
```pipeflow
[ [1,2,3],
  [4,5,6],
  [7,8,9]] />
{ // ready to parallel
  Rows: map(sum()) >>  // [6,15,24]
        mean() -> avg  // 15
        
  Cols: transpose() >> // [[1,4,7],[2,5,8],[3,6,9]]
        map(sum()) >>  // [12,15,18]
        mean() -> avg  // 15
} >> @ print()
```

### branch
```
range(1, 11) />
{
  Small: filter(x => x <= 3) >>       // [1,2,3]
         sum() -> smallSum      // 6
         
  Medium: filter(x => 3 < x <= 7) >>  // [4,5,6,7]
          product() -> medProd  // 840
          
  Large: filter(x => x > 7) >>       // [8,9,10]
         average() -> largeAvg  // 9
}
```

### data match
```
range(1, 6) >>           // [1,2,3,4,5]
map(x => x * 2) ++       // [2,4,6,8,10]
range(2, 7) >>           // [2,3,4,5,6]
combo() />               // [[2,4,6,8,10], [2,3,4,5,6]]
{
  Common: findCommon(_1, _2) >>  // [2,4,6]
          sum() -> commonSum     // 12
          
  Diff: findDifferent(_1, _2) >> // [8,10,3,5]
        sort() -> diffSorted     // [3,5,8,10]
}
```

### persisted pipeline
```
let processWithFactor = (factor: float, x: float) => (
  map(x => x * factor) >>
  filter(x => x > 10) >>
  sort()
)

data >> processWithFactor(2.5, _) -> result
```

### connect persisted pipelines
```
let pipeline1 = (x) => (
    map(x => x * 2) >>
    filter(x => x > 10)
)

let pipeline2 = (x) => (
    sort(x) >> take(5)
)

let combinedPipeline = pipeline1 |> pipeline2
data >> combinedPipeline -> result
```


## TODO
- use `package xxx/yyy/zzz`, `import "aaa/bbb/ccc"`
- error processing (use `empty`?)
- named table and its related operations
- strong typing systems
- functional programming
- parallel processing framework
- let data flow in/out temporary
