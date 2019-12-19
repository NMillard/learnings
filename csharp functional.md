# Functions

## Pure functions
- The same input should always generate the same result
- The method signature should carry all information about possible outcomes
    - e.g. if a method may throw, it should be described and anticipated
- The function may not produce side-effect, e.g. it does not alter input, ie. update an object's property


## Separation
Use Command-Query separation principle
- Commands produces side-effects
- Querys only return data