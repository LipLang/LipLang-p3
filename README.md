# LipLang - p3
> This is a reconstruction

## the current design:
1. use operators:
- '>>' For ordinary pipe transmission
- '/>' Used for branch operations
- '++' is used to merge data flows
- '->' Used for result assignment
- '@' is used for insert operations without affecting the data flow

2. use placeholder '#' for argumenting functions
3. use welded Pipelines for code reusability

## example
### filter and sum
```
range(10) >> filter(# > 5) >> map(# * 2) >> sum() -> result
// When the function has only one argument, and its data is transmitted from the pipe, the '#' can be omitted
// (such as the `sum` function above).

// result = sum([x * 2 for x in range(10) if x > 5])

result >> print()
// print(result)
```

### merge data flow
```
range(1, 4) >>         // [1,2,3]
map(# * 2) ++          // [2,4,6]
range(4, 7) >>         // [4,5,6]
map(# + 1) >>          // [5,6,7]
merge() >>             // [2,4,6,5,6,7]
sort() -> result       // [2,4,5,6,6,7]
```

### moving window
```
range(1, 8) >>                // [1,2,3,4,5,6,7,8]
window(#, size=3, step=1) >>  // [[1,2,3],[2,3,4],[3,4,5]...]
map(sum()) >>                 // [6,9,12,15,18,21]
@ print() -> result           // insert an op, but donot affect the data flow
```

### reduce
```
range(1, 6) >>              // [1,2,3,4,5]
reduce(sum()) >>            // [3,6,10,15]
@ print("Running Sum", #) >>
get(-1) -> result           // 15
```

### 2d data frame and another fork
```pipeflow
[ [1,2,3],
  [4,5,6],
  [7,8,9]] />
{ // ready to parallel
  Rows: map(sum()) >>     // [6,15,24]
        mean() -> rowAvg  // 15
        
  Cols: transpose() >>    // [[1,4,7],[2,5,8],[3,6,9]]
        map(sum()) >>     // [12,15,18]
        mean() -> colAvg  // 15
}
```

### branch
```
range(1, 11) />
{
  Small: filter(# <= 3) >>      // [1,2,3]
         sum() -> smallSum      // 6
         
  Medium: filter(3 < # <= 7) >>  // [4,5,6,7]
          product() -> medProd   // 840
          
  Large: Filter(# > 7) >>       // [8,9,10]
         average() -> largeAvg  // 9
}
```

### data match
```
range(1, 6) >>           // [1,2,3,4,5]
map(# * 2) ++            // [2,4,6,8,10]
range(2, 7) />           // [2,3,4,5,6]
{
  Common: findCommon() >>      // [2,4,6]
          sum() -> commonSum   // 12
          
  Diff: findDifferent() >>     // [8,10,3,5]
        sort() -> diffSorted   // [3,5,8,10]
}
```

### welded pipeline
```
let processWithFactor = (factor: float) => (
  map(# * factor) >>
  filter(# > 10) >>
  sort()
)

data >> processWithFactor(2.5) -> result
```

## advantage:
- clean, simple and intuitive
- flexible and efficient `fork` and `merge`
- be safe without the `own` and `borrow` problem (in `rust`)

---

## TODO

- Error processing

- Borrow strong typing systems from LINQ
```csharp
Enumerable.Range(1, 10)
    .Where(x => x > 5)
    .Select(x => x * 2)
    .Sum();
```

- Borrow functional programming from F#/OCaml
```fsharp
// F# 管道操作符 |>
[1..10]
|> List.filter (fun x -> x > 5)
|> List.map (fun x -> x * 2)
|> List.sum
```

- Borrow parallel processing framework from Apache Beam
```python
(p
 | beam.Create([1, 2, 3, 4, 5])
 | beam.Filter(lambda x: x > 2)
 | beam.Map(lambda x: x * 2))
```

- Borrow data analysis features from dplyr
```r
mtcars %>%
  filter(cyl == 6) %>%
  group_by(am) %>%
  summarise(mean_mpg = mean(mpg))
```

---

### Other lessons:
- Pipe operator of Elixir
```elixir
1..10
|> Enum.filter(&(&1 > 5))
|> Enum.map(&(&1 * 2))
|> Enum.sum()
```

- Julia is the pipeline operator
```julia
1:10 |>
x -> filter(y -> y > 5, x) |>
x -> map(y -> y * 2, x) |>
sum
```
